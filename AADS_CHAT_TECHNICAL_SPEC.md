# AADS 채팅 시스템 — 기능 분석 및 기술 명세서

> **버전**: v1.1 | **작성일**: 2026-03-11 | **최종 수정**: 2026-03-11
> **시스템**: aads.newtalk.kr/chat

---

## 1. 시스템 아키텍처

```
┌─────────────────┐    ┌──────────────────┐    ┌────────────────┐
│   Next.js 16.1  │───▶│   FastAPI 8100   │───▶│  PostgreSQL    │
│   (port 3100)   │SSE │   (uvicorn)      │    │  + pgvector    │
│   aads-dashboard│◀───│   aads-server    │    │  (port 5433)   │
└─────────────────┘    └───────┬──────────┘    └────────────────┘
                               │
                    ┌──────────┼──────────┐
                    ▼          ▼          ▼
              ┌──────────┐ ┌───────┐ ┌──────────┐
              │ LiteLLM  │ │ Redis │ │ MCP      │
              │ (4000)   │ │       │ │ Servers  │
              │ 40+ LLMs │ │       │ │ (3종)    │
              └──────────┘ └───────┘ └──────────┘
```

| 구성 요소 | 역할 | 포트 |
|----------|------|------|
| aads-dashboard | Next.js 프론트엔드 (3-패널 레이아웃) | 3100 |
| aads-server | FastAPI 백엔드 (SSE 스트리밍) | 8100 |
| aads-postgres | PostgreSQL + pgvector (벡터 검색) | 5433 |
| aads-redis | 세션 캐시, 큐 | 6379 |
| aads-litellm | LLM 게이트웨이 (40+ 모델 라우팅) | 4000 |
| MCP Servers | Filesystem, Git, Memory (3종) | IPC |

---

## 2. 프론트엔드 구조

### 2.1 메인 페이지

**파일**: `/root/aads/aads-dashboard/src/app/chat/page.tsx` (94.7KB)

#### 3-패널 레이아웃
```
┌──────────┬──────────────────────┬──────────────┐
│ 사이드바  │      채팅 영역        │  아티팩트    │
│ (280px)  │    (flex-1)          │  패널(420px) │
│          │                      │              │
│ 워크스페이스│  메시지 스트림        │  코드/차트/  │
│ 세션 목록 │  입력 영역            │  보고서/표   │
│ 검색     │  모델 선택기           │              │
└──────────┴──────────────────────┴──────────────┘
```

#### 테마
- 다크 모드 (`#0f0f23` 배경) / 라이트 모드 (`#f8f9fa` 배경)
- localStorage 저장, CSS 변수 기반 전환

### 2.2 컴포넌트 목록 (23개)

**경로**: `/root/aads/aads-dashboard/src/components/chat/`

| 컴포넌트 | 역할 |
|---------|------|
| `ChatInput.tsx` | 텍스트 입력, 파일 첨부(📎), 음성 입력(🎤), 모델 선택 |
| `ChatBubble.tsx` | 메시지 렌더링 (마크다운, 코드 하이라이트, 이미지, 링크) |
| `ModelSelector.tsx` | 40+ LLM 모델 드롭다운 (5개 기본 + 확장) |
| `ActionChips.tsx` | 빠른 액션 칩 (웰컴/동적) |
| `AIDrive.tsx` | 파일 드라이브 (업로드/다운로드/삭제) |
| `ArtifactChart.tsx` | 차트 아티팩트 렌더링 (Chart.js) |
| `ArtifactPanel.tsx` | 아티팩트 패널 (Full/Mini/Hidden, Ctrl+]) |
| `BriefingCard.tsx` | 접속 시 자동 브리핑 카드 (접기/펼치기) |
| `DiffEditor.tsx` | Monaco DiffEditor (코드 변경사항 비교) |
| `ResearchPanel.tsx` | 딥리서치 진행률 + 보고서 표시 |
| `ThinkingIndicator.tsx` | AI 사고 과정 표시 (Extended Thinking) |
| `ToolStatus.tsx` | 도구 실행 상태 표시 (이름, 결과 미리보기) |
| `StreamManager.tsx` | 멀티세션 SSE 스트림 관리 |
| `MessageQueue.tsx` | 스트리밍 중 메시지 큐잉/취소 |
| `SessionCost.tsx` | 세션별 누적 비용 표시 |
| `YellowLimitDialog.tsx` | 도구 사용 경고 다이얼로그 (Yellow 등급) |
| `ErrorReporter.tsx` | 프론트엔드 에러 자동 보고 |
| `SearchOverlay.tsx` | 메시지 전문 검색 |
| `BookmarkList.tsx` | 북마크된 메시지 목록 |
| `SessionList.tsx` | 세션 목록 (고정/이름변경/삭제) |
| `WorkspaceHub.tsx` | 워크스페이스 선택 허브 (7개) |
| `VoiceInput.tsx` | Web Speech API 음성 인식 |
| `CodePanel.tsx` | 코드 블록 복사/실행 패널 |

