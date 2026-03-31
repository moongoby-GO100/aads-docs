# CHAT-STATUS.md — AADS 채팅 시스템 현재 상태

_최종 업데이트: 2026-03-27_

---

## 아키텍처 개요

```
[CEO 브라우저]
    │
    ▼
[Next.js App Router — aads-dashboard]
    │  /chat  (page.tsx / layout.tsx)
    │  SSE 스트리밍 수신 (text/event-stream)
    │  multipart/form-data or JSON 전송
    │
    ▼
[FastAPI — aads-server]
    │  메인 라우터 : app/routers/chat.py   (AADS-170, /api/v1/)
    │  레거시 라우터: app/api/chat.py       (키워드 인텐트, /api/v1/chat)
    │  서비스 레이어: app/services/chat_service.py  (3969 줄)
    │
    ├── intent_router.py      — 65개 인텐트 분류 (Gemini Flash-Lite)
    ├── model_selector.py     — LLM 라우팅 (CLI Relay → LiteLLM → Agent SDK → Gemini)
    ├── tool_executor.py      — 도구 실행 핸들러
    ├── tool_registry.py      — 도구 스키마 + 인텐트-도구 매핑
    ├── context_builder.py    — 3계층 컨텍스트 (system/history/tool)
    ├── output_validator.py   — 빈 약속 응답 탐지 + 자동 재시도
    ├── context_compressor.py — 도구 출력 압축 + Observation Masking
    ├── compaction_service.py — 구조화 요약 (6섹션 Haiku)
    └── subagent_service.py   — 서브에이전트 병렬 실행
    │
    ▼
[PostgreSQL — aads-postgres]
    chat_workspaces / chat_sessions / chat_messages
    chat_artifacts / chat_drive_files / chat_templates
    chat_research_cache / session_notes / ai_observations
```

---

## 현재 작동 중인 기능

### 워크스페이스 (6개 슬롯)
- CRUD: 생성 / 수정 / 삭제
- 세션 격리: 워크스페이스별 독립 세션 목록
- 워크스페이스 전환 UI (사이드바 상단)

### 세션 관리
- 세션 생성 / 제목 수정 / 핀 고정 / 삭제
- 태그 편집 (우클릭 컨텍스트 메뉴 → 태그 추가/삭제)
- 태그 필터: 사이드바 상단 태그 칩으로 세션 필터링
- cursor 기반 페이지네이션 (기본 50개, offset 레거시 호환)
- 세션 내보내기 (markdown / txt 형식)
- 세션 이동 시 상태 복원: streaming-status API로 진행중 여부 확인

### SSE 스트리밍
- `POST /api/v1/chat/messages/send` — StreamingResponse (text/event-stream)
- SSE 이벤트 타입: delta / tool_use / tool_result / done / error / heartbeat / yellow_limit
- done 이벤트에 session_cost / session_turns 포함 (비용 추적)
- 백그라운드 완료 보장: 탭을 닫아도 LLM 응답이 DB에 저장됨 (with_background_completion)
- SSE 재연결: GET /stream-resume?offset=N — 끊긴 지점부터 이어서 스트리밍
- 서버 재시작 후 수동 이어가기: POST /{session_id}/resume
- Heartbeat: 5초 간격 (Cloudflare/Nginx 100초 타임아웃 대비)
- 인터럽트: 스트리밍 중 CEO 추가 지시 큐 삽입 (POST /{session_id}/interrupt)
- 스트리밍 강제 중단: POST /{session_id}/stop

### 도구 실행 UI
- 메시지 버블 내 도구 호출 인라인 표시 (아이콘 + 도구명 + 입력 요약)
- 도구별 아이콘 맵: 📄 read_remote_file / 📁 list_remote_dir / ✏️ write_remote_file 등
- yellow_limit 경고: Yellow 등급 도구 5회 연속 시 UI 경고 표시
- Diff 승인 UI: POST /chat/approve-diff — 코드 변경 승인/거절

