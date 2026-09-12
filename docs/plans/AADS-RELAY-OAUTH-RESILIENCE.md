# AADS-RELAY-OAUTH-RESILIENCE — 채팅 인증 만료 장애 항구 대책 PRD

> **작성**: 2026-09-12 KST
> **대상**: `claude-relay.service` (`/root/aads/aads-server/scripts/claude_relay_server.py`) + `aads-server` 모델 폴백 경로
> **상태**: 슬롯 라우팅 복구 완료·서비스 정상. 자격증명 파일 방식은 **docker_wrapper 모드에서 성립하지 않음이 확인되어 철회**(§3.5)
> **노출 경로**: <https://aads.newtalk.kr/docs> (project_docs.py 자동 스캔, plan 카테고리)
> **관련 인시던트**: 2026-09-11 19:39 KST 1차, 2026-09-12 06:07 KST 2차(동일 원인 재발)

---

## 1. 배경

2026-09-12 오전, AADS 채팅이 응답하지 않는 장애가 발생했다. 증상은 사용자 화면에
빈 응답 말풍선만 남고 실행이 `interrupted`로 종료되는 것이었다.

인프라는 정상이었다. 컨테이너 전부 healthy, 활성 슬롯 green(8102) 정상 라우팅,
`/api/v1/health` ok, 릴레이 동시성 15/15 전부 여유. 장애는 전적으로
**LLM 폴백 체인 전 구간의 동시 인증·쿼터 실패**였다.

같은 증상이 전날 19:39 KST에도 발생했고("터미널은 되는데 aads만 멈춤"),
그때의 조치가 약 10시간 뒤 수명을 다하며 재발한 것이다. 본 문서는 이 반복을
끝내기 위한 설계를 다룬다.

---

## 2. 장애 실측

### 2.1 실패한 실행

| 시각(KST) | 세션 | 결과 |
|---|---|---|
| 09-12 09:33 | 8bf0405a | `resume_single_stream_error: All LLM providers failed` |
| 09-12 09:19 | 5090a247 | 동일 |
| 09-12 06:11 | e8a62756 | 동일 |
| 09-12 06:06 | e8a62756 | **마지막 정상 응답** |

### 2.2 4중 동시 실패

| # | 경로 | 상태 | 근거 |
|---|---|---|---|
| 1 | slot2 `moongoby@naver.com` (우선순위 1) | **401 revoked** | DB 토큰 직접 호출 시 `OAuth access token has been revoked` |
| 2 | slot1 `moong76@gmail` (우선순위 2) | **429 주간한도** | `weekly limit · resets Sep 16, 3am (Asia/Seoul)` |
| 3 | Codex CLI (`gpt-6-astra`/`gpt-5.6-sol`/`gpt-5.6-terra`) | **세션 만료** | `codex exec` 직접 실행 시 `refresh_token_invalidated` |
| 4 | LiteLLM 유료 경로 | **코드에서 비활성** | `model_selector.py:2143` "paid route disabled (CEO 지시)" |

**1번이 단일 근본 원인**이다. 같은 시각 `/root/.claude/.credentials.json`의 토큰은
opus·sonnet·haiku 전부 200 OK였다. 계정과 쿼터는 멀쩡했고, 갱신된 토큰이 DB로
전달되지 않은 것뿐이다.

### 2.3 오진 유발 요소 (기록용)

- `claude_ai_usage_fetch_failed: 403` — 토큰 문제가 **아님**. Cloudflare 챌린지
  페이지(`Just a moment…`) 응답이다. 인증 장애로 오인하기 쉽다.
- Codex 릴레이가 인증 실패를 **HTTP 200 + 0토큰**으로 반환하여, 로그에는
  `REPORT_STRUCTURE_WEAK` / `TOO_SHORT`(응답이 너무 짧음)로 위장되어 나타난다.
- `unified_healer`의 contabo14 서비스 실패 경고 10만 건대는 본 건과 무관한
  기존 노이즈다.

---

## 3. 근본 원인