### 2.3 모델 선택기

**기본 모델 5개**:

| 모델명 | 용도 | 비용 |
|--------|------|------|
| Auto (기본) | 인텐트 기반 자동 라우팅 | — |
| Claude Sonnet 4.6 | 범용 (코드/분석/대화) | $3/$15 |
| Claude Opus 4.6 | 전략/아키텍처/Extended Thinking | $15/$75 |
| Gemini 3.1 Flash | 검색/grounding/저비용 | ~$0.1 |
| Deep Research | Gemini Pro 심층 리서치 | $2~5 |

**확장 모델 (40+)**: GPT-4o, GPT-5, o1, o3-mini, Gemini 2.5 Pro/Flash, Gemma 3, Claude Haiku 등

### 2.4 주요 UI 기능

| 기능 | 설명 |
|------|------|
| **SSE 스트리밍** | 실시간 토큰 단위 응답 표시 + heartbeat(8초) |
| **메시지 큐잉** | 스트리밍 중 추가 메시지 대기열 → 순차 전송 |
| **세션 관리** | 생성/이름변경/삭제/고정 + 자동 생성 |
| **워크스페이스** | 7개 허브 (CEO, AADS, KIS, GO100, SF, NTV2, NAS) |
| **파일 첨부** | 📎 클릭/드래그앤드롭 → Drive 업로드 → LLM 주입 |
| **음성 입력** | 🎤 Web Speech API (한국어) |
| **북마크** | 메시지별 즐겨찾기 |
| **검색** | 메시지 전문 검색 (워크스페이스별 필터) |
| **아티팩트** | 코드/차트/보고서/표 패널 (3단 토글) |
| **Diff 승인** | Monaco DiffEditor로 코드 변경사항 승인/거부 |
| **자동 브리핑** | 접속 시 미완료 지시서/알림/에러/서버상태 자동 표시 |
| **Extended Thinking** | AI 사고 과정 접기/펼치기 표시 |
| **다크/라이트 테마** | 원클릭 전환, localStorage 저장 |
| **메시지 수정 재전송** | ✏️ 인라인 편집 → 기존 메시지+AI응답 삭제 → 수정 내용 재전송 (방식A) |
| **메시지 재지시** | 🔄 입력창에 복사 → 수정 후 새 메시지로 전송, 기존 이력 유지 (방식B) |
| **파일 임시 컨텍스트** | 첨부파일 전문을 현재 턴에만 주입(Layer D), 다음 턴 자동 제거 |

---

## 3. 백엔드 API 엔드포인트

### 3.1 REST API (26개)

**라우터**: `/root/aads/aads-server/app/routers/chat.py`

#### 워크스페이스
| Method | 경로 | 설명 |
|--------|------|------|
| GET | `/chat/workspaces` | 워크스페이스 목록 |
| POST | `/chat/workspaces` | 워크스페이스 생성 |
| PUT | `/chat/workspaces/{id}` | 워크스페이스 수정 |
| DELETE | `/chat/workspaces/{id}` | 워크스페이스 삭제 |

#### 세션
| Method | 경로 | 설명 |
|--------|------|------|
| GET | `/chat/sessions?workspace_id=` | 세션 목록 |
| POST | `/chat/sessions` | 세션 생성 |
| PUT | `/chat/sessions/{id}` | 세션 수정 (제목, 고정) |
| DELETE | `/chat/sessions/{id}` | 세션 삭제 |

#### 메시지
| Method | 경로 | 설명 |
|--------|------|------|
| GET | `/chat/messages?session_id=` | 메시지 조회 (limit, offset) |
| **POST** | **`/chat/messages/send`** | **메시지 전송 (SSE 스트리밍 응답)** |
| PUT | `/chat/messages/{id}` | 사용자 메시지 내용 수정 (방식A: 수정재전송) |
| DELETE | `/chat/messages/{id}` | 메시지 + 다음 AI 응답 삭제 (방식A: 수정재전송) |
| PUT | `/chat/messages/{id}/bookmark` | 북마크 토글 |
| GET | `/chat/messages/search?q=` | 메시지 검색 |

#### 아티팩트
| Method | 경로 | 설명 |
|--------|------|------|
| GET | `/chat/artifacts?session_id=` | 아티팩트 목록 |
| POST | `/chat/artifacts` | 아티팩트 생성 |
| PUT | `/chat/artifacts/{id}` | 아티팩트 수정 |
| DELETE | `/chat/artifacts/{id}` | 아티팩트 삭제 |
| POST | `/chat/artifacts/{id}/export` | 내보내기 (PDF/MD/HTML) |

#### 드라이브
| Method | 경로 | 설명 |
|--------|------|------|
| GET | `/chat/drive?workspace_id=` | 파일 목록 |
| POST | `/chat/drive/upload` | 파일 업로드 (multipart) |
| GET | `/chat/drive/{id}/download` | 파일 다운로드 |
| DELETE | `/chat/drive/{id}` | 파일 삭제 |