### 파일 첨부 (멀티미디어)
- 지원 형식: 이미지(JPG/PNG/GIF/WebP), 텍스트(MD/TXT/PY/TS 등), PDF, 영상(MP4/WebM)
- 전송 방식: multipart/form-data 또는 JSON base64
- 서버 업로드: POST /chat/files/upload → file_id 기반 참조
- 최대 크기: 50MB (서버), 영상 20MB
- 업로드 중 전송 차단 (uploading state)
- 클립보드 붙여넣기 / 드래그앤드롭 지원
- AI Drive: POST /chat/drive/upload — 세션 간 재사용 파일 저장소
- read_uploaded_file 도구: 이전 업로드 파일 AI 재접근 (2026-03-11 추가)

### 아티팩트 패널
- 우측 패널: report / code / chart / dashboard 4개 탭
- 모드: hidden / mini / expanded (데스크탑), 모바일 스와이프
- 아티팩트 CRUD: GET|PUT|DELETE /chat/artifacts/{id}
- Directive 등록: 아티팩트 → Directive 원클릭 변환
- 내보내기: POST /chat/artifacts/{id}/export

### 분기 (Branch) 대화
- 메시지 우클릭 → "분기 대화 시작"
- 분기 모드 배너 + POST /chat/messages/{id}/branch 호출
- 분기 메시지: 좌측 들여쓰기 + 초록 세로선 표시
- 세션별 분기 목록: GET /chat/sessions/{id}/branches

### 인용 / 참고자료
- CitationCard: 검색 결과 출처 카드 표시
- SourceCard: 참고 URL + 제목 표시
- FactCheckCard: 팩트체크 결과 인라인 표시

### 메모리 컨텍스트
- GET /chat/sessions/{id}/memory-context — 5섹션 메모리 조회
- MemoryContextBar 컴포넌트: 메모리 상태 바 표시
- 20턴마다 session_notes 자동 저장

### 원격 쓰기/실행 도구 (Phase 1 — AADS-190)
- write_remote_file: SSH 파일 쓰기 + .bak_aads 자동 백업
- patch_remote_file: old_string→new_string 교체 (1-match 검증)
- run_remote_command: 화이트리스트 기반 원격 명령 실행 (31개 허용)
- git_remote_add / commit / push / status / create_branch

### 서브에이전트 (Phase 2 — AADS-190)
- spawn_subagent: 독립 LLM 호출 (sonnet/opus/haiku 선택)
- spawn_parallel_subagents: asyncio.gather 병렬 실행 (Semaphore 5)
- 최대 5턴 도구 루프 / 120초 타임아웃

### 기타
- 메시지 검색: GET /chat/messages/search
- 메시지 재생성: POST /chat/messages/{id}/regenerate
- 북마크 토글: PUT /chat/messages/{id}/bookmark
- 슬래시 커맨드 메뉴 (SlashCommandMenu)
- 단축키 도움말 (ShortcutHelp)
- 액션 칩 (ActionChips)
- Thinking 인디케이터 (ThinkingIndicator)
- Research 진행 표시 (ResearchProgress / DeepResearchProgress)
- 프론트엔드 에러 리포팅: errorReporter.ts — 6종 에러 자동 수집 + /chat/errors/report
- 멀티세션 백그라운드 스트리밍: streamManager.ts — 세션 전환 후에도 스트림 추적
- 다크/라이트 테마 (ThemeToggle + ThemeContext)
- 인증 키 순서 설정: GET|POST /settings/auth-keys
- 채팅 템플릿: GET|POST|DELETE /chat/templates

---

## 알려진 제한사항 / 이슈

| 항목 | 내용 | 상태 |
|------|------|------|
| Naver OAuth 리밋 시 자동 폴백 | LiteLLM order(Gmail 1순위) + num_retries:3 으로 완화 | 부분 완화 |
| 도구 실행 진행률 | 도구 호출 횟수만 표시, % 진행률 미구현 | 개선 예정 |
| Yellow 도구 경고 UI | 텍스트 경고만, [계속]/[중단] 버튼 없음 | 개선 예정 |
| 비용/턴 카운터 상시 표시 | SSE done 수신하나 입력란 옆 상시 표시 없음 | 개선 예정 |
| Agent SDK 턴/예산 | 100턴 / $50 (AGENT_SDK_MAX_TURNS, AGENT_SDK_MAX_BUDGET_USD 환경변수) | 운영중 |
| 도구 루프 최대 | 20턴 (MAX_TOOL_TURNS 환경변수) | 운영중 |
| 히스토리 조회 | 최대 200 메시지 (Observation Masking 적용) | 운영중 |
| 파일 업로드 최대 크기 | 50MB (서버), 영상 20MB | 운영중 |
| streaming_placeholder stale | 5분 초과 시 interrupted로 자동 정리 | 운영중 |

