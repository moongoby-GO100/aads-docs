# PRD — 채팅 턴 신뢰성: 잠금 기아·재개 루프·추가지시 유실

| 항목 | 내용 |
|---|---|
| 문서 ID | PRD-CHAT-TURN-RELIABILITY-v1.0 |
| 작성일 | 2026-09-16 (KST) |
| 지시 | 대표님 — "채팅창 응답이 너무 느린데 원인 파악이 가능한가?" → "원인 규명우선 진행해" → "신규 생성한 채팅창도 응답을 못하는데" → "진행하고 오류기록하고 설계 PRD파일 작성저장하고 구현 배포진행해" |
| 대상 | `aads-server` (`scripts/claude-docker-wrapper.sh`, `app/main.py`, `app/services/chat_service.py`, `app/routers/chat.py`) |
| 오류 사전 | `chat.execution_never_reaped`, `chat.slot_credential_lock_starvation`, `chat.resume_scanner_epoch_loop`, `codex.refresh_token_reused` |

---

## 0. 무엇을 고치나

하루 동안 채팅이 느리다는 신고에서 시작해, **서로 다른 세 가지 결함**이 나왔다.
셋 다 "느리다"로 보였지만 원인과 조치가 다르다.

| # | 결함 | 증상 | 범위 |
|---|---|---|---|
| A | 자격증명 잠금 기아 | **전 세션 무응답** (신규 포함) | 전역 |
| B | 재개 스캐너 epoch 루프 | 한 턴이 수백 회 재시도, 한도 소모 | 해당 턴 |
| C | 추가지시가 진행 중 답변을 밀어냄 | 만들던 응답이 버려짐, B의 방아쇠 | 해당 세션 |

여기에 **구조적 지연**(모델·프롬프트 크기)이 배경으로 깔려 있다(6절).

---

## 1. A — 자격증명 잠금 기아 (가장 심각)

### 실측

```
/root/.claude-relay-slots/slot1/.claude/.credentials.lock

PID 1977848   FLOCK WRITE(배타) 보유 — 6분 48초
  ↓ 대기
9개 프로세스가 flock -s 에서 정지    ← 신규 채팅창 포함
```

신규 세션 `4841c4fb`는 191초를 기다리다 `interrupted`로 끝났다.

**모델·계정·릴레이는 모두 정상이었다.** 같은 계정으로 CLI 를 직접 부르면 4.66초에
응답했고(opus-5), 릴레이 여유 슬롯 15/24, slot1 5시간 한도 14% 사용.

### 원인

`claude-docker-wrapper.sh`

```bash
if credential_requires_exclusive_lock "$CREDENTIAL_FILE"; then
    flock -x 9          # 갱신이 필요하면 배타
else
    flock -s 9
fi
...
docker exec ... claude -p ...   # ← 모델 호출이 끝날 때까지 안 놓는다
```

배타 잠금의 **의도**는 옳다. 토큰이 곧 만료될 때 두 호출이 동시에 갱신하면
refresh token 이 서로를 무효화한다. 그래서 하나만 갱신하게 직렬화한다.

문제는 **쥐고 있는 시간**이다. 갱신에 실제로 필요한 것은 몇 초인데, 잠금은
프로세스가 끝날 때 풀린다. opus 호출이 수 분이면 그동안 같은 슬롯의 모든
세션이 멈춘다. 대기에 상한이 없어 **한 호출의 지연이 전체 장애가 된다.**

여기에 갱신 창이 넓은 것이 겹쳤다.

```bash
REFRESH_LOCK_WINDOW_SEC=6000    # 100분
```

토큰 수명이 약 8시간이니 **수명의 20% 구간**에서 배타 잠금이 걸린다.

### 설계

두 가지를 함께 바꾼다. 하나만으로는 부족하다.

**① 대기에 상한을 둔다** — 장애 반경을 시간으로 묶는다.

