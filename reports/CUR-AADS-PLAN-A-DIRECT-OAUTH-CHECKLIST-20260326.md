# A안(Claude → Anthropic 직접 OAuth) 적용 — 정밀 체크리스트 (채팅 무영향 우선)

**문서**: `CUR-AADS-PLAN-A-DIRECT-OAUTH-CHECKLIST-20260326.md`  
**전제**: LiteLLM을 Claude 경로에서 제외(또는 최후 폴백만)하고, `CLAUDE_CODE_OAUTH_TOKEN`(OAuth)로 Anthropic 직접 호출.  
**최우선 제약**: **현재 CEO 채팅창 동작·가용성에 영향이 없어야 함** — 단일 점검 창구, 단계적 전환, 즉시 롤백 가능.

---

## 0. 무영향(Non-degradation) 원칙 — 반드시 준수

| 원칙 | 구현 요구 |
|------|-----------|
| **기본 경로 유지** | 전환 전·중·후 **동일 입력 → 동일 수준 응답 품질**(스트리밍·세션·MCP) 목표. 기능 플래그로 **구 경로(LiteLLM/기존 호스트 설정)를 항상 잔존**. |
| **기능 플래그** | 예: `AADS_CLAUDE_DIRECT_OAUTH=0|1`, `AADS_CLAUDE_DIRECT_OAUTH_CANARY_PCT=0–100`. **기본값 0**(기존 동작). |
| **세션 연속성** | CLI Relay `session_id` ↔ `--resume` 매핑(`/tmp/claude_relay_sessions.json`) **파일 포맷·키 불변**. 직접 OAuth 전환 시에도 **세션 깨짐 시 자동 신규 세션** 등 기존과 동일 복구 경로 유지. |
| **3단 폴백 보존** | `model_selector`: CLI Relay → Agent SDK → Gemini **순서 유지**. A안은 **1·2단에서만** “베이스 URL/키 소스” 변경; 3단(Gemini)은 **기존 LiteLLM/Gemini 경로 그대로**. |
| **관측 가능** | 전환 전후 **동일 smoke 시나리오**(짧은 텍스트 질의, MCP 미사용 / MCP 사용 각 1회) 로그 라벨(`claude_auth_mode=direct|proxy`)로 구분. |
| **롤백** | 플래그 **off + relay 재시작**만으로 **이전과 동일** 설정으로 복귀 가능해야 함(설정 백업·원클릭 스크립트). |

---

## 1. 사전 인벤토리 (변경 전 스냅샷)

- [ ] **호스트 Claude 환경**: `/root/.claude/settings.json`, `/root/.claude/current.env`, `/root/.claude/get_api_key.sh` — 현재 `ANTHROPIC_BASE_URL`, `apiKeyHelper`, `CLAUDE_CODE_OAUTH_TOKEN` 유무 기록.
- [ ] **OAuth 단일 원천**: `/root/.genspark/.env.oauth` — `CURRENT_OAUTH`, `OAUTH_TOKEN_1/2` 존재·권한(읽기 전용 권장은 운영 정책).
- [ ] **CLI Relay**: `claude_relay_server.py` — `create_subprocess_exec(..., env=...)` 가 **호스트 `os.environ` 상속**임을 확인; 직접 OAuth 시 **명시적 env 병합** 설계 위치 결정.
- [ ] **Agent SDK**: `scripts/claude-oauth-wrapper.sh` — 현재 `CURRENT_OAUTH` 반영 여부·컨테이너 마운트 경로.
- [ ] **aads-server**: `docker-compose.yml` 의 `ANTHROPIC_*`, `LITELLM_*`, `CLAUDE_RELAY_URL` — **채팅은 컨테이너→호스트 relay**이므로 relay만 바꿔도 1단 경로 변경 가능.
- [ ] **LiteLLM**: `litellm-config.yaml` + `.env.litellm` — **Gemini·기타 모델**은 계속 사용; Claude만 직접 OAuth 시 **문서상 역할 분리** 명시.
- [ ] **systemd**: `claude-relay.service` — 단위 파일·`Restart=`·실행 사용자·환경 drop-in.

---

## 2. 설계 결정 (채팅 무영향을 위한 분기)

- [ ] **직접 OAuth 활성 조건**: 플래그 ON **且** (선택) 특정 헤더/내부 사용자만 Canary — **전 사용자 일괄 전환 금지**直到 smoke 통과.
- [ ] **Relay subprocess env (권장 패턴)**  
  - 직접 OAuth 시: `unset` 또는 덮어쓰기 — `ANTHROPIC_API_KEY`(또는 helper가 넣는 proxy 키), **proxy `ANTHROPIC_BASE_URL`**  
  - 설정: `ANTHROPIC_BASE_URL=https://api.anthropic.com` (또는 공식 문서 기준)  
  - `CLAUDE_CODE_OAUTH_TOKEN=<CURRENT_OAUTH에 따른 sk-ant-oat01>`  
  - **API 키 `sk-ant-api03` 경로 금지** (프로젝트 규칙).
- [ ] **`get_api_key.sh`**: 직접 OAuth 모드에서는 **호출되지 않거나**, 호출돼도 **빈 값/false가 아닌** CLI 기대 동작과 충돌 없음을 확인 (settings.json과 CLI 버전별 동작 점검).
- [ ] **스위치 UI ↔ Relay**: UI가 바꾸는 값(`CURRENT_OAUTH` 등)이 **Relay 기동 시점이 아닌 요청 시점**에 읽히도록 할지(파일 매 read vs env 주입) 결정 — **채티 중 스위치 반영 지연** 허용 범위 문서화.

---