---

## 핵심 컴포넌트 경로

### 프론트엔드 (aads-dashboard)

| 파일 | 역할 |
|------|------|
| `src/app/chat/page.tsx` | 채팅 메인 페이지 (전체 상태 관리) |
| `src/app/chat/layout.tsx` | 채팅 레이아웃 |
| `src/app/chat/ChatSidebar.tsx` | 워크스페이스/세션 목록, 태그 필터 |
| `src/app/chat/ChatArtifactPanel.tsx` | 아티팩트 패널 |
| `src/app/chat/ChatInput.tsx` | 메시지 입력창 |
| `src/app/chat/types.ts` | 채팅 타입 정의 |
| `src/app/chat/api.ts` | 채팅 API 클라이언트 |
| `src/app/chat/MarkdownRenderer.tsx` | 마크다운 렌더러 |
| `src/components/chat/ChatBubble.tsx` | 메시지 버블 |
| `src/components/chat/CitationCard.tsx` | 인용 카드 |
| `src/components/chat/SourceCard.tsx` | 참고자료 카드 |
| `src/components/chat/FactCheckCard.tsx` | 팩트체크 카드 |
| `src/components/chat/MemoryContextBar.tsx` | 메모리 컨텍스트 바 |
| `src/components/chat/ThinkingIndicator.tsx` | Thinking 표시 |
| `src/components/chat/SlashCommandMenu.tsx` | 슬래시 커맨드 |
| `src/components/chat/ActionChips.tsx` | 빠른 명령 칩 |
| `src/components/chat/AIDrive.tsx` | AI Drive 파일 관리 |
| `src/hooks/useChatSSE.ts` | SSE 스트림 훅 |
| `src/hooks/useChatSession.ts` | 세션 상태 훅 |
| `src/services/chatApi.ts` | 채팅 API 서비스 |
| `src/services/errorReporter.ts` | 에러 리포팅 (6종 자동 수집) |
| `src/services/streamManager.ts` | 멀티세션 스트림 관리 |

### 백엔드 (aads-server)

| 파일 | 역할 |
|------|------|
| `app/routers/chat.py` | 메인 채팅 라우터 (1118줄, ~50개 엔드포인트) |
| `app/api/chat.py` | 레거시 인텐트 라우터 (키워드 기반) |
| `app/services/chat_service.py` | 채팅 비즈니스 로직 (3969줄) |
| `app/services/model_selector.py` | LLM 라우팅, 도구 루프, tool_choice |
| `app/services/intent_router.py` | 65개 인텐트 분류 |
| `app/services/tool_executor.py` | 도구 실행 핸들러 |
| `app/services/tool_registry.py` | 도구 스키마 정의 |
| `app/services/context_builder.py` | 3계층 컨텍스트 빌더 |
| `app/services/context_compressor.py` | 도구 출력 압축 + Observation Masking |
| `app/services/compaction_service.py` | 구조화 요약 (6섹션) |
| `app/services/output_validator.py` | 빈 약속 응답 검증기 |
| `app/services/subagent_service.py` | 서브에이전트 서비스 |
| `app/services/agent_sdk_service.py` | Claude Agent SDK 통합 |
| `app/api/ceo_chat_tools.py` | 원격 쓰기/실행 도구 9개 |
| `app/core/prompts/system_prompt_v2.py` | 시스템 프롬프트 v2 |
| `app/core/anthropic_client.py` | Anthropic 중앙 클라이언트 |
| `app/core/memory_recall.py` | 메모리 자동 주입 |
| `app/core/interrupt_queue.py` | 인터럽트 큐 |
