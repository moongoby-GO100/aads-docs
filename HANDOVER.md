# AADS HANDOVER v6.4
최종 업데이트: 2026-03-06 | 버전: v6.4 — AADS-128/129/130 10-Agent 풀사이클 통합 완료

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
| AADS | Phase 2 운영 | AADS-124 FLOW 체계 최종 Wrap up | 완료 |
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

## AADS-128~130 완료 사항 (2026-03-06)
- AADS-128: Full-Cycle Graph (ideation+execution 서브그래프 통합), project_artifacts DB, artifacts API 3개
- AADS-129: CEO 체크포인트 UI 4페이지 (select-item, approve-plan, full-cycle, reports), checkpoint sub-routes 4개
- AADS-130: E2E 3시나리오 검증(20/20 통과), models/ 모듈화(strategy/plan/artifact), services/db_recorder.py, debate-logs optional query
- 신규 DB 테이블: project_artifacts (산출물 통합 저장)
- 신규 API: POST/GET /api/v1/artifacts, POST /api/v1/projects/{id}/checkpoint/{action}
- 테스트: test_full_cycle.py(10/10), test_e2e_scenarios.py(20/20), 기존 183 통과

## AADS-124 완료 사항 (2026-03-06)
- CEO-DIRECTIVES v2.8: D-016 FLOW, R-014 Wrap up 의무화, R-015 교훈 등록, 9-3 파일명 확장
- auto_trigger.sh WRAP 게이트: P0/P1 WRAP 미완료 시 다음 작업 차단 (10분 대기)
- claude_exec.sh 자동 health-check: P2/P3 완료 후 5분 → pipeline_healthy 확인, 실패 시 WRAP 자동 생성
- 전체 검증 체크리스트 20개 항목 수행 + PLN-AADS-001-v2 Wrap up 완료

## 상세 참조
- AADS 전용 지식: /root/aads/aads-server/docs/knowledge/AADS-KNOWLEDGE.md
- 공유 교훈: shared/lessons/INDEX.md
- CEO 지침: CEO-DIRECTIVES.md (v2.8)
- 이전 HANDOVER 전문: archive/HANDOVER-v5.39-full.md
- WRAP 보고서: shared/verify/AADS-WRAP-124_FLOW체계전체검증.md