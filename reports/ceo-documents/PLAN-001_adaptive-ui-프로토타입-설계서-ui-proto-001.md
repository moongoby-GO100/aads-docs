# PLAN-001: Adaptive UI 프로토타입 설계서 (UI-PROTO-001)

## 개요
AADS CEO 대시보드를 위한 Adaptive UI 프로토타입 설계.
3-Layer Progressive Disclosure UI 구조와 Adaptive Router를 통한 키워드-UI 매핑 13종 지원.

## 3-Layer Progressive Disclosure UI
- Layer 1: 단순 요약 카드 (기본 뷰)
- Layer 2: 상세 패널 (클릭 확장)
- Layer 3: 전체 화면 모달 (deep-dive)

## Adaptive Router 키워드-UI 매핑 (13종)
| 키워드 | UI 컴포넌트 | 데이터 소스 |
|--------|------------|------------|
| 비용 | cost-chart | /api/v1/costs |
| 상태 | status-board | /api/v1/watchdog |
| 에러 | error-list | /api/v1/watchdog/errors |
| 기획서 | doc-list | /api/v1/documents?tag=plan |
| 기술서 | doc-list | /api/v1/documents?tag=tech |
| 보고서 | doc-list | /api/v1/documents?tag=research |
| 상황 | doc-list | /api/v1/documents?tag=status |
| 대화창 | channel-list | /api/v1/channels |
| 프로젝트 | project-board | /api/v1/projects |
| 메모리 | memory-view | /api/v1/memory |
| 승인 | approval-queue | /api/v1/approval |
| 배포 | deploy-status | /api/v1/watchdog |
| 매니저 | agent-list | /api/v1/context/system/agents |

## 설계 원칙
- 키워드 입력 → UI 자동 전환 (라우팅)
- 최소 클릭 원칙 (CEO는 타이핑만)
- 모바일 우선 반응형

## 생성 일시
2026-03-05 22:00 KST (소급 저장: 2026-03-06 KST)