#### 기타
| Method | 경로 | 설명 |
|--------|------|------|
| GET | `/chat/research?session_id=` | 리서치 이력 |
| POST | `/chat/approve-diff` | Diff 승인/거부 |
| POST | `/chat/errors/report` | 프론트엔드 에러 보고 |

### 3.2 SSE 이벤트 타입 (15종)

```
data: {"type": "delta", "content": "안녕"}           ← 텍스트 토큰
data: {"type": "thinking", "thinking": "..."}        ← Extended Thinking
data: {"type": "tool_use", "tool_name": "..."}       ← 도구 호출 시작
data: {"type": "tool_result", "tool_name": "...", "content": "..."} ← 도구 결과
data: {"type": "sources", "sources": [...]}          ← 출처/인용
data: {"type": "research_start", "message": "..."}   ← 딥리서치 시작
data: {"type": "research_progress", "phase": "..."}  ← 딥리서치 진행
data: {"type": "research_complete", ...}             ← 딥리서치 완료
data: {"type": "done", "model": "...", "cost": "...", "intent": "..."} ← 완료
data: {"type": "error", "content": "..."}            ← 오류
data: {"type": "heartbeat"}                          ← 연결 유지 (8초)
data: {"type": "sdk_session", "session_id": "..."}   ← Agent SDK 세션
data: {"type": "sdk_complete"}                       ← Agent SDK 완료
data: {"type": "yellow_limit", ...}                  ← 도구 등급 경고
data: {"type": "diff_preview", ...}                  ← 코드 변경 미리보기
```

---

## 4. 핵심 서비스

### 4.1 메시지 처리 파이프라인

**파일**: `/root/aads/aads-server/app/services/chat_service.py` (~1,290 lines)

```
사용자 메시지 수신
  ↓
1. 세션 검증 + 사용자 메시지 DB 저장
  ↓
2. 첨부파일 처리 (Ephemeral Document Context)
   ├─ 파일 추출: PDF(pdfplumber), Excel(openpyxl), 텍스트(200KB)
   ├─ 30K 토큰 이하 → 전문 삽입, 초과 → 앞뒤 분할(6K+6K)
   ├─ Layer D 생성 (현재 턴에만 주입, 다음 턴 자동 제거)
   └─ 참조 요약 1줄을 DB content에 추가 (히스토리 오염 방지)
  ↓
3. 재참조 감지 ("아까 그 파일", "위 문서" 등 7패턴)
   └─ 이전 첨부파일 DB 조회 → Layer D 재구성
  ↓
4. 대화 히스토리 로드 (최근 200건, ASC)
  ↓
5. 4-레이어 컨텍스트 빌드 (context_builder)
   ├─ Layer 1: 시스템 프롬프트 (정적, ~1400 토큰)
   ├─ Layer 2: 동적 상태 (시간, 세션 비용, 도구 가이드, ~300 토큰)
   ├─ Layer 2+: 메모리 주입 (memory_recall, 5섹션, ~2000 토큰)
   └─ Layer D: 임시 문서 컨텍스트 (첨부파일 전문, 현재 턴 only)
  ↓
6. 시맨틱 코드 검색 컨텍스트 주입 (옵션)
  ↓
7. 자동 압축 (80K 토큰 초과 시 compaction_service)
  ↓
8. 인텐트 분류 (intent_router → Gemini Flash-Lite, ~200ms)
  ↓
9. 모델/도구 라우팅
   ├─ Gemini Direct → grounding / deep_research
   ├─ Agent SDK → execute / code_modify
   ├─ AutonomousExecutor → cto_*, pipeline_c (max 25 iterations)
   └─ Standard → model_selector.call_stream()
  ↓
10. LLM 스트리밍 (도구 루프 포함)
  ↓
11. Output Validator (빈 약속 응답 재시도)
  ↓
12. 응답 DB 저장 + 비용 누적
  ↓
13. 20턴마다 세션 노트/관찰 자동 저장
```

### 4.2 인텐트 분류

**파일**: `/root/aads/aads-server/app/services/intent_router.py`

#### 도구 불필요 인텐트
| 인텐트 | 모델 | 설명 |
|--------|------|------|
| casual | gemini-flash-lite | 일상 대화 |
| greeting | gemini-flash-lite | 인사 |
| strategy | claude-opus (thinking) | 전략/방향성 |
| planning | claude-sonnet | 기획 |
| decision | claude-sonnet | 의사결정 |
| design / design_fix | claude-sonnet | 디자인 |
| deep_research | gemini-pro | Gemini 심층 리서치 |
| cto_strategy | claude-opus (thinking) | CTO 전략 |
| image_analyze | claude-sonnet | 이미지 분석 |
| video_analyze | gemini-3-flash-preview | 영상 분석 |