## 3. 구현 체크리스트 (코드/설정)

### 3.1 CLI Relay (`claude_relay_server.py`)

- [ ] **환경 병합 함수** 추가: `build_claude_env(session_id=None) -> dict` — 플래그에 따라 proxy vs direct 분기.
- [ ] **로그**: 토큰 값 **절대 미출력**; `oauth_slot=1|2`, `mode=direct|proxy` 만.
- [ ] **타임아웃/세마포어**: 기존 `_MAX_CONCURRENT`, `600s` 등 **변경 없음** 확인.
- [ ] **MCP / `--agents`**: CLI 인자 **동일** 유지 (채팅 기능 회귀 방지).

### 3.2 호스트 설정 (선택·플래그 연동)

- [ ] `current.env` 또는 systemd `EnvironmentFile` — direct 모드용 템플릿 **별도 파일** 권장 (`current.env.direct.oauth.example` — **시크릿 커밋 금지**).
- [ ] 배포 스크립트: 플래그 전환 시 **relay만 reload** 가능한지 확인.

### 3.3 Agent SDK 경로

- [ ] `claude-oauth-wrapper.sh`와 Relay의 **동일한 소스**(예: `.env.oauth`의 `CURRENT_OAUTH`)로 **1·2단 계정 일치**.
- [ ] 컨테이너에 **호스트 경로 마운트** 없으면 대체( env 동기화 또는 API로 토큰 슬롯 전달) — **채팅 2단 실패율** 모니터링.

### 3.4 `model_selector.py`

- [ ] 에러 시 기존과 동일하게 **2단·3단 폴백**; direct 모드에서만 발생하는 예외 메시지 **사용자 노출 최소화**(내부 로그만 상세).

### 3.5 LiteLLM / 기타

- [ ] Claude 직접 OAuth 후에도 **Gemini 라우팅·키** 변경 없음 확인.
- [ ] 다른 서비스가 **동일 LiteLLM 4000**에 의존하면 **Claude 트래픽만 분리**되었는지 문서화.

---

## 4. 검증 매트릭스 (회귀 = 채팅 기능 저하 금지)

각 행: **플래그 OFF(기준)** vs **ON(신규)** 동일 시나리오 통과.

| # | 시나리오 | 기준 | 신규(direct) | 비고 |
|---|----------|------|----------------|------|
| V | 짧은 텍스트 질의, MCP 없음 | 스트리밍 정상 | 동일 | |
| V | 동일 세션 2턴 (`--resume`) | 세션 이어짐 | 동일 | 세션 맵 백업 후 테스트 |
| V | MCP 도구 1회 이상 | 도구 호출·응답 | 동일 | |
| V | 이미지/멀티모달(해당 시) | stream-json 경로 | 동일 | relay 분기 확인 |
| V | `CURRENT_OAUTH=1` / `2` 각각 | 기대 계정(로그 슬롯만) | 동일 | Anthropic 응답/리밋으로 간접 확인 |
| V | Relay 비가용 | Agent SDK 폴백 | **변경 없음** | |
| V | Claude 전부 실패 | Gemini 폴백 | **변경 없음** | |

- [ ] **부하**: 동시 `_MAX_CONCURRENT` 내에서 지연·에러율 **기준선 대비 악화 없음**.

---

## 5. 배포 절차 (단계적, 채팅 보호)

1. [ ] **백업**: `claude_relay_sessions.json`, `settings.json`, `current.env`, relay systemd drop-in.
2. [ ] **Stg 또는 단일 테스트 세션**: 플래그 ON + Canary 0% — **수동 헤더/내부 플래그만** direct.
3. [ ] **Smoke V 전부 PASS** 후 Canary 1% → 10% → 50% → 100% (각 단계 **최소 1 영업일 또는 사전 합의 기간**).
4. [ ] **모니터링**: `429`, `5xx`, relay `CLI timeout`, 평균 **첫 토큰 지연** — 임계치 초과 시 **즉시 Canary 축소 또는 플래그 OFF**.

---

## 6. 롤백

- [ ] 플래그 `AADS_CLAUDE_DIRECT_OAUTH=0` (또는 동등) **즉시 적용** 경로(환경 + relay reload) 문서화.
- [ ] 직접 OAuth용으로 수정한 **호스트 전역 파일**이 있으면 **백업본으로 복구** 스크립트 1개.
- [ ] 롤백 후 **V 매트릭스 1·2행만** 재실행해 “채팅 정상” 확인.

---

## 7. 보안·규정

- [ ] 리포트/슬랙/로그에 **토큰 전체 값 금지** (접두사 8자 + 해시만).
- [ ] `sk-ant-api03` 미사용 grep 확인.
- [ ] 커밋 전: 민감 파일·`.env` 제외 확인.

---

## 8. 문서 동기화

- [ ] `reference_claude_auth_setup.md` (또는 AADS 인증 가이드): **Claude = 직접 OAuth / 기타 = LiteLLM** 다이어그램 갱신.
- [ ] 운영 런북: 플래그 위치, 롤백, “채팅 안 됨” 1차 조치(플래그 OFF + relay restart).

---

## 9. 완료 정의 (Definition of Done)

- [ ] 플래그 OFF일 때 **현재와 바이너리 동일 동작**(설정 diff 없음).
- [ ] 플래그 ON + Canary 100% 시 **검증 매트릭스 전부 PASS**.
- [ ] 롤백 드릴 1회 이상 실제 수행·기록.

---

*본 체크리스트는 “A안 적용”과 “채팅 기능 무영향”을 동시 만족하도록 **플래그·단계 배포·폴백·롤백**을 필수 항목으로 둔다.*
