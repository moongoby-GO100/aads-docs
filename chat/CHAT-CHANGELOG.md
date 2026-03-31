# CHAT-CHANGELOG.md — AADS 채팅 시스템 변경 이력

_최신 항목이 위에 오도록 역순 정렬_

---

## 2026-03-27 — 채팅 문서 체계 수립

- aads-docs/chat/ 디렉토리 생성
- CHAT-STATUS.md / CHAT-ROADMAP.md / CHAT-CHANGELOG.md 문서 3종 작성
- 이후 채팅 관련 작업 완료 시 3개 문서 갱신 필수

---

## 2026-03-24 — LiteLLM 이중 OAuth 폴백 파이프라인 (보고서: 20260324_chat_litellm_dual_oauth_pipeline.md)

**증상**: Naver OAuth 리밋 소진 시 Gmail 계정으로 자동 전환 안 됨

**원인**:
- `call_stream` 메인 경로가 CLI Relay → Agent SDK만 사용 (단일 계정)
- LiteLLM `simple-shuffle` 설정에서 Naver 쪽 반복 배정

**수정**:
- `model_selector.call_stream`: CLI Relay 실패 후 LiteLLM `/v1/messages` 단계 삽입
- `litellm-config.yaml`: Claude 배포에 `order: 1`(Gmail) / `order: 2`(Naver) 설정
- `router_settings.enable_pre_call_checks: true`, `num_retries: 3` 추가

**수정 파일**: `app/services/model_selector.py`, `litellm-config.yaml`

---

## 2026-03-11 — 파일 업로드 장애 수정 + read_uploaded_file 도구 추가 (보고서: 20260311_chat_file_upload_fix.md)

**증상**:
- 파일 업로드 후 "파일이 샌드박스에 도달하지 않았습니다" 응답
- DB에 `attachments=[]` 저장

**근본 원인**:
- `chat_drive_files.metadata` JSONB 컬럼이 `_JSONB_FIELDS` 화이트리스트에 누락
- `_row_to_dict()`가 문자열 `'{}'`을 dict로 파싱하지 않음 → DriveFileOut 검증 실패
- API 500 → 프론트엔드 pendingAttachments 미등록 → attachments: []

**수정 내역**:
- `chat_service.py`: `_JSONB_FIELDS`에 `"metadata"` 추가, 첨부파일 디버그 로깅 추가
- `page.tsx`: 업로드 중 전송 차단 (uploading state), 업로드 진행 표시기 추가

**신규 기능**:
- `read_uploaded_file` 도구: 이전 대화에서 업로드한 파일을 AI가 재접근 (compaction 후에도 유효)
- `intent_router.py`: `file_read` 인텐트 추가
- SSH 경로 정규식 확장: 한글/공백/비표준 하이픈(U+2011) 허용

**수정 파일**: `app/services/chat_service.py`, `app/services/tool_executor.py`, `app/services/tool_registry.py`, `app/services/intent_router.py`, `src/app/chat/page.tsx`

---

## 2026-03-10 — 무한 대화 + 자동 컨텍스트 관리 (보고서: 20260310_infinite_conversation_report.md)

**구현 항목 (6/6)**:

1. **도구 출력 자동 압축** — `context_compressor.py` (신규, 300줄)
   - 도구별 규칙 기반 압축 (LLM 호출 없음, $0 비용)
   - health_check → 1줄 요약 / read_remote_file → 앞 80줄+뒤 20줄 / query_database → 30행 / list_remote_dir → 50항목 / 기본 2000자 절단
   - 프론트에는 원본, 컨텍스트에는 압축본

2. **Observation Masking** — 슬라이딩 윈도우 10턴, 60K 토큰 초과 시 5턴
   - 이전 턴 도구 결과: `[도구 결과: {tool_name} — 상세 내용 생략]` 플레이스홀더
   - AI 추론/결정 텍스트는 100% 보존

3. **자동 구조화 요약** — `compaction_service.py` 전면 개편
   - 6섹션 강제 템플릿: 현재 목표 / 수정된 파일 / 내려진 결정 / 실패한 접근 / 보류 작업 / 활성 Directive
   - 증분 병합: Haiku로 기존 요약에 새 정보 병합
   - 80K 토큰 초과 시 `context_builder`에서 자동 트리거

