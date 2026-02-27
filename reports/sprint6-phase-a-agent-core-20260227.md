# Sprint 6 Phase A: 에이전트 코어 아키텍처

- **일시**: 2026-02-27 KST
- **Sprint**: 6 — Phase A (Task 6.0 ~ 6.2)

## 개요

AADS Level 3 에이전트 시스템의 핵심 3개 모듈 구현

### Task 6.0: 기획 인터뷰 에이전트

| 항목 | 내용 |
|------|------|
| 모듈 | `core/agents/interview_agent.py` |
| API | `api/interview_routes.py` (`/interview/{id}/start`, `/answer`, `/state`, `/prd`) |
| UI | `dashboard/src/app/projects/[id]/interview/page.tsx` |
| 인터뷰 단계 | 7단계 (개요→타겟→기능→비기능→디자인→기술→PRD) |

### Task 6.1: 에이전트 전문 프롬프트 시스템

| 항목 | 내용 |
|------|------|
| 모듈 | `core/agents/prompts.py` |
| 에이전트 수 | 7개 (PM, 아키텍트, 디자이너, 개발자, QA, DevOps, SRE) |
| 컨텍스트 주입 | PRD + 이전 단계 결과 자동 포함 |

### Task 6.2: 컨텍스트 체인 + 게이트 피드백

| 항목 | 내용 |
|------|------|
| 모듈 | `core/agents/context_chain.py` |
| API | `api/gate_routes.py` (`/gate/{id}/feedback`, `/context/{phase}`, `/history`) |
| 기능 | 이전 결과 전달, 게이트 피드백(승인/거부/수정지시), 재실행, 버전 관리 |

## Level 3 아키텍처 비교

| 기능 | 이전 | 현재 |
|------|------|------|
| 기획 수집 | 한 줄 아이디어 | 7단계 인터뷰 → PRD |
| 에이전트 전문성 | 동일 프롬프트 | 역할별 전문 프롬프트 |
| 컨텍스트 | 미전달 | 이전 결과 체인 전달 |
| 게이트 | 승인/거부만 | 승인/거부/수정지시 + 재실행 |
| 결과 품질 | 추측 기반 | PRD 기반 검증된 결과 |

## 검증 결과

- **Python import**: InterviewAgent(7 steps), AgentPrompts(7 agents), ContextChain(7 phases), InterviewRouter(4 routes), GateRouter(3 routes), API main — 통과
- **Dashboard build**: Next.js 빌드 성공, `/projects/[id]/interview` 라우트 포함
- **API**:
  - `GET /health`: 200, `status: healthy`
  - `POST /interview/test-project/start`: 200, step 1 질문 3개 반환
  - `GET /gate/test-project/context/planning`: 200, prd/previous_results/phase_order 반환
- **서비스**: aads-api, aads-dashboard 재시작 후 정상

## 변경 파일

- `core/agents/__init__.py`
- `core/agents/interview_agent.py`
- `core/agents/prompts.py`
- `core/agents/context_chain.py`
- `api/interview_routes.py`
- `api/gate_routes.py`
- `api/main.py` (interview_router, gate_router 등록)
- `dashboard/src/app/projects/[id]/interview/page.tsx`

## 상태

- 적용: ✅
- 배포: ✅ (서비스 재시작 완료)

## 다음 작업 제안

- Sprint 6 Phase B: 파이프라인 엔진이 PRD/컨텍스트체인/전문프롬프트 연동
- 인터뷰 완료 시 PRD 생성 AI 호출 연동 (현재는 상태만 저장, 실제 LLM 호출 미구현)
