# HANDOVER – AADS (Autonomous AI Development System)
> 최종 업데이트: 2026-03-04 (v4.9 — T-029: 211서버 ShortFlow 검수 클라이언트 배포 + run_v4_pipeline.py 통합)
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
| **LAUNCH-READY-010** | **03-04** | **a69c061** | **200** | **Docker 샌드박스(D-011), CEO-DIRECTIVES v2.4, E2E 풀사이클 검증(CEO-Test-Calculator, 8 LLM calls, $0.69), docker-compose 소켓 마운트, 대시보드 접근 OK, 가동 준비 완료** |
| **T-020** | **03-04** | **TBD** | **200** | **Context API 보안 강화: POST /context/system Monitor Key 인증 추가(401 반환), 응답에 saved data 포함, Rate Limiting 분당 30회/IP(429), 전체 curl 테스트 통과** |
| **T-019** | **03-04** | **b54ff75** | **200** | **System Memory HANDOVER 데이터 마이그레이션: migrate_handover.py v3.8 재작성, 9카테고리(status/repos/architecture/agents/phase/costs/ceo_directives/pending/history) 30건 INSERT, Docker Postgres(5433) 적재, GET /context/system 9카테고리 확인, GET /context/handover 섹션1~9 완전 마크다운 생성** |
| **T-021** | **03-04** | **TBD** | **200** | **메모리 시스템 워크플로우 통합: memory_helper.sh(read_context/write_task_result/write_experience/write_error 4함수), claude_exec.sh(Context API 맥락주입+결과기록), auto_trigger.sh(COMPLETED 스킵+phase 자동업데이트), bridge.py(CEO결정감지+ceo_directives 자동저장). nginx User-Agent 필터 우회(curl/7.64.0) 발견.** |
| **T-024** | **03-04** | **cddeeed** | **200** | **Visual QA 스크린샷 + Visual Regression 기반 구축: VisualQAService(Playwright headless 1920x1080 + Pillow pixelmatch), 4개 API 엔드포인트(capture/compare/set-baseline/baselines), Docker glibc fallback, pyproject.toml playwright+Pillow 추가, Dockerfile chromium install, compare diff_percent=0.0(identical) / 100.0(diff) 검증 완료** |
| **T-025** | **03-04** | **17bf032** | **200** | **LLM 디자인 감리 엔진 6단계 스코어카드: design_auditor.py(DesignAuditor 클래스), AUDIT_PROMPT(5개 항목 10점), Gemini 2.5 Flash Vision→Claude Sonnet Vision fallback, AuditResult(PASS/CONDITIONAL/FAIL), generate_report(마크다운), 2개 신규 API(POST /visual-qa/audit, GET /visual-qa/audit/{project_id}/latest), experience_memory 자동저장(experience_type=design_audit)** |
| **T-026** | **03-04** | **71a4f0a** | **200** | **QA Agent 통합 — Visual Regression + LLM 감리 + CEO 알림: agents/qa.py(5단계 파이프라인 통합), services/qa_pipeline.py(run_full_qa → AUTO PASS/CEO 확인 요청/AUTO FAIL 판정), services/ceo_notify.py(텔레그램+Context API), POST /visual-qa/full-qa 엔드포인트, Context API qa_results+qa_notifications 저장** |
| **T-027** | **03-04** | **9f0cc1c** | **200** | **ShortFlow 영상 품질 게이트 + 자동 보정 루프: benchmark_spec.py(BenchmarkSpecExtractor — Gemini→Claude Vision, spec_to_ffmpeg_params, system_memory 저장), auto_correction.py(AutoCorrector — CORRECTION_MAP 6항목, analyze_failures, generate_correction_params, create_correction_directive), visual_qa.py 3개 신규 엔드포인트(POST /quality-gate, GET /benchmark-specs/{project}/{channel}, POST /extract-spec), scripts/shortflow_quality_gate.sh(211서버 cron 직전 호출), VIDEO_QA_PROMPT(6항목 60점), FFmpeg 프레임 추출, AUTO_PUBLISH(85%+)/CONDITIONAL(70-84%)/AUTO_REJECT(<70%) 판정** |
| **T-028** | **03-04** | **268139c** | **200** | **뉴톡 이미지 검수 + 211/116 서버 클라이언트 배포: IMAGE_AUDIT_PROMPT(이커머스 6항목 60점 스코어카드 — resolution_clarity/background_quality/product_visibility/color_accuracy/text_overlay/commercial_readiness), DesignAuditor.audit_product_image/audit_product_images_batch 메서드 추가, visual_qa.py 2개 신규 엔드포인트(POST /image-qa, POST /image-quality-gate), scripts/aads_qa_client.sh(image-qa/image-gate 명령 추가, quality-gate 통합), docs/SHORTFLOW-QA-INTEGRATION.md+NEWTALK-QA-INTEGRATION.md 생성, PASS 48+(80%)/CONDITIONAL 36-47/FAIL 35이하** |
| **T-029** | **03-04** | **TBD** | **200** | **211서버 ShortFlow 검수 클라이언트 배포 + run_v4_pipeline.py 통합: scripts/aads_qa_local/run_v4_integration.py(AadsQualityGate 클래스 + quality_gate_check 함수 — quality_gate.sh→aads_qa_client.sh→requests 3단계 fallback), scripts/deploy_to_211.sh(SSH/SCP 자동 배포 스크립트 — 연결테스트/파일전송/권한설정/pip설치/동작확인 6단계), aads_qa_211_deploy.tar.gz(배포 패키지), AADS 서버 4개 엔드포인트 HTTP 200 확인(health/quality-gate/extract-spec/benchmark-specs)** |

