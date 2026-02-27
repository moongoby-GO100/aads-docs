# Sprint 6 Phase B: 에이전트 엔진 LLM 연동

- **일시**: 2026-02-27 KST (검증: 2026-02-28)
- **Sprint**: 6 — Phase B (Task 6.3 ~ 6.7)

## 개요
Phase A의 에이전트 틀에 실제 AI 호출을 연동하여 Level 3 파이프라인 완성

## Task 요약

| Task | 내용 | 모듈 |
|------|------|------|
| 6.3 | 인터뷰 AI 분석 + PRD 생성 | core/agents/interview_engine.py |
| 6.4 | Level 3 파이프라인 엔진 | core/agents/pipeline_engine.py, api/pipeline_routes.py |
| 6.5 | 게이트 수정지시 → 재실행 | api/gate_routes.py (revise-and-rerun) |
| 6.6 | 대시보드 UI 연동 | api.ts, page.tsx 업데이트 |
| 6.7 | 통합 빌드 + 검증 | 전체 import, API, 빌드 테스트 |

## 핵심 흐름 (Level 3 완성)

```
사용자 → 기획 인터뷰 (7단계) → AI 분석 → PRD 생성
    ↓
PRD → Ideation(PM) → Planning(아키텍트) → Design(디자이너) → Development(개발자)
    → Testing(QA) → Deployment(DevOps) → Monitoring(SRE)
    ↑                                                    ↑
    └── 게이트 수정지시 → 재실행 ─────────────────────────┘
```

각 단계: 전문 프롬프트 + 이전 결과 컨텍스트 + 게이트 피드백 반영

## API 엔드포인트 (신규)

| Method | Endpoint | 설명 |
|--------|----------|------|
| POST | /pipeline/{id}/run | 전체 파이프라인 실행 |
| POST | /pipeline/{id}/run/{phase} | 단일 단계 실행 |
| GET | /pipeline/{id}/status | 진행 상태 |
| POST | /interview/{id}/answer | AI 분석 + 다음 질문 |
| POST | /interview/{id}/generate-prd | PRD 수동 생성 |
| POST | /gate/{id}/revise-and-rerun/{phase} | 수정 후 재실행 |

## 검증 (2026-02-28 실행)

### 1) 전체 import 테스트 (9개 모듈) — PASS
- InterviewAgent: 7 steps
- InterviewEngine: analyze_step, generate_prd, analyze_and_advance
- AgentPrompts: 7 agents
- ContextChain: 7 phases
- PipelineEngine: run_phase, run_pipeline
- InterviewRouter: 6 routes
- GateRouter: 5 routes
- PipelineRouter: 3 routes
- API main: all routers registered

### 2) 대시보드 빌드 — PASS
- Next.js 16.1.6 (Turbopack), Node 20
- Compiled successfully in 18.3s
- Static pages 13/13 생성

### 3) API 검증
| 항목 | 결과 |
|------|------|
| GET /health | `{"status":"healthy","time":"2026-02-28T07:53:46+09:00"}` |
| POST /interview/test-6b/start | step 1, title "프로젝트 개요", 3 questions, total_steps 7 |
| GET /pipeline/test-6b/status | 7 phases, progress_percent 0, JSON 정상 |
| GET /gate/test-6b/context/planning | phase_order 7단계, current_phase planning |
| GET 인터뷰 페이지 | HTTP 307 (리다이렉트, 정상) |

### 4) 서비스 재시작
- systemctl restart aads-api ✅
- systemctl restart aads-dashboard ✅

## 결과
- 적용: ✅ 소스 반영
- 배포: ✅ 빌드·재시작 완료
- 검증: import 9/9, API 4/4, npm build 성공, 인터뷰 페이지 307
