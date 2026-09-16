# PRD — LLM 계정 런타임 바인딩 (코덱스 계정 다중화 + 클로드 슬롯3 + 사용량 가시화)

| 항목 | 내용 |
|---|---|
| 문서 ID | PRD-LLM-ACCOUNT-RUNTIME-BINDING-v1.0 |
| 작성일 | 2026-09-16 |
| 지시 | 2026-09-16 대표님 — "진아서버에 코덱스 계정추가하라고 지시했는데 반영된거 맞아?" → "모두 진행하고 설계 PRD 작성 파일로 저장해" |
| 대상 서버 | 116 (`5.104.86.116`, 호스트명 `vmi3267555`) |
| 관련 커밋 | `7002bbe8` feat(codex) 어댑터, `cea8720f` (현재 HEAD) |

---

## 0. 용어 정리

이 문서에서 쓰는 말의 뜻을 먼저 고정한다. 같은 이름이 시스템마다 다른 것을 가리켜 혼선이 있었다.

| 용어 | 뜻 | 실체 위치 |
|---|---|---|
| **레지스트리 등록** | 계정 정보를 AADS 중앙 키 저장소에 기록하는 것 | DB 테이블 `llm_api_keys` |
| **런타임 바인딩** | CLI/어댑터가 실제 호출할 때 그 계정으로 인증하는 것 | `auth.json` / `.credentials.json` 파일 |
| **릴레이 세션** | 오비스가 코덱스를 호출할 때 세션마다 만드는 독립 작업 디렉터리. 계정 단위가 아니다 | `/root/.codex-relay/<session_id>/.codex/` |
| **슬롯 (slot)** | 클로드 릴레이 전용 개념. **OAuth 계정 1개 = 슬롯 1개** | `/root/.claude-relay-slots/slotN/` |
| **계정 홈 (account home)** | 이 PRD에서 **신설**하는 코덱스 쪽 계정 단위 디렉터리. 클로드의 슬롯에 대응한다 | `/root/.codex-accounts/<key_name>/` (신규) |
| **CODEX_HOME** | 코덱스 CLI가 인증·설정·세션기록을 읽고 쓰는 홈 디렉터리 | 릴레이가 세션별로 지정 |
| **rollout 파일** | 코덱스 세션 1회의 전체 기록. 사용량은 이 안 `token_count` 이벤트에 있다 | `<CODEX_HOME>/sessions/YYYY/MM/DD/rollout-*.jsonl` |
| **진아서버** | 244 서버. LLM 계정 원본 보유처. 이 서버(116)와 다른 장비다 | - |

핵심은 **레지스트리 등록과 런타임 바인딩이 자동으로 연결되지 않는다**는 것이다. API 키 방식(gemini·groq 등)은 DB 등록만으로 동작하지만, 코덱스와 클로드는 OAuth 구독 계정이라 CLI 홈 디렉터리 작업이 별도로 필요하다. 이번 사고의 원인이 정확히 이 지점이다.

---

## 1. 배경과 문제

2026-09-15 지시(*"진아서버에 llm 계정들이 많이 추가되었다고 한다. 확인하고 리스트 요금제등 정리해서 보고하고 aads에 모두 반영해줘"*)에 따라 `register_jinah_llm_accounts.py`가 API 키 12종을 DB에 등록했다. 등록 자체는 정상 완료됐다.

그러나 OAuth 구독 계정 2건은 **DB에만 들어가고 런타임에 닿지 않았다.**

### 1.1 실측 현황 (2026-09-16 확인)

**코덱스 — 등록 O / 런타임 X**

```
provider | key_name          | label                                              | created_at
codex    | CODEX_OAUTH_JINAH | jinah-biseo(244) ChatGPT Pro / jswworld@gmail.com   | 2026-09-15
```

- 릴레이가 세션 홈을 만들 때 `auth.json`을 **`/root/.codex/auth.json`으로 하드코딩 심볼릭 링크** (`claude_relay_server.py:1407-1416`)
- 링크 대상 계정 = `moongoby@gmail.com` — 릴레이 세션 35개 전수가 같은 파일을 가리킨다
- 어댑터 `app/core/codex_oauth.py`는 작성돼 있으나 **호출부가 0건**이다(`grep -rn "codex_oauth" app` → 없음). 즉 죽은 코드다
- `llm_models`의 코덱스 모델 8종은 `execution_backend: codex_cli` 로 등록돼 있어, 어댑터가 아니라 **CLI 경로로 흐른다**