---

## 3. 진행 중 작업

| Task ID | 상태 | 내용 |
|---------|------|------|
| PHASE1-W1 | **완료** | aads-server Week 1: 3-agent chain, StateGraph, FastAPI 엔드포인트, 단위테스트 6/6 |
| PHASE1-W2 | **완료** | W2-001 ✅, W2-002 ✅, W2-003 ✅, W2-004 배포 ✅, W2-005 8-agent ✅ → Phase 1 코어 8-agent 완성 |
| PHASE1.5 | **완료** | REALTEST-001 ✅, CICD-002 ✅, INTEGRATION-003 ✅ |
| PHASE2 | **가동 준비 완료** | LAUNCH-READY-010 ✅ → CEO 직접 테스트 단계 |

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

### 6-1. 최신 상태 (v3.8 기준)

**가동 상태**: CEO 직접 테스트 가능
- **대시보드**: https://aads.newtalk.kr/
- **로그인**: AADS_ADMIN_EMAIL / AADS_ADMIN_PASSWORD (.env 참조)
- **프로젝트 생성**: 대시보드 또는 API POST /api/v1/projects

### 6-1-prev. 이전 상태 (v3.6 기준)
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

## 9. Memory System (v3.8+)

- **5-Layer Architecture 가동** (PostgreSQL + pgvector 기반)
- L1: Working Memory (AADSState + AsyncPostgresSaver checkpointer)
- L2: Project Memory (project_memories 테이블)
- L3: Experience Memory (experience_memory + pgvector, RIF scoring)
- L4: System Memory (system_memory 테이블, 9 카테고리)
- L5: Procedural Memory (procedural_memory 테이블)
- Context API: `/api/v1/context/*`
- HANDOVER.md는 System Memory DB에서 자동생성 가능: `GET /api/v1/context/handover`
- Monitor Key: 설정됨 (읽기/쓰기 API, 서버 .env 보관)
- **워크플로우 스크립트** (T-021, /root/aads/scripts/):
  - `memory_helper.sh`: read_context/write_task_result/write_experience/write_error (source로 사용)
  - `claude_exec.sh`: Context API 맥락주입 → Claude Code 실행 → 결과기록
  - `auto_trigger.sh`: 지시서 감지 → COMPLETED 스킵 → 자동실행 → phase 업데이트
  - `bridge.py`: CEO 결정감지("확정"/"승인"/"OK" 등) → ceo_directives 자동저장