#### 도구 사용 인텐트 (전부 group="all")
| 인텐트 | 모델 | 설명 |
|--------|------|------|
| system_status | claude-sonnet | 시스템 상태 |
| health_check | claude-sonnet | 헬스체크 |
| dashboard | claude-sonnet | 대시보드 현황 |
| diagnosis | claude-sonnet | 종합 진단 |
| search | gemini-3-flash (grounding) | 웹 검색 |
| directive / directive_gen | claude-opus (thinking) | 지시서 |
| execute | claude-opus | 실행/배포 |
| code_modify | claude-opus | 코드 수정 |
| **pipeline_c** | **claude-sonnet** | **자율 작업 파이프라인** |
| cto_code_analysis | claude-opus (thinking) | 코드 분석 |
| cto_verify | claude-opus (thinking) | 작업 검증 |
| cto_impact | claude-opus (thinking) | 영향 분석 |
| service_inspection | claude-sonnet | 서비스 점검 |
| task_query | claude-sonnet | 작업 현황 조회 |
| status_check | claude-sonnet | 상태 확인 |
| news_search ~ knowledge_search | gemini-3-flash | Naver 특화 검색 (8종) |

#### 분류 방법
1. **Primary**: Gemini 2.5 Flash-Lite LLM 분류 (~200ms)
2. **Fallback**: 키워드 기반 규칙 매칭
3. **CEO 명령형 인식**: "확인하고 보고하라", "~해줘" 패턴 → status_check/execute

### 4.3 도구 시스템

**레지스트리**: `/root/aads/aads-server/app/services/tool_registry.py`
**실행기**: `/root/aads/aads-server/app/services/tool_executor.py`

#### 전체 도구 목록 (49개)

**Tier 1 — 상시 로드 (즉시, 무료)**
| 도구 | 설명 |
|------|------|
| `health_check` | 서버 68/211/114 헬스체크 |
| `get_all_service_status` | 6개 서비스 병렬 상태 조회 |
| `check_directive_status` | 지시사항 진행 종합 확인 |
| `read_remote_file` | 원격 서버 소스 코드/설정 읽기 |
| `list_remote_dir` | 원격 디렉토리 탐색/검색 |
| `query_database` | AADS PostgreSQL SELECT |
| `query_project_database` | 프로젝트별 원격 DB SELECT (KIS/GO100/SF/NTV2) |
| `task_history` | 작업 이력 |
| `dashboard_query` | 파이프라인 현황 |
| `server_status` | Docker 컨테이너 상태 |
| `read_github_file` | GitHub 문서 읽기 |
| `pipeline_c_start` | Pipeline C 시작 |
| `pipeline_c_status` | Pipeline C 상태 확인 |
| `pipeline_c_approve` | Pipeline C 승인/거부 |

**Tier 2 — 분석 (온디맨드, 3~15초)**
| 도구 | 설명 |
|------|------|
| `code_explorer` | 함수 호출 체인 추적 (depth 3) |
| `semantic_code_search` | 벡터 기반 코드 검색 |
| `analyze_changes` | Git 변경 + 위험도 분석 |
| `inspect_service` | 서비스 종합 점검 |

**Tier 3 — 액션 (즉시 실행)**
| 도구 | 설명 |
|------|------|
| `directive_create` | 지시서 생성 |
| `generate_directive` | 지시서 자동 생성 |
| `delegate_to_agent` | 복잡 작업 위임 → AutonomousExecutor 실제 실행 + directive_lifecycle DB 등록 + 채팅방 결과 보고 |
| `delegate_to_research` | 심층 리서치 위임 |
| `spawn_subagent` | 서브에이전트 생성 |
| `spawn_parallel_subagents` | 병렬 서브에이전트 |
| `save_note` / `recall_notes` | 메모리 저장/조회 |
| `learn_pattern` | 패턴 학습 |
| `observe` | 관찰 기록 |
| `cost_report` | 비용 분석 |
| `export_data` | 데이터 내보내기 (Excel/CSV/PDF) |
| `schedule_task` / `unschedule_task` / `list_scheduled_tasks` | 동적 스케줄러 |

**Tier 4 — 외부 검색 (API 비용)**
| 도구 | 설명 |
|------|------|
| `web_search_brave` / `web_search` | 통합 웹 검색 (Google/Naver/Kakao 폴백) |
| `jina_read` | URL 페이지 텍스트 추출 |
| `crawl4ai_fetch` | 크롤링 |

**Tier 5 — 고비용 (CEO 명시 요청)**
| 도구 | 설명 |
|------|------|
| `deep_research` | Gemini Deep Research ($2~5, 3~10분) |
| `deep_crawl` | 다수 URL 동시 크롤링 |
| `search_all_projects` | 6개 프로젝트 동시 검색 |

**Tier 6 — 브라우저 (온디맨드)**
| 도구 | 설명 |
|------|------|
| `browser_navigate` | URL 이동 |
| `browser_snapshot` | 접근성 트리 추출 |
| `browser_screenshot` | PNG 스크린샷 |
| `browser_click` / `browser_fill` | 요소 클릭/입력 |
| `browser_tab_list` | 탭 목록 |

