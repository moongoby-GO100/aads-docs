# UI-PROTO-001: AADS Adaptive UI 프로토타입 설계서

> **문서 ID**: UI-PROTO-001
> **유형**: 설계서 (design)
> **작성일**: 2026-03-06 KST
> **Task**: T-095
> **상태**: APPROVED

---

## 1. 개요

AADS 대시보드의 Adaptive UI 레이아웃 및 컴포넌트 설계서. Genspark AI UI를 참고하여 CEO가 직관적으로 멀티 매니저·문서·모델을 운용할 수 있는 통합 인터페이스를 정의한다.

---

## 2. 전체 레이아웃 와이어프레임 (ASCII)

```
┌─────────────────────────────────────────────────────────────────────────┐
│  AADS                                                     CEO 대시보드   │
├──────┬──────────────────────────────────────────────────────────────────┤
│  🏠  │                                                                   │
│  📊  │   ┌─────────────────────────────────────────────────────────┐    │
│  💬  │   │   상단 통계 카드 영역 (4열 그리드)                        │    │
│  📌  │   │   [프로젝트수]  [총 대화]  [에이전트]  [알림]             │    │
│  🌐  │   └─────────────────────────────────────────────────────────┘    │
│  📁  │                                                                   │
│  🎯  │   ┌────────────────────────┐  ┌────────────────────────────┐    │
│  📋  │   │  시스템 상태 카드       │  │  최근 대화 카드             │    │
│  🔧  │   │  API / DB / Redis       │  │  최근 5건 목록              │    │
│  ⚙️  │   └────────────────────────┘  └────────────────────────────┘    │
│      │                                                                   │
│      │   ┌────────────────────────┐  ┌────────────────────────────┐    │
│      │   │  Genspark AI 바로가기   │  │  CEO 문서 카드              │    │
│      │   │  🌐 클릭 → /genspark    │  │  기획서/기술서/보고서 목록   │    │
│      │   └────────────────────────┘  └────────────────────────────┘    │
│      │                                                                   │
│──────┤   ┌─────────────────────────────────────────────────────────┐    │
│ v0.5 │   │  CEO Chat (하단 패널, 항상 접근 가능)                     │    │
│      │   │  ┌──────────────────────────────────────────────────┐   │    │
│      │   │  │  [🟣Claude Opus ▼]  입력창...             [전송]  │   │    │
│      │   │  └──────────────────────────────────────────────────┘   │    │
│      │   └─────────────────────────────────────────────────────────┘    │
└──────┴──────────────────────────────────────────────────────────────────┘
```

### 2.1 사이드바 상세 (좌측 아이콘 그리드)

```
┌──────┐
│ AADS │  ← 브랜드 로고
├──────┤
│  🏠  │  Dashboard
│  📊  │  Project Status
│  💬  │  Conversations
│  📌  │  대화창 관리
│  🌐  │  Genspark AI  ← NEW (T-104)
│  📁  │  CEO 문서
│  🎯  │  CEO Decisions
│  📋  │  Tasks
│  👥  │  Managers
│  🔧  │  Pipeline
│  ⚙️  │  Settings
├──────┤
│ v0.5 │
└──────┘
```

---

## 3. 컴포넌트 트리

```
App (layout.tsx)
├── ClientLayout
│   ├── Sidebar
│   │   └── NavItems[] (동적 렌더링)
│   └── main
│       ├── /               → DashboardPage
│       │   ├── Header
│       │   ├── StatCards[4]
│       │   ├── SystemStatusCard
│       │   ├── RecentConvsCard
│       │   ├── GensparkCard  ← NEW (T-104)
│       │   └── CeoDocumentsCard
│       ├── /channels        → ChannelsPage (T-103)
│       │   ├── Header
│       │   ├── ChannelGrid
│       │   │   └── ChannelCard[] (🔗 ✏️ 🗑)
│       │   ├── AddChannelModal
│       │   │   ├── InputFields (id/name/desc/url/project/server)
│       │   │   ├── SystemPromptTextarea
│       │   │   └── ContextDocsList (role + URL 행 동적 추가/삭제)
│       │   └── ContextPackagePreview
│       ├── /genspark         → GensparkPage  ← NEW (T-104)
│       │   ├── IframeView (https://www.genspark.ai/)
│       │   └── FallbackView (iframe 실패 시 새 탭 버튼)
│       ├── /ceo-chat         → CeoChatPage
│       │   ├── ModelSelector  ← NEW (T-104)
│       │   │   └── ModelChip[] (5개 모델)
│       │   ├── ChatMessages[]
│       │   └── ChatInput
│       └── ... (기타 페이지)
```

---

## 4. 신규 컴포넌트 상세

### 4.1 ModelSelector (T-104)

위치: `/src/components/chat/ModelSelector.tsx`

```
┌──────────────────────────────────────────────────────┐
│  모델 선택:                                            │
│  [🟣 Claude Opus] [🔵 Claude Sonnet*] [🟡 Gemini Flash] │
│  [🟢 GPT-5 mini]  [🔴 혼합 에이전트]                   │
│  * = 기본 선택                                         │
│  선택된 모델 비용: $3/$15 per M tokens                  │
└──────────────────────────────────────────────────────┘
```

Props:
- `value: string` - 현재 선택된 모델 ID
- `onChange: (modelId: string) => void`