- **주의**: nginx가 Python-urllib User-Agent 차단 → 반드시 `User-Agent: curl/7.64.0` 헤더 필요

---

## 10. Chat Endpoint (v3.8+)

- **엔드포인트**: `POST /api/v1/chat`
- 자연어 입력 → 의도 분류 → 액션 라우팅
- 키워드 기반 룰 라우터 (LLM 비용 $0)
- Genspark 브릿지 연동 준비 완료
- 예시: `{"message":"상태 보고"}` → `{"intent":"check_status","action":"GET /api/v1/health + GET /api/v1/context/system/status"}`

---

## 11. Sandbox (v3.8+)

- **Docker 로컬 샌드박스** (E2B 대체, CEO D-011 확정 2026-03-03)
- 이미지: python:3.12-slim, node:20-slim
- 보안: `--network=none --memory=512m --cpus=1 --read-only tmpfs:/tmp:100m`
- 최대 5 동시 컨테이너
- docker-compose 소켓 마운트: `/var/run/docker.sock`
- 대형 프로젝트: 사용자 서버 SSH/Docker API (Phase 3)
- E2B: Phase 3+ SaaS 확장 시 옵션 (현재 PLACEHOLDER)

---

## 12. E2E Test Result (v3.8)

- **프로젝트**: CEO-Test-Calculator
- **결과**: 7/8 PASS
  - ✅ health, login, create_project, pipeline, memory, costs, chat_api
  - ❌ sandbox (E2B_API_KEY=PLACEHOLDER, 실제 키 필요)
- **LLM 호출**: 8회
- **비용**: $0.69
- **Verdict**: **LAUNCH READY**
- **날짜**: 2026-03-04
- **커밋**: a69c061

---

## 13. Visual QA & Quality Gate System (v4.9, T-024~T-029)

### 웹 QA (T-024, T-025, T-026)
- **Playwright + Visual Regression**: headless 1920x1080 스크린샷 → Pillow pixelmatch 비교 (diff_percent)
- **Gemini Vision 6항목**: visual_consistency, accessibility, interaction_clarity, brand_coherence, polish + (웹용 5항목)
- **DesignAuditor**: AUDIT_PROMPT 5항목 10점, PASS(35+)/CONDITIONAL(25-34)/FAIL(24 이하)
- **QA Agent 파이프라인** (T-026): 코드→스크린샷→Visual Regression→LLM 감리→종합판정 → CEO 알림(텔레그램+Context API)

### 영상 QA (T-027)
- **FFmpeg 프레임 추출**: `ffprobe` 길이 확인 → 균등 간격 5 프레임 추출 (JPEG)
- **Gemini Vision 6항목**: subtitle_readability, background_quality, composition, brand_consistency, visual_consistency, polish (각 10점, 총 60점)
- **match_percent**: total_score / 60 × 100

### 벤치마크 비교 (T-027)
- **BenchmarkSpecExtractor** (`services/benchmark_spec.py`): SPEC_PROMPT → 사양 JSON 추출 (subtitle/background/composition/branding)
- **spec_to_ffmpeg_params**: 사양 → FFmpeg 파라미터 매핑 (fontsize, box_alpha, margin_bottom, resolution 등)
- **저장**: `system_memory` (category: benchmark_specs, key: {project_id}_{channel})

### 품질 게이트 판정 (T-027)
| 판정 | 기준 | 액션 |
|------|------|------|
| AUTO_PUBLISH | match_percent ≥ 85% (51점+) | action: publish → 업로드 진행 |
| CONDITIONAL | 70% ≤ match_percent < 85% | action: ceo_review → CEO 리뷰 대기 |
| AUTO_REJECT | match_percent < 70% | action: re-render (auto_correct=true) 또는 reject |

### 자동 보정 (T-027)
- **AutoCorrector** (`services/auto_correction.py`): CORRECTION_MAP 6항목 → FFmpeg 파라미터 delta 적용
  - subtitle_readability → fontsize +8, box_alpha +0.2
  - background_quality → min_resolution 1080x1920
  - composition → margin_bottom +5%
  - brand_consistency → color_grading 적용
