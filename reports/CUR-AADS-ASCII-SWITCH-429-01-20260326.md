# CUR-AADS-ASCII-SWITCH-429-01-20260326.md

## 1) 요약
`claude-opus-4-6` 요청에서 `429 rate_limit_error (Available Model Group Fallbacks=None)`가 발생했는데, UI에서 토큰 스위치를 했음에도 실제 호출 경로에서 “naver로 전환”이 반영되지 않은 정황이 확인됨.

## 2) 확인한 사실(근거)
1. 현재 OAuth 선택값
   - `/root/.genspark/.env.oauth`의 `CURRENT_OAUTH=1` 확인
2. AADS 서버용 인증 토큰(호스트/.env 기준)
   - `/root/aads/aads-server/.env`에 `ANTHROPIC_AUTH_TOKEN`과 `ANTHROPIC_AUTH_TOKEN_2`가 모두 존재(서명 해시로 확인)
   - 따라서 “Agent SDK wrapper에서 어떤 값을 CLAUDE_CODE_OAUTH_TOKEN으로 넣는지”가 결과에 직접 영향
3. LiteLLM 환경/라우팅 설정(고정 키 + order)
   - `/root/aads/aads-server/litellm-config.yaml`에서 `ANTHROPIC_API_KEY_1`(order=1) / `ANTHROPIC_API_KEY_2`(order=2)로 구성
   - 같은 파일에 `router_settings.model_group_fallbacks`가 존재
4. LiteLLM 요청 응답에 “Available Model Group Fallbacks=None”가 반복 노출
   - LiteLLM에 최소 요청을 보내도 동일 문구가 포함된 응답이 발생(즉, 모델그룹 폴백이 런타임에서 적용되지 않는 정황)
5. Claude Code 호스트 경로는 OAuth 토큰 스위치와 무관할 수 있음(호스트 고정)
   - `/root/.claude/get_api_key.sh`는 고정으로 `sk-litellm`만 출력
   - `/root/.claude/current.env`에는 `CLAUDE_CODE_OAUTH_TOKEN`이 unset이고 `ANTHROPIC_API_KEY=sk-litellm`만 설정
6. Agent SDK wrapper는 토큰2(nav er)를 사용하지 않도록 고정되어 있음
   - `/root/aads/aads-server/scripts/claude-oauth-wrapper.sh`에서 `unset ANTHROPIC_AUTH_TOKEN_2`
   - `CLAUDE_CODE_OAUTH_TOKEN`은 `ANTHROPIC_AUTH_TOKEN`(단일값)에서만 가져오도록 되어 있음
   - 결과적으로 “naver 토큰으로 스위치”가 되어도 Agent SDK 경로의 CLAUDE_CODE_OAUTH_TOKEN이 바뀌지 않을 가능성이 큼
7. 운영 상태 확인(HTTP)
   - `http://localhost:8100/api/v1/health/api-keys` HTTP `200`
   - `http://localhost:4000/health/readiness` HTTP `200`

## 3) 왜 스위치가 “안 먹는 것처럼” 보였나(결론)
다음 불일치가 “토큰 스위치 버튼 → 실제 호출 계정 변경”을 끊는 요인으로 관측됨.

1) CEO 채팅 Claude 호출의 핵심 1차 경로는 CLI Relay(호스트 `claude`)인데, 호스트 `claude`는 OAuth 토큰을 스위치해서 쓰는 구조가 아니라 `sk-litellm`으로 고정되어 있음.

2) CEO 채팅의 2차 경로(Agent SDK)는 토큰2(`ANTHROPIC_AUTH_TOKEN_2`)를 무시하도록 래퍼에서 고정되어 있음.
   - `claude-oauth-wrapper.sh`는 `ANTHROPIC_AUTH_TOKEN_2`를 unset하고 `ANTHROPIC_AUTH_TOKEN`만 `CLAUDE_CODE_OAUTH_TOKEN`으로 내보냄
   - 즉, 네이버(토큰2) 계정 사용량이 0%여도 Agent SDK 단계의 실제 인증은 gmail(토큰1)로 고정될 수 있음.

3) LiteLLM 쪽은 모델그룹 폴백 문구가 `None`으로 반복되며, 최소 요청에서도 동일 문구가 확인되어 “요금제 한도 도달 시 폴백 체인”이 기대대로 적용되지 않을 수 있음.

## 4) 검증 결과(통과/실패)
- 검증(HTTP 건강상태): PASS (200)
- 검증(호스트/래퍼 인증 소스 일치성): PASS (호스트 get_api_key.sh / current.env 고정, Agent SDK wrapper 단일토큰 고정 확인)
- 검증(LiteLLM 응답의 폴백 문구): PASS (Available Model Group Fallbacks=None 반복 노출)

## 5) 적용/배포 상태
- 적용: ✅ `claude-oauth-wrapper.sh` 수정 반영
  - 백업: `claude-oauth-wrapper.sh.bak_20260326_122801` 생성
  - 변경: `/root/.genspark/.env.oauth`의 `CURRENT_OAUTH(1/2)`를 읽어 `CLAUDE_CODE_OAUTH_TOKEN`을 선택 주입하도록 수정
  - 검증(dry-run, 현재 naver 테스트용): `AADS_OAUTH_WRAPPER_DRYRUN=1` 실행 시 `selected_oauth_index=2` / `current_oauth=2` 확인
- 배포: ❌ (컨테이너 재시작/강제배포 없음. 파일 볼륨 마운트 기반이라 다음 Agent SDK 실행부터 반영 예상)

테스트(채팅창 UI 직접):
- 브라우저 자동화로 `aads.newtalk.kr` 접속은 가능했지만, 로그인 화면에서 비밀번호 인증이 필요했고 자동 로그인이 불가하여 실제 채팅 전송 테스트는 진행 중단
  - 단, 인증 스위치 기준값은 `CURRENT_OAUTH=2(naver)`로 변경 완료 상태

## 6) 리스크/주의사항
- `Available Model Group Fallbacks=None`이 뜬 429은 “모델그룹 폴백 체인이 런타임에서 적용되지 않음”을 시사함.
- Agent SDK 래퍼가 `ANTHROPIC_AUTH_TOKEN_2`를 무시하도록 작성되어 있어, UI에서 네이버를 선택해도 실제 인증이 gmail로 남는 구조적 리스크가 큼.
- LiteLLM 로그에서 특정 `request_id`를 직접 매칭하기 어려워(로그 포맷 차이) 최종 원인 확정은 “구조적 원인 + 런타임 폴백 미적용” 기준으로 결론 처리.

## 7) 후속 체크(다음에 필요한 작업)
- Agent SDK 경로에서 “네이버 스위치가 실제로 토큰2로 호출되는지” 1회 재확인 필요
- CLI Relay(host claude) 경로에서 “토큰 스위치”가 실제로 주입되는지 확인 필요(현재 호스트는 sk-litellm 고정)
- LiteLLM 폴백 설정이 실제로 적용되도록 `litellm-config.yaml`의 `router_settings.model_group_fallbacks` 형식/적용 가능성 재검증 필요

## 8) 보안/스캔
- `security_scan.sh` / `path_check.sh`: `/root` 및 `/root/aads` 하위에서 스크립트 파일을 찾지 못해 미실행