### 3.1 릴레이가 갱신 불가능한 토큰을 나른다

`claude_relay_server.py:461 _build_claude_env()`는 HOME을 격리한 뒤
`CLAUDE_CODE_OAUTH_TOKEN`에 **액세스 토큰만** 주입한다.

```python
env["HOME"] = relay_home                    # /tmp/.claude-relay (공용)
env["CLAUDE_CODE_OAUTH_TOKEN"] = token      # 액세스 토큰만 — refreshToken 없음
```

이 환경변수에는 refreshToken이 담길 자리가 없다. 따라서 CLI는 갱신할 수단 자체가
없고, 액세스 토큰 수명(약 8시간)이 끝나면 그대로 죽는다.

### 3.2 CLI 레벨 검증 (2026-09-12 실측)

| 조건 | 결과 |
|---|---|
| `CLAUDE_CODE_OAUTH_TOKEN`=폐기 토큰 + 정상 자격증명 파일 | **401 실패** — env가 파일보다 우선 |
| env 변수 없음 + 정상 자격증명 파일 | 성공 |
| env 변수 없음 + 만료시각을 2023년으로 **강제 조작**한 파일 | **성공** — CLI가 refreshToken으로 자동 갱신 후 파일에 되써서 `expiresAt` 자동 전진 |

세 번째가 핵심이다. 자격증명 **파일**을 주면 CLI가 스스로 갱신하고 영속화한다.
사람 개입도, 동기화 잡도, DB 쓰기도 필요 없다.

### 3.3 잘못된 대응이 문제를 고착시켰다

2026-09-11 20:03 KST, 컨테이너 내부에서 `/tmp/update_token2.py`를 직접 실행하여
`llm_api_keys.ANTHROPIC_AUTH_TOKEN_2`의 값을 교체했다. 복사한 것은
`claudeAiOauth.accessToken` — 수명 몇 시간짜리 액세스 토큰이었다. refreshToken은
복사하지 않았고, 갱신분을 DB로 되돌리는 장치도 없다.

이 방식이 구조적으로 불가능함은 2026-09-12 작업 중 실증되었다. DB 재동기화 12분 뒤
운영자가 `/login`을 실행하자 방금 넣은 토큰이 **즉시 revoke**되어 채팅이 다시 끊겼다.
타이밍 문제가 아니라 설계 오류다.

부수적으로, 이 경로는 감사 대상인 `PUT /api/v1/llm-keys/{id}`를 우회하므로
`llm_key_audit_logs` 2,126행(4/23~현재) 어디에도 값 변경 기록이 없다. 추적 가능한
단서는 `notes` 필드의 `synced 2026-09-11 via terminal (naver Max account)`
문자열뿐이었다.

### 3.5 자격증명 파일 방식이 통하지 않는 이유 (2026-09-12 확인)

§4의 "슬롯별 자격증명 파일" 설계는 **CLI가 이 호스트에서 직접 실행될 때만** 성립한다.
운영 경로는 그렇지 않다.

```
릴레이(호스트) → claude-docker-wrapper.sh → docker exec → 컨테이너 내부 CLI
                 └ CLAUDE_CODE_OAUTH_TOKEN 만 -e 로 전달. HOME 은 전달하지 않음
                                               └ claude-oauth-wrapper.sh 가
                                                 HOME=/tmp/.claude-sdk 로 고정
```

따라서 `/root/.claude-relay-slots/slotN` 은 CLI에 **닿지 않는다**. 그 상태에서
`CLAUDE_CODE_OAUTH_TOKEN` 을 비우면 컨테이너 래퍼가 자기 `.env` 의
`ANTHROPIC_AUTH_TOKEN`(=slot1)으로 폴백하여 **어느 슬롯을 지정하든 slot1 로 나간다.**

컨테이너 래퍼의 주석이 이 회귀를 이미 경고하고 있었다:

