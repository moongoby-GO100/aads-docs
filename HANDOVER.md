# AADS HANDOVER v6.5
최종 업데이트: 2026-03-06 | 버전: v6.5 — AADS-130 E2E 검증 완료 + 소스코드 모듈화 확정

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
자동복구: 12건 상시 가동 (pipeline_monitor, watchdog, cross_validator, approval_queue)
세션 관리: 글로벌 ≤4, 서버별 동적 1~3슬롯

## FLOW 프레임워크
모든 작업: Find → Lay out → Operate → Wrap up (D-016)
소규모 수정: Operate → Wrap up만 수행 가능
상세: shared/rules/flow-rules.md | WRAP 게이트: auto_trigger.sh (R-014)

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

## CEO-DIRECTIVES 원칙 (현행 v2.8)
- D-016: FLOW 프레임워크 (Find→Layout→Operate→Wrap up) 모든 작업 의무
- D-017: 소스코드 모듈화 원칙 — agents/graphs/models/services 4개 디렉토리 독립 모듈
- 모든 산출물 DB화 원칙: project_artifacts 테이블 통합 저장

## 상세 참조
- AADS 전용 지식: /root/aads/aads-server/docs/knowledge/AADS-KNOWLEDGE.md
- 공유 교훈: shared/lessons/INDEX.md
- CEO 지침: CEO-DIRECTIVES.md (v2.8)
- 이전 HANDOVER 전문: archive/HANDOVER-v5.39-full.md
- WRAP 보고서: AADS-130_WRAPUP_REPORT.md