#### 도구 등급 (Green/Yellow/Red)
- **Green**: 읽기 전용, 항상 허용 (read_remote_file, query_database 등)
- **Yellow**: 쓰기/부작용, 연속 사용 제한 + CEO 확인 (write_remote_file, run_remote_command 등)
- **Red**: 항상 차단 (directive_create 직접 호출, submit_directive)

### 4.4 메모리 시스템

**파일**: `/root/aads/aads-server/app/core/memory_recall.py`

#### 5-섹션 자동 주입 (~2,000 토큰)

| 섹션 | 소스 테이블 | 예산 | 설명 |
|------|-----------|------|------|
| 최근 세션 노트 | `session_notes` | ~500 토큰 | 최근 3개 세션 요약 |
| CEO 선호사항 | `ai_observations` (preference) | ~300 토큰 | 도구/커뮤니케이션 선호 |
| 도구 전략 | `ai_observations` (tool_strategy) | ~400 토큰 | 도구 사용 패턴/최적화 |
| 활성 지시사항 | `directive_lifecycle` | ~400 토큰 | 진행 중인 지시서 |
| 발견사항 | `ai_observations` (discovery) | ~400 토큰 | 학습된 인사이트 |

#### 자동 저장
- **20턴마다**: 세션 노트 자동 저장 (`_auto_save_session_note`)
- **20턴마다**: 관찰 자동 저장 (`_auto_observe_session`)
- **Compaction 시**: 7-섹션 구조화 요약 → `session_notes` + `ai_observations` 동기화

### 4.5 컨텍스트 빌더

**파일**: `/root/aads/aads-server/app/services/context_builder.py`

```
Layer 0: Checkpointer (LangGraph 상태 — 현재 미사용)
Layer 1: 시스템 프롬프트 (정적, 워크스페이스별, ~1,400 토큰)
Layer 2: 동적 상태 (현재 시간, 세션 비용, 도구 가이드, ~300 토큰)
Layer 2+: 메모리 주입 (memory_recall 5섹션, ~2,000 토큰)
Layer 3: 대화 히스토리 (Observation Masking 적용)
  ├─ 최근 20턴: 도구 결과 유지
  └─ 이전 턴: 도구 결과 플레이스홀더 교체
Layer D: 임시 문서 컨텍스트 (현재 턴에만 주입, 다음 턴 자동 제거)
  ├─ 파일 첨부 시: <ephemeral_document_context> XML로 전문 삽입
  ├─ 30K 토큰 이하: 전문 삽입, 초과: 앞뒤 분할 (CHUNK_MAX=6K)
  ├─ 재참조 감지: "아까 그 파일" 등 → 이전 첨부 DB 조회 → 재주입
  └─ DB에는 1줄 참조 요약만 저장 (히스토리 오염 방지)
```

**환경변수**:
- `DOCUMENT_FULL_INSERT_MAX_TOKENS=30000` — 전문 삽입 최대 토큰
- `DOCUMENT_CHUNK_MAX_TOKENS=6000` — 분할 시 청크 크기

### 4.6 압축 (Compaction)

**파일**: `/root/aads/aads-server/app/services/compaction_service.py`

| 설정 | 값 | 환경변수 |
|------|-----|---------|
| 트리거 | 80,000 토큰 | `COMPACTION_TRIGGER_TOKENS` |
| 유지 턴 수 | 최근 20턴 | `COMPACTION_KEEP_RECENT` |
| 요약 모델 | Claude Haiku | — |
| Cascade 임계값 | 12,000자 | — |

#### 7-섹션 구조화 요약 템플릿
1. 현재 목표 및 진행 상태
2. CEO 주요 지시사항
3. 에이전트 위임 현황
4. 수정된 파일/서비스 목록
5. 발견된 문제/해결된 사항
6. 중요한 기술적 결정
7. 다음 단계 및 미완료 항목

### 4.7 Pipeline C (자율 작업 파이프라인)

**파일**: `/root/aads/aads-server/app/services/pipeline_c.py`

```
Phase 1: Claude Code SSH 자율 작업 (30분 타임아웃)
  ↓  → 채팅방: 🔧 시작 보고
Phase 2: AI 자동 검수 (Claude Sonnet)
  ↓  → 채팅방: ✅ 통과 / 🔄 실패+재지시
Phase 3: 수정 반복 루프 (최대 3회)
  ↓  → 채팅방: 매 사이클 기록
Phase 4: CEO 승인 대기 (awaiting_approval)
  ↓  → 채팅방: 🔔 승인 요청 (diff 포함)
Phase 5: 배포 (git commit + push + 서비스 재시작)
  ↓  → 채팅방: 🚀 배포 진행
Phase 6: Health 검증
  ↓
Phase 7: 완료 (done) → 채팅방: ✅ 최종 보고
```

**지원 프로젝트**: KIS, GO100, SF, NTV2, AADS (크로스 프로젝트: 어느 세션에서든 모든 프로젝트 작업 가능)

**채팅방 연동**: `_post_to_chat()` 헬퍼로 매 Phase 전환 시 `chat_messages`에 직접 삽입. `contextvars`로 session_id 전달.

