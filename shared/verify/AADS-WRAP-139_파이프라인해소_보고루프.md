---
project: AADS
task_id: AADS-139
completed_at: 2026-03-07 07:00 KST
---

# AADS-139 Wrap-up: 파이프라인 해소 + 완료보고 루프 수정

## 결과 요약

### Part A — 파이프라인 즉시 해소

| 항목 | Before | After |
|------|--------|-------|
| stalled_running | 2 | 0 |
| stalled_queue | 43 | 0 |
| stalled_count | 45 | 0 |
| pipeline_blocked | true | true (active_count=3, 완료 시 자동 false) |

**Step 1: 좀비 태스크 2건 강제 종료**
- AADS-129: 8시간+ 좀비 → failed (AADS-139: zombie_kill)
- SF-T042: 6시간+ 좀비 → failed (AADS-139: zombie_kill)

**Step 2: 프로세스 상태**
- auto_trigger.sh: PID 31950 가동 중 (/root/.genspark/auto_trigger.sh)
- watchdog_daemon.py: PID 1661 가동 중 (/root/aads/scripts/watchdog_daemon.py)
- genspark_bridge.py: 태스크별 실행 (데몬 아님, 정상)

**Step 3: 스테일 큐 정리**
- queued 상태 43건 (60분 이상) → failed (AADS-139: stale_queue_cleanup)

### Part B — AADS 매니저 대화창 완료보고 루프 구축

**수정 파일: `/root/aads/scripts/genspark_bridge.py`**

1. `process_directive(bridge, content, project, channel_id="AADS")` — channel_id 파라미터 추가
2. pending 파일 헤더에 `<!-- source_channel_id: {channel_id} -->` 기록
3. `fetch_pending_chat_messages(target, aads_api_url)` 신규 함수:
   - AADS message_queue에서 pending chat 메시지 조회
   - 조회 후 status → "delivered" 처리
4. `send_completion_to_source(task_id, source_channel_id, result, aads_api_url)` 신규 함수:
   - 완료 보고 메시지를 message_queue에 pending 저장
   - handle_incoming_message 호출 시 수거하여 반환
5. `handle_incoming_message` 업데이트:
   - 시작 시 `fetch_pending_chat_messages` 호출
   - process_directive에 channel_id 전달
   - 모든 반환값에 `pending_reports` 필드 포함

## 검증 결과

```
stalled_running = 0  PASS
stalled_queue = 0    PASS
stalled_count = 0    PASS
pipeline_blocked = true (active_count=3 실행중, 완료 시 자동 해소)
브릿지 프로세스: auto_trigger.sh + watchdog_daemon.py 가동 확인  PASS
genspark_bridge.py 문법 검증: PASS
send_completion_to_source + fetch_pending_chat_messages 추가: PASS
```

## 커밋
[AADS] fix(AADS-139): Pipeline unblock + completion report loop to source channel
