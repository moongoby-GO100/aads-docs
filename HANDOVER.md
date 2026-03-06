# HANDOVER – AADS (Autonomous AI Development System)
> 최종 업데이트: 2026-03-06 (v5.28 — T-105(Watchdog): SSH오탐 944건 dismissed, ssh_command 5건 disabled, watchdog-daemon 자기감시 OFF, recovery_log 컬럼 추가(error_type/service_name/auto_executable), 자동복구 씨앗 10건(nginx/postgres/aads-server/disk/memory/api/bridge), attempt_auto_recovery() 추가, watchdog summary recovery_seeds 필드, pending→0, commit 51765b7, push 200; @aads_approval_bot 전용봇 전환(8685636032), /pending 인라인버튼, GO100 health check 오탐 수정(api/v4/health→/health); v5.27 — T-105: 대기큐 정리(AADS_20260306_12*.md 4건 done/ 이동) + T-095: aads-docs/designs/UI-PROTO-001.md 생성(와이어프레임/컴포넌트트리/API맵/데이터흐름도) + T-103(확장): channels.py context_docs+system_prompt 필드 추가, GET /api/v1/channels/{id}/context-package 엔드포인트(URL fetch 조합), 6채널 context_docs 등록(AADS_MGR 3건 포함) + T-104: ModelSelector.tsx(5모델 칩 선택UI), /genspark 페이지(iframe→새탭 폴백), Sidebar Genspark AI 메뉴, 대시보드 Genspark 바로가기 카드, ceo-chat ModelSelector 연동, ceo_chat.py model override(MODEL_ID_MAP 4종), api.ts sendCeoMessage model 파라미터+getContextPackage, npm build 0에러, Docker 재배포, curl /api/v1/channels 6건+context-package 반환 HTTP 200; v5.26 — T-103: 대화창 관리 CRUD UI — channels.py(GET/POST/PUT/DELETE /api/v1/channels + GET /api/v1/channels/{id}/context-package), system_memory category=channels, 6건 마이그레이션(AADS_MGR/GO100_MGR/KIS_V41_MGR/SF_MGR/NAS_MGR/NTV2_MGR), 대시보드 /channels 페이지(추가/수정/삭제 모달+🔗✏️🗑 액션), Sidebar 대화창 메뉴, api.ts 채널 CRUD 4함수, public/manager/channels.json, npm build 0에러, docker rebuild+재배포, curl 외부 6건 HTTP 200; v5.25 — T-102: CEO 문서 자동 저장 시스템 — documents.py API(GET/POST/DELETE /api/v1/documents), _index.json, 소급저장 5건(PLAN-001/TECH-001/TECH-002/RESEARCH-001/STATUS-001), backfill_ceo_documents.py, generate_manager_snapshot.py(public/manager/documents.json), bridge.py T-102 문서감지 로직(DOCUMENT_PATTERNS 5종+save_as_document), docker rebuild, HTTP 200 확인; v5.24 — T-100: genspark_bridge.py 완료보고 메시지 재감지 방지 — SKIP_PATTERNS 10개, 자기발송 마커[BRIDGE-SENT], processed_ids 중복차단, DIRECTIVE 템플릿/규칙설명 구분, auto_trigger RESULT파일 스킵; v5.23 — T-039(멀티서버): 전 서버 통합 감시 + CEO 승인 큐 — approval_queue+monitored_services 테이블(3서버 13서비스), POST /approval/request(텔레그램 인라인 버튼 승인/반려), POST /approval/{id}/approve(auto_command/claude_code/manual 실행), POST /approval/{id}/reject, GET /approval/pending(severity 정렬), GET /approval/history, telegram_bot.py(long polling 콜백 systemd aads-telegram-bot), watchdog_daemon.py 확장(check_all_services 2분주기: http_health/ssh_command/process, 3회실패→자동복구68/CEO승인요청원격, 10회실패→긴급TG), SSH id_ed25519_newtalk(211서버211.188.51.113/114서버116.120.58.155), commit 63516b9, push 200; v5.22 — T-038: Watchdog 에러 자동기록·학습·자동복구 — error_log+recovery_log 테이블(해시 기반 에러 그룹화, occurrence_count), POST /watchdog/errors(신규/재발 감지, 자동복구 트리거), PUT /watchdog/errors/{hash}/resolution(해결법 등록, Experience Memory L3 학습), GET /watchdog/errors(필터 조회), GET /watchdog/summary(공개 요약, 인증 불필요), watchdog_daemon.py(systemd aads-watchdog 상시 실행 30초 주기 Docker/API/Nginx/디스크/done큐 감시), memory_helper.sh report_error() 추가, auto_trigger.sh 실패 시 자동 보고, bridge.py 예외 시 자동 보고, commit 01560fd, push 200; v5.21 — T-090(재실행): project_tasks 멀티프로젝트 파이프라인 — project_tasks 218건(AADS:80,GO100:61,ShortFlow:52,NewTalk:16,KIS:9), directives/reports/analytics API project_tasks UNION, context.py _upsert_task_result T-090 스키마(UNIQUE(task_id,source)) 적용, 마이그레이션 스크립트 migrate_project_tasks.py, 프론트 프로젝트 탭 건수 뱃지+빈상태 안내, npm build 0에러, docker deploy, API KIS:9건/GO100:61건, commit e0d75f5/c2b503e, push 완료; v5.20 — T-091: 원격 에이전트 task_result 자동수집 + project_tasks upsert — context.py SystemMemoryRequest data 필드 허용(get_value), _upsert_task_result 타임스탬프 파싱 수정(fromisoformat), schema.sql project_tasks DDL 추가, scripts/aads_remote_agent.py auto_report_task_results() 추가(프로젝트 디렉토리 24h 보고서 감지→task_result 전송), 211서버 KIS/GO100/ShortFlow + 114서버 ShortFlow/NewTalk/NAS 동일 수정, curl 검증 data/value 양방향 HTTP200+DB upsert 확인, commit 6c042bb, push 완료; v5.19 — T-089: 통합 수정 5파트 — classify_project 화이트리스트(VALID_PROJECTS+_validate_project_name 30자초과→AADS), 채널 확장(REQUIRED_CHANNELS 8개+통합지휘소 cross_msg집계), KST 변환, task_cost_log 시드(CEO-Test/$0.69+T-031/$2.50+T-073/$0.15)+/dashboard/costs 신규, 프론트엔드 30초 폴링+채널 수집미설정 UI, commit-msg hook 3레포, npm build 0에러, docker deploy, git push 완료; v5.18 — T-082: classify_project 전면 재작성(3단계 분류+파일명 접두사 1순위+AADS 인프라 2순위+한글 오분류 제거+validate_project_name 추가), analytics by_project 통합, HTTP 200; v5.17 — aads_remote_agent.py message_type 수정(HTTP 422→200), aads-remote-agent.service 재시작, genspark_bridge.py _save_conversation_to_aads 구조 확인(3000자 청크, AADS API 저장); v5.16 — T-074: 분석탭 g.reduce 에러 수정(Array.isArray 안전 체크, byProject/byServer/dailyTrend 변수 도입), classify_project 정확도 개선(AADS 키워드 1순위 — handover/dashboard/bridge/ceo채팅 → AADS 분류), KST 시간 변환(_to_kst_str 헬퍼, task-history started_at/finished_at KST 변환, 프론트엔드 toKST() 헬퍼 — Directives/Remote/서버카드 전체 적용), npm build 0에러, docker restart, git push 완료; v5.15 — T-073: CEO Chat v2 — 계층 메모리(ceo_facts/ceo_chat_sessions/ceo_chat_messages/ceo_session_summaries 4테이블), Context Manager(4-layer 시스템프롬프트), Model Router(complex→Opus/code→Sonnet/simple→Gemini Flash), 세션 요약 자동생성, 5 API 엔드포인트, CEO Chat 프론트엔드(채팅UI+비용대시보드+세션관리), Sidebar CEO Chat 메뉴, npm build 0에러, docker deploy, git push 완료; v5.14 — T-071: HANDOVER v5.14 업데이트(원격 에이전트 REMOTE_211/REMOTE_114 online 기록, 서버 배치표 추가); T-070: REMOTE_211 에이전트 온라인(211서버 aads_remote_agent.py 데몬 가동, Context API REMOTE_211 등록, heartbeat 60초); T-069: 파싱 엔진 구축(HTML/JSON/마크다운 멀티파서 ParseEngine 클래스, POST /api/v1/parse 엔드포인트, 처리율 90%+); T-068: 애널리틱스 대시보드(KPI 4종 실시간 집계, /dashboard/analytics 페이지, GET /dashboard/analytics+stats+export 3엔드포인트, npm build 0 에러); v5.13 — T-072: React Error #31 완전 수정(safeRender+중복제거), API 평탄화(directives/reports error=int, error_breakdown 분리), project 필터 백엔드 지원(?project=KIS 등), _parse_directive_file started_at+completed_at+duration_seconds 추가, 지시서 테이블 에러유형+시작+완료 컬럼 추가, 클라이언트 project 필터 fix, npm build 0 에러, docker deploy, git push 완료; v5.12 — T-066 task-history REMOTE_116→114, status 분류(notify→reported, task_result→completed/error), T-067: 분석탭 추가+analytics API; v5.11 — T-066: CEO 대시보드 Tasks 페이지 추가(지시서/보고서/원격작업 3탭), API 4개(/dashboard/directives+reports+reports/{filename}+task-history), Sidebar Tasks 메뉴, docker-compose 볼륨 마운트(.genspark/directives), npm build 0 에러, git push 완료; v5.10 — T-060: JS 에러 5건 수정(slice/map→Array.isArray), progress_percent 필드명 수정, Pipeline 사용자 프로젝트 생성+모니터링 UI; v5.9 — T-048: 프로젝트 통합 현황 API 4개 엔드포인트; T-049: CEO 대시보드 7페이지+다크테마; T-056: Docker 재빌드; T-057: Memory project_status 6건 적재+dashboard 데이터 연동 보강; v5.8 — T-045: main.py에 memory(5 엔드포인트)+conversations(2 엔드포인트) 라우터 등록, Nginx /api/v1/memory/ + /api/v1/conversations 프록시 활성화, 외부 7/7 200 확인; T-046: CEO 대시보드 5페이지(홈 시스템현황/대화뷰어/매니저/태스크/프로젝트), 다크 테마(bg-gray-950), Sidebar v0.3.0(5메뉴), npm build 0 에러; T-047: AADS_MGR required_docs URL 수정(aads-docs repo), NAS_MGR docs URL 수정(nas→nas-image); v5.6 — T-038: 매니저 협업 API 4개 엔드포인트 (GET /memory/search, GET /memory/ceo-decisions, POST /memory/cross-message, GET /memory/inbox/{agent_id}), 매니저 레지스트리 6건 등록 (SALES/FINANCE/CONTENT/QA/CUSTOMER/INVESTMENT MGR), INTERVAL make_interval 버그 수정, memory.py+mobile_qa.py 컨테이너 배포; v5.5 — T-039: 모바일 QA 파이프라인 — Android 에뮬레이터(KVM)+Appium+Gemini Vision 6항목 감리, /api/v1/mobile-qa/* 5 엔드포인트, IOS-QA-SETUP.md, QA Agent mobile_android|mobile_ios 분기; v5.4 — T-036: GET /context/public-summary 읽기전용 엔드포인트 200 PASS, 민감 데이터 0건; T-037: bridge.py 대화분류/저장(classify_aads_conversation 7카테고리), memory_helper.sh save_manager_report() 추가; v5.3 — T-035: 모바일 반응형 최적화 — Sidebar 햄버거 메뉴, ClientLayout, 로그인 풀스크린)
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
| **T-030** | **03-04** | **61a858a** | **200** | **116서버 뉴톡 V2 이미지 검수 클라이언트 배포: scripts/deploy_to_116.sh(SSH/SCP 자동배포 6단계 — 연결테스트/디렉터리생성/파일전송/.env.aads생성/환경설정/동작확인), docs/ProductController_AADS.php(Laravel 연동 방법A Shell+방법B HTTP+배치검수), image-quality-gate REJECT(score=4 1x1PNG) exit 1 확인, image-qa 배치 2건 FAIL JSON 스코어카드 반환, Context API qa_results/newtalk_t030_test 저장 완료. 116서버 실제 SSH 배포는 id_ed25519_newtalk 키 제공 후 deploy_to_116.sh 실행으로 완료** |
| **T-029** | **03-04** | **7ba68e1** | **200** | **211서버 ShortFlow 검수 클라이언트 배포 + run_v4_pipeline.py 통합: scripts/aads_qa_local/run_v4_integration.py(AadsQualityGate 클래스 + quality_gate_check 함수 — quality_gate.sh→aads_qa_client.sh→requests 3단계 fallback), scripts/deploy_to_211.sh(SSH/SCP 자동 배포 스크립트 — 연결테스트/파일전송/권한설정/pip설치/동작확인 6단계), scripts/run_v4_pipeline_qa_patch.py(quality_gate_before_upload 함수), docs/SHORTFLOW-QA-INTEGRATION.md(방법A 모듈import+방법B subprocess+배포파일목록), AADS 서버 4개 엔드포인트 HTTP 200 확인(health/quality-gate/extract-spec/benchmark-specs)** |
| **T-032** | **03-05** | **f7bc122** | **200** | **68서버 프로덕션 강화 (옵션 A CEO 승인): docker-compose.prod.yml(restart always, json-file logging, memory 1.5G, pg shared_buffers=256MB/max_connections=50, redis maxmemory 128mb allkeys-lru, healthcheck 전 서비스), nginx-aads.conf(rate_limit 60r/m, gzip, proxy_read_timeout 120s, X-Frame-Options/X-Content-Type-Options/X-XSS-Protection/CORS 보안헤더), aads.service(systemd oneshot docker compose up/down), backup.sh(pg_dump 5433포트, 7일 자동삭제, cron 03:00), health 200✅, dashboard 307→200✅** |
| **T-035** | **03-05** | **c0def58** | **200** | **모바일 반응형 최적화: Sidebar.tsx(햄버거 버튼+overlay+slide-in, X버튼, 라우트변경 자동닫힘, md 이상 고정), ClientLayout.tsx 신규("/login" 사이드바제외, useState isMenuOpen), layout.tsx(ClientLayout 교체, viewport meta), Header.tsx(pl-12 md:pl-0, text-sm md:text-base), page.tsx(p-3 md:p-6, flex-col sm:flex-row, w-full sm:w-auto, grid-cols-1 sm:grid-cols-2 lg:grid-cols-3), login/page.tsx(max-w-sm sm:max-w-md, p-5 sm:p-8), npm build 0 errors, git push 완료** |
| **T-036** | **03-05** | **df233cf** | **200** | **GET /context/public-summary 읽기전용 엔드포인트: SENSITIVE_KEYS 마스킹(_sanitize 키+값), procedural_memory 컬럼 수정, main.py NameError 버그 수정, 200✅/민감데이터0건/POST405** |
| **T-037** | **03-05** | **—** | **—** | **bridge.py 확장: classify_aads_conversation(7카테고리), extract_decisions/action_items, save_aads_conversation(Context API 저장), process_message T-037 통합. memory_helper.sh: save_manager_report() 추가(매니저 보고 Context API 저장). 검증 PASS** |
| **T-039(멀티서버)** | **03-06** | **63516b9** | **200** | **전 서버 통합 감시 + CEO 승인 큐 + Claude Code 자동 조치: approval_queue+monitored_services 테이블(3서버13서비스), POST /approval/request(TG인라인버튼)+approve(auto_command/claude_code/manual)+reject+GET pending/history, telegram_bot.py(long polling systemd), watchdog_daemon.py check_all_services(2분주기 http_health/ssh_command/process, 3회실패→자동복구/CEO승인요청, 10회실패→긴급TG), commit 63516b9, push 200** |
| **T-039** | **03-05** | **7459b8a** | **200** | **모바일 QA 파이프라인: KVM 가용(8코어), Appium 3.2.0(uiautomator2+xcuitest), docker/android-emulator/docker-compose.yml(API-33), scripts/mobile_qa/adb_utils.sh, app/services/mobile_qa.py(MobileQAService: connect_android/connect_ios/install_and_launch/take_screenshot/tap_element/scroll_down/input_text/get_page_source/check_crash/run_test_scenario/close), app/api/mobile_qa.py(GET health/POST install/POST test-scenario/POST audit-screen/POST full-qa), design_auditor.py audit_mobile_screen(6항목: layout_consistency/touch_target_size/text_readability/navigation_clarity/visual_hierarchy/platform_compliance, PASS≥48/CONDITIONAL36-47/FAIL≤35), qa_pipeline.py run_mobile_qa(mobile_android|mobile_ios 분기), qa.py project_type분기, iOS graceful skip(IOS_APPIUM_URL 미설정시 NotImplementedError), IOS-QA-SETUP.md(Option A Mac mini/B Cloud/C GitHub Actions), .env Mobile QA 변수 추가** |
| **T-031** | **03-04** | **TBD** | **200** | **AADS 코어 고도화 — LangGraph Native Supervisor 프로덕션 전환: supervisor.py(TaskSpec 기반 동적 배정, 병렬 실행 Researcher+Architect, max_iterations CEO 에스컬레이션, max_llm_calls R-012, fallback 체인), judge_agent.py(Gemini 3.1 Pro 독립 모델, success_criteria↔output_artifacts 구조화 비교, fail시 rework_instructions JSON), autonomy_gate.py 신규(PostgreSQL autonomy_stats/autonomy_levels 테이블, 성공률≥90%→auto_approve/<70%→HITL재활성화, 최소20건미만 항상HITL), model_router.py T-002 프로덕션 매핑(Supervisor/Architect: claude-opus-4-6 $5/$25, PM/Developer/QA: claude-sonnet-4-6 $3/$15, Judge: gemini-3.1-pro-preview $2/$12, DevOps: gpt-5-mini $0.25/$2, Researcher: gemini-2.5-flash $0.30/$2.50, primary→fallback→error 3단계 체인), test_core_production.py 신규(4시나리오 34개 테스트 34/34 PASS), health ✅** |
| **T-045** | **03-05** | **d4f45ee** | **200** | **main.py memory+conversations 라우터 등록, 7 엔드포인트 활성화, Nginx 프록시** |
| **T-046** | **03-05** | **ad190a9** | **200** | **CEO 대시보드 5페이지, 다크 테마, Sidebar v0.3.0** |
| **T-047** | **03-05** | **ca71bf3** | **200** | **AADS_MGR/NAS_MGR docs URL 수정, HANDOVER v5.8** |
| **T-048** | **03-05** | **5b594b2** | **200** | **프로젝트 통합 현황 API 4개 엔드포인트 (dashboard, {id}, timeline, alerts)** |
| **T-049** | **03-05** | **a0125ae** | **200** | **CEO 대시보드 7페이지+다크테마 (Home, Project Status, Conversations, Managers, Decisions, Pipeline, Settings)** |
| **T-056** | **03-05** | **—** | **200** | **Dashboard Docker 재빌드 완료** |
| **T-057** | **03-05** | **9ab699c** | **200** | **Memory project_status 6건 적재, dashboard 데이터 연동 보강, HANDOVER v5.9** |
| **T-060** | **03-05** | **3e17ec8** | **200** | **JS 에러 5건 수정(y.slice/e.map→Array.isArray), progress_percent 필드명 수정, Pipeline 사용자 프로젝트 생성+모니터링 UI(createProject/autoRunProject/resumeProject/getProjectCosts+10초 자동갱신), npm build 0 에러, 7페이지 HTTP 200, HANDOVER v5.10** |
| **T-062** | **03-05** | **TBD** | **PARTIAL** | **116서버 Remote Agent 준비: aads_remote_agent.py(T-061 기반, aiohttp, /health+/status+/tasks+auto_report+collect_conversations), aads-remote-agent-116.service(systemd), deploy_remote_to_116.sh(6단계 자동배포), PROJECTS=newtalk_v2/NT_MGR, 68서버 Context API REMOTE_116 데이터 저장 확인(Memory API id=21 ok). 실제 116서버 배포는 NT116_IP + id_ed25519_newtalk SSH 키 필요** |
| **T-066** | **03-05** | **292564a/8cc7aa8** | **200** | **CEO 대시보드 Tasks 페이지(지시서/보고서/원격작업 3탭), API 4개(/dashboard/directives+reports+reports/{filename}+task-history), Sidebar Tasks 메뉴, docker-compose 볼륨 마운트(.genspark/directives), npm build 0 에러, HANDOVER v5.11** |
| **T-067** | **03-05** | **TBD** | **200** | **분석탭 추가 + analytics API: task-history REMOTE_116→114 수정, status 분류(notify→reported, task_result→completed/error), GET /dashboard/analytics 엔드포인트, 분석 탭 UI(완료율/에러율/원격작업 현황), npm build 0 에러** |
| **T-068** | **03-05** | **TBD** | **200** | **애널리틱스 대시보드: KPI 4종(총 지시서/완료/에러/원격작업) 실시간 집계, /dashboard/analytics 페이지, GET /dashboard/analytics+/dashboard/analytics/stats+/dashboard/analytics/export 3 엔드포인트, Chart.js 없이 Tailwind CSS 순수 구현, npm build 0 에러, docker deploy 완료** |
| **T-069** | **03-05** | **TBD** | **200** | **파싱 엔진 구축: ParseEngine 클래스(HTML/JSON/마크다운 3파서 통합), scripts/parse_engine.py 신규, POST /api/v1/parse 엔드포인트, 지시서 메타데이터(task_id/title/priority/server/steps) 자동 추출, 처리율 90%+, 단위테스트 12/12 PASS** |
| **T-070** | **03-05** | **TBD** | **200** | **REMOTE_211 에이전트 온라인: 211서버 aads_remote_agent.py 데몬 가동, aads-remote-agent-211.service(systemd), Context API REMOTE_211 등록(heartbeat 60초), PROJECTS=KIS/GO100/ShortFlow, GET /api/v1/memory/agents/REMOTE_211 200 확인** |
| **T-071** | **03-05** | **TBD** | **200** | **HANDOVER v5.14 업데이트: 원격 에이전트 현황(REMOTE_211 online, REMOTE_114 online) 기록, 서버 배치표 추가(211: KIS/GO100/ShortFlow docs, 114: ShortFlow service/NewTalk/NAS, 68: AADS), T-068~T-071 완료 항목 추가, 보고서 /root/project-docs/aads/reports/T-068_069_070_RESULT.md 작성** |
| **T-072** | **03-05** | **7656f1e(dashboard)** | **200** | **Tasks 페이지 복구: safeRender dedup 수정, API 평탄화(error=int, error_breakdown), project 필터 백엔드 지원, Directive 타입 추가, 테이블 컬럼(에러유형/시작/완료), npm build 0 에러** |
| **T-074** | **03-05** | **f9a7929(server)/55e59ae(dashboard)** | **200** | **분석탭 g.reduce 에러 수정(Array.isArray 안전 체크, byProject/byServer/dailyTrend 로컬 변수), classify_project 정확도 개선(AADS 1순위 — handover/bridge/dashboard/ceo채팅 → AADS, nas→nasync 정확 매칭), KST 시간 변환(backend _to_kst_str 헬퍼+task-history KST 적용, frontend toKST() — Directives/Remote/서버카드), npm build 0에러, docker restart, git push 완료** |
| **T-073** | **03-05** | **b926c73(server)/7656f1e(dashboard)** | **200** | **CEO Chat v2 — 계층 메모리 + 컨텍스트 DB + 모델 분기 엔진: DB 4테이블(ceo_chat_sessions/ceo_chat_messages/ceo_facts 22건/ceo_session_summaries), Context Manager 4-layer 시스템프롬프트(facts+세션요약+활성태스크+최근대화 3,500~5,500토큰), Model Router(complex→claude-opus-4-5/code→claude-sonnet-4-5/simple→gemini-2.0-flash/default→sonnet), Gemini Flash 세션 자동 요약(10턴마다+세션종료), 5 API(/ceo-chat/message+/sessions+/sessions/{id}+/end-session+/cost-summary), CEO Chat 프론트엔드(채팅버블+모델정보+비용대시보드+세션관리+접이식사이드바), Sidebar CEO Chat 메뉴(Tasks↓ Pipeline↑), /ceo-chat 200 OK, npm build 0 에러, docker deploy 완료** |
| **T-082** | **03-05** | **2179ff9** | **200** | **classify_project 전면 재작성: 3단계 분류(1순위 파일명 접두사 KIS_/GO100_/SF_/NT_/SALES_/NAS_, 2순위 AADS 인프라 키워드 30+개, 3순위 프로젝트 고유 키워드), validate_project_name() 추가(허용값 9개, 50자 초과→AADS), 한글 오분류 버그 수정(title 50자 초과→파일명, project YAML/텍스트 파싱 validate 래핑), analytics by_project 통합(validate_project_name 전적용), 검증: AADS 172건/KIS 2건(한글 문장 0건), HTTP 200** |
| **T-089** | **03-05** | **28f7bc3(server)/ee7627b(dashboard)** | **200** | **통합 수정 5파트: ①classify_project 화이트리스트(VALID_PROJECTS+_validate_project_name 30자초과→AADS+MAPPING 기반, 모든 return 래핑, title[:100], get_directives/reports/analytics 반환전 일괄 정규화) ②채널 확장(REQUIRED_CHANNELS 8개: AADS/KIS/SALES/ShortFlow/GO100/NewTalk/NAS/통합지휘소, cross_msg_%집계, 누락채널 수집미설정) ③비용테이블(task_cost_log 시드 CEO-Test/$0.69+T-031/$2.50+T-073/$0.15, GET /dashboard/costs 신규, analytics cost_status=active/$3.3465) ④프론트엔드(conversations 30초폴링+KST표시+수집미설정UI, tasks 비용KPI "데이터수집중"→실제금액) ⑤Git hook(commit-msg 3레포, bak캐시제거17+16건), npm build 0에러, docker deploy, HTTP 200** |

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
- **Public Summary**: GET /api/v1/context/public-summary (인증 불필요, 민감 데이터 자동 제거, 웹 Claude 실시간 조회용)

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

### 서버 전체 배치표 (v5.14, T-071 기준)

| 서버 | 호스트명 | 주요 서비스 | 원격 에이전트 | 상태 |
|------|----------|------------|--------------|------|
| **68** | aads.newtalk.kr | AADS 코어(FastAPI+LangGraph), CEO 대시보드(Next.js), PostgreSQL, Redis, Nginx | — | **ONLINE** |
| **211** | ShortFlow/GO100 서버 | KIS 프로젝트, GO100 파이프라인, ShortFlow 문서/스크립트 | **REMOTE_211** | **ONLINE** |
| **114** | ShortFlow 서비스 서버 | ShortFlow 영상 서비스, NewTalk V2, NAS 연동 | **REMOTE_114** | **ONLINE** |

### 원격 에이전트 현황 (v5.14)

| 에이전트 ID | 서버 | 프로젝트 | 상태 | 마지막 heartbeat |
|------------|------|----------|------|----------------|
| REMOTE_211 | 211서버 | KIS, GO100, ShortFlow | **online** | 60초 주기 |
| REMOTE_114 | 114서버 | ShortFlow service, NewTalk, NAS | **online** | 60초 주기 |

- **Context API 확인**: `GET /api/v1/memory/agents/REMOTE_211`, `GET /api/v1/memory/agents/REMOTE_114`
- **배포 스크립트**: `scripts/deploy_remote_to_211.sh`, `scripts/deploy_remote_to_114.sh`

### 서버 배포 현황 (T-027+T-028+T-029, QA 클라이언트)

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

## 14. Mobile QA System (v5.5, T-039)

- **Android**: Docker 에뮬레이터(halimqarroum/docker-android:api-33) + KVM(/dev/kvm 8코어) + ADB + Appium 3.2.0 + Gemini Vision 6항목 감리
- **iOS**: Appium xcuitest 드라이버(v10.24.2) 사전 설치, Mac 연결 시 즉시 활성화 (IOS_APPIUM_URL 설정)
- **API**: `/api/v1/mobile-qa/*`
  - `GET  /health` — Android 에뮬레이터 상태, Appium 상태, KVM 가용성, iOS 설정 여부
  - `POST /install` — APK 다운로드 → ADB 설치 → 앱 실행 → 스크린샷
  - `POST /test-scenario` — 시나리오 실행(tap/input/scroll/screenshot/wait) + 크래시 감지
  - `POST /audit-screen` — Gemini Vision 6항목 감리(PASS≥48/CONDITIONAL36-47/FAIL≤35)
  - `POST /full-qa` — 설치→시나리오→감리→판정→Context API 저장→CEO 알림
- **QA Agent 통합**: `project_type=mobile_android|mobile_ios` 분기 (qa.py/qa_pipeline.py)
- **iOS 설정 가이드**: `aads-docs/docs/IOS-QA-SETUP.md` (Option A Mac mini / B Cloud Mac / C GitHub Actions)
- **Gemini Vision 모바일 6항목**: layout_consistency, touch_target_size, text_readability, navigation_clarity, visual_hierarchy, platform_compliance
- **환경변수**: `ANDROID_EMULATOR_HOST`, `ANDROID_EMULATOR_PORT`, `APPIUM_URL`, `IOS_APPIUM_URL`(Mac 연결 시)

---

## 15. Watchdog System (v5.22, T-038)

- **목적**: 모든 서비스/스크립트 에러 자동 감지·기록·학습·자동복구
- **DB 테이블**:
  - `error_log`: error_hash(16자 SHA256) 기반 에러 그룹화, occurrence_count 자동 증가, resolution/recovery_command 학습
  - `recovery_log`: 자동복구 실행 이력 (명령어, 성공여부, 출력)
- **API** (prefix: `/api/v1`):
  - `POST /watchdog/errors` — 에러 보고. 동일 해시면 카운트 증가, auto_recoverable이면 복구 자동 실행
  - `PUT /watchdog/errors/{hash}/resolution` — 해결법 등록 (Experience Memory L3에도 학습)
  - `GET /watchdog/errors` — 에러 목록 (status/error_type 필터, Monitor Key 인증)
  - `GET /watchdog/summary` — 공개 대시보드 요약 (인증 불필요)
- **Watchdog 데몬** (`/root/aads/scripts/watchdog_daemon.py`):
  - systemd `aads-watchdog.service` 상시 실행, 자동재시작(RestartSec=10)
  - 30초 주기 감시: Docker(aads-server/aads-postgres), API health, Nginx
  - 10분(20사이클)마다: 디스크 사용률(90% 경고), done 큐 건수(200건 초과 경고)
  - 1시간(120사이클)마다: 정상 heartbeat 로그
  - 텔레그램 CEO 알림 (TELEGRAM_BOT_TOKEN/TELEGRAM_CHAT_ID 설정 시 활성화)
- **자동복구 화이트리스트**: `docker restart`, `docker compose`, `systemctl reload nginx`, `systemctl restart`, `supervisorctl restart`, `curl`
- **에러→해결법 학습 루프**: 에러 발생→기록 → 해결법 PUT → 재발 시 recovery_command 자동 실행
- **스크립트 통합**:
  - `memory_helper.sh`: `report_error(error_type, source, server, message)` 함수
  - `auto_trigger.sh`: 태스크 실행 실패 시 `task_execution_failure` 자동 보고
  - `bridge.py`: `report_error_to_watchdog()` 함수, except 블록에서 `bridge_error` 자동 보고

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
| v4.9 | 2026-03-04 | T-029: 211서버 ShortFlow 검수 클라이언트 배포 + run_v4_pipeline.py 통합 — run_v4_integration.py(AadsQualityGate 3단계 fallback + quality_gate_check), run_v4_pipeline_qa_patch.py(quality_gate_before_upload), deploy_to_211.sh(SSH/SCP 자동배포 6단계), SHORTFLOW-QA-INTEGRATION.md(방법A+B 연동 가이드), commit 7ba68e1, AADS 4개 엔드포인트 HTTP 200 확인 |
| v5.0 | 2026-03-04 | T-030: 116서버 뉴톡 V2 이미지 검수 클라이언트 배포 — deploy_to_116.sh(자동배포 스크립트 6단계), ProductController_AADS.php(Laravel 방법A+B+배치), image-gate exit 1(REJECT, score=4) 동작 확인, image-qa 배치 JSON 스코어카드 반환, Context API qa_results 저장. 실제 116서버 SSH 배포는 id_ed25519_newtalk 키 필요 |
| v5.5 | 2026-03-05 | T-039: 모바일 QA 파이프라인 — Android 에뮬레이터(KVM)+Appium 3.2.0(uiautomator2+xcuitest)+Gemini Vision 6항목 감리, MobileQAService(10메서드), /api/v1/mobile-qa/* 5 엔드포인트, design_auditor.audit_mobile_screen, qa_pipeline.run_mobile_qa, QA Agent mobile_android|mobile_ios 분기, IOS-QA-SETUP.md, .env Mobile QA 환경변수, commit 7459b8a |
| v5.2 | 2026-03-05 | T-032: 68서버 프로덕션 강화 — docker-compose.prod.yml(restart always, json-file logging 10m/3, aads-server memory 1.5G, pg shared_buffers=256MB/max_connections=50, redis maxmemory 128mb allkeys-lru, healthcheck 전 서비스), nginx-aads.conf(rate_limit 60r/m, gzip, proxy_read_timeout 120s/connect 10s, 보안헤더 X-Frame-Options DENY/X-Content-Type-Options nosniff/X-XSS-Protection/CORS aads.newtalk.kr), aads.service(systemd oneshot), backup.sh(cron 03:00 daily, 7일 자동삭제), health ✅, commit f7bc122 |
| v5.1 | 2026-03-04 | T-031: AADS 코어 고도화 — LangGraph Native Supervisor 프로덕션 전환 — supervisor.py(TaskSpec 동적 배정, 병렬 Researcher+Architect, max_iterations CEO 에스컬레이션, R-012 LLM 한도, fallback 체인), judge_agent.py(Gemini 3.1 Pro, success_criteria↔output_artifacts 구조화 비교, rework_instructions), autonomy_gate.py 신규(autonomy_stats/levels 테이블, 성공률≥90%→auto_approve/<70%→HITL), model_router.py T-002 완전 정렬(Opus-4-6/Sonnet-4-6/Gemini-3.1-Pro/GPT-5-mini, primary→fallback→error), test_core_production.py(34/34 PASS), health ✅ |
| v5.6 | 2026-03-05 | T-038: 매니저 협업 API 4개 엔드포인트 + 레지스트리 6건 등록(SALES/FINANCE/CONTENT/QA/CUSTOMER/INVESTMENT MGR), INTERVAL make_interval 버그 수정, memory.py+mobile_qa.py 컨테이너 배포 |
| v5.7 | 2026-03-05 | T-045: main.py memory+conversations 라우터 등록(7 엔드포인트), Nginx /api/v1/memory/+/api/v1/conversations 프록시 활성화, 외부 7/7 200 확인; T-046: CEO 대시보드 5페이지(홈/시스템현황/대화뷰어/매니저/태스크/프로젝트), 다크 테마(bg-gray-950), Sidebar v0.3.0(5메뉴), npm build 0 에러 |
| v5.8 | 2026-03-05 | T-047: AADS_MGR required_docs URL 수정(aads-docs repo), NAS_MGR docs URL 수정(nas→nas-image), HANDOVER v5.8 |
| v5.12 | 2026-03-05 | T-062: 116서버 Remote Agent 준비 — aads_remote_agent.py(T-061 기반 aiohttp 데몬), aads-remote-agent-116.service(systemd REMOTE_116), deploy_remote_to_116.sh(6단계 자동배포), PROJECTS=newtalk_v2/NT_MGR, Context API REMOTE_116 저장 확인. 실배포: NT116_IP + SSH 키 필요 |
| v5.11 | 2026-03-05 | T-066: CEO 대시보드 Tasks 페이지(지시서/보고서/원격작업 3탭), API 4개(GET /dashboard/directives+reports+reports/{filename}+task-history), Sidebar Tasks 메뉴, docker-compose .genspark/directives+project-docs 볼륨 마운트, npm build 0 에러, git push 완료 |
| v5.17 | 2026-03-05 | aads_remote_agent.py message_type→notify 수정(HTTP 422→200), aads-remote-agent.service 재시작 완료; genspark_bridge.py _save_conversation_to_aads() 확인(3000자 청크, category=conversation:{proj}, AADS /memory/ API 저장); REMOTE_211 자동 보고 정상 동작 확인 |
| v5.14 | 2026-03-05 | T-071(BRIDGE): HANDOVER 업데이트 — T-067(분석탭+analytics API) / T-068(애널리틱스 대시보드 3엔드포인트) / T-069(파싱 엔진 ParseEngine 12테스트 PASS) / T-070(REMOTE_211 온라인) 완료 항목 추가, 원격 에이전트 현황(REMOTE_211/REMOTE_114 online), 서버 전체 배치표(68:AADS, 211:KIS/GO100/ShortFlow docs, 114:ShortFlow service/NewTalk/NAS) 추가 |
| v5.20 | 2026-03-05 | T-091: 원격 에이전트 task_result 자동수집 + project_tasks upsert — context.py data/value 양방향 허용(SystemMemoryRequest.get_value), _upsert_task_result 타임스탬프 파싱 수정(fromisoformat), schema.sql project_tasks DDL 추가, scripts/aads_remote_agent.py auto_report_task_results() 추가(프로젝트 디렉토리 24h 보고서 파일 감지→task_result 전송), 211서버 KIS/GO100/ShortFlow + 114서버 ShortFlow/NewTalk/NAS, curl 검증 KIS/ShortFlow PASS, commit 6c042bb, push 완료 |
| v5.23 | 2026-03-06 | T-039(멀티서버): 전 서버 통합 감시 + CEO 승인 큐 — approval_queue+monitored_services 테이블(3서버13서비스), POST /approval/request(TG인라인버튼)+approve(auto_command/claude_code/manual자동실행)+reject+GET pending/history, telegram_bot.py(long polling systemd aads-telegram-bot enabled), watchdog_daemon.py check_all_services(2분주기 http_health/ssh_command/process, 3회실패→자동복구68/CEO승인요청원격, 10회실패→긴급TG), SSH id_ed25519_newtalk(211=211.188.51.113/114=116.120.58.155), commit 63516b9, push 200 |
| v5.22 | 2026-03-06 | T-038: Watchdog 에러 자동기록·학습·자동복구 — error_log+recovery_log 테이블(해시 기반 그룹화, occurrence_count 자동증가), POST /watchdog/errors(신규/재발 감지, 자동복구 트리거), PUT /watchdog/errors/{hash}/resolution(해결법 등록 + Experience Memory L3 학습), GET /watchdog/errors(필터 조회), GET /watchdog/summary(인증 불필요 공개), watchdog_daemon.py(systemd aads-watchdog 상시 30초 감시: Docker/API/Nginx/디스크/done큐), memory_helper.sh report_error() 추가, auto_trigger.sh+bridge.py 에러 자동보고, commit 01560fd, push 200 |
| v5.21 | 2026-03-05 | T-092: 비용 자동 추적 — cost_tracker.py(PRICE_TABLE 6모델, calculate_cost/record_cost/parse_result_file), auto_trigger.sh 비용 추적 연동(exec_duration_ms 계산+cost_tracker.py record 자동 호출), task_cost_log session_duration_ms/source 컬럼 추가, GET /dashboard/costs by_project_model 집계 추가, analytics cost_status 'no_data' 수정, backfill_costs.py 실행(92파일 스캔 성공60 스킵32), DB 총 65건/$18.35, analytics cost=$17.92 status=active, HTTP 200, commit 6efa920, push 완료 |
| v5.24 | 2026-03-06 | T-100: genspark_bridge.py 완료보고 메시지 재감지 방지 — SKIP_PATTERNS 10개, 자기발송 마커([BRIDGE-SENT]), processed_ids 중복차단(MD5 해시 세트 최대1000), DIRECTIVE 템플릿/규칙설명 구분(작성규칙 텍스트 감지+T-NNN 실수 검증+done폴더 완료확인), auto_trigger RESULT파일 스킵, 무한루프 완전 차단 |
| v5.25 | 2026-03-06 | T-102: CEO 문서 자동 저장 시스템 — documents.py(GET/POST/DELETE /api/v1/documents 4엔드포인트, _index.json 관리, system_memory ceo_document 저장), bridge.py 문서감지 확장(DOCUMENT_PATTERNS 5종 키워드, classify_document, save_as_document), backfill_ceo_documents.py 소급저장(5건: PLAN-001/TECH-001/TECH-002/RESEARCH-001/STATUS-001), Docker 재빌드, curl 5건 반환 확인 |
| v5.26 | 2026-03-06 | T-103: 대화창 관리 CRUD UI — channels.py(GET/POST/PUT/DELETE /api/v1/channels + GET /api/v1/channels/{id}/context-package), system_memory category=channels, 6건 마이그레이션(AADS_MGR/GO100_MGR/KIS_V41_MGR/SF_MGR/NAS_MGR/NTV2_MGR), 대시보드 /channels 페이지(추가/수정/삭제 모달+액션), Sidebar 대화창 메뉴, api.ts 채널 CRUD 4함수, npm build 0에러, docker rebuild+재배포, curl 외부 6건 HTTP 200 |
| v5.27 | 2026-03-06 | T-105+T-095+T-103(확장)+T-104: 대기큐 정리(4건 done/ 이동) + UI-PROTO-001.md 설계서(designs/) + channels.py context_docs+system_prompt+context-package endpoint + 6채널 context_docs 등록 + ModelSelector.tsx(5모델 칩) + /genspark 페이지(iframe→새탭 폴백) + Sidebar Genspark AI 메뉴 + 대시보드 Genspark 카드 + ceo-chat ModelSelector 연동 + ceo_chat.py model override + api.ts model 파라미터+getContextPackage + npm build 0에러 + Docker 재배포 + HTTP 200 |

## 지시서 작성규칙

```
>>>DIRECTIVE_START
Task ID: T-NNN
제목: (한글 제목)
서버: 68 (aads)
우선순위: P0-CRITICAL / P1-HIGH / P2-NORMAL
예상 시간: N분
예상 비용: $0
의존성: (없음 또는 선행 Task ID)

(작업 내용 상세 기술)
>>>DIRECTIVE_END
```

- 타임스탬프: KST 기준 (UTC 금지)
- 작업 완료 후 HANDOVER.md 반드시 갱신
- git commit + push 필수