```bash
_excl_wait=${CLAUDE_SLOT_EXCLUSIVE_LOCK_WAIT_SEC:-300}
_shared_wait=${CLAUDE_SLOT_SHARED_LOCK_WAIT_SEC:-120}

배타 요청:  flock -x -w $_excl_wait  실패 → 공유로 강등
공유 요청:  flock -s -w $_shared_wait 실패 → 잠금 없이 진행 (경고 기록)
```

- 배타를 못 잡았다 = 다른 호출이 이미 갱신 중이다. 그 결과를 쓰면 되므로
  공유로 내려가는 것이 맞다.
- 공유마저 시간이 차면 **잠금 없이 진행한다.** 근거: 자격증명은 진입 시점에
  `validate_credential` 로 이미 검증했고, 되쓰기는 `.sync` 잠금과 digest 비교가
  따로 지킨다(`sync_container_credential`). 막혀서 못 하는 것보다 낫다.

**② 배타가 켜지는 창을 좁힌다** — 애초에 덜 걸리게 한다.

```bash
REFRESH_LOCK_WINDOW_SEC: 6000초(100분) → 600초(10분)
```

갱신에 필요한 것은 몇 초다. 배타 구간이 토큰 수명의 20% → 2% 로 줄어든다.

### 하지 않기로 한 것

**배타 잠금 제거.** 갱신 직후 공유로 강등하는 안도 검토했으나, 그러면 만료
임박 구간에서 여러 호출이 동시에 갱신해 refresh token 이 서로를 무효화할 수
있다. 관측된 장애(전역 무응답)는 시간 상한으로 막을 수 있고, 제거는 아직
관측된 적 없는 다른 장애를 부를 수 있다. **관측된 것을 고치고 관측되지 않은
것은 건드리지 않는다.**

### 검증

| 항목 | 기준 |
|---|---|
| 대기 상한 | 배타 보유 중 공유 요청이 설정 시간에 진행 (실측: 3초 설정 → 3초 후 진행) |
| 전역 무응답 | `/proc/locks` 에 WRITE 보유가 있어도 신규 세션이 응답 |
| 갱신 정합 | 만료 10분 이내에서만 배타 발생 |

---

## 2. B — 재개 스캐너 epoch 루프

### 실측

```
실행 c0189a7c (세션 #119 상한가따라잡기 전략관리자)
  owner_epoch 420 · 3시간 51분 · 누적 내용 93자
  6시간 기준: 실패 세대 1,091건 대 완료 73건
  정리 대상 총 71건, 최대 epoch 524
```

### 원인

```
① 추가지시가 턴을 밀어냄 → status='retrying' 로 남음
② 5초 주기 재개 스캐너가 running/retrying 을 집어감  (main.py:3054)
③ _claim_execution_lease 가 owner_epoch 만 +1, retry_count 는 그대로
④ 재개해도 같은 자리에서 다시 밀려남 → ②로 복귀
```

재시도 예산 가드는 `retry_count >= 5` 를 본다(`chat_service.py:1246`). 그런데
스캐너 경로는 `retry_count` 를 올리지 않는다. **실측 owner_epoch 524 대
retry_count 1** — 가드가 영영 발동하지 않았다.

### 설계

**상한을 소유권을 주는 자리 한 곳에 둔다.** 처음에는 스캐너 조회에만 걸었는데
구멍이 남았다 — `interrupted` 로 내린 실행을 **다른 호출자**가 2초 만에 다시
집어가 epoch 가 419 → 420 으로 올랐다. `_claim_execution_lease` 호출부는
여섯 군데다. 조회마다 막으면 반드시 하나를 빠뜨린다.

```sql
WHERE id = $1
  AND status IN ('running','retrying','interrupted')
  AND ($6::boolean OR COALESCE(owner_epoch,0) < $7::int)   -- 기본 50
```