**Watchdog**: 2분 주기 스톨 감지 (10분 이상 같은 phase 유지 시 ⚠️ 채팅방 경고)

**AADS 자기수정 안전장치**: 재시작 전 `phase='restarting'` DB 선저장 → 서버 재시작 후 `recover_interrupted_jobs()`가 자동 복구 → 채팅방 기록

**AADS 실행 경로**: Docker 컨테이너에서 `host.docker.internal`로 SSH → 호스트의 Claude CLI 사용 (컨테이너 내 미설치 해결)

### 4.8 Agent SDK 서비스

**파일**: `/root/aads/aads-server/app/services/agent_sdk_service.py`

- **인텐트**: `execute`, `code_modify`
- **모델**: Claude Opus 4.6
- **기능**: 코드 수정, Bash 실행, git 작업 자율 수행
- **안전장치**: 위험 명령 자동 차단 + 세션 resume 지원
- **Fallback**: SDK 실패 시 AutonomousExecutor로 자동 전환

### 4.9 Output Validator

**파일**: `/root/aads/aads-server/app/services/output_validator.py`

- 빈 약속 응답 감지 ("~하겠습니다", "~할게요" 로 끝나면서 실제 내용 없는 경우)
- 위반 시 retry_prompt 생성 → 자동 재시도

---

## 5. 데이터 모델

### 5.1 Pydantic 모델

**파일**: `/root/aads/aads-server/app/models/chat.py`

| 모델 | 주요 필드 |
|------|----------|
| `WorkspaceCreate` | name, slug, description, icon, color |
| `WorkspaceOut` | id, name, slug, system_prompt, settings, files, color, icon |
| `SessionCreate` | workspace_id, title |
| `SessionOut` | id, workspace_id, title, summary, message_count, cost_total, pinned |
| `MessageSendRequest` | session_id, content, attachments, model_override |
| `MessageUpdateRequest` | content (수정 내용) |
| `MessageOut` | id, session_id, role, content, model_used, intent, cost, tokens_in/out, bookmarked, attachments, sources, tools_called, thinking_summary, **edited_at** |
| `ArtifactOut` | id, session_id, type, title, content, metadata |
| `DriveFileOut` | id, workspace_id, filename, file_path, file_type, file_size |

### 5.2 DB 스키마

#### `chat_workspaces`
| 컬럼 | 타입 | 설명 |
|------|------|------|
| id | UUID (PK) | 워크스페이스 ID |
| name | VARCHAR | 이름 |
| slug | VARCHAR (UNIQUE) | URL slug |
| description | TEXT | 설명 |
| system_prompt | TEXT | 워크스페이스별 시스템 프롬프트 |
| settings | JSONB | 설정 |
| files | JSONB | 첨부 파일 참조 |
| icon | VARCHAR | 아이콘 |
| color | VARCHAR | 테마 색상 |
| created_at / updated_at | TIMESTAMPTZ | 타임스탬프 |

#### `chat_sessions`
| 컬럼 | 타입 | 설명 |
|------|------|------|
| id | UUID (PK) | 세션 ID |
| workspace_id | UUID (FK) | 워크스페이스 |
| title | VARCHAR | 세션 제목 |
| summary | TEXT | 요약 |
| message_count | INTEGER | 메시지 수 |
| cost_total | NUMERIC | 누적 비용 ($) |
| pinned | BOOLEAN | 고정 여부 |
| settings | JSONB | 세션 설정 (sdk_session_id 등) |
| created_at / updated_at | TIMESTAMPTZ | 타임스탬프 |

#### `chat_messages`
| 컬럼 | 타입 | 설명 |
|------|------|------|
| id | UUID (PK) | 메시지 ID |
| session_id | UUID (FK) | 세션 |
| role | VARCHAR | user / assistant |
| content | TEXT | 메시지 본문 |
| model_used | VARCHAR | 사용 모델 |
| intent | VARCHAR | 인텐트 |
| cost | NUMERIC | 비용 ($) |
| tokens_in / tokens_out | INTEGER | 토큰 수 |
| bookmarked | BOOLEAN | 북마크 |
| attachments | JSONB | 첨부파일 목록 |
| sources | JSONB | 출처/인용 |
| tools_called | JSONB | 호출된 도구 목록 |
| thinking_summary | TEXT | Extended Thinking 요약 |
| is_compacted | BOOLEAN | 압축 여부 |
| edited_at | TIMESTAMPTZ | 수정 시간 (NULL=미수정) |
| created_at | TIMESTAMPTZ | 생성 시간 |

#### `chat_artifacts`
| 컬럼 | 타입 | 설명 |
|------|------|------|
| id | UUID (PK) | 아티팩트 ID |
| session_id | UUID (FK) | 세션 |
| type | VARCHAR | report / code / chart / dashboard / table |
| title | VARCHAR | 제목 |
| content | TEXT | 본문 |
| metadata | JSONB | 메타데이터 |
| created_at / updated_at | TIMESTAMPTZ | 타임스탬프 |

