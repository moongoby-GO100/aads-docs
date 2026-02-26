# Sprint 2 진행 보고서
- 기간: 2026-02-26 ~
- 최종 수정: 2026-02-26

## 태스크 현황

| # | 태스크 | 상태 | 비용 |
|---|--------|------|------|
| 2.0 | 모델 ID 수정 + 4모델 테스트 | ✅ 완료 | ~$0.02 |
| 2.0.1 | 최신 모델 전면 교체 (Opus 4.6, Sonnet 4.6, Haiku 4.5) | ✅ 완료 | $0 |
| 2.1 | LangGraph 오케스트레이션 (graph.py) | ✅ 완료 | $0 |
| 2.2 | 에이전트 7종 (planner/designer/developer/qa/devops/ops/cost) | ✅ 완료 | $0 |
| 2.3 | 파이프라인 실행 엔진 (pipeline.py) | ✅ 완료 | $0 |
| 2.4 | FastAPI 엔드포인트 (api/main.py) | ✅ 완료 | $0 |
| 2.5 | 파이프라인 통합 테스트 | ✅ 완료 | ~$0.05 |
| 2.6 | 대시보드 (Next.js) | ⏳ 대기 | - |

## API 엔드포인트

| Method | Path | 기능 |
|--------|------|------|
| GET | / | 서비스 정보 |
| GET | /health | 헬스체크 |
| POST | /projects | 프로젝트 생성 |
| GET | /projects | 프로젝트 목록 |
| GET | /projects/{id} | 상태 조회 |
| POST | /projects/{id}/run | 현재 단계 실행 |
| POST | /projects/{id}/run-all | 자동 전체 실행 |
| POST | /projects/{id}/approve | 게이트 승인 |
| POST | /projects/{id}/reject | 게이트 거부 |
| GET | /models | 모델 목록 |

## 에이전트 구성

| 에이전트 | 파일 | Tier | 역할 |
|----------|------|------|------|
| Planner | agents/planner/agent.py | 1 (Opus) | 아이디어 → 기획서 |
| Designer | agents/designer/agent.py | 1 (Opus) | 기획서 → 설계서 |
| Developer | agents/developer/agent.py | 2 (Sonnet) | 설계서 → 코드 |
| QA | agents/qa/agent.py | 2 (Sonnet) | 코드 리뷰 + 테스트 |
| DevOps | agents/devops/agent.py | 2 (Sonnet) | 배포 설정 생성 |
| Ops | agents/ops/agent.py | 3 (Gemini) | 모니터링 + 개선 |
| Cost | agents/cost/agent.py | 4 (Haiku) | 비용 추적 + 경고 |