- 정상 운영은 한 자릿수, 배포 전환이 겹쳐도 수십을 넘지 않는다.
- **수동 재개는 예외**(`allow_any_epoch`). 상한은 자동 재개가 무한히 도는 것을
  막으려는 것이지, 사람이 버튼을 눌러 되살리는 복구까지 막을 이유는 없다.

스캐너 조회에는 `superseded`·`newer_user` 사유 제외를 함께 둔다(이중 방어).

---

## 3. C — 추가지시가 진행 중 답변을 밀어냄

### 실측

```
최근 2일 추가지시 49건 중 45건이 '더 새 사용자 메시지' 로 오판정
코드 주석 기록: 54,301자짜리 진행 중 응답이 버려짐 (session_relay.py:133)
```

### 원인

판정을 **접두 문자열**로 했다.

```sql
AND content NOT LIKE '[추가 지시]%'    -- 닫는 대괄호를 바로 요구
```

그런데 실제 접두는 계속 는다. 2026-09-16 하루에만 네 가지가 나왔다.

```
[추가 지시]
[추가 지시 · 12:26 KST]
[CEO 지시 · 마일스톤 확정 · 09:05 KST · 같은 건 보강, 새 질문 아님]
[CEO 승인 · 배포 지시 · 08:57 KST]
```

뒤 셋은 패턴에 걸리지 않아 **평범한 새 사용자 메시지**로 취급됐고, 진행 중
턴을 `stale_superseded_by_newer_user_message` 로 중단시켰다. 그것이 B의 방아쇠다.

### 설계

**판정을 `intent` 로 옮긴다.** intent 는 접수 시점에 이미 찍혀 있어 접두가
무엇이든 새지 않는다.

```sql
AND COALESCE(intent,'') NOT IN (
    'system_trigger','queued_interrupt','interrupt_completed',
    'recovered_interrupt','interrupt_expired'
)
```

적용 후 '밀어내기로 판정되는 추가지시' **45건 → 0건**.

이 하나로 "이어쓰기"가 함께 해결된다. 추가지시가 더 이상 턴을 밀어내지 않으므로
만들던 답변이 버려지지 않고 그대로 이어진다. **별도 이어쓰기 엔진이 필요 없다.**

### 곁들인 것

- 수거 SQL·정규식 접두를 `[추가 지시%` 로 넓혀, 프로세스 메모리에만 있던
  45건이 DB 에서 복구되게 했다. 스트림 단절·재시작 시 소실되던 경로다.
- 중복 차단 창 30초 → 120초, 접두를 뗀 본문으로 비교. 실측 중복쌍이 92초
  벌어져 있었고 접두가 서로 달라 둘 다 못 잡았다. 적용 기준 11건이 걸린다.

---

## 4. 작업 목록

| # | 내용 | 파일 | 상태 |
|---|---|---|---|
| A1 | flock 대기 상한 (배타 300초 / 공유 120초) | `scripts/claude-docker-wrapper.sh` | 구현 |
| A2 | 갱신 창 6000초 → 600초 | 〃 | 구현 |
| B1 | 스캐너 조회에 사유 제외 + epoch 상한 | `app/main.py` | 배포 완료 |
| B2 | `_claim_execution_lease` 에 epoch 상한(50) | `app/services/chat_service.py` | 구현 |
| B3 | 수동 재개는 `allow_any_epoch` 예외 | `app/routers/chat.py` | 구현 |
| C1 | 추가지시 판정을 intent 기준으로 | `app/services/chat_service.py` | 배포 완료 |
| C2 | 수거 SQL·정규식 접두 확장 | 〃, `app/routers/chat.py` | 배포 완료 |
| C3 | 중복 차단 창·정규화 | `app/routers/chat.py` | 배포 완료 |

---

## 4-1. D — 버려진 실행이 수거되지 않음 (2026-09-17 추가)

### 실측