4. **턴/예산 제한 제거**
   - Agent SDK: `_MAX_TURNS=0`, `_MAX_BUDGET_USD=0` (무제한)
   - 도구 루프: 5 → 20턴 (`MAX_TOOL_TURNS` 환경변수)
   - 히스토리 조회: 25 → 200 메시지
   - done 이벤트에 `session_cost`, `session_turns` 포함

5. **Yellow 도구 연속 실행 제한** — 11종 Yellow 도구 5회 연속 시 `yellow_limit` SSE 이벤트
   - 경고만 (차단 아님), `StreamState.yellowLimitWarning` 추가

6. **Prompt Caching** — 시스템 프롬프트 Layer 1 + 도구 정의 `cache_control: ephemeral`
   - `build_cached_tools()`: 1024토큰 이상 시 자동 활성화
   - 캐시 히트율 로깅: `prompt_cache: read=N create=N`

**수정 파일**: `context_compressor.py` (신규), `compaction_service.py`, `context_builder.py`, `model_selector.py`, `chat_service.py`, `agent_sdk_service.py`, `src/services/chatApi.ts`, `src/hooks/useChatSSE.ts`

---

## 2026-03-10 — AADS-190: Phase 0+1+2 (보고서: 20260310_AADS190_phase0_phase1_phase2_report.md)

### Phase 0 — 기반 인프라

- **에러 리포팅 시스템**: `POST /api/v1/chat/errors/report` 엔드포인트
  - 6가지 에러 타입: SSE_DISCONNECT / API_ERROR / STREAM_TIMEOUT / SESSION_SWITCH / TOOL_FAILURE / UNHANDLED
  - `src/services/errorReporter.ts` (신규), `ClientLayout.tsx`에서 1회 초기화
- **멀티세션 백그라운드 스트리밍**: `src/services/streamManager.ts` (신규)
  - 인메모리 세션별 스트림 상태 저장소, 5분 이상 경과 자동 정리
- **CEO Chat 메모리 통합**: `build_memory_context()` 자동 주입 (5섹션, ~2000토큰)
- **임베딩 API 검증**: `gemini-embedding-001` (3072차원) 정상 동작 확인

### Phase 1 — 원격 쓰기/실행 도구 (8개)

| 도구 | 등급 | 기능 |
|------|------|------|
| write_remote_file | Yellow | SSH 파일 쓰기 + .bak_aads 자동 백업 |
| patch_remote_file | Yellow | old_string→new_string 교체 (1-match 검증) |
| run_remote_command | Yellow | 화이트리스트 31개 허용 명령 실행 |
| git_remote_add | Yellow | 원격 git add |
| git_remote_commit | Yellow | 원격 git commit |
| git_remote_push | Yellow | 원격 git push (force push 차단) |
| git_remote_status | Green | 원격 git status |
| git_remote_create_branch | Yellow | 원격 브랜치 생성 |

보안 3단계: blocked regex → whitelist → pipe/체인 제한

### Phase 2 — 서브에이전트 + 확장

- `spawn_subagent` / `spawn_parallel_subagents` 도구 추가
  - 독립 LLM 호출, 읽기 도구 7종, asyncio.gather 병렬, Semaphore(5)
  - 단일 972ms, 병렬 3개 ~740ms 확인
- Agent SDK 턴/예산: 30턴/$10 → 100턴/$50 (환경변수)
- `COMPACTION_TRIGGER_TURNS` 환경변수로 조정 가능 (기본 20)

**수정 파일**: `app/api/ceo_chat_tools.py`, `app/routers/chat.py`, `app/services/tool_executor.py`, `app/services/tool_registry.py`, `app/services/agent_sdk_service.py`, `app/services/agent_hooks.py`, `app/services/subagent_service.py` (신규), `src/services/errorReporter.ts` (신규), `src/services/streamManager.ts` (신규), `src/hooks/useChatSSE.ts`, `src/hooks/useChatSession.ts`

---

## 2026-03-09 — AADS-188C: 3-Phase Chat Improvement (보고서: 20260309_AADS-188C_3phase_chat_improvement_report.md)

**문제**: CEO Chat에서 "확인하겠습니다", "알겠습니다" 등 빈 약속 응답 반복

**Phase 1 — 시스템 프롬프트 구조 개선**:
- `app/core/prompts/system_prompt_v2.py`
  - `LAYER1_BEHAVIOR`: 행동 원칙 4개 절대 규칙 (빈 약속 금지 / 행동 우선 / 불가능 명시 / 최소 기준)
  - `LAYER1_CEO_GUIDE`: CEO 비격식 표현 → 도구 매핑
  - `LAYER1_ROLE`: Orchestrator 역할 명시

