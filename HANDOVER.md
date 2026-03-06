# AADS HANDOVER v6.2
최종 업데이트: 2026-03-06 | 버전: v6.2 — AADS-123 Dashboard 교훈 탭 + FLOW 파이프라인 시각화

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
| AADS | Phase 2 운영 | AADS-123 Dashboard 교훈탭+FLOW 시각화 완료 | 진행중 |
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
모든 작업: Find → Lay out → Operate → Wrap up
소규모 수정: Operate → Wrap up만 수행 가능
상세: shared/rules/flow-rules.md

## AADS-123 완료 사항 (2026-03-06)
- Dashboard /lessons 페이지: 교훈 카드 그리드, 카테고리/프로젝트 필터, 상세 모달
- Dashboard /flow 페이지: 프로젝트별 FLOW 4단계 파이프라인 시각화
- Sidebar: 교훈(💡), FLOW(🔄) 메뉴 추가
- api.ts: getLessons(), getLesson(), getOpsDirectiveLifecycleByProject() 추가
- npm build 에러 0건, Docker 재배포 완료

## AADS-122 완료 사항 (2026-03-06)
- GET/POST /api/v1/lessons — 교훈 CRUD API
- DB: lessons 테이블 생성 (10번째 테이블)
- 교훈 8건 DB 등록 (L-001~L-008)
- claude_exec.sh: aads_lesson_check() 자동 등록 함수
- genspark_bridge.py: 교훈 자동첨부 로직 (_attach_relevant_lessons)
- 유지보수 모드 첫 실전 적용 (AADS-116)

## 상세 참조
- AADS 전용 지식: /root/aads/aads-server/docs/knowledge/AADS-KNOWLEDGE.md
- 공유 교훈: shared/lessons/INDEX.md
- CEO 지침: CEO-DIRECTIVES.md
- 이전 HANDOVER 전문: archive/HANDOVER-v5.39-full.md
- 기술서: TECH-002 (지시서 자동화 시스템)