#### `chat_drive_files`
| 컬럼 | 타입 | 설명 |
|------|------|------|
| id | UUID (PK) | 파일 ID |
| workspace_id | UUID (FK) | 워크스페이스 |
| filename | VARCHAR | 원본 파일명 |
| file_path | VARCHAR | 서버 저장 경로 |
| file_type | VARCHAR | 확장자 |
| file_size | INTEGER | 파일 크기 (bytes) |
| uploaded_by | VARCHAR | 업로드 주체 |
| metadata | JSONB | 메타데이터 |
| created_at | TIMESTAMPTZ | 생성 시간 |

---

## 6. 메모리 관련 DB 테이블

#### `session_notes`
| 컬럼 | 설명 |
|------|------|
| id | PK |
| session_id | 세션 |
| summary | 요약 텍스트 |
| projects_discussed | TEXT[] — 논의된 프로젝트 배열 |
| created_at | 생성 시간 |

#### `ai_observations`
| 컬럼 | 설명 |
|------|------|
| key | UNIQUE — 관찰 키 |
| category | preference / tool_strategy / discovery |
| content | 관찰 내용 |
| project | 프로젝트 (NULL=글로벌) |
| confidence | 신뢰도 (0.0~1.0) |
| created_at / updated_at | 타임스탬프 |

#### `directive_lifecycle`
| 컬럼 | 설명 |
|------|------|
| id | PK |
| title | 지시서 제목 |
| status | pending / running / queued / done / cancelled |
| project | 프로젝트 |
| assignee | 담당자 |
| created_at / updated_at | 타임스탬프 |

#### `pipeline_jobs`
| 컬럼 | 설명 |
|------|------|
| job_id | PK (pc-{timestamp}-{uuid}) |
| chat_session_id | 채팅 세션 |
| project | 프로젝트 |
| instruction | 작업 지시 |
| claude_session_id | Claude Code 세션 |
| phase | 현재 단계 |
| cycle | 검수 횟수 |
| max_cycles | 최대 검수 횟수 |
| status | queued / running / awaiting_approval / deploying / done / failed |
| logs | JSONB — 실행 로그 |
| result_output | 실행 결과 |
| git_diff | 변경사항 diff |
| review_feedback | 검수 피드백 |
| updated_at | 타임스탬프 |

---

## 7. 환경 변수

| 변수 | 기본값 | 설명 |
|------|--------|------|
| `COMPACTION_TRIGGER_TOKENS` | 80000 | 압축 트리거 토큰 수 |
| `COMPACTION_KEEP_RECENT` | 20 | 압축 시 유지할 최근 턴 수 |
| `OBSERVATION_WINDOW_SIZE` | 20 | 도구 결과 유지 관찰 윈도우 |
| `EXTENDED_THINKING_ENABLED` | true | Extended Thinking 활성화 |
| `AGENT_SDK_ENABLED` | true | Agent SDK 활성화 |
| `AGENT_SDK_MAX_TURNS` | 0 (무제한) | SDK 최대 턴 |
| `AGENT_SDK_MAX_BUDGET_USD` | 0 (무제한) | SDK 최대 비용 |
| `AGENT_SDK_CWD` | /root/aads | SDK 작업 디렉토리 |
| `LITELLM_BASE_URL` | http://litellm:4000 | LLM 게이트웨이 |
| `CHAT_UPLOAD_DIR` | /root/aads/uploads/chat | 첨부파일 저장 경로 |
| `DATABASE_URL` | — | PostgreSQL 연결 URL |
| `MCP_ENABLED` | true | MCP 서버 활성화 |
| `DOCUMENT_FULL_INSERT_MAX_TOKENS` | 30000 | Layer D 전문 삽입 최대 토큰 |
| `DOCUMENT_CHUNK_MAX_TOKENS` | 6000 | Layer D 분할 시 청크 크기 |
| `MAX_THINKING_TOKENS` | 8000 | Extended Thinking 최대 토큰 |
| `CONFIDENCE_CEO_PREF` | 0.2 | CEO 선호 관찰 신뢰도 임계값 |
| `CONFIDENCE_TOOL_STRATEGY` | 0.3 | 도구 전략 관찰 신뢰도 임계값 |
| `CONFIDENCE_DISCOVERY` | 0.4 | 발견사항 관찰 신뢰도 임계값 |
| `LANGFUSE_ENABLED` | true | Langfuse 트레이싱 활성화 |

---

## 8. 파일 시스템 경로

### 프론트엔드
```
/root/aads/aads-dashboard/
├── src/app/chat/page.tsx              ← 메인 페이지 (94.7KB)
├── src/components/chat/               ← 23개 컴포넌트
│   ├── ChatInput.tsx                  ← 입력 (파일/음성/모델)
│   ├── ChatBubble.tsx                 ← 메시지 렌더링
│   ├── ModelSelector.tsx              ← 40+ 모델 선택
│   ├── ActionChips.tsx                ← 빠른 액션
│   ├── ArtifactPanel.tsx              ← 아티팩트 패널
│   ├── BriefingCard.tsx               ← 자동 브리핑
│   ├── DiffEditor.tsx                 ← 코드 diff
│   └── ...
├── src/services/chatApi.ts            ← 채팅 API 클라이언트
└── src/services/driveApi.ts           ← 드라이브 API
```

