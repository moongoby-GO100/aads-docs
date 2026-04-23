# AADS Claude DB 우선순위 폴백 및 문서 허브 확장 보고 (2026-04-23 KST)

## 요약

- Claude 계정 선택 기준을 `.env.oauth` 우선에서 `DB(llm_api_keys.provider=anthropic)` 우선으로 전환했다.
- 채팅 API, Claude relay, 채팅 UI 상태 표시가 모두 같은 DB priority를 기준으로 동작하도록 정렬했다.
- Claude 계정은 사용자가 설정한 슬롯 우선순위를 먼저 따르고, rate limit이 기록된 계정만 자동으로 뒤로 미루는 하이브리드 폴백으로 보강했다.
- `/docs` 통합 페이지는 전 프로젝트 문서를 더 넓게 볼 수 있도록 개선했고, 빠져 있던 프로젝트 경로를 반영했다.

## 원인 및 조치

### 1. Claude 계정 우선순위 불일치

- 기존 문제:
  - 백엔드 일부는 DB 키 순서를 사용하고, relay는 `.env.oauth CURRENT_OAUTH`를 따르며, UI는 `Naver/Gmail` 하드코드 라벨을 사용했다.
  - 운영자가 DB priority를 바꿔도 실제 Claude relay current slot과 채팅창 표시가 즉시 일치하지 않았다.
- 조치:
  - `aads-server/app/core/auth_provider.py`
    - `get_oauth_key_records_async()` 추가
    - `set_token_order_async()` 추가
    - DB priority 변경 시 relay `/oauth/switch` 동기화 추가
  - `aads-server/app/core/llm_key_provider.py`
    - provider별 key record 조회 함수 추가
  - `aads-server/app/services/model_selector.py`
    - Claude 폴백 순서를 DB priority 기반으로 재구성
    - quota/rate-limit 오류 시 DB `rate_limited_until` 갱신
  - `aads-server/app/api/health.py`
    - `anthropic.db_keys`와 실제 relay current slot을 함께 노출
  - `aads-dashboard/src/app/chat/page.tsx`
    - 계정 전환 버튼을 실제 key_name/slot 기반으로 수정
    - 하드코드 `Naver/Gmail` 제거

### 2. Claude relay의 DB 파싱 실패

- 기존 문제:
  - relay가 DB 상태를 `docker exec aads-server ... python`으로 읽을 때 `db_pool_initialized`, `db_pool_closed` 로그가 JSON 앞뒤에 섞여 파싱에 실패했다.
  - 그 결과 relay는 DB가 아니라 `.env.oauth`로 폴백해 잘못된 current slot을 계속 사용했다.
- 조치:
  - `aads-server/scripts/claude_relay_server.py`
    - DB OAuth row 캐시 추가
    - 로그 줄 사이의 JSON payload만 추출하는 파서 추가
    - DB `rate_limited_until`을 읽어 current slot 결정 시 반영
    - 라벨을 실제 계정명으로 노출
    - 기본 동시성 `3 -> 1`로 축소
    - NDJSON 청크 파서 도입으로 Codex 긴 응답 line 처리 안정화

### 3. /docs 통합 페이지 누락 및 가독성 문제

- 기존 문제:
  - AADS 문서 스캔 범위가 `/app/docs`, `/app/reports` 중심이라 `aads-docs`, `aads-dashboard`, `aads-core` 일부 문서가 누락됐다.
  - SF/NTV2 운영 경로가 실제 서버 경로와 달라 일부 문서가 비어 보였다.
  - 리스트와 본문 폭이 고정적이라 문서 내용 확인성이 떨어졌다.
- 조치:
  - `aads-server/app/api/project_docs.py`
    - AADS 스캔 경로를 `aads-docs`, `aads-dashboard`, `aads-core`까지 확장
    - SF `/data/shortflow/docs`, NTV2 `/srv/newtalk-v2/docs`로 수정
    - `full_path` 필드 추가
    - 문서 타입 분류를 세분화
  - `aads-server/docker-compose.prod.yml`
    - 해당 문서 디렉터리 마운트 추가
  - `aads-dashboard/src/app/docs/page.tsx`
    - 전체 경로 표시
    - 좌우 리사이즈 지원
    - 문서 타입 필터 및 세분화 라벨 반영

## 검증

- Claude relay health:
  - `2026-04-23 12:22 KST`
  - `oauth_slot=2`, `oauth_label=moongoby@gmail`
- API health 내부 검증:
  - `anthropic.cli.account=2`
  - `anthropic.cli.label=moongoby@gmail`
  - `anthropic.db_keys[0]=moongoby@gmail / priority=1 / slot=2`
  - `anthropic.db_keys[1]=moong76@gmail / priority=2 / slot=1 / rate_limited_until=2026-04-23T03:19:15.376656+00:00`
- `/docs` 스캐너:
  - AADS 268
  - KIS 78
  - GO100 799
  - SF 134
  - NTV2 135
  - 총 1414

## 배포

- `aads-api` supervisor restart 완료
- `claude-relay` systemd restart 완료
- `aads-dashboard` 이미지 재빌드 및 컨테이너 재기동 완료

## 운영 메모

- 현재 Claude current slot은 DB priority와 DB rate-limit 상태를 함께 따른다.
- 우선순위는 사용자가 DB에서 지정한 순서를 먼저 따르며, 한도 초과 계정만 자동으로 뒤로 보낸다.
- relay는 여전히 DB를 `docker exec`로 읽는다. 다음 단계로는 relay가 내부 서명 API 또는 직접 DB 조회를 사용하도록 바꾸는 것이 더 단순하다.