> 이전 버전은 ANTHROPIC_AUTH_TOKEN 을 우선 사용해 relay 가 선택한 slot2 토큰을
> 덮어써 모든 요청이 slot1 로만 흘러갔고, slot1 의 seven_day 한도 소진 후
> 슬롯 폴백이 무효화됨

즉 이미 고쳐져 있던 결함을 되돌린 것이다. 결과로 slot2 요청이 slot1 계정으로
응답했고, **"두 계정 모두 주간 100%" 라는 관측 자체가 오독이었다** — slot2(naver)는
줄곧 14% 였다. 계정 상태를 릴레이 경유로만 측정하고 계정별 자격증명으로
교차 확인하지 않은 것이 오진의 직접 원인이다.

**현재 방침**: docker_wrapper 모드에서는 슬롯 토큰을 반드시 주입한다(`d2467a71`).
자격증명 파일 경로는 호스트 직접 실행일 때만 사용한다. 토큰 자동 갱신이라는
원래 목표를 이 구조에서 달성하려면 컨테이너에 슬롯별 자격증명을 마운트하고
`claude-oauth-wrapper.sh` 가 HOME 을 슬롯별로 잡도록 바꿔야 한다(미착수).

---

### 3.4 부수 결함

| 위치 | 결함 |
|---|---|
| `auth_provider.py:114 _assign_slots()` | 슬롯 번호를 **컨테이너 env 토큰 값과 문자열 비교**로 결정. DB 토큰만 교체하면 매칭이 깨져 두 계정이 같은 슬롯에 배정된다. 2026-09-12 복구 작업 중 실제 발생. |
| `model_selector.py:2069,2116` | `quota_reset_parsed`가 "resets Sep 16"를 파싱하고도 300초 쿨다운만 설정. `main.py` `rate_limit_recovery`가 1분마다 이를 다시 지워, 소진된 계정을 계속 재시도한다. |
| `chat_service.py:7305` | resume 체인을 `['claude-opus-5','gpt-6-astra','gpt-5.6-sol','gpt-5.6-terra']`로 선언하고도 chain_pos 0 실패 직후 `All LLM providers failed`로 종료. 1~3번은 시도조차 안 함. |
| `aads-relay-config-apply.service` | 2026-08-31부터 failed 상태 |

---

## 4. 해결책

> **원칙: 액세스 토큰을 나르지 말고, 갱신 능력을 나른다.**

### 4.1 슬롯별 자격증명 디렉터리

```
/root/.claude-relay-slots/          (700)
  README.md
  slot1/.claude/.credentials.json   ← moong76@gmail       (accessToken + refreshToken)
  slot2/.claude/.credentials.json   ← moongoby@naver.com
```

`/tmp/.claude-relay`가 아닌 `/root` 하위에 두는 이유는 재부팅 시 휘발을 막기 위함이다.

### 4.2 릴레이 변경

```python
def _build_claude_env(token, slot):
    relay_home = _ensure_slot_home(slot)        # 슬롯별 HOME
    env = {...}
    env["HOME"] = relay_home
    # CLAUDE_CODE_OAUTH_TOKEN 주입하지 않음 — CLI가 파일에서 읽고 스스로 갱신
    env["CLAUDE_CODE_MAX_OUTPUT_TOKENS"] = "16384"
    return env
```

§3.2 1행에서 확인했듯 env 변수가 파일보다 우선하므로, **반드시 주입을 제거해야 한다.**
남겨두면 파일이 정상이어도 401이 난다.

### 4.3 책임 재분배

| 저장소 | 역할 |
|---|---|
| 디스크(`/root/.claude-relay-slots`) | 자격증명 단일 출처. 갱신 주체는 CLI 자신 |
| DB(`llm_api_keys`) | 계정 메타데이터만 — `label`, `priority`, `rate_limited_until`. **토큰 값에서 손을 뗀다** |

refreshToken은 2026-10-09까지 유효하며 **갱신 때마다 함께 회전**하므로 만료일이 계속
밀린다. 사람이 손대지 않는 한 영구히 살아 있다.