**Phase 2 — 메타 도구 3개 + 인텐트 매핑**:
- `check_directive_status`: 작업 이력 + 서비스 상태 통합
- `delegate_to_agent`: Agent SDK 자율 실행 위임
- `delegate_to_research`: Deep Research 위임
- `task_query` / `status_check` 인텐트 추가 (기존 누락)
- `INTENT_REQUIRED_TOOLS` 매핑: 특정 인텐트 → 필수 도구 연결

**Phase 3 — Output Validator + 동적 tool_choice**:
- `output_validator.py` (신규): EMPTY_PROMISE / NO_TOOL_FOR_ACTION / TOO_SHORT 3종 탐지
  - 탐지 시 자동 재시도 (시스템 경고 + 재호출)
- `model_selector.py`: status_check / dashboard / task_query 등 → `tool_choice: any` (반드시 도구 호출)
- greeting / casual → tool_choice 생략

**수정 파일**: `system_prompt_v2.py`, `agent_hooks.py`, `agent_sdk_service.py`, `chat_service.py`, `intent_router.py`, `model_selector.py`, `tool_executor.py`, `tool_registry.py`, `output_validator.py` (신규)

---

## 2026-03-09 — 채팅창 전수 조사 및 수정 17건 (보고서: 20260309_chat_full_audit_and_fixes.md)

**CRITICAL (3건)**:
- `ceo_chat.py`: `await` on sync function 수정, Pydantic 필드 오류 수정
- `ceo_chat_tools.py`: SQL injection — UNION / INTO OUTFILE / LOAD_FILE 차단 추가
- `mcp/config.py`: 존재하지 않는 postgres MCP를 ALWAYS_ON에서 제거

**HIGH (7건)**:
- DB 연결 누수 수정 (try/finally conn.close() 보장)
- 헬스체크 외부 URL → localhost:8080으로 변경
- structlog kwargs 크래시: stdlib logger → structlog.get_logger() 전환 (7개 파일)
- `Sidebar.tsx`: session_id vs id 필드 불일치 수정 (`s.session_id ?? s.id`)
- `chatApi.ts`: API 응답 필드명 불일치 수정 (tokens_in/out/cost 호환)
- 메시지 limit=50 → 기본 200, 최대 1000으로 증가

**MEDIUM (3건)**:
- SSE 타임아웃: 30초 → 90초 (Extended Thinking 대비)
- 폴링 fallback: 가장 오래된 응답 → reverse().find()로 최신 선택
- Git MCP root: 빈 디렉토리 → 실제 git repo 경로로 변경

**DB 정리 (3건)**:
- research_archive FK: ON DELETE CASCADE로 변경
- stale cross_msg 308건, 빈 세션 1건 삭제
- 6개 소형 테이블 VACUUM ANALYZE

**기능 추가**:
- 백그라운드 스트리밍: 탭 닫아도 LLM 응답 DB 저장 보장 (`with_background_completion`)

**인프라**:
- 볼륨 마운트 추가: `/root/aads/aads-server` + `/root/aads/aads-dashboard`
- 디스크 정리: 99% → 40% (Docker 캐시 84GB + 이미지 8.5GB + 로그 0.86GB)
- Docker 로그 제한: 20MB x 3파일
- 타임존: `Asia/Seoul` 명시

---

## 2026-03-09 초 — AADS-182: CEO Chat UI 구축 (커밋 817dee1)

- recovered 메시지 tool UI 복원
  - streaming_placeholder에 tool_events 축적
  - 프론트 인라인 렌더 (재연결 시 도구 호출 UI 표시 유지)

---

## 2026-03-09 초 — 스트리밍 버블 2개 + 끊김 후 대화 불가 근본 수정 (커밋 8e3e68c)

- 스트리밍 중 버블이 2개 렌더링되는 문제 수정
- SSE 끊김 이후 새 메시지를 보낼 수 없는 문제 수정
- `_streaming_state` 추적 로직 보강

---

## 이전 이력 (AADS-170 이전)

AADS-170: CEO Chat-First 시스템 — app/routers/chat.py 신규 구축
- 워크스페이스 / 세션 / 메시지 / 아티팩트 / Drive CRUD
- SSE 스트리밍, 분기 대화, cursor 기반 페이지네이션
- chat_service.py 3969줄 서비스 레이어
