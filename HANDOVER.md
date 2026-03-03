# HANDOVER – AADS (Autonomous AI Development System)
> 최종 업데이트: 2026-03-03 (v3.7 — STABILITY-006: JWT 86자 보안키, PostgreSQL 로컬 연결, MCP SSE ping 수정, 118 PASS)
> 관리자: CEO (moongoby)
> 용도: 모든 AI 세션(웹 Claude, Cursor, Claude Code) 시작 시 필수 읽기

---

## 1. 프로젝트 개요

- **AADS (Autonomous AI Development System)**: 멀티 AI 에이전트가 조직처럼 협업하여 소프트웨어를 자율 개발하는 시스템
- **AADS 자체가 프로덕트** — 먼저 AADS를 완성하여 서비스 배포, 이후 첫 프로젝트 HealthMate
- 저장소: github.com/moongoby-GO100/aads-docs (문서), github.com/moongoby-GO100/aads-server (서버), github.com/moongoby-GO100/aads-dashboard (Phase 2 대시보드)
- **대시보드**: https://aads.newtalk.kr/ (Next.js 16, React, Tailwind CSS)
- 서버: aads.newtalk.kr
- 파이프라인 대시보드: https://aads.newtalk.kr/projects/2a0647ca
- GitHub PAT 'aads-server': 권한 repo + workflow, 만료 2026-05-27

### 업무 체계
- **웹 Claude (지휘 AI)**: CEO와 직접 대화, 전략 수립, 지시서 작성, 교차검증
- **Cursor (서버 작업자)**: 서버에서 코드 작성, 분석, 보고서 생성, GitHub push
- **Claude Code (서버 작업자)**: Cursor와 동일 역할, 터미널 기반
- 웹 Claude는 서버 접근 불가 → 본 HANDOVER.md로 맥락 복구

---

## 2. 완료된 작업