이로써 `.env.oauth` 폴백 경로, 컨테이너 내 직접 SQL, `notes` 필드 수동 동기화 관행,
§3.4의 `_assign_slots` env 값 매칭 결함이 모두 불필요해지거나 자연 해소된다.

---

## 5. 단계별 적용 계획

### 1단계 — 슬롯 디렉터리 구축 (완료, 무중단)

- `/root/.claude-relay-slots/{slot1,slot2}/.claude/` 생성 (700)
- slot2에 자격증명 배치, 인증 실측 통과
- 릴레이 미변경(uptime 유지), 채팅 경로 정상 확인
- 작업 전후 `/root/.claude/.credentials.json` sha256 대조로 회전 없음 확인

### 2단계 — slot2 독립 인증 (완료, 무중단)

1단계의 slot2는 `/root/.claude`를 **복사**해 만들었으므로 두 곳이 **같은
refreshToken을 공유**했다. refreshToken은 사용 시 회전하므로 먼저 갱신하는 쪽이
다른 쪽을 무효화한다. 릴레이가 slot2를 쓰지 않는 동안은 무해하나 3단계로
연결하는 순간 실제 충돌이 되므로, 그 전에 독립 grant를 부여했다.

```
HOME=/root/.claude-relay-slots/slot2 claude auth login
```

같은 계정이라도 별도 로그인은 독립 grant이므로 서로 간섭하지 않는다.

> `claude setup-token`(장수명 토큰)도 존재하나, 본 장애의 교훈이
> "갱신 능력 없는 정적 토큰을 나르지 말 것"이므로 자동 갱신되는 `auth login`을 썼다.

**검증 결과 (2026-09-12)**

| 항목 | 결과 |
|---|---|
| slot2 자격증명 독립성 | accessToken·refreshToken **양쪽 모두 `/root/.claude`와 상이** |
| slot2 인증 | 통과 |
| **자동 갱신** | `expiresAt`을 2023년으로 강제 조작 후 호출 → 성공, `expiresAt` 자동 전진 |
| `/root/.claude` 무손상 | sha256 `f9a6e5287e8e5745` 작업 전후 동일, API 호출 200 |
| 채팅 경로 생존 | 전 과정 무중단 |

세 번째 행이 §6 합격 기준 2번을 실제 운영 위치에서 충족한 것이다. 이로써 남은
작업은 릴레이가 이 디렉터리를 바라보게 하는 것뿐이다.

### 3단계 — 릴레이 전환 (커밋 완료, 전환 대기)

**코드 (커밋 `38122fa5`, origin/main 푸시 완료)**

| 항목 | 내용 |
|---|---|
| `_ensure_slot_home(slot)` | 슬롯 자격증명 파일이 있으면 HOME 경로, 없으면 `None` |
| `_build_claude_env(token, slot)` | 파일 있음 → 슬롯 HOME + **`CLAUDE_CODE_OAUTH_TOKEN` 미주입** / 없음 → 기존 env 경로 |
| 가시성 | 호출마다 `claude_auth_source=` 로그, `/health`에 `slot_credentials` 노출 |

자격증명 파일이 없는 슬롯은 기존 경로를 유지하므로 **점진 전환**이며, 전환 중
어느 슬롯도 깨지지 않는다.

AUTH 보호 pre-commit 훅이 이 파일을 차단하므로 `ALLOW_AUTH_COMMIT=1`이 필요하다
(CEO 승인 게이트). `--no-verify` 우회는 AGENTS.md 9항 위반이므로 사용하지 않는다.

**전환 (완료, 무중단)**

`restart-claude-relay-when-idle.sh`는 `lease_count==0`을 15초(5초×3회) 연속
확인한 뒤에만 재시작하여 진행 중 대화를 보호한다. 1차 시도(240초)는 채팅 3건이
겹쳐 돌아 스킵되었고, 7200초 waiter로 재무장하여 **2026-09-12 10:51 KST 전환 완료**.