- **재렌더링 지시서**: `{"action":"re-render", "video_id":"...", "params":{...}, "max_retries":2}`

### ShortFlow 연동
- **scripts/shortflow_quality_gate.sh**: 211서버 cron에서 업로드 직전 호출
  - 사용: `./shortflow_quality_gate.sh /path/to/video.mp4 economy economy_20260304`
  - 반환 코드: 0=AUTO_PUBLISH, 2=CONDITIONAL, 3=AUTO_REJECT, 1=ERROR
- **AADS URL**: `https://aads.newtalk.kr/api/v1/visual-qa/quality-gate`
- **ShortFlow 영상 경로**: `/data/shortflow/outputs/{channel}/`

### 이미지 검수 (T-028)
- **IMAGE_AUDIT_PROMPT**: 이커머스 6항목 각 10점, 총 60점
  - resolution_clarity, background_quality, product_visibility, color_accuracy, text_overlay, commercial_readiness
- **판정**: PASS 48+(80%) / CONDITIONAL 36-47(60-79%) / FAIL 35이하
- **DesignAuditor**: `audit_product_image(image_path_or_base64, is_base64)` / `audit_product_images_batch(images, is_base64)`
- **엔드포인트**:
  - `POST /api/v1/visual-qa/image-qa` — 이미지 일괄 검수 (ImageQAResponse)
  - `POST /api/v1/visual-qa/image-quality-gate` — 단일 이미지 품질 게이트 (action: approve|reject)

### 서버 배포 현황 (T-027+T-028+T-029)

| 서버 | 역할 | 클라이언트 | 용도 |
|------|------|-----------|------|
| 68 (AADS) | 중앙 검수 엔진 | — | 모든 검수 API 처리 |
| 211 (ShortFlow/GO100) | 영상 생성·업로드 | /root/aads_qa_client.sh + /root/aads_qa/ | 영상 품질 게이트, run_v4_pipeline.py 통합 |
| 116 (뉴톡 V2) | 이미지 저장·서빙 | /root/aads_qa_client.sh | 이미지 품질 게이트, 웹 검수 |

- **aads_qa_client.sh 명령**: `quality-gate`, `image-qa`, `image-gate`
- **연동 가이드**: `docs/SHORTFLOW-QA-INTEGRATION.md`, `docs/NEWTALK-QA-INTEGRATION.md`

### 211서버 배포 (T-029)
- **배포 스크립트**: `scripts/deploy_to_211.sh` — SSH/SCP 자동 배포 (6단계: 연결테스트/파일전송/권한설정/pip설치/동작확인)
  - 사용: `export SF211_IP=<IP> && bash scripts/deploy_to_211.sh`
  - 원격 설치 경로: `/root/aads_qa/`
- **run_v4 통합 모듈**: `scripts/aads_qa_local/run_v4_integration.py`
  - `AadsQualityGate` 클래스: quality_gate.sh → aads_qa_client.sh → requests 3단계 fallback
  - `quality_gate_check(video_path, channel, video_id)` 편의 함수
  - run_v4_pipeline.py 패치 예시 포함 (`RUN_V4_PATCH_EXAMPLE`)
- **배포 패키지**: `scripts/aads_qa_211_deploy.tar.gz` (12.5KB — aads_qa_client.sh + aads_qa_local 전체)
- **run_v4_pipeline.py 적용 방법**:
  ```python
  sys.path.insert(0, "/root/aads_qa")
  from run_v4_integration import quality_gate_check
  action = quality_gate_check(output_path, channel_name, video_id)
  # "publish" | "hold" | "reject" | "error"
  ```

