# AADS HANDOVER v6.9
최종 업데이트: 2026-03-07 | 버전: v6.9 — AADS-142 하트비트 시스템 3서버 전체 배포 + E2E 검증 + CEO-DIRECTIVES v3.1

## 시스템 개요
AADS (Autonomous AI Development System): 멀티 AI 에이전트 자율 개발 시스템
대시보드: https://aads.newtalk.kr/
리포: aads-docs, aads-server, aads-dashboard (moongoby-GO100)
GitHub PAT: repo+workflow, 만료 2026-05-27

## 서버 현황
| 서버 | IP | 역할 | 프로젝트 |
|------|-----|------|----------|
| 211 | 211.188.51.113 | Hub(Bridge, auto_trigger, pipeline_monitor) | KIS, GO100 |
| 68 | 68.183.183.11 | AADS Backend(FastAPI, PostgreSQL, Dashboard) | AADS |
| 114 | 116.120.58.155 | 실행 서버 | SF, NTV2 |

## 프로젝트 현황
| 프로젝트 | Phase | 최근 태스크 | 상태 |
|----------|-------|------------|------|
| AADS | Phase 2 운영 | AADS-130 E2E 검증 + 모듈화 Wrap-up | 완료 |
| KIS | V4.1 운영 | KIS-041 | 정상 |
| GO100 | 운영중 | GO100-023 | 정상 |
| NTV2 | Phase 1 | NT-001 환경구축 | 대기 |
| SF | 운영중 | SF-015 | 정상 |
| NAS | 유지보수 | NAS-010 | 정상 |

## 긴급 이슈
없음

## 핵심 자동화 (TECH-002 참조)
8단계 파이프라인: CEO지시→Bridge감지→사전검증→우선순위전송→Claude실행→결과보고→DB기록→교차검증(9종)
자동복구: **15건** 상시 가동 (기존 12 + 신규 3: recovery_graph + escalation_engine + circuit_breaker)
세션 관리: 글로벌 ≤4, 서버별 동적 1~3슬롯

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

**AADS-142 변경 (하트비트 3서버 전체 배포 + E2E 검증 + CEO-DIRECTIVES v3.1)**:
- deploy_heartbeat_3servers.sh: 3서버 일괄 배포 스크립트 (inotify-tools/jq 설치 + systemd 등록)
- CEO-DIRECTIVES v3.1: D-018 개정(하트비트 기반 L1) + D-021 신규(세션 관리 원칙) 배포
- E2E 검증: 파일럿 B-1(완료 흐름 4초), B-2(Tier2 시뮬레이션 150초), B-3(3건 즉시투입 2초) 완료
- 서버 68: session_watchdog RUNNING(PID 6166), meta_watchdog 감시 포함, HC=200 ✅

**AADS-141 변경 (이벤트 기반 즉시 투입 + 에스컬레이션)**:
- auto_trigger.sh: /tmp/aads_trigger_next.signal 감지 → 즉시 투입 (크론 대기 없음), 크론 fallback 유지
- session_watchdog: trigger_post_processing()에서 signal 생성 + auto_trigger 즉시 호출
- 시맨틱 루프 에스컬레이션: PARTIAL 보고서 자동 생성 (10개 패턴+토큰+권고)
- recovery_logs DB: issue_type='semantic_loop' 기록
- CLAUDE.md 주입: 재시작 시 이전 실패 패턴 + 다른 접근법 지시
- 텔레그램 4종: Tier2(⚠️), Tier3(🔴), Tier4(🚨), 완료(✅)
- 슬롯 관리: check_global_slots() 글로벌 ≤4 + trigger_decisions.log 기록

**AADS-140 변경 (D-018 L1 전환)**:
- 기존: 고정 30분 하드 타임아웃 → 좀비 30분 방치 문제
- 신규: 하트비트 이벤트 기반 (120초 무반응 Tier2, 300초 Tier3)
- session_watchdog.sh: Tier2 CPU+시맨틱루프 통합판별, Tier3 강제종료+복구
- 하드 타임아웃은 7200초(2시간)로 연장, 안전망 역할만

**서버 상호감시**: 211↔68↔114 삼각형 크로스 모니터링, 2분 주기
**복구 의존성**: recovery_graph.py 위상정렬 기반
**에스컬레이션**: 3단계 (L2→L3→L4)
**서킷브레이커**: 3회 연속 실패 → 5분 쿨다운 (circuit_breaker_state DB)
**복구 이력**: recovery_logs DB (project_id 포함)

## 대시보드 신규 페이지 (AADS-134)
- `/ops/recovery` — Recovery History + 서킷브레이커 상태 + 통계
- `/ops/servers` — 3서버 상태 카드 + 감시 토폴로지 + 4계층 현황