| 검증 항목 | 결과 |
|---|---|
| 기동 로그 | `slot1 auth source: slot_credentials` / `slot2 auth source: slot_credentials` |
| `/health` | `slot_credentials: {'1': True, '2': True}` |
| 호출 경로 | `Direct OAuth: slot=2 label=moongoby@naver.com` → `claude_auth_source=slot_credentials slot=2` |
| CLI 세션 | `oauth_slot=2`, `claude_auth_mode=direct` |
| 무중단 | 진행 중 대화 유실 0건 |

**전환 자체는 적용되었으나, 이 방식은 현 실행 구조에서 성립하지 않는다.** §3.5 참조.

### 4단계 — 이중화 복원 (Claude 완료, Codex 미완)

`claude auth login`을 슬롯 HOME으로 격리 실행하여 두 계정 모두 독립 grant 확보.

| 슬롯 | 계정 | 상태 |
|---|---|---|
| slot1 | `moong76@gmail.com` | 인증 완료. 계정이 주간한도(2026-09-16 03:00 KST 해제) — **인증이 미리 되어 있으므로 해제 시각에 자동 복귀** |
| slot2 | `moongoby@naver.com` | 인증 완료, 가동 중 |

`/root/.claude`·slot1·slot2 세 자격증명의 refreshToken이 **상호 전부 상이**함을
확인했다. 회전 충돌 가능성이 구조적으로 제거되었다.

> 계정 정체는 추정하지 않고 `claude auth status`로 직접 확인했다. slot1의
> `rateLimitTier`가 과거 기록(`max_5x`)과 달리 `max_20x`로 나와 동일 계정 중복
> 로그인을 의심했으나, 이메일 확인 결과 정상(계정이 그 사이 업그레이드됨).

**미완**: Codex 재로그인(`codex login`). 대화형이며 URL 중계가 불가능한 방식이라
운영자 직접 수행이 필요하다. 이것이 되어야 Claude 양 계정 소진 시 받아줄 곳이 생긴다.

---

## 6. 합격 기준

| # | 기준 | 검증 방법 |
|---|---|---|
| 1 | 슬롯별 인증 동작 | `/stream`에 `oauth_slot=1,2` 각각 호출 → `is_error:false` |
| 2 | 자동 갱신 동작 | 슬롯 자격증명 `expiresAt`을 과거로 조작 후 호출 → 성공 + `expiresAt` 자동 전진 |
| 3 | **무개입 만료 통과** | 액세스 토큰 만료 시점을 사람 개입 없이 통과 |
| 4 | DB 비의존 | `llm_api_keys`의 토큰 값이 낡아도 채팅 정상 |
| 5 | 무중단 | 전환 중 진행 대화 유실 0건, 5분 P0/P1 모니터링 무오류 (AGENTS.md 8항) |

3번이 진짜 합격선이다. 1·2번은 통과하고도 3번에서 실패하면 대책이 아니다.

---

## 7. 현재 상태 (2026-09-12 마감 시점)

| 슬롯 | 계정 | 주간 사용률 | 리셋 |
|---|---|---|---|
| slot1 | `moong76@gmail` | 100% (rejected) | 2026-09-16 03:00 KST |
| slot2 | `moongoby@naver.com` | **14% (allowed)** — 현재 서비스 담당 | 2026-09-13 00:00 KST |
| Codex | ChatGPT Pro | 100% | 2026-09-16 15:05 KST |
| LiteLLM 유료 | — | — | 코드 비활성(CEO 지시) |

채팅은 slot2 로 정상 동작한다. 두 계정이 동시에 막힌 적은 없었다 — 그렇게 보인 것은
§3.5 의 오독이었다.

### 오늘 드러난 구조적 결함 (모두 조치됨)