### 백엔드
```
/root/aads/aads-server/
├── app/routers/chat.py                ← 26개 REST 엔드포인트
├── app/models/chat.py                 ← Pydantic 데이터 모델
├── app/services/
│   ├── chat_service.py                ← 핵심 비즈니스 로직 (~1,290줄)
│   ├── intent_router.py               ← 57종 인텐트 분류 (동적 라우팅)
│   ├── model_selector.py              ← LLM 모델 라우팅
│   ├── context_builder.py             ← 3-레이어 컨텍스트
│   ├── compaction_service.py          ← 토큰 압축
│   ├── tool_registry.py               ← 49개 도구 등록
│   ├── tool_executor.py               ← 도구 실행/디스패치
│   ├── autonomous_executor.py         ← 자율 실행 엔진
│   ├── agent_sdk_service.py           ← Agent SDK 통합
│   ├── output_validator.py            ← 출력 검증
│   └── pipeline_c.py                  ← Pipeline C 워크플로우
├── app/core/
│   ├── memory_recall.py               ← 5-섹션 메모리 시스템
│   ├── document_context.py            ← Ephemeral Document Context (Layer D)
│   └── project_config.py              ← 프로젝트별 서버/경로 매핑
└── app/api/
    ├── ceo_chat_tools.py              ← 도구 구현 (브라우저, Pipeline C)
    ├── ceo_chat_tools_db.py           ← 프로젝트 DB 쿼리
    ├── ceo_chat_tools_export.py       ← 데이터 내보내기
    └── ceo_chat_tools_scheduler.py    ← 동적 스케줄러
```

### 데이터
```
/root/aads/uploads/chat/               ← 첨부파일 저장 (볼륨 마운트)
/var/www/aads_exports/                  ← 내보내기 파일 (Excel/CSV/PDF)
```

---

## 9. 보안

| 항목 | 구현 |
|------|------|
| 인증 | JWT Bearer Token (FastAPI Depends) |
| API 키 | .env 전용, docker-compose에서 `${VAR:-}` 참조 |
| 도구 등급 | Green/Yellow/Red 3단계 제어 |
| SSH 접근 | 원격 서버 키 기반 인증 |
| SQL Injection | asyncpg 파라미터 바인딩 |
| 파일 업로드 | UUID 프리픽스 + 확장자 검증 |
| 도메인 화이트리스트 | 브라우저 도구 URL 제한 |
| 위험 명령 차단 | Agent SDK 자동 차단 리스트 |

---

## 10. 알려진 제한사항 및 개선 필요 항목

| # | 항목 | 우선순위 |
|---|------|---------|
| 1 | ~~DB 커넥션 풀 없음~~ → asyncpg.create_pool 구현 완료 (min=5, max=20, command_timeout=30s) | DONE |
| 2 | 메시지 카운트 Race Condition (트랜잭션 미래핑) | HIGH |
| 3 | ~~directive_lifecycle 72건 stale~~ → 71건 stuck 정리 완료 (정상 활성만 유지) | DONE |
| 4 | 백그라운드 태스크 메모리 누수 (finally 미정리) | HIGH |
| 5 | ~~memory_recall 직렬~~ → asyncio.gather 병렬 완료 (AADS-CRITICAL-FIX #30) | DONE |
| 6 | ai_observations upsert race condition | MEDIUM |
| 7 | session_notes 중복 방지 없음 | MEDIUM |
| 8 | 이미지/PDF vision 블록 미지원 (텍스트만 주입) | MEDIUM |
| 9 | ~~Pipeline B delegate_to_agent 미실행~~ → AutonomousExecutor 통합 완료 (DB 등록 + 채팅방 보고) | DONE |
| 10 | ~~Pipeline C AADS localhost claude 미설치~~ → host.docker.internal SSH 경유 해결 | DONE |
| 11 | ~~Pipeline C 원격 600초 타임아웃~~ → 1800초(30분) 확장 | DONE |

---

## 11. 변경 이력

| 버전 | 날짜 | 주요 변경 |
|------|------|----------|
| v1.0 | 2026-03-11 | 최초 작성 — 전체 시스템 기술 명세 |
| v1.1 | 2026-03-11 | 메시지 수정/재지시(방식A+B), Ephemeral Document Context(Layer D), Pipeline B 실제실행 통합, Pipeline C 채팅방연동+크로스프로젝트+Watchdog+AADS자기수정안전장치+타임아웃확장, 환경변수 7개 추가 |

---

*이 문서는 AADS 채팅 시스템 v2.1 기준으로 작성되었습니다.*
*문의: aads.newtalk.kr*
