# AADS HANDOVER HISTORY
최종 업데이트: 2026-03-09 | AADS-188D/188E 배포 완료 기록

---

## AADS-188D/188E 완료 (2026-03-09)

- **제목**: Monaco DiffEditor + approve-diff API + E2E 테스트 배포
- **188D**: 에이전트 코드 수정 시 diff 미리보기(Monaco DiffEditor) 및 승인/거절/편집 후 반영 API·UI 구현. 백엔드: ApproveDiffRequest/Out, agent_hooks diff_preview(original_content/modified_content), POST /api/v1/chat/approve-diff, _diff_approval_store. 프론트: CodeDiffViewer, CodePanel, useDiffApproval, code-editor.css, chat/page.tsx 연동.
- **188E**: Agent SDK 자율 실행 E2E 테스트 추가 (execute_stream, 3턴 이상, approve-diff approve/reject/400) — tests/test_e2e_agent_sdk.py.
- **커밋**: aads-server main b6dc996, aads-dashboard feature/188d-monaco-diff 9fb4620(lockfile), aads-docs 511cd1e.
- **배포**: docker compose -f docker-compose.prod.yml build aads-server aads-dashboard && up -d 완료 (2026-03-09).
- **보고서**: https://github.com/moongoby-GO100/aads-docs/blob/main/reports/AADS-188D-188E-monaco-approve-diff-report_20260307.md

---

## AADS-186E-2 완료 (2026-03-09)

- **제목**: Extended Thinking 활성화 + Programmatic Tool Calling + 4계층 영속 메모리
- **내용**: CTO 인텐트 4개(cto_strategy/cto_code_analysis/cto_verify/cto_impact) → claude-opus + thinking(budget_tokens=10000, max_tokens=16000). ptc_executor.py 신규(PTC 병렬 실행, code_execution 도구). memory_manager.py + 024_memory_tables.sql(session_notes, ai_meta_memory). save_note/recall_notes/learn_pattern 도구, context_builder에 \<recent_sessions\>/\<learned_patterns\> 주입. chat_service 자동 노트 저장.
- **커밋**: aads-server 004508e, aads-docs a39365f
- **테스트**: 41/41 통과

---

## AADS-186D 완료 (2026-03-09)

- **제목**: 전체 통합 + 나머지 프로젝트 CKP + Tool Search Tool + Prompt Caching 최적화
- **내용**: KIS/GO100/SF/NTV2/NAS .claude/projects/{project}/ CKP 5종(25개 파일) 생성. ckp_manager.scan_remote_project() 개선(.claude/projects 우선, HANDOVER 폴백). Tool Search Tool 도구 카테고리 안내. cache_config.py Prompt Caching. 주간 CEO 브리핑 매주 월요일 09:00 KST. 통합 테스트 31/31.
- **커밋**: aads-server 4587714
- **문서**: HANDOVER v12.13, STATUS.md 갱신

---

## AADS-186E-1 완료 (2026-03-09)

- **제목**: 검색·크롤링 완전 해방 — Jina Reader + Crawl4AI + deep_crawl 도구
- **내용**: jina_reader_service.py, crawl4ai_service.py, deep_crawl_service.py(research_crawl 파이프라인). tool_registry crawl 그룹: jina_read, crawl4ai_fetch, deep_crawl. intent_router url_read/deep_crawl. docker-compose.crawl4ai.yml(선택). tests/test_crawl_tools.py 8/8.
- **커밋**: aads-server dde8147
- **문서**: HANDOVER v12.12, STATUS.md 갱신

---

## 채팅 서버검색+프론트 반영 (2026-03-09)

- **목적**: 채팅창에서 클로드봇에게 서버(SSH) 파일 검색 지시 가능 + 작업현황 확인용 웰컴 칩 추가
- **백엔드**:
  - `tool_registry.py`: `list_remote_dir` 도구 스키마 추가, `read_remote_file`에 `project`+`path` 반영
  - `tool_executor.py`: `_list_remote_dir`(ceo_chat_tools.tool_list_remote_dir), `_read_remote_file`(project+path → ceo_chat_tools)
  - 인텐트 매핑: `server_file` → list_remote_dir/read_remote_file
  - `intent_router.py`: 인텐트 `server_file` 추가, 키워드 폴백(서버 검색, 원격 서버, SSH 파일, KIS/SF/NTV2 서버)