모델 목록:
| id | name | icon | cost | desc | default |
|---|---|---|---|---|---|
| claude-opus-4-6 | Claude Opus 4.6 | 🟣 | $5/$25 | 복잡한 전략·설계 | false |
| claude-sonnet-4-6 | Claude Sonnet 4.6 | 🔵 | $3/$15 | 일반 개발·분석 | **true** |
| gemini-2.0-flash | Gemini 2.0 Flash | 🟡 | $0.05/$0.40 | 가벼운 조회·요약 | false |
| gpt-5-mini | GPT-5 mini | 🟢 | $0.25/$2 | DevOps·스크립트 | false |
| mixture | 혼합 에이전트 | 🔴 | 자동 | 여러 모델 자동 라우팅 | false |

### 4.2 GensparkPage (T-104)

위치: `/src/app/genspark/page.tsx`

동작:
1. `<iframe src="https://www.genspark.ai/" ...>` 로딩
2. `onError` 시 `iframeError = true`
3. fallback: "Genspark AI는 iframe을 지원하지 않습니다." + 새 탭 버튼

### 4.3 ChannelsPage 확장 (T-103 컨텍스트 주입)

위치: `/src/app/channels/page.tsx`

추가 필드:
- `context_docs`: `[{role: string, url: string}]` — 행 추가/삭제 가능
- `system_prompt`: textarea
- `[컨텍스트 패키지 미리보기]` 버튼 → `/api/v1/channels/{id}/context-package` 호출

---

## 5. API 맵

### 5.1 기존 API (운용 중)

| 메서드 | 엔드포인트 | 용도 |
|---|---|---|
| GET | /api/v1/health | 시스템 상태 |
| GET | /api/v1/channels | 대화창 목록 |
| POST | /api/v1/channels | 대화창 추가 |
| PUT | /api/v1/channels/{id} | 대화창 수정 |
| DELETE | /api/v1/channels/{id} | 대화창 삭제 |
| GET | /api/v1/documents | CEO 문서 목록 |
| GET | /api/v1/documents/{id} | CEO 문서 본문 |
| POST | /api/v1/documents | CEO 문서 등록 |
| POST | /api/v1/ceo-chat/message | CEO 채팅 메시지 전송 |

### 5.2 신규 API (T-103/T-104)

| 메서드 | 엔드포인트 | 용도 | Task |
|---|---|---|---|
| GET | /api/v1/channels/{id}/context-package | 컨텍스트 패키지 조합 반환 | T-103 |

### 5.3 수정 API (T-104)

| 엔드포인트 | 변경 내용 |
|---|---|
| POST /api/v1/ceo-chat/message | `model` 파라미터 추가 (선택, 없으면 자동 라우팅) |

---

## 6. 데이터 흐름도

### 6.1 대화창 컨텍스트 자동 주입 흐름

```
CEO (Genspark) ──지시서 작성──▶ genspark_bridge.py
                                       │
                         GET /api/v1/channels/{id}/context-package
                                       │
                              aads-server channels.py
                                       │
                         context_docs[] 각 URL fetch (httpx)
                                       │
                         마크다운 조합 반환
                                       │
                         지시서 앞에 컨텍스트 삽입
                                       │
                    ──주입된 지시서──▶ /root/.genspark/directives/pending/
                                       │
                              Cursor AI (Claude Code) 실행
```

### 6.2 모델 선택 흐름 (T-104)

```
CEO ──모델 선택 (ModelSelector)──▶ selectedModel state (localStorage)
                                        │
                    POST /api/v1/ceo-chat/message { message, model }
                                        │
                           ceo_chat.py route_model()
                                        │
                   model 파라미터 있으면 override, 없으면 자동 라우팅
                                        │
                        call_llm(model, system_prompt, messages)
                                        │
                           응답 + model_used 반환
```

### 6.3 Genspark WebAI 통합 흐름 (T-104)

```
CEO ──사이드바 "Genspark AI" 클릭──▶ /genspark 페이지
                                            │
                              iframe src=https://www.genspark.ai/
                                            │
                           로딩 성공 ──▶ 전체화면 iframe 표시
                           로딩 실패 ──▶ 안내 메시지 + 새 탭 버튼
```

---

## 7. 정적 스냅샷 연동 패널

`/manager/*.json` 연동:
- `public/manager/channels.json` ← GET /api/v1/channels 스냅샷
- `public/manager/documents.json` ← GET /api/v1/documents 스냅샷
- `public/manager/agents.json` ← 에이전트 목록 스냅샷

CEO가 오프라인에서도 `/manager/channels.json` URL로 즉시 확인 가능.

---

## 8. 구현 우선순위

| 우선순위 | 컴포넌트 | Task | 상태 |
|---|---|---|---|
| P0 | 대화창 CRUD UI (channels/page.tsx) | T-103 | 완료 |
| P0 | Channels API (GET/POST/PUT/DELETE) | T-103 | 완료 |
| P1 | context_docs + context-package endpoint | T-103 | 구현 예정 |
| P1 | ModelSelector 컴포넌트 | T-104 | 구현 예정 |
| P1 | Genspark WebAI 페이지 | T-104 | 구현 예정 |
| P1 | CEO Chat 모델 override | T-104 | 구현 예정 |
| P2 | CEO Documents 패널 (대시보드) | T-095 | 구현 예정 |
| P2 | 정적 스냅샷 channels.json | T-103 | 구현 예정 |

---

*작성: Claude Code (T-095) | 2026-03-06 KST*