```
세션 #310 전파동 사이클 — 답변 버블이 아예 안 나옴
  06:37 사용자 메시지 → 실행 3e650f17
        status=interrupted · assistant_message_id = NULL
        → 그릴 것이 없어 버블 자체가 없었다

같은 세션에 남아 있던 실행들의 나이
  13.7h · 42.9h · 48.4h · 58.2h · 65.7h
```

한 실행의 재개 간격을 보면 성격이 드러난다.

```
attempt 1   09-16 17:19:36
attempt 2   09-17 07:01:54   ← 13시간 42분 휴면
attempt 3~13  6초 간격으로 폭주
```

### 원인 — 그물은 있었고, 이들만 그물을 빠져나갔다

처음에는 "종결을 찍는 주체가 없다" 고 적었다. **틀렸다.** 검증 중에
`stale_execution_watchdog` 이 90초마다 돌며 running/retrying 을 `interrupted`
로 내리고 있는 것이 확인됐다 — 합성 행을 넣었더니 6분 만에 처리했다.

빠져나간 이유는 워치독 후보 조회에 있다.

```python
_claimable = [r for r in candidates
              if str(r["session_id"]) not in _active_sids]   # ← 여기
```

`_active_sids` 는 `_active_bg_tasks` — **프로세스 로컬 표**다. 재개 스캐너가
5초마다 그 세션의 백그라운드 태스크를 새로 띄우면 세션은 영원히 "진행 중"
으로 보이고, 워치독은 영원히 건너뛴다. 게다가 이 표는 슬롯이 바뀌면 비므로
블루/그린 전환 뒤에는 근거 자체가 사라진다.

실측이 이를 뒷받침한다.

| 항목 | 값 |
|---|---|
| 살아남은 12건의 `watchdog_settled_without_retry` | **0건** (워치독이 손댄 적 없음) |
| `owner_epoch` | 2 → 17 (반복 claim) |
| `retry_count` | 0 ~ 2 (스캐너 경로는 이 값을 올리지 않는다 — B 참고) |

재개 스캐너의 `updated_at > NOW()-2h` 창은 이들을 **조회에서 뺄 뿐** 종결
시키지 않는다. 행은 `retrying` 인 채 살아 있다가 무언가 건드리면 되살아났다.

### 설계 — 세 겹

하나만으로는 또 샌다. 어제 스캐너 조회만 막았다가 다른 호출자가 2초 만에
되살린 전례가 있다(B 참고).

| # | 지점 | 내용 |
|---|---|---|
| 1 | 스캐너 조회 | `started_at` 기준 나이 상한. `updated_at` 은 재개 때마다 갱신돼 무력하다 |
| 2 | `_claim_execution_lease` | 같은 상한. 조회를 막아도 소유권을 주면 되살아난다 |
| 3 | 수거기 (5분 주기) | 오래되고 **멈춰 있는** `running`/`retrying` 을 **`cancelled`** 로 종결 |

**`cancelled` 여야 한다.** `interrupted` 는 `_claim_execution_lease` 의 WHERE 에
그대로 들어 있어 다시 집어간다.

**`_active_bg_tasks` 로 거르지 않는다.** 그 표가 바로 이 사고의 사각지대다.
대신 DB 에 남는 하트비트로 "정말 멈췄나" 를 본다.

**수거 대상에 `interrupted` 를 넣지 않는다.** 전수로 잡으면 과거 기록
5,580건(최고 145일)을 다시 쓰게 된다. 오래된 `interrupted` 의 부활은 2번이 막는다.

**자리표시자는 지우지 않는다.** 내용이 있으면 `interrupted_partial` 로 살리고,
없으면 안내 문구로 바꾼다. 지우면 대표님이 보시는 것이 정확히 이 절의 증상
— "버블이 아예 안 나온다" — 이 된다.

### 나이만으로 자르면 정상 턴을 죽인다

처음 잡은 2시간은 **위험한 값이었다.** 검증 중 실측.