결과: `jswworld@gmail.com` 계정은 단 한 번도 호출에 쓰인 적이 없다.

**클로드 — 등록 O / 런타임 X (의도된 차단 + 자격증명 부족)**

```
anthropic | ANTHROPIC_AUTH_TOKEN   | p1 | moong76@gmail       (2026-04-20)  → slot1 O
anthropic | ANTHROPIC_AUTH_TOKEN_2 | p2 | moongoby@naver.com  (2026-04-20)  → slot2 O
anthropic | ANTHROPIC_AUTH_TOKEN_3 | p3 | jinah-biseo(244)    (2026-09-14)  → slot3 없음
```

- `/root/.claude-relay-slots/` 에 `slot3` 디렉터리가 없다
- 릴레이 코드에 차단이 명시돼 있다 — `claude_relay_server.py:510` *"Slot 3 is the jinah account (2026-09-15): it must never be selected implicitly, because using it spends someone else's weekly quota."*
- 저장된 값이 **평문 `sk-...` 108자 = access token 단독**이다. refresh token이 없어, 지목해도 구형 env 토큰 경로로 떨어지고 발급 약 8시간 뒤 401로 죽는다(슬롯 README 2026-09-12 검증 기록). 9/14 등록분이므로 이미 무효다
- `logs/relay-restart-slot3.log`(9/15 00:27)가 **0바이트**로 남아 있다 — 시도 후 중단된 흔적

### 1.2 이로 인한 2차 피해 — 한도 소진

계정이 하나로 묶인 채 전량이 `moongoby@gmail.com` 에 쏠렸다.

- 최근 72시간 rollout 80건: 정상 31 / **사용 한도 초과 실패 19** / 턴 미완료 30
- `default` 세션은 10건 연속 한도 초과로 사실상 정지
- 실패 원문: `You've hit your usage limit. ... try again at Sep 19th, 2026 5:13 PM`
- 마지막 유효 스냅샷(50시간 전) 기준 주간 한도 **100% 소진**, 크레딧 잔액 0

### 1.3 왜 아무도 몰랐나 — 사용량 가시화 부재

- ChatGPT 구독 인증이라 API 청구 대시보드가 존재하지 않는다
- codex CLI 0.154.0 에 `usage`/`status` 서브커맨드가 없다(`doctor`는 설치 진단 전용)
- 한도 정보는 rollout 파일 안에만 남고, 어디에서도 집계되지 않는다

---

## 2. 목표 / 비목표

### 목표

| # | 목표 | 완료 기준 |
|---|---|---|
| G1 | 코덱스가 계정 여러 개를 쓸 수 있다 | 릴레이 세션이 계정 홈을 골라 바인딩하고, 한도 걸린 계정은 자동 우회 |
| G2 | 진아 코덱스 계정(`jswworld@gmail.com`)이 실제 호출에 쓰인다 | 해당 계정 홈의 rollout에 정상 `token_count` 기록 |
| G3 | 클로드 슬롯3이 살아난다 | `slot3/.claude/.credentials.json`에 refresh token 포함, 자동 갱신 확인 |
| G4 | 사용량을 언제든 확인할 수 있다 | CLI 1줄 + 대시보드 카드 + 한도 임박 알림 |
| G5 | 같은 사고가 재발하지 않는다 | 등록만 되고 바인딩 안 된 계정을 검출하는 점검이 상시 동작 |

### 비목표

- 코덱스 유료 크레딧 구매 — 대표님 결정 사항이며 이 PRD 범위 밖이다
- `app/core/codex_oauth.py` 어댑터의 챗 경로 연결 — CLI 경로와 목적이 다르다. 별건으로 분리
- 244 서버 자체의 구성 변경 — 이 PRD는 116 서버만 다룬다

---

## 3. 설계

### P1. 코덱스 계정 홈 도입 (핵심)

클로드 슬롯에서 검증된 구조를 코덱스에 그대로 옮긴다.

**3.1 디렉터리**

```
/root/.codex-accounts/
  CODEX_OAUTH_MAIN/auth.json     # moongoby@gmail.com  (기존 /root/.codex/auth.json 승계)
  CODEX_OAUTH_JINAH/auth.json    # jswworld@gmail.com  (DB에서 신규 구성)
```

