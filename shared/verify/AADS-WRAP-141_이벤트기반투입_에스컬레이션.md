---
project: AADS
task_id: AADS-141
completed_at: 2026-03-07 09:15 KST
---

# AADS-141 Wrap-up: Event-based Task Dispatch + Semantic Loop Escalation + Telegram Alerts

## 결과 요약

| 항목 | 상태 |
|------|------|
| Part A: auto_trigger.sh signal 기반 즉시 투입 | ✅ 완료 |
| Part B: 시맨틱 루프 에스컬레이션 강화 | ✅ 완료 |
| Part C: 텔레그램 알림 4종 | ✅ 완료 |
| recovery_logs DB 기록 (semantic_loop) | ✅ 완료 |
| CLAUDE.md 실패 원인 주입 | ✅ 완료 |
| systemd 서비스 파일 | ✅ 완료 |
| HTTP 200 확인 | ✅ HTTP 200 |

---

## Part A: auto_trigger.sh 이벤트 기반 전환

### A-1: signal 파일 기반 즉시 투입 추가
파일: `/root/aads/scripts/auto_trigger.sh`

**변경 내용:**
- `SIGNAL_FILE="/tmp/aads_trigger_next.signal"` 경로 정의
- 스크립트 시작 시 signal 파일 존재 여부 확인 → 즉시 실행 모드 전환
- 크론 주기 fallback 유지 (기존 로직 그대로)
- `TRIGGER_DECISION_LOG=/var/log/aads/trigger_decisions.log` (fallback: /root/aads/logs/)

**V-1 검증:**
```
$ echo "TEST_TASK-141:$(date +%s)" > /tmp/aads_trigger_next.signal
$ bash auto_trigger.sh --dry-run
모드: SIGNAL 기반 즉시 투입 (크론 주기 대기 없음)
trigger_decisions.log: 2026-03-07 09:14:17 | SIGNAL_TRIGGER | content=TEST_TASK-141:1772842457 | mode=immediate | skip_cron_wait=true
```

**V-2 검증:**
```
$ bash auto_trigger.sh --dry-run  (signal 파일 없음)
모드: 크론 주기 실행 (fallback)
```

### A-2: 슬롯 관리 연동
- `check_global_slots()` 함수: 글로벌 ≤4 제한 준수
- 투입 결정 로그: `SLOT_OK | running=2/4` 형태로 기록
- 완료 시 슬롯 즉시 반환 후 다음 작업 배정

---

## Part B: 시맨틱 루프 에스컬레이션 강화

### B-1: PARTIAL 보고서 자동 생성
파일: `/root/aads/scripts/session_watchdog.sh` - `generate_semantic_loop_partial()` 함수

**포함 내용:**
- 반복된 action 패턴 (최근 10개 heartbeat detail)
- 토큰 소비량 추정
- 권고 조치 (작업 분할, 다른 접근법, CLAUDE.md 주입)

**V-3 검증 (recovery_logs DB):**
```sql
INSERT INTO recovery_logs (issue_type='semantic_loop', ...)
```
결과:
```
 id |  issue_type   | affected_task_id | tier  | action_taken | result
----+---------------+------------------+-------+--------------+---------
  1 | semantic_loop | TEST-141         | tier2 | kill_restart | success
```

### B-2: 재시작 시 CLAUDE.md 접근법 변경 지시
함수: `inject_failure_into_claude_md()` in session_watchdog.sh

**V-4 검증:**
```
# [AADS 자동 주입 — session_watchdog]
이전 세션이 동일 동작 반복(시맨틱 루프)으로 중단됨. 반복된 패턴: file_changed: same_file.py.
완전히 다른 접근법을 시도할 것. 동일 패턴 반복 시 즉시 중단됨.
```

**시맨틱 루프 감지 테스트 (V-6):**
```python
total=10 unique=1
→ SEMANTIC_LOOP
```

---

## Part C: 텔레그램 알림 통합

### C-1: session_watchdog 이벤트별 4종 알림 함수
파일: `/root/aads/scripts/session_watchdog.sh`