```
최근 30일 완료된 턴 4,284건
  p99   77.4분
  최대  289.5분 (4시간 50분)
  2시간 초과 정상 완료  8건
```

2시간이면 이 8건을 잘랐다. 그런데 같은 8건을 다시 보면 답이 나온다.

```
실행시간 289.5분 → 하트비트 289.5분까지 갱신
        218.2분 →         218.2분까지
        213.8분 →         213.8분까지     (8건 전부 동일)
```

**살아 있는 하트비트가 곧 "진짜 일하는 중" 이다.** 그래서 수거 조건을 셋으로
한다 — 셋 다 만족해야 자른다.

```sql
WHERE status IN ('running', 'retrying')
  AND started_at < NOW() - ($1 * INTERVAL '1 hour')          -- 기본 6시간
  AND (lease_expires_at IS NULL OR lease_expires_at < NOW())
  AND COALESCE(heartbeat_at, updated_at, started_at)
        < NOW() - ($2 * INTERVAL '1 minute')                 -- 기본 30분
```

운영 데이터로 확인: 나이 문턱을 **0시간**으로 낮춰 돌려도 현재 실행 중인 턴은
하나도 걸리지 않는다(리스·하트비트가 살아 있으므로).

두 곳이 `AADS_EXECUTION_MAX_AGE_HOURS`(기본 6) 하나를 같이 읽는다. 한쪽만
바꾸면 수거되기 전에 되살아나는 창이 생긴다.

### 회귀 방지

`tests/unit/test_execution_lease_contract.py` 가 세 겹과 실측 기본값을 검사한다.
기본값을 낮추려면 위 지속시간 실측부터 다시 해야 테스트가 통과한다.

---

## 5. 운영 복구 절차

증상별로 어디를 보는지 적어 둔다. 오늘 전부 실제로 쓴 절차다.

**전 세션 무응답**
```bash
grep FLOCK /proc/locks | grep WRITE          # 배타 보유자 확인
ls -l /proc/<pid>/fd | grep credentials.lock # 어느 슬롯인지
kill -TERM <pid>                             # 종료하면 즉시 대기열이 풀린다
```

**한 턴이 끝없이 재시도**
```sql
SELECT id, owner_epoch, retry_count, status FROM chat_turn_executions
 WHERE status IN ('running','retrying') AND owner_epoch > 20;
-- interrupted 로 내리면 다시 잡힌다. cancelled 로 내려야 끊긴다.
UPDATE chat_turn_executions SET status='cancelled', completed_at=NOW() WHERE ...;
```

**스피너가 멈춰 있음** — 내용이 있으면 `interrupted_partial` 로 살리고, 없으면
`interruption_notice` 로 바꾼다. 내용을 지우지 않는다.

---

## 6. 남은 구조적 지연 (이 PRD 범위 밖)

위 셋을 고쳐도 체감 지연의 큰 몫이 남는다. 실측값이다.

| 요인 | 측정 | 조치 시 영향 |
|---|---|---|
| 모델이 opus-5 (95턴 중 80턴) | 같은 한 줄 프롬프트에 opus 11.6초 vs sonnet 4.5초 | 품질 판단 필요 |
| 도구 142개 = 76,245자(≈25,000토큰) 매 턴 주입 | MCP 기동 자체는 0.3초로 쌈 | 도구 누락 위험 |
| 프롬프트 크기 | 중앙값 19,031자, 최대 416,441자. 신규 세션 한 줄 질문에도 36,271자 | 맥락 손실 위험 |
| 히스토리 | 턴당 71~118개 메시지 | 〃 |

첫 토큰까지 평균 39.9초, 턴 전체 중앙값 254초다. **대표님이 물으신
"컨텍스트·프롬프트를 맥락순 순차 투입 또는 선검색 후 반영"이 정확히 이
문제를 겨냥한다** — 별도 설계로 다룬다.