`auth.json`은 **실파일**이며 권한 0600이다. 심볼릭 링크가 아니다.

**3.2 DB → 계정 홈 구성 (materialize)**

`CODEX_OAUTH_JINAH` 의 암호화 값은 단일 토큰이 아니라 JSON 객체다. 구조 확인 결과:

```
auth_mode / client_id / account_id / refresh_token(211자) / token_endpoint
responses_endpoint / model
```

**refresh_token이 들어 있다.** 따라서 설계 시점에는 재로그인 없이 계정 홈을 만들 수 있다고 봤다. codex CLI가 읽는 형식으로 변환해 기록한다.

> **2026-09-16 실측 — 이 전제는 깨졌다.** 저장된 refresh_token으로 토큰 교환을 시도하니
> `HTTP 401 refresh_token_reused`("Your refresh token has already been used to generate a
> new access token")가 돌아왔다. ChatGPT OAuth의 refresh_token은 **1회용이고 사용할 때마다
> 회전**한다. DB 사본은 누군가 한 번 쓰는 순간 죽은 값이 된다.
> 그래서 진아 코덱스 계정도 **재로그인 또는 244 서버에서 현재 auth.json 재추출**이 필요하다.
> 오류 사전 `codex.refresh_token_reused` 에 등록했다.
>
> 부수 확인: `access_token`을 빈 값으로 두면 CLI가 refresh_token으로 자동 발급하지 않고
> `Missing bearer or basic authentication in header`로 죽는다. 최초 1회는 스크립트가
> 교환해 채워줘야 한다.

```json
{ "auth_mode": "chatgpt",
  "OPENAI_API_KEY": null,
  "tokens": { "refresh_token": "...", "account_id": "..." },
  "last_refresh": null }
```

access_token은 **일부러 비운다.** CLI가 첫 호출에서 refresh_token으로 직접 발급하고 파일에 되쓰게 한다. 스크립트가 직접 갱신하면 refresh token 회전 시 DB 사본과 어긋나 양쪽이 죽는다. 클로드 슬롯 README가 *"tokens never need manual syncing"* 으로 같은 판단을 이미 기록해 두었다.

이후로는 **파일이 진실의 원천**이고 DB 값은 최초 1회 씨앗으로만 쓴다.

**3.3 계정 선택**

`llm_api_keys` 의 기존 메커니즘을 그대로 재사용한다. 새 테이블을 만들지 않는다.

```sql
SELECT ... FROM llm_api_keys
WHERE provider='codex' AND is_active
  AND (rate_limited_until IS NULL OR rate_limited_until <= NOW())
ORDER BY priority ASC, id ASC
```

`app/core/llm_key_provider.py:get_provider_key_records()` 가 이미 이 질의를 한다. 클로드 슬롯1 복귀 스크립트(`restore_claude_slot1.sh`)가 의존하는 것과 동일한 규약이므로 운영 방식이 통일된다.

- 세션 단위 **고정(sticky)**: 한 릴레이 세션은 중간에 계정을 바꾸지 않는다. 섞이면 한도 추적과 로그 추적이 모두 깨진다
- 선택 결과는 세션 홈에 `account.json` 으로 남겨 사후 추적을 가능하게 한다

**3.4 세션 홈 바인딩 변경**

`_build_codex_home(session_id)` 의 하드코딩 링크를 선택된 계정 홈으로 바꾼다.

```
# 기존
session/.codex/auth.json -> /root/.codex/auth.json              (고정)
# 변경
session/.codex/auth.json -> /root/.codex-accounts/<선택계정>/auth.json
```

심볼릭 링크 방식 자체는 유지한다. CLI가 갱신한 토큰이 계정 홈 원본에 바로 반영돼 세션 간 공유되는 이점이 있고, 이것이 원래 주석에 적힌 401 회피 의도였다.

**3.5 한도 감지 → 자동 우회**

rollout의 `task_complete.error.message` 에서 `usage limit` 과 `try again at <시각>` 을 파싱해 `rate_limited_until` 에 기록한다. 그러면 3.3의 질의가 자동으로 다음 계정을 고른다. 클로드가 이미 쓰는 패턴과 같다.

### P2. 클로드 슬롯3

**대화형 로그인 1회가 반드시 필요하다.** DB의 `ANTHROPIC_AUTH_TOKEN_3` 은 access token 단독이라 자동 구성이 불가능하다. 이것만은 자동화로 우회할 수 없다.

1. `/root/.claude-relay-slots/slot3/.claude/` 골격 생성 (자동)
2. `HOME=/root/.claude-relay-slots/slot3 claude auth login` — **대표님 실행 필요**
3. refresh token 포함 여부 검증 (자동)
4. 릴레이 `_build_claude_env()` 의 `elif slot in ("1","2")` 조건을 자격증명 파일이 있는 모든 슬롯으로 확장

**암묵 선택 차단 유지 여부는 대표님 결재 사항이다.** 현재 코드는 *"타인의 주간 할당량을 소모한다"* 는 이유로 슬롯3을 자동 선택에서 제외한다. 설정 플래그 `CLAUDE_RELAY_ALLOW_SLOT3_IMPLICIT`(기본 off)로 분리해, 대표님이 켜기 전까지는 현 정책을 유지한다.

### P3. 사용량 가시화

**3단 구성**

| 층 | 산출물 | 주기 |
|---|---|---|
| 수집 | `scripts/codex_usage.py` — 계정 홈별 rollout에서 최신 `rate_limits` 추출 | cron 10분 |
| 저장 | `llm_api_keys.rate_limited_until` 갱신 + 사용률 스냅샷 적재 | 수집과 동시 |
| 표시 | CLI 출력 + 대시보드 카드 | 상시 |

**스냅샷 신선도 문제와 대책**

파일 수집만으로는 값이 낡는다. 한도에 걸린 호출은 사용률을 갱신해주지 않기 때문이다(실측: 최신 스냅샷이 50~121시간 전). 사용률이 가장 낮은 계정으로 최소 프롬프트를 하루 1~2회 실행해 값을 강제 갱신하는 **액티브 프로브**를 둔다. 계정당 수십 토큰 수준이라 소모는 무시 가능하다.

**대시보드 카드 — 상태별 목업**

실측 데이터로 그린다. 계정 단위로만 표시하며, 릴레이 세션 36개를 나열하지 않는다. 같은 숫자를 36번 보여주는 오해 유발 화면이 되기 때문이다.

정상:
```
┌─ 코덱스 사용량 ──────────────────────────────┐
│  MAIN  moongoby@gmail.com      Pro           │
│  ██████░░░░░░░░░░░░  32%   리셋 9/19 17:13   │
│  JINAH jswworld@gmail.com      Pro           │
│  ██░░░░░░░░░░░░░░░░   9%   리셋 9/22 14:43   │
│  갱신 4분 전                                  │
└───────────────────────────────────────────────┘
```

경고(80% 이상):
```
┌─ 코덱스 사용량 ──────────────────────────────┐
│  MAIN  moongoby@gmail.com      Pro       ⚠   │
│  ████████████████░░  91%   리셋 9/19 17:13   │
│  JINAH jswworld@gmail.com      Pro           │
│  ████░░░░░░░░░░░░░░  18%   리셋 9/22 14:43   │
│  MAIN 한도 임박 — 신규 세션은 JINAH로 배정됨  │
└───────────────────────────────────────────────┘
```

한도 초과:
```
┌─ 코덱스 사용량 ──────────────────────────────┐
│  MAIN  moongoby@gmail.com      Pro    ⛔정지 │
│  ████████████████████ 100%  9/19 17:13 복귀  │
│  JINAH jswworld@gmail.com      Pro           │
│  ██████████░░░░░░░░░  52%   리셋 9/22 14:43  │
│  가용 계정 1 / 2 · 최근 24h 한도실패 19건     │
└───────────────────────────────────────────────┘
```

데이터 없음:
```
┌─ 코덱스 사용량 ──────────────────────────────┐
│  수집된 사용량 기록이 없습니다                 │
│  마지막 시도 9/16 08:20 · 다음 수집 10분 후    │
│  [지금 수집]                                  │
└───────────────────────────────────────────────┘
```

전 계정 정지:
```
┌─ 코덱스 사용량 ──────────────────────────────┐
│  ⛔ 가용 계정 없음                            │
│  MAIN  100% · 9/19 17:13 복귀                 │
│  JINAH 100% · 9/22 14:43 복귀                 │
│  코덱스 호출은 모두 실패합니다                 │
└───────────────────────────────────────────────┘
```

### P3-1. 사용량은 실시간 조회가 원본이다 (2026-09-16 추가)

설계 당시 "codex CLI에 usage 서브커맨드가 없다"는 이유로 rollout 파일 수집만 검토했는데, 확인해 보니 **릴레이가 이미 실시간 조회 경로를 갖고 있었다**.

```
codex app-server  →  JSON-RPC  account/rateLimits/read
```

`claude_relay_server.py:_query_codex_rate_limits()`가 그것이다. `CODEX_HOME`을 계정 홈으로 지정하면 **계정별로** 물을 수 있다. 실측 응답:

```json
{"rateLimits": {"limitId": "codex", "planType": "pro",
  "primary": {"usedPercent": 100, "windowDurationMins": 10080, "resetsAt": 1789805594},
  "rateLimitReachedType": "rate_limit_reached"}}
```

그래서 수집 순서를 바꾼다. **실시간 조회가 1순위, rollout 수집이 폴백**이다. 이로써 P3의 "스냅샷 신선도" 문제와 액티브 프로브 계획이 함께 사라진다 — 토큰을 전혀 쓰지 않고 항상 현재 값을 얻는다.

### P5. 화면에서 재로그인 (2026-09-16 지시)

> 대표님 지시: "화면에 로그인이 필요한건 표시하고 재로그인버튼을 반영해서 재로그인이 가능하게 조치해줘"

**왜 릴레이가 CLI를 띄우나.** 자격증명 파일을 만들 수 있는 것은 각 CLI뿐이고, 그 파일은 호스트에 있다. API 컨테이너에는 `/root/.codex-accounts`도 `/root/.claude-relay-slots`도 마운트돼 있지 않다. 그래서 호스트에서 도는 릴레이가 CLI를 pty로 띄우고, 화면은 진행 상태만 받는다. 두 CLI 모두 TTY가 아니면 아무것도 출력하지 않아 pty가 필수다(실측).

**두 CLI의 흐름이 다르다.**

| | 코덱스 | 클로드 |
|---|---|---|
| 명령 | `codex login --device-auth` | `claude auth login --claudeai` |
| 화면에 띄울 것 | 인증 URL + 일회용 코드 | 인증 URL |
| 대표님이 할 일 | 브라우저에서 코드 입력 | 인증 후 받은 코드를 화면에 붙여넣기 |
| 완료 방식 | CLI가 스스로 폴링해 종료 | 코드를 stdin으로 돌려줘야 종료 |
| 플래그 | `needs_code: false` | `needs_code: true` |

**상태 전이**: `starting → awaiting_browser → (awaiting_code → verifying) → success / failed / expired / cancelled`

성공 판정은 출력 문구가 아니라 **파일**로 한다. 문구는 CLI 버전마다 바뀐다. 코덱스는 `tokens.access_token`, 클로드는 `refreshToken` 존재를 본다 — 클로드에서 refreshToken이 없으면 8시간 뒤 401로 죽으므로 성공으로 치면 안 된다.

**지키는 것**
- 일회용 코드 수명과 같은 15분에 프로세스를 죽인다
- 같은 대상에 진행 중인 세션이 있으면 새로 띄우지 않는다. 버튼 연타로 로그인 프로세스가 쌓이면 어느 것이 파일을 쓸지 알 수 없다
- 출력에서 토큰 모양(`sk-`, `rt.`, `ey…`)을 걸러 화면·로그로 새지 않게 한다 (R-KEY)
- 관리자 인증(`require_internal_admin`) + 릴레이 shared secret 이중으로 막는다

**경로**

```
대시보드 → API /llm-keys/account-login → 릴레이 /account-login → CLI(pty)
대시보드 → API /llm-keys/account-bindings → 릴레이 /account-bindings → 파일 존재 확인
```

### P4. 재발 방지

`scripts/check_account_binding.py` — 등록만 되고 런타임에 닿지 않은 계정을 검출한다.

- `llm_api_keys` 의 `provider IN ('codex','anthropic')` 각 행에 대응하는 계정 홈/슬롯 파일이 있는지
- 그 파일에 refresh token이 들어 있는지 (access token 단독이면 경고)
- 대응 파일이 최근 호출에 실제로 쓰였는지

일 1회 실행하고 불일치가 있으면 보고서에 올린다. 이번 건은 이 점검이 있었다면 9/15에 잡혔다.

---

## 4. 작업 분해

| # | 작업 | 담당 | 선행 |
|---|---|---|---|
| T1 | `/root/.codex-accounts/` 생성, MAIN 계정 홈으로 기존 auth.json 승계 | 자동 | - |
| T2 | `materialize_codex_accounts.py` — DB에서 JINAH 계정 홈 구성 | 자동 | T1 |
| T3 | `_build_codex_home()` 계정 선택·바인딩 개조 | 자동 | T2 |
| T4 | 한도 파싱 → `rate_limited_until` 기록 | 자동 | T3 |
| T5 | `codex_usage.py` 정식 배치 + cron 10분 | 자동 | T1 |
| T6 | slot3 골격 생성 | 자동 | - |
| T7 | **`claude auth login` 1회 실행** | **대표님** | T6 |
| T8 | `_build_claude_env()` 슬롯 확장 | 자동 | T7 |
| T9 | 대시보드 카드 | 자동 | T5 |
| T10 | `check_account_binding.py` + 일 1회 cron | 자동 | T5 |
| T11 | 릴레이 재기동 | **대표님 승인 후** | T3·T4·T8 |

---

## 5. 검증 계획

| 항목 | 방법 | 통과 기준 |
|---|---|---|
| JINAH 계정 바인딩 | JINAH 홈으로 `codex exec "1+1"` | 정상 응답 + rollout에 `token_count` 기록 |
| 계정 회전 | MAIN의 `rate_limited_until` 을 미래로 세팅 후 신규 세션 생성 | 세션이 JINAH로 배정됨 |
| 토큰 자동 갱신 | JINAH `auth.json` 의 `last_refresh` 변화 관찰 | CLI가 스스로 갱신하고 되씀 |
| 세션 고정 | 한 세션에서 다중 턴 실행 | 계정이 중간에 바뀌지 않음 |
| 슬롯3 | `HOME=slot3 claude -p "ping"` | 8시간 뒤 재실행에서도 401 없음 |
| 사용량 수집 | cron 1주기 대기 후 CLI 조회 | 계정별 사용률이 최신으로 표시됨 |
| 회귀 | 기존 세션 경로로 코덱스 호출 | 변경 전과 동일 동작 |

---

## 6. 리스크와 대응

| 리스크 | 영향 | 대응 |
|---|---|---|
| refresh token 회전으로 DB 사본 무효화 | JINAH 재구성 불가 | 파일을 진실의 원천으로 두고 스크립트가 갱신하지 않는다. DB는 씨앗 전용 |
| 릴레이 재기동 중 진행 중 세션 중단 | 작업 유실 | 대표님 승인 후 유휴 시간대 재기동. 코드 변경은 프로세스 재시작 전까지 무영향 |
| JINAH 계정 한도까지 소진 | 전 계정 정지 | P3 경보를 80%에서 울린다. 전 계정 정지 상태를 대시보드에 명시 |
| 타인 계정 소모 논란 | 운영 신뢰 | 슬롯3 암묵 선택은 기본 off. 계정별 사용량을 분리 표시해 누구 몫인지 항상 보이게 한다 |
| 계정 홈 권한 노출 | 자격증명 유출 | 0600 고정, 값은 로그·DB·보고서 어디에도 출력하지 않는다 |

## 7. 롤백

T3·T4·T8 은 모두 단일 파일 수정이다. `claude_relay_server.py` 를 변경 전 사본으로 되돌리고 릴레이를 재기동하면 원상복구된다. 계정 홈 디렉터리는 남겨도 무해하다(아무도 참조하지 않는 상태가 됨).

---

## 8. 대표님 결재 필요 항목

1. **T2·T7 — 로그인 2회.** 둘 다 자동화 불가로 확정됐다.
   - 코덱스 진아: DB의 refresh_token이 `refresh_token_reused`로 무효. 244 서버에서 현재
     `auth.json`을 재추출하거나 이 서버에서 `codex login`
   - 클로드 슬롯3: 저장값이 access token 단독이라 `claude auth login`
2. **T11 — 릴레이 재기동 시점.** 운영 중 프로세스다
3. **슬롯3 암묵 선택 허용 여부.** 현재는 타인 할당량 보호를 이유로 차단돼 있다
4. **코덱스 크레딧 구매 여부.** 계정 2개로 늘려도 9/19까지 MAIN은 정지 상태다