## FLOW 프레임워크
모든 작업: Find → Lay out → Operate → Wrap up (D-016)
소규모 수정: Operate → Wrap up만 수행 가능
상세: shared/rules/flow-rules.md | WRAP 게이트: auto_trigger.sh (R-014)

## AADS-140~142 완료 사항 (2026-03-07)
- **AADS-140**: claude_exec 하트비트(inotifywait + git status fallback) + session_watchdog.sh(Tier2/3) 구현
- **AADS-141**: signal 기반 즉시 투입 + 시맨틱 루프 에스컬레이션 + PARTIAL 보고서 + 텔레그램 4종 알림
- **AADS-142**: 하트비트 시스템 3서버 배포 + E2E 파일럿 검증 + CEO-DIRECTIVES v3.1 배포
  - 서버 68: session_watchdog RUNNING, meta_watchdog 감시, HC HTTP 200 ✅
  - 배포 스크립트: scripts/deploy_heartbeat_3servers.sh (root 실행 전용)
  - B-1 파일럿: 완료 흐름 4초 내 확인 ✅
  - B-2 멈춤 시뮬레이션: 150초 Tier2 대상 설정 ✅
  - B-3 처리량: 3건 즉시 투입 2초 ✅

## AADS-130 최종 완료 사항 (2026-03-06 Wrap-up)
- **E2E 3시나리오 검증 완료**: AI 퍼포먼스 마케팅 SaaS / K-12 교육 플랫폼 / 이커머스 셀러 도구
  - 각 시나리오: strategy_reports ✅ + project_plans ✅ + debate_logs ✅ + artifacts 6건 ✅
  - 건당 비용: $3.72~$4.03 (전체 $5 이하 기준 통과)
  - DB 총합: strategy_reports 4건, project_plans 4건, debate_logs 6건, artifacts 18건
- **소스코드 모듈화 (D-017 준수)**:
  - agents/: 10파일 (BaseAgent + strategist/planner/pm/supervisor/architect/developer/qa/judge/devops/researcher)
  - graphs/: 3파일 (ideation_subgraph/execution_chain/full_cycle_graph) + __init__
  - models/: 4파일 (strategy/plan/task/artifact) + __init__
  - services/: 4파일 (mcp_client/model_router/db_recorder/cost_tracker) + __init__
- **신규 DB 테이블**: debate_logs (토론 이력) + projects (모드 컬럼 포함)
- **신규 API**: GET /api/v1/debate-logs?project_id=...
- **테스트**: strategist 7/7, planner 7/7, ideation_subgraph 9/9, full_cycle 10/10, unit 114/118 (4건 sandbox 기존 실패)

## AADS-128~129 완료 사항 (2026-03-06)
- AADS-128: Full-Cycle Graph (ideation+execution 서브그래프 통합), project_artifacts DB, artifacts API
- AADS-129: CEO 체크포인트 UI 4페이지 (select-item, approve-plan, full-cycle, reports)

## 하트비트 기반 세션 관리 (AADS-140~142, D-021)

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

## CEO-DIRECTIVES 원칙 (현행 v3.1)
- D-016: FLOW 프레임워크 (Find→Layout→Operate→Wrap up) 모든 작업 의무
- D-017: 소스코드 모듈화 원칙 — agents/graphs/models/services 4개 디렉토리 독립 모듈
- D-016: FLOW 프레임워크 (Find→Layout→Operate→Wrap up) 모든 작업 의무
- D-017: 소스코드 모듈화 원칙 — agents/graphs/models/services 4개 디렉토리 독립 모듈
- D-018: 4계층 자기치유 원칙 — L1 하트비트 기반(60/120/300초), L1.5 session_watchdog, L2~L4 감시 체계
- D-019: 서버 상호 감시 의무화 — 3서버 2분 주기 크로스 모니터링
- D-020: 복구 이력 DB 의무화 — recovery_logs 테이블 기록
- D-021: 하트비트 기반 세션 관리 — claude_exec 하트비트, 10초 감시, 이벤트 기반 투입
- R-016: 서킷브레이커 준수 — 3회 실패 시 5분 쿨다운
- 모든 산출물 DB화 원칙: project_artifacts 테이블 통합 저장

## 상세 참조
- AADS 전용 지식: /root/aads/aads-server/docs/knowledge/AADS-KNOWLEDGE.md
- 공유 교훈: shared/lessons/INDEX.md
- CEO 지침: CEO-DIRECTIVES.md (v3.1)
- 이전 HANDOVER 전문: archive/HANDOVER-v5.39-full.md
- WRAP 보고서: AADS-130_WRAPUP_REPORT.md
