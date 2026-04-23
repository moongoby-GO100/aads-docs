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
- `aads-dashboard` blue-green 배포 완료 (`3100 -> 3101`)

## 운영 메모

- 현재 Claude current slot은 DB priority와 DB rate-limit 상태를 함께 따른다.
- 우선순위는 사용자가 DB에서 지정한 순서를 먼저 따르며, 한도 초과 계정만 자동으로 뒤로 보낸다.
- relay는 더 이상 DB를 `docker exec`로 읽지 않는다.
- relay는 `GET /api/v1/health/claude-relay/oauth-state` 내부 엔드포인트를 shared secret으로 호출해 OAuth 상태를 받는다.
- shared secret 파일은 `aads-server/scripts/claude_relay_secret.txt` 로컬 파일을 사용하며 Git에는 포함하지 않는다.

## 2026-04-23 추가 조치 (2차)

### 1. 권한 제약 검토 결과

- 현재 채팅창에서 막히는 권한 문제는 실제로 존재한다.
- 원인:
  - Docker daemon 접근
  - `systemctl`/`nginx reload` 같은 호스트 운영 명령
  - `.git/logs/*` 쓰기 및 pre-commit hook의 Docker smoke test
- 결론:
  - 채팅창이 직접 호스트 권한 상승을 요청할 수 없는 구조라면, 위 종류 작업은 서버 내부 privileged path로 우회하지 않는 한 실패 가능성이 높다.
  - 특히 배포, 운영 복구, 일부 검증 작업은 여전히 권한 경계 밖에 있다.

### 2. Claude relay의 Docker 기반 DB 조회 제거

- 이전:
  - relay가 `docker exec aads-server ... python` 으로 DB-backed OAuth 상태를 읽었다.
  - Docker 권한/출력 로그 혼입에 취약했다.
- 현재:
  - `app/api/health.py`에 `GET /api/v1/health/claude-relay/oauth-state` 추가
  - relay는 local HTTP + shared secret 헤더로 OAuth 상태를 읽는다.
- 장점:
  - Docker socket 의존 제거
  - JSON 파싱 단순화
  - 권한 이슈 감소
- 남은 고려사항:
  - shared secret 파일 유실 시 relay는 env fallback으로 떨어질 수 있다.
  - 이 파일은 서버 로컬 자산으로 운영 관리가 필요하다.

### 3. Dashboard blue-green 실배포

- 이전:
  - nginx는 `/` 를 `127.0.0.1:3100` 단일 포트로만 보았다.
  - `aads-dashboard`는 컨테이너 교체형이라 엄밀한 무중단이 아니었다.
- 현재:
  - `docker-compose.prod.yml`에 `aads-dashboard-green` 추가 (`3101`)
  - `nginx-aads-upstream.conf`에 `upstream aads_dashboard` 추가
  - nginx `/` 라우팅을 `aads_dashboard` upstream으로 전환
  - `aads-dashboard/deploy.sh`를 실제 blue-green 스위치 스크립트로 교체
- 실배포 결과:
  - `2026-04-23 12:51 KST`
  - active slot: `green`
  - active container: `aads-dashboard-green`
  - external health: `/login` 정상

### 4. Claude 계정 상태판

- `health/api-keys` 응답에 다음 필드를 추가했다.
  - `cli.status`
  - `cli.auth_mode`
  - `cli.token_available`
  - `db_keys[].key_name`
  - `db_keys[].is_current`
  - `db_keys[].last_used_at`
  - `db_keys[].last_verified_at`
  - `db_keys[].notes`
- 채팅창에는 Claude 상태판 팝오버를 추가했다.
- 표시 항목:
  - 현재 relay slot
  - 현재 active 계정
  - priority/slot
  - rate limit 시각
  - 마지막 검증 시각
  - 마지막 사용 시각