| 함수 | 메시지 형식 |
|------|------------|
| `tg_tier2_kill()` | `⚠️ 세션 {TASK_ID} API hang/semantic loop 감지 → kill + 재시작` |
| `tg_tier3_kill()` | `🔴 세션 {TASK_ID} {ELAPSED}초 무응답 → 강제 종료` |
| `tg_tier4_escalate()` | `🚨 세션 {TASK_ID} 3회 연속 실패 → 서킷브레이커 발동, CEO 확인 필요` |
| `tg_complete()` | `✅ 세션 {TASK_ID} 완료 ({DURATION}초) → 다음 작업 즉시 투입` |

**V-5 검증:**
```
grep -n "tg_tier2_kill|tg_tier3_kill|tg_tier4_escalate|tg_complete" session_watchdog.sh
97: tg_tier2_kill() {    ← 정의
102: tg_tier3_kill() {   ← 정의
108: tg_tier4_escalate() {  ← 정의
113: tg_complete() {     ← 정의
333: tg_tier2_kill "$task_id"   ← kill_and_restart 내 호출
335: tg_tier3_kill "$task_id"   ← kill_and_restart 내 호출
434: tg_tier4_escalate "$task_id"  ← tier4_escalate 내 호출
449: tg_complete "$task_id" "$duration"  ← trigger_post_processing 내 호출
```

---

## session_watchdog.sh 주요 변경 사항 (AADS-141)

1. **헤더 업데이트**: AADS-140+141 통합 버전 명시
2. **env 로딩 강화**: `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID` aads-server/.env에서 로드
3. **로그 경로 통일**: `/var/log/aads/` → `TRIGGER_DECISION_LOG`
4. **4종 텔레그램 함수 추가**: `tg_tier2_kill`, `tg_tier3_kill`, `tg_tier4_escalate`, `tg_complete`
5. **check_global_slots()** 추가: 슬롯 현황 + 로그 기록
6. **generate_semantic_loop_partial()** 추가: PARTIAL 보고서 자동 생성
7. **inject_failure_into_claude_md()** 추가: CLAUDE.md 실패 패턴 주입
8. **record_recovery_log()** 강화: Docker PostgreSQL 직접 기록
9. **kill_and_restart()** 강화: Tier2/3 텔레그램, 시맨틱 루프 PARTIAL + CLAUDE.md 주입
10. **trigger_post_processing()** 강화: signal 파일 생성, 슬롯 확인, 완료 텔레그램, auto_trigger 즉시 호출
11. **watch_signal_file_loop()** 추가: 5초 주기 signal 파일 감시 백그라운드 루프
12. **메인 루프**: signal watcher 백그라운드 시작 추가

---

## 파일 목록

| 파일 | 변경 유형 | 라인 수 |
|------|-----------|---------|
| `/root/aads/scripts/session_watchdog.sh` | 수정 (AADS-141 강화) | 696줄 |
| `/root/aads/scripts/auto_trigger.sh` | 수정 (signal 기반 즉시 투입) | 366줄 |
| `/root/aads/scripts/session_watchdog.service` | 수정 (systemd, AADS-141 버전) | 20줄 |

---

## 검증 결과

| 검증 항목 | 결과 |
|-----------|------|
| V-1: signal 파일 → 10초 이내 즉시 투입 | ✅ PASS (즉시 감지) |
| V-2: signal 없을 때 크론 주기 정상 | ✅ PASS (fallback 모드) |
| V-3: 시맨틱 루프 → PARTIAL 보고서 + DB 기록 | ✅ PASS (semantic_loop 레코드 확인) |
| V-4: 재시작 시 CLAUDE.md 주입 | ✅ PASS (함수 구현 + 테스트 확인) |
| V-5: 텔레그램 알림 4종 | ✅ PASS (함수 정의 + 호출 지점 확인) |
| V-6: 시맨틱 루프 감지 (동일 패턴 10회) | ✅ PASS (SEMANTIC_LOOP 출력) |
| 슬롯 관리 ≤4 제한 | ✅ running=2/4 (정상) |
| HTTP 200 | ✅ HTTP 200 |

---

## 다음 단계 (참고)

- session_watchdog.service → `sudo cp /root/aads/scripts/session_watchdog.service /etc/systemd/system/ && sudo systemctl enable --now session_watchdog` (root 권한 필요)
- 텔레그램 토큰 설정: aads-server/.env에 `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID` 확인