- **프론트**:
  - `ChatStream.tsx`: `_TOOL_DISPLAY_NAMES`에 list_remote_dir "서버 파일 검색 (SSH)", read_remote_file "원격 서버 파일 읽기"
  - `ActionChips.tsx`: 웰컴 칩 "작업현황", "서버 파일 검색" 추가; 동적 칩(서버/원격/작업 관련)
- **배포**: 2026-03-09 68서버 `docker compose -f docker-compose.prod.yml build aads-server aads-dashboard` 후 `up -d` 완료
- **보고서**: aads-docs/reports/chat_server_search_frontend_report_20260306.md

---

## AADS-171 완료 사항 (2026-03-08)
- **LiteLLM Proxy Docker 설치**: docker-compose.prod.yml에 aads-litellm 서비스 추가 (ghcr.io/berriai/litellm:main-latest, port 4000, mem 512m, aads_network)
- **litellm-config.yaml**: Gemini 3종(flash-lite/flash/pro) + Claude 3종(haiku/sonnet/opus) 모델 등록, 일 $5 예산 상한, spend_logs 활성화
- **model_router.py 확장**: INTENT_MODEL_MAP(18 인텐트) + resolve_intent_model() 비동기 함수 + check_monthly_budget_warning()
  - 일 $5 초과 시 claude-opus → claude-sonnet 자동 다운그레이드
  - 월 $150 초과 시 경고 로그
- **신규 API 엔드포인트**:
  - GET /api/v1/chat/cost-summary — LiteLLM spend/logs 기반 일별/월별 비용 요약 + Opus 차단 여부
  - GET /api/v1/chat/intent-model-map — 인텐트→모델 매핑 테이블 확인
- **.env.example 업데이트**: GEMINI_API_KEY, LITELLM_MASTER_KEY, LITELLM_BASE_URL, 예산 변수 추가
- **aads-server commit**: 5f62e5e — https://github.com/moongoby-GO100/aads-server/commit/5f62e5e

---

## AADS-161 완료 사항 (2026-03-08)
- **AADS 문서 전체 업데이트**: 매니저 자기인식 프로토콜 + bridge.py 자동화 명시 + CEO 전달 금지
- **HANDOVER.md v10.2**: 매니저 자기인식 프로토콜 섹션, 지시서 자동화 파이프라인 섹션, 6개 프로젝트 라우팅 테이블 추가
- **CEO-DIRECTIVES.md v3.4**: §0 운영 원칙, D-035(bridge 자동 감지), D-036(매니저 자기인식), D-037(CEO 전달 금지), R-022(위반 처리)
- **HANDOVER-RULES.md v1.1**: §6-2 매니저 역할 보강, §6-2-1 자기인식 프로토콜, §6-14 라우팅 테이블
- **WORKFLOW-PIPELINE.md v3.3**: Step 2 bridge 자동 감지 명시
- **RULE-MATRIX.md v1.3**: D-035/D-036/D-037/R-022 행 추가 (23→27규칙)

---

## AADS-160 완료 사항 (2026-03-07)
- **CEO 직접 검수**: AADS-157/158/159 검수 → 버그 4건 발견 및 수정
- CEO Chat TypeError 2건: logger.info structlog kwargs → f-string 변환
- Dashboard React Error #31: object를 React children으로 렌더링 → typeof 체크 + JSON.stringify
- Ops toLocaleString/toFixed crash: undefined 값에 메서드 호출 → nullish coalescing
- 모델 선택 드롭박스: 버튼 7개 → select 드롭박스 29개 모델
- KST 타임존 동기화: 8개 파일 날짜를 Asia/Seoul timezone으로 통일

---