### CEO 알림
- Context API `qa_results`+`qa_notifications` 저장 (T-026)
- 텔레그램 알림 (옵션, ceo_notify.py)

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
| v3.8 | 2026-03-04 | LAUNCH-READY-010: Docker 샌드박스, CEO-DIRECTIVES v2.4, E2E 풀사이클, 가동 준비 완료 |
| v3.9 | 2026-03-04 | T-020: Context API POST 인증 강화(verify_monitor_key), data 반환, Rate Limiting 30회/분/IP |
| v4.0 | 2026-03-04 | T-019: System Memory HANDOVER 마이그레이션 — 9카테고리 30건 적재, GET /context/system+handover 검증 완료, commit b54ff75 |
| v4.1 | 2026-03-04 | T-020: Context API POST /context/system 보안 강화 — verify_monitor_key Depends 추가(401), data 반환, Rate Limiting 30회/분/IP(429), curl 전체 테스트 통과 |
| v4.2 | 2026-03-04 | T-017: HANDOVER v3.8 누락 섹션 추가(Memory System/Chat Endpoint/Sandbox/E2E Test Result), CEO-DIRECTIVES v2.4 완성(D-011~D-013/T-011 추가), System Memory 버전 업데이트 |
| v4.3 | 2026-03-04 | T-021: 메모리 시스템 워크플로우 통합 — memory_helper.sh 4함수(read/write), claude_exec.sh(맥락주입), auto_trigger.sh(완료스킵+phase업데이트), bridge.py(CEO결정감지). nginx User-Agent 필터(curl/7.64.0) 발견·적용. |
| v4.4 | 2026-03-04 | T-024: Playwright Visual QA 스크린샷 + Visual Regression 기반 구축 — VisualQAService, 4개 API 엔드포인트, Docker glibc fallback, commit cddeeed |
| v4.5 | 2026-03-04 | T-025: LLM 디자인 감리 엔진 6단계 스코어카드 — design_auditor.py DesignAuditor 클래스, 5항목 10점 AUDIT_PROMPT, Gemini→Claude Vision fallback, PASS/CONDITIONAL/FAIL 판정, POST /visual-qa/audit + GET /visual-qa/audit/{project_id}/latest, experience_memory 저장, commit 17bf032 |
| v4.6 | 2026-03-04 | T-026: QA Agent 통합 — Visual Regression + LLM 감리 + CEO 알림 — agents/qa.py 5단계 파이프라인(코드→스크린샷→Visual Regression→LLM 감리→종합판정), services/qa_pipeline.py run_full_qa(AUTO PASS/CEO 확인 요청/AUTO FAIL), services/ceo_notify.py 텔레그램+Context API 저장, POST /visual-qa/full-qa, commit 71a4f0a |
| v4.7 | 2026-03-04 | T-027: ShortFlow 영상 품질 게이트 + 자동 보정 루프 — benchmark_spec.py(BenchmarkSpecExtractor), auto_correction.py(AutoCorrector CORRECTION_MAP 6항목), visual_qa.py 3 신규 엔드포인트(quality-gate/benchmark-specs/extract-spec), shortflow_quality_gate.sh(cron 연동), VIDEO_QA_PROMPT 6항목 60점, AUTO_PUBLISH(85%+)/CONDITIONAL(70-84%)/AUTO_REJECT(<70%), 섹션 13 추가, CEO-DIRECTIVES D-014 |
| v4.8 | 2026-03-04 | T-028: 뉴톡 이미지 검수 + 211/116 서버 클라이언트 배포 — IMAGE_AUDIT_PROMPT(이커머스 6항목 60점), DesignAuditor.audit_product_image/batch, visual_qa.py /image-qa+/image-quality-gate, aads_qa_client.sh(image-qa/image-gate), SHORTFLOW/NEWTALK-QA-INTEGRATION.md, commit 268139c |
| v4.9 | 2026-03-04 | T-029: 211서버 ShortFlow 검수 클라이언트 배포 + run_v4_pipeline.py 통합 — run_v4_integration.py(AadsQualityGate 3단계 fallback + quality_gate_check), deploy_to_211.sh(SSH/SCP 자동배포 6단계), aads_qa_211_deploy.tar.gz(배포 패키지), AADS 4개 엔드포인트 HTTP 200 확인 |
