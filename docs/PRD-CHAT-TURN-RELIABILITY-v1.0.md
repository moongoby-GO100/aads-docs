# PRD — 채팅 턴 신뢰성: 잠금 기아·재개 루프·추가지시 유실

| 항목 | 내용 |
|---|---|
| 문서 ID | PRD-CHAT-TURN-RELIABILITY-v1.0 |
| 작성일 | 2026-09-16 (KST) |
| 지시 | 대표님 — "채팅창 응답이 너무 느린데 원인 파악이 가능한가?" → "원인 규명우선 진행해" → "신규 생성한 채팅창도 응답을 못하는데" → "진행하고 오류기록하고 설계 PRD파일 작성저장하고 구현 배포진행해" |
| 대상 | `aads-server` (`scripts/claude-docker-wrapper.sh`, `app/main.py`, `app/services/chat_service.py`, `app/routers/chat.py`) |
| 오류 사전 | `chat.slot_credential_lock_starvation`, `chat.resume_scanner_epoch_loop`, `codex.refresh_token_reused` |

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