## AADS-159 완료 사항 (2026-03-07)
- **CEO Chat Playwright 브라우저 자동화**: 6개 도구 추가 (T-003 Phase 2 확장)
- ceo_chat_tools.py: browser_navigate/snapshot/screenshot/click/fill/tab_list
- 도메인 화이트리스트: *.newtalk.kr, github.com, raw.githubusercontent.com, localhost
- Playwright Python 싱글턴 컨텍스트 (asyncio.Lock, headless Chromium)

---

## AADS-158 완료 사항 (2026-03-07)
- **Pending 대기큐 정리**: 11개 완료/중복 지시서 → archived/ 이동
- STATUS.md: last_completed=AADS-157 → 갱신 완료

---

## AADS-157 완료 사항 (2026-03-07)
- **CEO Chat v2 Core Engine 연결**: Intent Classifier + DashboardCollector + Tool-use 루프 + Directive Submit
- classify_intent(): 5분류 (dashboard/diagnosis/research/execute/strategy)
- ceo_chat_tools.py: read_file, read_github, search_logs, query_db, fetch_url
- directives.py: POST /api/v1/directives/submit

---

## AADS-148 완료 사항 (2026-03-08)
- **HANDOVER 4계층 전면 재작성**: v10.0 (11개 섹션, 운영 원칙 최상단, 토큰 상한 폐기)
- HANDOVER-RULES.md v1.0 신규 생성 (13개 섹션, 파이프라인/매니저/작업자/효율성12전략)
- CEO-DIRECTIVES.md v3.3 (D-023 v2, D-033/D-034/R-021)
- RULE-MATRIX.md v1.2 (19→23규칙), WORKFLOW-PIPELINE.md v3.2
- 대시보드 채널 카드 문서 링크 + 편집 기능 (DB+auto-push+30초 갱신)
- 트리거 메시지 편집 기능 (DB+auto-push+30초 갱신)
- project-docs/trigger-messages 백엔드 API

---

## AADS-149 완료 사항 (2026-03-07)
- **파이프라인 전수조사 버그 5건 Wrap 보고서**: reports/AADS-149-WRAP_pipeline-audit-5bugs.md
- **BUG-1 (Critical)**: auto_trigger.sh — SCP 실패 시 seen_tasks 영구 차단
  - 수정: SCP 실패 직후 `unset seen_tasks["$task_id"]` 롤백 로직 추가
- **BUG-2 (High)**: auto_trigger.sh — RESULT 폴러 타임아웃 25분 < HARD_TIMEOUT 30분
  - 수정: `seq 1 50` → `seq 1 80` (40분, HARD_TIMEOUT + 10분 여유)
- **BUG-3 (Medium)**: auto_trigger.sh — `aads_lifecycle_queued()` 호출 시 _rt/_title 변수 미정의
  - 수정: 변수 추출 코드를 lifecycle 호출 앞으로 이동
- **BUG-4 (Critical)**: claude_exec.sh — Claude가 pending/running 경로 삭제 가능 + /proc grep 금지 없음
  - 수정: CONTEXT_HEADER에 디렉토리 조작 금지 + /proc grep -r 금지 규칙 주입
- **BUG-5 (High)**: done_watcher.sh — 서버 114 SSH 포트 7916 미지정
  - 수정: `get_project_ssh_port()` 함수 추가, SSH/SCP 명령에 `-p/-P` 옵션 적용
- **교훈 L-011 등록**: shared/lessons/infra/L-011_pipeline-audit-critical-patterns.md (5대 패턴)
- **적용 서버**: 211(전체), 68(auto_trigger+claude_exec), 114(auto_trigger+claude_exec)

---

## AADS-145 완료 사항 (2026-03-07)
- **Tasks 시스템 통합**: scripts/claude_exec.sh + claude_exec.sh(main)
  - CLAUDE_CODE_TASK_LIST_ID 환경변수 자동 생성 및 export
  - 지시서 실행 시 ~/.claude/tasks/{task_id}.json 생성 (in_progress → done/failed)
  - 세션 복구: Tasks JSON 상태 체크로 PENDING/DONE 이중관리 제거
  - auto_trigger.sh: _process_directive에 Tasks 상태 사전 조회 추가