| Task ID | 날짜 | 커밋 | HTTP | 핵심 결과 |
|---------|------|------|------|-----------|
| RESEARCH-AI-MODELS | 02-28 | — | — | 프론티어 LLM 100+모델 조사, 가격표, 에이전트별 모델 배정 |
| RESEARCH-OPENSOURCE | 02-28 | — | — | 수학/추론 7개, 코딩 6개, 퀀트 20+, RAG 8개, 벡터DB 6개 |
| RESEARCH-INFRA-MCP | 02-28 | — | — | 샌드박스 7개, PaaS 3개, MCP 8,600+, 앱빌더 8개, 에이전트 12개 |
| PAT-VERIFY | 02-26 | — | — | GitHub PAT 검증: repo scope, 만료 2026-05-27 |
| PIPELINE-CHECK | 02-26 | — | — | 서버 파이프라인 실행 확인 |
| WORKFLOW-SCOPE | 02-28 | — | — | PAT에 workflow scope 추가 |
| PROCESS-SETUP | 02-28 | — | — | 웹Claude↔Cursor/ClaudeCode 3자 협업 체계 수립 |
| INIT-DOCS | 02-28 | 6145a6c | 200 | HANDOVER.md + CEO-DIRECTIVES.md v1.0 생성 |
| DIRECTIVES-v1.1 | 02-28 | 3778986 | 200 | CEO-DIRECTIVES v1.1 (브라우저 URL 보고 규칙) |
| ARCH-v1.0-DRAFT | 02-28 | 3778986 | 200 | Phase 1 설계서 v1.0-draft |
| REPORT-PLACEHOLDER | 02-28 | 3778986 | 200 | 보고서 placeholder 3건 |
| PUSH-006 | 02-28 | 7dfd454 | 200 | CEO-DIRECTIVES v2.0 + 설계서 v1.1-final (21건 수정) + 보고서 3건 실제 내용 + HANDOVER v2.0 |
| **PUSH-007** | **03-01** | **0f6b5a1** | **200** | **설계서 v1.1 통합본 신규(27섹션), CEO-DIRECTIVES v2.1, 기존 설계서 DEPRECATED, 6건 불일치 해소** |
| **PUSH-008** | **03-02** | **ac0c85f** | **200** | **Phase 1 Week 1 구현 완료: aads-server 신규 저장소, 3-agent chain (PM→Supervisor→Developer), LangGraph 1.0.10 StateGraph, AsyncPostgresSaver 3.0.4, FastAPI 0.134.0, E2B AsyncSandbox, R-012 비용추적, 6/6 단위테스트 통과** |
| **PUSH-009** | **03-02** | **26aad5f** | **200** | **model_router T-002 가격/모델 정정 (10건), sandbox.py tenacity retry 추가** |
| **CAPABILITY-MAP-001~004** | **03-02** | **42e2e1b** | **200** | **AADS 능력 확장 전방위 검토: 12개 영역 분석, 9개 자체구축+3개 외부API 확정** |
| **INFRA-STRATEGY-001** | **03-02** | **42e2e1b** | **200** | **인프라 전략: DO→Contabo 이전 계획, 월 $64~$172 목표, DO Droplet 리사이즈 승인** |
| **CEO-DIRECTIVES-v2.2** | **03-02** | **6fcc547** | **200** | **CEO-DIRECTIVES v2.2 — Genspark 통합지휘 규칙 추가 (섹션 4, 9-1~9-8)** |
| **PHASE1-W2-001** | **03-02** | **f698b4c** | **200** | **PHASE1-W2-001 — QA/Judge 에이전트 추가, 5-agent chain (PM→Supervisor→Developer→QA→Judge), 23/23 PASS** |
| **PHASE1-W2-002** | **03-02** | **7e59e18** | **200** | **PHASE1-W2-002 — MCP 서버 연결 (상시4+온디맨드3), MCPClientManager, graceful degradation, 36/36 PASS** |
| **PHASE1-W2-003** | **03-02** | **0bae6a1** | **200** | **PHASE1-W2-003 — E2B 실제 연동 + E2E 테스트 (39 PASS, 5 SKIP, E2B PLACEHOLDER), 5-agent+MCP+E2B 파이프라인 완성** |
| **PHASE1-W2-004** | **03-02** | **d2d0944** | **200** | **PHASE1-W2-004 — Docker Compose 배포 + Nginx 리버스 프록시, aads.newtalk.kr/api/v1/health ✅** |
| **PHASE1-W2-005** | **03-02** | **d4f7636** | **200** | **PHASE1-W2-005 — 8-agent chain 완성 (Architect/DevOps/Researcher 추가), 45/45 PASS** |
| **PHASE15-REALTEST-001** | **03-02** | **3fb1fd5** | **200** | **CUR-AADS-PHASE15-REALTEST-001 — HITL 체크포인트(6단계 auto_approve), 비용추적 강화(Redis), /costs API, 56/56 PASS** |
| **PHASE15-CICD-002** | **03-02** | TBD | **200** | **CUR-AADS-PHASE15-CICD-002 — GitHub Actions CI/CD (unit+E2E 자동화), README 8-agent 반영, HANDOVER 버전이력 정렬** |
| **PHASE2-INTEGRATION-003** | **03-02** | TBD | **200** | **8-agent 통합 실행 검증: SSE 스트리밍(/projects/{id}/stream), 프로젝트 상태 API, 3개 E2E 시나리오 테스트** |
| **PHASE2-POLISH-004** | **03-02** | **2133437** | **200** | **CUR-AADS-PHASE2-POLISH-004 — auth.py hmac.compare_digest, 전역 예외 핸들러, structlog 표준화, API 문서 강화, TS 타입 안정성 0오류** |
| **PHASE2-MCP-LIVE-005** | **03-03** | **046111f** | **200** | **CUR-AADS-PHASE2-MCP-LIVE-005 — MCP 실구동 서버 3개(Filesystem/Git/Memory FastMCP SSE), supervisord.conf, Dockerfile 갱신, MCPClientManager HTTP ping 검증, 에이전트 8개 프롬프트 전면 개선, 테스트 118개(+73), 커버리지 43%→62%** |
| **PHASE2-DASHBOARD-001** | **03-02** | TBD | **200** | **Phase 2 대시보드 기초: Next.js + React + Tailwind, 6개 페이지, Docker Compose 3100포트, aads.newtalk.kr/** |
| **PHASE2-DASHBOARD-002** | **03-02** | TBD | **200** | **대시보드 인증(JWT), AgentStatus 파이프라인 시각화, CostTracker 바 차트, 에러 페이지, HANDOVER 섹션6** |
| **PHASE2-LLM-CONNECT-003** | **03-02** | **4fa9341** | **200** | **실제 LLM 연동: state.py _last_value 리듀서, supervisor 병렬 버그 수정, E2B graceful degradation, 8-agent E2E completed ✅, 45/45 PASS** |
| **PHASE2-STABILITY-006** | **03-03** | TBD | **200** | **CUR-AADS-PHASE2-STABILITY-006 — JWT 86자 보안키, AADS_ADMIN_PASSWORD 갱신, 호스트 PostgreSQL 연결(host.docker.internal+iptables), MCP SSE ping stream 수정, checkpointer 3단계 fallback, 118 PASS** |

---

## 3. 진행 중 작업

| Task ID | 상태 | 내용 |
|---------|------|------|
| PHASE1-W1 | **완료** | aads-server Week 1: 3-agent chain, StateGraph, FastAPI 엔드포인트, 단위테스트 6/6 |
| PHASE1-W2 | **완료** | W2-001 ✅, W2-002 ✅, W2-003 ✅, W2-004 배포 ✅, W2-005 8-agent ✅ → Phase 1 코어 8-agent 완성 |
| PHASE1.5 | **완료** | REALTEST-001 ✅, CICD-002 ✅, INTEGRATION-003 ✅ |
| PHASE2 | **진행 중** | DASHBOARD-001 ✅, DASHBOARD-002 ✅, LLM-CONNECT-003 ✅, POLISH-004 ✅, MCP-LIVE-005 ✅, **STABILITY-006 ✅** → 다음: E2E-FULLCYCLE-007 (P1), PHASE3-PLAN-008 (P2) |

---

## 4. 보류/미시작

| 항목 | 선행조건 | 우선순위 |
|------|----------|----------|
| AADS 코어 구현 (LangGraph Native Supervisor + 8 에이전트) | 설계서 확정 ✅ | ★★★★★ |
| AADS 서비스 배포 (Fly.io + Vercel) | 코어 완료 | ★★★★★ |
| CI/CD (.github/workflows) | 코어 완료 | ★★★★ |
| AADS 대시보드 (Phase 2) | ✅ DASHBOARD-001 완료 | ★★★ |
| MCP 서버 구성 가이드 | 코어 구현 시 동시 | ★★★ |
| HealthMate | AADS 완성 후 | ★★ |

---

## 5. 핵심 발견 (누적)

### AI 모델 (2026-02-28, 검증 가격)
- Claude Opus 4.6: $5/$25, Sonnet 4.6: $3/$15, Haiku 4.5: $1/$5
- GPT-5.3 Codex: $1.75/$14, GPT-5 mini: $0.25/$2, GPT-5 nano: $0.05/$0.40
- Gemini 3.1 Pro Preview: $2/$12, Gemini 2.5 Flash: $0.30/$2.50
- GPT-5.2 Pro: $21/$168 (극고가, 사용 비추천)
- AADS 최적 혼합 캐싱 후 월 ~$55

### MCP 에코시스템
- 8,600+ 서버, Phase 1 필수 7개
- MultiServerMCPClient, supervisord 프로세스 관리, SSE transport (localhost)
- langgraph-supervisor MCP 루프 버그 #249 → Native StateGraph 사용

### 인프라
- E2B $150-250/mo, Fly.io 2GB RAM $20-50/mo, Supabase Direct:5432 $25-40/mo
- Fly.io 60초 idle timeout → 20초 SSE keepalive
- 월 예상 $320 (D-004 $500 내)

### 아키텍처 핵심 결정 (21건 수정 + 6건 불일치 해소)
- LangGraph >= 1.0.10, Native Tool-Based Supervisor
- 8 에이전트: Supervisor, PM, Architect, Developer, QA, Judge, DevOps, Researcher
- 6단계 사용자 체크포인트 (경쟁사 차별점)
- 구조화 JSON TaskSpec 통신 (Pydantic BaseModel, 12필드)
- Judge Agent 독립 검증 (정확도 ~10%→~70%)
- 점진적 자율성 게이트 (90% 자동승인, 70% 재활성화)
- Graceful Degradation: LLM fallback, E2B "코드만" 모드, Supabase InMemory 임시
- 동시성: thread 10, E2B 5, DB 15, Redis 글로벌 비용 카운터
- 작업당 LLM 호출 최대 15회 (R-012)
- MCP SSE transport (supervisord 환경)
- SaaS 4-tier 과금: Starter(무료)/Pro($49)/Business($149)/Enterprise(커스텀)
- BEP 15~20명, 안정기 마진 85~90%

### CEO 확정 사항 (2026-03-02)
- **능력 확장**: 12개 영역 중 9개 자체구축 + 3개 외부API (Groq Whisper/Google TTS/DALL-E 3)
- **인프라 전략**: DO→Contabo VPS 30 이전 (2026-04-01 목표), 월 $84 절감 (88%)
- **aads-server 배포**: 68서버 Docker Compose, aads.newtalk.kr Nginx 리버스 프록시, 포트 8100 (W2-004 완료)
- **월 운영비 목표**: Contabo $12 + LLM $50~150 + 외부API $2~10 = **$64~172/월**
- **GPU 구매 안 함**: 외부 API 활용으로 12개월 $1,936~$2,540 절감


---

## 6. 웹 Claude 인수인계 사항

### 6-1. 최신 상태 (v3.6 기준)
- **Phase 1 완료**: 8-agent chain, LangGraph 1.0.10, FastAPI, MCP 7개, HITL 체크포인트
- **Phase 1.5 완료**: REALTEST-001, CICD-002, INTEGRATION-003 — 63/63 테스트 PASS
- **Phase 2 진행 중**: aads-dashboard (Next.js 16, https://aads.newtalk.kr/), JWT 인증, SSE 모니터
  - MCP-LIVE-005 ✅: Filesystem/Git/Memory 실구동 서버 + supervisord.conf 완성
  - 에이전트 8개 프롬프트 전면 개선 (역할/원칙/출력형식 구체화)
  - 테스트 118개, 커버리지 62%
- CEO-DIRECTIVES v2.2 확정 (Genspark 통합지휘 규칙 포함)
- 설계서 v1.1 통합본(27섹션): `design/aads-architecture-v1.1.md`
- GitHub: aads-server (**046111f**), aads-dashboard (2ea0348), aads-docs (최신)

### 6-2. 웹 Claude가 해야 할 일
1. Phase 2 대시보드 고도화 지시 (E2B 실전 연동, 대시보드 추가 기능)
2. Phase 3 기획 시작 (SaaS 멀티유저, 결제 연동, HealthMate 첫 프로젝트)
3. 구현 진행 중 교차검증 및 이슈 대응

### 6-3. 대표님 확인 사항
- [x] INIT-DOCS push 완료
- [x] 설계서 v1.1-final 확정
- [x] Phase 1 코어 8-agent 완성 (45/45 PASS)
- [x] Phase 1.5 실전 검증 완료 (63/63 PASS)
- [x] GitHub Actions CI/CD 구축
- [x] aads.newtalk.kr Docker 배포 (API + 대시보드)
- [x] JWT 인증 기초 구현
- [ ] AADS_ADMIN_PASSWORD 실제 값 설정 (서버 .env)
- [ ] E2B 실제 API 키 적용 (실전 코드 생성 테스트)
- [ ] Phase 3 서비스 범위 확정 (SaaS, HealthMate 등)

### 6-4. 주의사항
- PAT 토큰 웹 Claude 세션 간 비유지 → Cursor/Claude Code가 push
- 웹 Claude 서버 접근 불가 → Cursor/Claude Code 경유
- `langgraph-supervisor` 프로덕션 사용 금지 (R-010)
- Supabase Supavisor 경유 금지 (R-011)
- 작업당 LLM 호출 최대 15회 (R-012)
- JWT_SECRET_KEY, AADS_ADMIN_PASSWORD → 서버 .env에만 보관 (git 커밋 금지)
- 차트 라이브러리 추가 금지 (Tailwind CSS 순수 구현)

---

## 7. 업데이트 규칙
- 모든 Task 완료 시 이 문서 업데이트 필수
- push 후 raw URL HTTP 200 확인:
  `curl -s -o /dev/null -w "%{http_code}" https://raw.githubusercontent.com/moongoby-GO100/aads-docs/main/HANDOVER.md`
- 보고서 마지막 줄: `HANDOVER.md 업데이트 완료: {커밋해시}`

---

## 8. 버전 이력

| 버전 | 날짜 | 변경 |
|------|------|------|
| v1.0 | 2026-02-28 | 초판 — 리서치 완료, 인계서 구축 |
| v2.0 | 2026-02-28 | 대규모 개정 — CEO-DIRECTIVES v2.0, 설계서 v1.1-final, 보고서 3건 완성, 21건 수정 반영 |
| v2.1 | 2026-03-01 | PUSH-007 — 설계서 v1.1 통합본(27섹션), CEO-DIRECTIVES v2.1, 기존 설계서 DEPRECATED, 6건 불일치 해소 |
| v2.2 | 2026-03-02 | PUSH-008 — Phase 1 Week 1 구현 완료: aads-server 3-agent chain, 6/6 테스트 |
| v2.3 | 2026-03-02 | PUSH-009 완료 확인, CAPABILITY-MAP 001~004 + INFRA-STRATEGY-001 추가, CEO 확정사항(9자체구축+3외부API, DO→Contabo 전략, $64~$172/월) 반영 |
| v2.4 | 2026-03-02 | CEO-DIRECTIVES v2.2 Genspark 통합지휘 규칙 추가, PHASE1-W2-001 QA/Judge 5-agent chain 완료 (23/23 PASS) |
| v2.5 | 2026-03-02 | PHASE1-W2-002 MCP 서버 연결 (상시4+온디맨드3), MCPClientManager, README 업데이트, 36/36 PASS |
| v2.6 | 2026-03-02 | PHASE1-W2-003 E2B 연동 + E2E 테스트 (39 PASS, 5 SKIP), 5-agent+MCP+E2B 파이프라인 완성 |
| v2.7 | 2026-03-02 | PHASE1-W2-004 Docker Compose 배포 + Nginx (aads.newtalk.kr/api/v1/health ✅), Phase 1 Week 2 완료 |
| v2.8 | 2026-03-02 | PHASE1-W2-005 8-agent chain 완성: Architect/DevOps/Researcher 추가, 45/45 PASS |
| v2.9 | 2026-03-02 | PHASE15-REALTEST-001: HITL 체크포인트, 비용추적 강화(Redis), /costs API, 56/56 PASS |
| v3.0 | 2026-03-02 | PHASE15-CICD-002: GitHub Actions CI/CD, README 8-agent 반영, 버전이력 정렬 |
| v3.1 | 2026-03-02 | PHASE2-INTEGRATION-003: SSE 스트리밍, 상태 API, E2E 3시나리오 테스트 |
| v3.2 | 2026-03-02 | PHASE2-DASHBOARD-001: Next.js 대시보드 기초, Docker Compose, Nginx 통합 |
| v3.3 | 2026-03-02 | PHASE2-DASHBOARD-002: JWT 인증, 시각화 고도화, 에러 페이지 |
| v3.4 | 2026-03-02 | PHASE2-LLM-CONNECT-003: 실제 LLM 연동, 8-agent E2E completed, 45/45 PASS |
| v3.5 | 2026-03-02 | PHASE2-POLISH-004: auth.py 보안 강화(hmac.compare_digest), 전역 예외 핸들러, structlog 표준화, API 문서 강화, aads-dashboard TS 타입 안정성(빌드 오류 0건) |
| v3.6 | 2026-03-03 | PHASE2-MCP-LIVE-005: MCP 실구동 서버 3개(Filesystem/Git/Memory FastMCP SSE), supervisord.conf, Dockerfile 갱신, MCPClientManager HTTP ping, 에이전트 8개 프롬프트 전면 개선, 테스트 118개(+73), 커버리지 43%→62% |
| v3.7 | 2026-03-03 | PHASE2-STABILITY-006: JWT 86자 보안키, AADS_ADMIN_PASSWORD 갱신, 호스트 PostgreSQL 연결(host.docker.internal+iptables 5432), MCP SSE ping stream 수정, checkpointer 3단계 fallback(local→supabase→memory), 118 PASS |
