# AADS HANDOVER ARCHIVE
최종 업데이트: 2026-03-07 | D-023 3계층 분리 — 구버전 상세 이력

---

## 4계층 자기치유 체계 (AADS-131~142, 2026-03-07)

| 계층 | 컴포넌트 | 주기/타임아웃 | 위치 |
|------|---------|-------------|------|
| L1 | claude_exec 하트비트 + inotifywait | 이벤트 기반 | 3서버 공통 |
| L1 | claude_exec 하드 타임아웃 (안전망) | 2시간 | 3서버 공통 |
| L1 | bridge 셀프체크 | 60초 | 서버 211 |
| L1.5 | session_watchdog.sh | 10초 주기 | 3서버 공통 |
| L2 | watchdog_daemon | 30초 | 서버 68 |
| L2 | pipeline_monitor | 2분 | 서버 211 |
| L2 | bridge_monitor | 60초 | 서버 211 |
| L3 | meta_watchdog.sh | cron */1 | 서버 211 |
| L4 | UptimeRobot / GitHub Actions | 외부 | 외부 |

**서버 상호감시**: 211↔68↔114 삼각형 크로스 모니터링, 2분 주기
**복구 의존성**: recovery_graph.py 위상정렬 기반
**에스컬레이션**: 3단계 (L2→L3→L4)
**서킷브레이커**: 3회 연속 실패 → 5분 쿨다운 (circuit_breaker_state DB)
**복구 이력**: recovery_logs DB (project_id 포함)

## 대시보드 신규 페이지 (AADS-134)
- `/ops/recovery` — Recovery History + 서킷브레이커 상태 + 통계
- `/ops/servers` — 3서버 상태 카드 + 감시 토폴로지 + 4계층 현황

## 하트비트 기반 세션 관리 파라미터 (AADS-140~142, D-021)

| 항목 | 값 |
|------|-----|
| 하트비트 발신 | inotifywait 또는 git status fallback |
| 감시 주기 | 10초 (session_watchdog.sh) |
| 경고 기준 | 60초 미갱신 |
| Tier 2 진단 | 120초 → CPU + 시맨틱루프 판별 → kill + 재시작 |
| Tier 3 강제종료 | 300초 → kill + recovery_logs 기록 |
| 하드 타임아웃 | 2시간 (안전망) |
| 완료 즉시 투입 | signal 파일 → auto_trigger.sh (크론 대기 없음) |
| 텔레그램 알림 | 4종: Tier2(⚠️), Tier3(🔴), Tier4(🚨), 완료(✅) |

## AADS 핵심 자동화 목록
8단계 파이프라인: CEO지시→Bridge감지→사전검증→우선순위전송→Claude실행→결과보고→DB기록→교차검증(9종)
자동복구: **15건** 상시 가동 (기존 12 + 신규 3: recovery_graph + escalation_engine + circuit_breaker)
세션 관리: 글로벌 ≤4, 서버별 동적 1~3슬롯

## 이전 버전 HANDOVER 전문
- v5.39: archive/HANDOVER-v5.39-full.md
- v7.0 이전 상세: HANDOVER-HISTORY.md 참조

---

## 참조
- Core: HANDOVER.md
- History: HANDOVER-HISTORY.md
- CEO-DIRECTIVES: CEO-DIRECTIVES.md (v3.2)