- **투기적 실행 (AADS-141 확장)**:
  - 하트비트 "type":"final_commit" 이벤트 추가 (scripts/claude_exec.sh)
  - /tmp/aads_final_commit_{task_id}.signal 파일로 신호 전달
  - auto_trigger.sh: _speculative_preload() 함수 — final_commit 감지 시 다음 작업 git pull 병렬 실행
  - 후처리 실패시 _preload_fail 플래그로 프리로드 취소
- **컨텍스트 자동관리**:
  - claude_exec.sh(main): _ctx_monitor() 백그라운드 — JSON output-format token usage 파싱
  - 70% 도달: 로그에 /compact 권고 기록
  - 90% 도달: 중간 결과 저장 + 재시작 프롬프트로 재실행
  - scripts/claude_exec.sh: 행 수 기반 토큰 추정 모니터링 + tee로 출력 캡처
  - 2회 연속 Edit 실패 감지 → CTX-EDIT-FAIL 경고

---

## AADS-144 완료 사항 (2026-03-07)
- **CEO-DIRECTIVES v3.2**: D-022~D-025 신규 추가
  - D-022: 지시서 포맷 v2.0 (필수/선택 필드 + 기본값 체계)
  - D-023: HANDOVER 3계층 분리 (Core/HISTORY/ARCHIVE)
  - D-024: 모델 라우팅 (XS→haiku, S/M→sonnet, L/XL→sonnet/opus)
  - D-025: 우선순위큐 impact/effort 정렬
- **HANDOVER 3계층 분리**: Core(≤1500토큰) + HISTORY.md + ARCHIVE.md 구조 적용
- **claude_exec.sh**: 지시서 model 필드 동적 읽기 + D-024 size 기반 자동 라우팅
- **auto_trigger.sh**: _select_next_file에 impact/effort 정렬 추가
- **WORKFLOW-PIPELINE v3.1**: D-024 모델 라우팅 섹션 추가
- **RULE-MATRIX v1.1**: D-022~D-025 행 추가

## AADS-143 완료 사항 (2026-03-07)
- **WORKFLOW-PIPELINE v3.0**: shared/rules/WORKFLOW-PIPELINE.md 생성 (8단계 파이프라인 + 라우팅 구조)
- **git-push 감시**: claude_exec.sh commit SHA 기록 + auto_trigger.sh verify_git_push() (3회 backoff + Telegram + recovery_logs)
- **HANDOVER 표준화**: AADS HANDOVER 3섹션 삽입 (작업 파이프라인/매니저 권한 한계/AI 작업자 규칙) + 5개 프로젝트 staged
- **NTV2/NAS CEO-DIRECTIVES**: NTV2-CEO-DIRECTIVES.md + NAS-CEO-DIRECTIVES.md 신규 배포
- **RULE-MATRIX.md**: 15개 규칙 × 8단계 매핑 매트릭스 생성
- **대시보드 트리거 통합**: /channels 트리거 전송 버튼 + /managers HANDOVER 링크

## AADS-142 완료 사항 (2026-03-07)
- deploy_heartbeat_3servers.sh: 3서버 일괄 배포 스크립트 (inotify-tools/jq 설치 + systemd 등록)
- CEO-DIRECTIVES v3.1: D-018 개정(하트비트 기반 L1) + D-021 신규(세션 관리 원칙) 배포
- E2E 검증: 파일럿 B-1(완료 흐름 4초), B-2(Tier2 시뮬레이션 150초), B-3(3건 즉시투입 2초) 완료
- 서버 68: session_watchdog RUNNING(PID 6166), meta_watchdog 감시 포함, HC=200 ✅

## 참조
- Core: HANDOVER.md
- Archive: HANDOVER-ARCHIVE.md
- CEO-DIRECTIVES: CEO-DIRECTIVES.md (v3.4)