| 결함 | 영향 | 조치 |
|---|---|---|
| 슬롯을 토큰 "값" 으로 배정 | 우선순위가 흔들리면 두 계정이 한 슬롯에 접혀 교차 폴백 소멸 | `063a88fd` 키 이름 기준 배정 |
| 슬롯 충돌을 조용히 폐기 | 폴백이 1계정으로 줄어도 로그 없음 | 같은 커밋에서 경고 추가 |
| 컨테이너 모드 토큰 미주입 | 전 요청이 slot1 로 | `d2467a71` |
| `claude_max_usage_snapshot` 무한 증가 | 51.8만 행 / 238MB | `8a5fba35` 30일 보존 |
| 계정별 쿼터 가시성 없음 | 소진까지 아무도 못 봄 | `3a2ad43d` 수집·표시·70/90% 경보 |
| `codex_auth_sync.sh` 미등록 + 결함 3건 | Codex 인증 폐기 방치 | `9c5d01e8` + 크론 등록 |
| nginx `/reports` include 중복 | **모든 배포 cutover 차단, 재시작 시 전체 기동 실패** | 중복 2줄 제거 후 reload |

### 남은 과제

| 과제 | 내용 |
|---|---|
| 토큰 자동 갱신 미달성 | §3.5 — 컨테이너에 슬롯별 자격증명 마운트가 필요. 현재는 DB 액세스 토큰에 의존 |
| resume 체인 조기 종료 | §3.4 — 폴백 체인 4개를 선언하고 1개만 시도 |
| 슬롯 2개 고정 | `_ACCOUNT_SLOTS` 가 1·2만 상정. 계정 증설이 코드 변경을 요구 |
| 단일 계정 의존 | 9/16 까지 slot2 하나. 여기가 막히면 받아줄 곳이 없다 |

## 8. 운영 수칙 (신규)

1. 액세스 토큰을 DB·환경변수·설정파일에 **복사하지 않는다.** 자격증명 파일을 배치한다.
2. `llm_api_keys`는 메타데이터 전용이다. 토큰 값 변경이 필요하면
   `PUT /api/v1/llm-keys/{id}`를 쓴다 — 컨테이너 내 직접 SQL은 감사 로그를 우회한다.
3. 인증 장애 1차 판별: `/root/.claude/.credentials.json` 토큰으로
   `api.anthropic.com/v1/messages` 직접 호출. 200이면 계정이 아니라 **전달 경로** 문제다.
4. `claude_ai_usage_fetch_failed: 403`은 인증 장애 신호가 아니다(Cloudflare).
5. 릴레이 재시작은 반드시 `restart-claude-relay-when-idle.sh` 경유.
6. 계정 상태는 **계정별 자격증명으로 직접** 확인한다. 릴레이 경유 관측만으로
   판단하면 라우팅 결함이 계정 소진으로 오독된다(§3.5).
7. 인증 경로를 바꾸기 전에 **CLI가 어디서 실행되는지**(`claude_cmd_mode`) 먼저
   확인한다. 호스트 실행과 컨테이너 실행은 환경 전달 규칙이 다르다.
8. 배포가 막히면 게이트를 우회하지 말고 차단 사유를 제거한다. 2026-09-12 의
   #325·#326 차단은 둘 다 실제 결함(미추적 파일, nginx 중복 include)이었다.
9. **검증 호출이 의도한 코드 경로를 실제로 통과했는지 로그로 확인한다.**
   2026-09-12 에 같은 형태의 오판이 세 번 반복되었다.
   - 슬롯 자격증명 전환을 "성공"으로 보고했으나 컨테이너 래퍼가 무효화하고 있었다
   - 계정 상태를 릴레이 경유로만 보고 "두 계정 모두 소진"이라 판단했으나 slot2 는 14% 였다
   - 슬롯별 세션을 검증하며 필드명을 `aads_session_id` 로 잘못 보내(릴레이는
     `session_id` 를 읽는다) 세션 없는 경로만 통과시켰고, 그 사이 실제 요청은
     500 을 받고 있었다. 로그의 `aads=none` 이 이미 경고하고 있었다.
   통과했다는 결과값이 아니라 **경로 표식**(`claude_auth_source=`, `aads=`,
   `resume=`, `oauth_slot=`)을 확인하고 나서 성공으로 판단한다.
