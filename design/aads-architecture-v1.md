# AADS Phase 1 Architecture Design Document
> Version: 1.1-final | Date: 2026-02-28 | Author: CEO-AI (web Claude)
> Status: **확정** — CEO 승인 완료, 구현 착수 가능
> Previous: v1.0-draft → v1.1-final (21건 수정 반영)

---

## 1. 문서 개요

### 1.1 목적
AADS(Autonomous AI Development System)의 Phase 1 핵심 아키텍처를 확정한다.
이 문서는 Cursor/Claude Code가 구현에 착수하기 위한 최종 설계 기준이며,
CEO-DIRECTIVES.md의 모든 원칙(D-001~D-010)과 기술 지시(T-001~T-009)를 반영한다.

### 1.2 적용 범위
- Phase 1 (Core LangGraph + Agent Chain + 사용자 체크포인트 흐름) 구현 범위만 다룬다.
- Phase 2(Dashboard UI 고도화) 이후는 별도 설계서로 분리한다.
- HealthMate는 AADS 완성 후 첫 프로젝트이며, 이 문서의 범위 밖이다.

### 1.3 필수 참조 문서
| 문서 | 위치 |
|------|------|
| HANDOVER.md | `aads-docs/HANDOVER.md` |
| CEO-DIRECTIVES.md | `aads-docs/CEO-DIRECTIVES.md` |
| AI Models Survey | `aads-docs/reports/aads-ai-models-survey-20260228.md` |
| Open-source Tools Survey | `aads-docs/reports/aads-opensource-tools-survey-20260228.md` |
| Infra/MCP/App-builder Survey | `aads-docs/reports/aads-infra-mcp-appbuilder-survey-20260228.md` |

### 1.4 v1.0 → v1.1 주요 변경사항 (21건)
| 구분 | 건수 | 내용 |
|------|------|------|
| 가격/버전 오류 수정 | 8건 | Opus 4.6 가격 정정($5/$25), LangGraph 1.0.8, Gemini Flash $0.30/$2.50, E2B $150-250, 모델 ID 감사, 비용 재시뮬레이션(input+output), PM Agent Phase 1 포함 |
| 실전 인프라 문제 반영 | 5건 | Supabase 직접 연결(Supavisor 금지), Fly.io SSE heartbeat, langgraph-supervisor MCP 루프 버그 대체, MCP 프로세스 관리, Fly.io 2GB RAM |
| 아키텍처 보강 | 4건 | Judge Agent 추가, 구조화 JSON 통신, 점진적 자율성 게이트, PM Agent 사용자 대면 역할 |
| SaaS 사용자 흐름 | 4건 | 6단계 체크포인트, State Schema 확장, SSE 스트리밍 UX, 경쟁사 차별점 정의 |

---

## 2. 시스템 전체 아키텍처

### 2.1 아키텍처 패턴: LangGraph Native Supervisor (2-Level Hierarchy)

AADS는 **LangGraph 1.0 네이티브 tool-based supervisor 패턴**을 채택한다.
`langgraph-supervisor` 라이브러리는 MCP 도구 사용 시 무한 루프 버그(GitHub #249, 2025-10-18, 미해결)가 있어 사용하지 않는다.
LangGraph `StateGraph`를 직접 구성하여 supervisor를 구현한다.

```
┌──────────────────────────────────────────────────────────┐
│                      AADS SYSTEM                          │
│                                                           │
│  ┌─────────────────────────────────────────────────────┐ │
│  │              사용자 인터페이스 (Next.js)               │ │
│  │         채팅 + 체크포인트 승인 + 실시간 진행 스트리밍    │ │
│  └──────────────────────┬──────────────────────────────┘ │
│                          │ SSE / REST                     │
│  ┌──────────────────────▼──────────────────────────────┐ │
│  │              FastAPI Server (Fly.io, 2GB RAM)        │ │
│  └──────────────────────┬──────────────────────────────┘ │
│                          │                                │
│  ┌──────────────────────▼──────────────────────────────┐ │
│  │           TOP SUPERVISOR (Orchestrator)               │ │
│  │           Model: Claude Opus 4.6                     │ │
│  │           Role: 작업 라우팅 · 위임 · 품질 최종 검증     │ │
│  └───┬─────────┬─────────┬─────────┬─────────┬────────┘ │
│      │         │         │         │         │           │
│  ┌───▼──┐ ┌───▼──┐ ┌───▼──┐ ┌───▼──┐ ┌───▼──┐        │
│  │  PM  │ │Archi-│ │ Dev  │ │  QA  │ │DevOps│         │
│  │Agent │ │tect  │ │Agent │ │Agent │ │Agent │         │
│  └──────┘ └──────┘ └──────┘ └──────┘ └──────┘         │
│      │                                                    │
│  ┌───▼──────────────────────────────────────────┐        │
│  │  Judge Agent (독립 검증 — 별도 컨텍스트)        │        │
│  └──────────────────────────────────────────────┘        │
│                                                           │
│  ┌────────────── Researcher Agent (on-demand) ────────┐  │
│  └────────────────────────────────────────────────────┘  │
│                                                           │
│  ┌─────────────────────────────────────────────────────┐ │
│  │             SHARED INFRASTRUCTURE                    │ │
│  │  PostgreSQL Checkpointer (Supabase Direct:5432)     │ │
│  │  Upstash Redis · MCP Servers · E2B Sandbox          │ │
│  └─────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────┘
```

### 2.2 핵심 설계 원칙

**P1. Sandbox-as-Tool**: 에이전트는 Fly.io에서 실행, 코드 실행 시 E2B API 호출. API 키가 샌드박스 밖에 있어 보안 우수. 에이전트 업데이트 시 컨테이너 재빌드 불필요.

**P2. Checkpointed State Machine**: 모든 노드 실행 후 PostgreSQL 체크포인트 저장. 장애 복구, HITL, time travel 가능.

**P3. Bounded Context MCP**: 각 MCP 서버는 단일 책임, stateless·idempotent 설계.

**P4. 비용 우선(D-004)**: 월 $500 이하. 모델 라우팅 + Prompt Caching + Batch API.

**P5. 사용자 체크포인트 기반 품질 보증**: 6단계 체크포인트로 멀티에이전트 누적 오류를 각 구간에서 리셋.

**P6. Graceful Degradation**: Primary 장애 시 Fallback 자동 전환. E2B 장애 시 "코드 작성까지만" 모드. Supabase 장애 시 InMemory 임시 전환.

---

## 3. SaaS 사용자 흐름 설계

### 3.1 6단계 사용자 체크포인트

AADS SaaS의 핵심 차별점: "프롬프트 → 블랙박스 → 결과"가 아닌 단계별 사용자 확인.

```
사용자 요청: "건강관리 앱 만들어줘"
      │
      ▼
┌─────────────────────────────────────────────────────────┐
│ Stage 1: 요구사항 대화 (PM Agent ↔ 사용자)               │
│ PM이 핵심 질문 → 사용자 답변 → 요구사항 정리              │
│ 소요: 1~3분                                              │
└─────────────────────┬───────────────────────────────────┘
                      ▼
            ┌── Checkpoint ① ──┐
            │ 기획서 승인 요청    │
            │ [승인] [수정 요청]  │
            └────────┬─────────┘
                     ▼
┌─────────────────────────────────────────────────────────┐
│ Stage 2: 설계 (Architect Agent)                          │
│ 화면 구조 + DB 스키마 + 기술 선택                          │
│ 소요: 1~2분 (자율)                                       │
└─────────────────────┬───────────────────────────────────┘
                      ▼
            ┌── Checkpoint ② ──┐
            │ 설계 확인 요청      │
            │ 화면 와이어프레임    │
            │ [승인] [수정 요청]  │
            └────────┬─────────┘
                     ▼
┌─────────────────────────────────────────────────────────┐
│ Stage 3: 구현 (Developer + QA, 자율 실행)                 │
│ 실시간 진행 스트리밍: ✅완료 / ⏳진행중 / ⬜대기            │
│ 소요: 5~30분                                              │
└─────────────────────┬───────────────────────────────────┘
                      ▼
            ┌── Checkpoint ③ ──┐
            │ 중간 결과 확인      │
            │ [라이브 프리뷰 링크] │
            │ [계속] [수정 요청]  │
            └────────┬─────────┘
                     ▼
┌─────────────────────────────────────────────────────────┐
│ Stage 4: 마무리 (Developer + QA + Judge)                  │
│ 잔여 기능 + 테스트 + 독립 품질 검증                        │
│ 소요: 5~15분                                              │
└─────────────────────┬───────────────────────────────────┘
                      ▼
            ┌── Checkpoint ④ ──┐
            │ 최종 결과 확인      │
            │ [라이브 프리뷰 링크] │
            │ 테스트 결과 요약     │
            │ [배포] [수정 요청]  │
            └────────┬─────────┘
                     ▼
┌─────────────────────────────────────────────────────────┐
│ Stage 5: 배포 (DevOps Agent)                              │
│ 자동 배포 + URL 발급                                      │
└─────────────────────┬───────────────────────────────────┘
                      ▼
            ┌── Checkpoint ⑤ ──┐
            │ 배포 완료 알림      │
            │ [서비스 URL]       │
            └──────────────────┘
```

### 3.2 체크포인트별 상세

| 체크포인트 | 트리거 | 사용자에게 보여주는 것 | 사용자 액션 | 소요 시간 |
|-----------|--------|---------------------|-----------|----------|
| ① 기획서 승인 | PM 요구사항 정리 완료 | 기능 목록, 화면 수, 예상 소요 | 승인 / 수정 요청 | 30초~1분 |
| ② 설계 확인 | Architect 설계 완료 | 와이어프레임, DB 구조, 기술 스택 | 승인 / 수정 요청 | 1분 |
| ③ 중간 결과 | 핵심 기능 50%+ 완성 | 라이브 프리뷰 링크 | 계속 / 수정 요청 | 2~3분 |
| ④ 최종 결과 | 전체 완성 + 테스트 통과 | 프리뷰 + 테스트 리포트 | 배포 / 수정 요청 | 2~3분 |
| ⑤ 배포 완료 | DevOps 배포 성공 | 서비스 URL | 확인 | 10초 |

### 3.3 수정 요청 처리
사용자가 "수정 요청" 선택 시: PM이 수정 내용을 TaskSpec으로 변환 → Supervisor가 해당 에이전트에 재작업 위임 → 같은 체크포인트 재제시. 최대 3회 수정, 초과 시 PM 재상담.

---

## 4. Agent 구성 및 모델 할당

### 4.1 Agent 역할 정의

| Agent | 역할 | Primary Model (API ID) | 입력/출력 $/1M | Fallback Model (API ID) | 입력/출력 $/1M | 사용자 대면 |
|-------|------|----------------------|---------------|------------------------|---------------|-----------|
| **PM** | 사용자 대화, 요구사항 수집, 작업 분해, 체크포인트 관리 | `claude-sonnet-4-6` | $3 / $15 | `gpt-5.2-chat-latest` | $1.75 / $14 | **Yes** |
| **Supervisor** | 내부 작업 라우팅, 위임, 결과 취합 | `claude-opus-4-6` | $5 / $25 | `gemini-3.1-pro-preview` | $2 / $12 | No |
| **Architect** | 시스템 설계, 코드 구조, 기술 선택 | `claude-opus-4-6` | $5 / $25 | `gemini-3.1-pro-preview` | $2 / $12 | No |
| **Developer** | 코드 생성, 수정, 리팩토링 | `claude-sonnet-4-6` | $3 / $15 | `gpt-5.3-codex` | $1.75 / $14 | No |
| **QA** | 테스트 생성, 코드 리뷰, 버그 탐지 | `claude-sonnet-4-6` | $3 / $15 | `gpt-5-mini` | $0.25 / $2 | No |
| **Judge** | 독립 품질 검증 (별도 컨텍스트) | `claude-sonnet-4-6` | $3 / $15 | `gemini-3.1-pro-preview` | $2 / $12 | No |
| **DevOps** | 배포, CI/CD, 인프라 관리 | `gpt-5-mini` | $0.25 / $2 | `claude-haiku-4-5` | $1 / $5 | No |
| **Researcher** | 정보 검색, 데이터 수집 | `gemini-2.5-flash` | $0.30 / $2.50 | `gpt-5-nano` | $0.05 / $0.40 | No |

### 4.2 모델 라우팅 전략

```python
from enum import Enum

class ModelTier(Enum):
    REASONING = "reasoning"       # Opus — 설계, 복잡한 판단
    BALANCED = "balanced"         # Sonnet — 코드 생성, 대화
    EFFICIENT = "efficient"       # Mini/Haiku — 분류, 간단 작업
    MINIMAL = "minimal"           # Nano/Flash — 라벨링, 데이터

def get_model(tier: ModelTier, fallback: bool = False):
    models = {
        ModelTier.REASONING: ("claude-opus-4-6", "gemini-3.1-pro-preview"),
        ModelTier.BALANCED: ("claude-sonnet-4-6", "gpt-5.3-codex"),
        ModelTier.EFFICIENT: ("gpt-5-mini", "claude-haiku-4-5"),
        ModelTier.MINIMAL: ("gemini-2.5-flash", "gpt-5-nano"),
    }
    primary, backup = models[tier]
    return backup if fallback else primary
```

### 4.3 비용 시뮬레이션 (월간, input+output 반영)

하루 50~100개 사용자 작업 기준. 작업당 평균 input 3K, output 8K tokens.

| 항목 | 최소 | 기대 | 최대 |
|------|------|------|------|
| Claude Opus 4.6 (Supervisor/Architect) | $15 | $40 | $80 |
| Claude Sonnet 4.6 (PM/Dev/QA/Judge) | $20 | $55 | $130 |
| GPT-5 mini/nano (DevOps/분류) | $3 | $10 | $25 |
| Gemini 2.5 Flash (Researcher) | $3 | $15 | $35 |
| **모델 API 합계** | **$41** | **$120** | **$270** |
| E2B Pro | $150 | $180 | $250 |
| Fly.io (2GB RAM) | $15 | $30 | $50 |
| Vercel Pro | $20 | $20 | $25 |
| Supabase Pro | $25 | $30 | $40 |
| Upstash Redis | $0 | $5 | $10 |
| LangSmith Free | $0 | $0 | $39 |
| **월 총합** | **$251** | **$385** | **$684** |

기대 $385/mo → D-004 $500 이내. 최대 시나리오 초과 가능 → 비용 제어 필수.

### 4.4 비용 절감 전략

**Prompt Caching**: 반복 시스템 프롬프트 캐싱, 입력 최대 90% 절감. **Batch API**: 비긴급 작업(Researcher, Judge)에 50% 할인. **모델 다운그레이드**: 분류/라우팅은 nano($0.05). **조기 종료**: 충분한 결과 시 추가 호출 중단.

---

## 5. 기술 스택 상세

### 5.1 핵심 프레임워크

| 계층 | 기술 | 버전/사양 | 선택 근거 |
|------|------|----------|----------|
| Agent Orchestration | LangGraph (native supervisor) | >= 1.0.8 | 안정 릴리스, StateGraph 직접 구성 |
| State Persistence | langgraph-checkpoint-postgres | >= 2.0 | AsyncPostgresSaver, AES 암호화 |
| API Server | FastAPI | >= 0.115 | 비동기 I/O, SSE + heartbeat |
| Agent Tools | langchain-mcp-adapters | >= 0.1 | MultiServerMCPClient |
| Code Sandbox | E2B code-interpreter | Pro plan | 150ms cold start |
| Database | PostgreSQL 16 + pgvector (Supabase) | — | 체크포인트 + 벡터 검색 통합 |
| Cache | Upstash Redis | — | 세션, rate limiting |
| Frontend | Next.js + React + Tailwind | latest stable | Phase 2 |
| Auth/BaaS | Supabase (Auth + DB + Edge Functions) | Pro $25/mo | RLS, 실시간 |
| Observability | LangSmith Free (5K traces/mo) | — | LangGraph 네이티브 trace |

### 5.2 Python 의존성

```
# core
langgraph>=1.0.8
langgraph-checkpoint-postgres>=2.0
langchain>=1.0
langchain-core>=1.0
langchain-anthropic>=0.3
langchain-openai>=0.3
langchain-google-genai>=2.0
langchain-mcp-adapters>=0.1

# api
fastapi>=0.115
uvicorn[standard]>=0.30
uvloop>=0.20
pydantic>=2.10
sse-starlette>=2.0

# tools & sandbox
e2b-code-interpreter>=1.0
mcp>=1.0

# database
psycopg[binary]>=3.2
asyncpg>=0.30
pgvector>=0.3

# cache
redis>=5.2

# observability
langsmith>=0.2
structlog>=24.0
prometheus-client>=0.21

# security
python-jose[cryptography]>=3.3
slowapi>=0.1

# resilience
tenacity>=9.0
circuitbreaker>=2.0

# utilities
httpx>=0.27
```

---

## 6. LangGraph 그래프 설계

### 6.1 Top-Level Supervisor Graph

```python
from langgraph.graph import StateGraph, START, END
from langgraph.checkpoint.postgres.aio import AsyncPostgresSaver
from langgraph.types import Command, interrupt

# === Worker Agents (create_react_agent 또는 직접 함수) ===
# 각 에이전트는 tools + model + system prompt로 구성

# === Supervisor Graph 조립 ===
graph = StateGraph(AADSState)
graph.add_node("pm", pm_agent)
graph.add_node("supervisor", supervisor_node)
graph.add_node("architect", architect_agent)
graph.add_node("developer", developer_agent)
graph.add_node("qa", qa_agent)
graph.add_node("judge", judge_agent)
graph.add_node("devops", devops_agent)
graph.add_node("researcher", researcher_agent)
graph.add_node("user_checkpoint", user_checkpoint_node)

graph.add_edge(START, "pm")  # 사용자 요청은 항상 PM이 먼저 받음
# 이후 라우팅은 supervisor_node의 Command.goto로 동적 결정

# === Compile ===
checkpointer = AsyncPostgresSaver.from_conn_string(
    "postgresql://postgres.[ref]:[pw]@db.[ref].supabase.co:5432/postgres",
    pool_config={"min_size": 5, "max_size": 15}
)
await checkpointer.setup()
app = graph.compile(checkpointer=checkpointer)
```

### 6.2 State Schema

```python
from typing import Annotated, TypedDict, Literal
from langgraph.graph import add_messages
from langchain_core.messages import BaseMessage

class TaskSpec(TypedDict):
    """구조화된 태스크 명세 (에이전트 간 통신 프로토콜)"""
    task_id: str
    description: str
    assigned_agent: str
    dependencies: list[str]
    success_criteria: list[str]
    constraints: list[str]
    max_iterations: int

class AADSState(TypedDict):
    """AADS 글로벌 상태"""
    messages: Annotated[list[BaseMessage], add_messages]
    current_task_spec: TaskSpec | None
    task_plan: list[TaskSpec]
    task_status: Literal["requirements", "planning", "designing",
                         "developing", "testing", "deploying", "complete", "failed"]
    next: str  # 다음 실행할 노드
    # 사용자 체크포인트
    user_checkpoint_stage: Literal[
        "requirements_gathering", "plan_review", "design_review",
        "development", "midpoint_review", "final_review",
        "deploying", "deployed"
    ]
    user_feedback: list[str]
    approved_stages: list[str]
    revision_count: int
    # 산출물
    files_modified: list[str]
    test_results: dict
    preview_url: str | None
    # 비용 제어
    cost_accumulated: float
    cost_limit: float
    llm_call_count: int
    # 시스템
    hitl_required: bool
    hitl_reason: str
    judge_verdict: Literal["pass", "fail", "conditional_pass"] | None
    judge_feedback: str
    error_log: list[str]
    # 점진적 자율성
    auto_approve_enabled: bool
    success_rate_history: dict
```

### 6.3 실행 흐름

```
사용자 요청
   │
   ▼
[PM Agent] ← 사용자 대화, 요구사항 수집
   │ 요구사항 완료
   ▼
[Checkpoint ① 기획서 승인] ← interrupt
   │ 승인
   ▼
[Supervisor] ← task_plan에 따라 라우팅
   │
   ▼
[Architect] ← 설계
   │ 완료
   ▼
[Checkpoint ② 설계 확인] ← interrupt
   │ 승인
   ▼
[Developer] → [QA] ← 자율 반복
   │ 50%+ 완료
   ▼
[Checkpoint ③ 중간 확인] ← interrupt
   │ 승인
   ▼
[Developer] → [Judge] ← 독립 검증
   │ 합격
   ▼
[Checkpoint ④ 최종 확인] ← interrupt
   │ 배포 승인
   ▼
[DevOps] ← 배포
   │
   ▼
[Checkpoint ⑤ 배포 완료]
```

### 6.4 Interrupt 기반 체크포인트

```python
from langgraph.types import interrupt

def user_checkpoint_node(state: AADSState) -> dict:
    stage = state["user_checkpoint_stage"]
    summary = generate_checkpoint_summary(state, stage)

    user_response = interrupt({
        "type": "user_checkpoint",
        "stage": stage,
        "summary": summary,
        "actions": ["approve", "request_changes"]
    })

    if user_response["action"] == "approve":
        return {
            "approved_stages": [*state["approved_stages"], stage],
            "revision_count": 0
        }
    else:
        revision_count = state["revision_count"] + 1
        if revision_count > 3:
            return {"user_checkpoint_stage": "requirements_gathering"}
        return {
            "user_feedback": [*state["user_feedback"], user_response["feedback"]],
            "revision_count": revision_count
        }
```

---

## 7. 에이전트 간 구조화 통신 프로토콜

### 7.1 문제
에이전트 간 자연어 핸드오프 시 "왜 이 결정을 했는지", "제약 조건이 뭐였는지" 소실 → 체이닝 오류의 핵심 원인 (arXiv 2503.13657: 79%).

### 7.2 해결: TaskSpec JSON
모든 작업 위임은 TaskSpec JSON으로 수행. 자연어는 메시지 히스토리에 보존하되, 핵심 지시는 구조화 형태.

```json
{
    "task_id": "dev-001",
    "description": "사용자 운동 기록 입력 화면 구현",
    "assigned_agent": "developer",
    "dependencies": ["arch-001"],
    "success_criteria": [
        "운동 종류 선택 (드롭다운)",
        "시간/칼로리 입력 필드",
        "저장 버튼 → Supabase 저장",
        "입력 검증 (빈 값 방지)"
    ],
    "constraints": ["Next.js + Tailwind CSS", "Supabase RLS 적용", "모바일 반응형"],
    "max_iterations": 5
}
```

### 7.3 Judge 검증 결과

```json
{
    "task_id": "dev-001",
    "verdict": "conditional_pass",
    "criteria_results": [
        {"criterion": "운동 종류 선택", "met": true},
        {"criterion": "시간/칼로리 입력", "met": true},
        {"criterion": "Supabase 저장", "met": true},
        {"criterion": "입력 검증", "met": false, "note": "빈 값 에러 메시지 없음"}
    ],
    "overall_score": 0.75,
    "recommendation": "입력 검증 추가 후 재검증 필요"
}
```

---

## 8. MCP 서버 통합 설계

### 8.1 Phase 1 필수 MCP 서버 (7개)

| MCP Server | 용도 | 바인딩 Agent | 관리 |
|------------|------|-------------|------|
| Filesystem | 파일 읽기/쓰기/검색 | Developer, Architect, QA, Judge | 상시 |
| Git | commit, diff, log, branch | Developer, DevOps | 상시 |
| GitHub | PR, Issue, 코드 검색 | Developer, DevOps | 온디맨드 |
| PostgreSQL | 스키마 조회, 쿼리 실행 | Developer, Architect | 상시 |
| Brave Search | 웹 검색 (2,000 free/mo) | Researcher | 온디맨드 |
| Memory | 지식 그래프 | PM, Supervisor, Architect | 상시 |
| Fetch | HTTP 요청, 웹 크롤링 | Researcher | 온디맨드 |

### 8.2 프로세스 관리

FastAPI lifespan에서 `MultiServerMCPClient` 1회 초기화, 앱 수명 내내 유지.

```python
from contextlib import asynccontextmanager
from langchain_mcp_adapters.client import MultiServerMCPClient

mcp_client: MultiServerMCPClient | None = None

@asynccontextmanager
async def lifespan(app: FastAPI):
    global mcp_client
    mcp_client = MultiServerMCPClient({
        "filesystem": {"command": "npx", "args": ["@modelcontextprotocol/server-filesystem", "/workspace"], "transport": "stdio"},
        "git": {"command": "uvx", "args": ["mcp-server-git"], "transport": "stdio"},
        "memory": {"command": "npx", "args": ["@modelcontextprotocol/server-memory"], "transport": "stdio"},
        "postgresql": {"command": "npx", "args": ["@modelcontextprotocol/server-postgres", POSTGRES_URI], "transport": "stdio"},
    })
    await mcp_client.__aenter__()
    yield
    await mcp_client.__aexit__(None, None, None)
```

**상시 4개**: Filesystem, Git, Memory, PostgreSQL (~200-400MB). **온디맨드 3개**: GitHub, Brave Search, Fetch — 필요 시만 스폰.

### 8.3 Phase 2 추가
Puppeteer, Sentry, Slack, Supabase MCP.

---

## 9. 샌드박스 설계

### 9.1 Sandbox-as-Tool 패턴

```python
from e2b_code_interpreter import Sandbox
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(stop=stop_after_attempt(3), wait=wait_exponential(min=1, max=8))
async def execute_code(code: str, language: str = "python") -> dict:
    sandbox = Sandbox()  # ~150ms cold start
    try:
        execution = sandbox.run_code(code)
        return {
            "stdout": execution.logs.stdout,
            "stderr": execution.logs.stderr,
            "results": [r.text for r in execution.results],
            "error": execution.error.message if execution.error else None
        }
    finally:
        sandbox.kill()
```

### 9.2 E2B 장애 시 Fallback
E2B 장애 시 Developer는 "코드 작성까지만 완료, 실행 보류" 모드. QA에 "수동 검증 필요" 플래그 전달.

---

## 10. 인프라 & 배포

### 10.1 배포 구성도

```
┌──────────────┐     ┌───────────────┐     ┌──────────────┐
│   Vercel      │     │   Fly.io       │     │   E2B Cloud   │
│   (Frontend)  │────▶│   (API/Agent)  │────▶│   (Sandbox)   │
│   $20/mo      │     │   $20-50/mo    │     │   $150+/mo    │
└──────────────┘     └───────┬───────┘     └──────────────┘
                             │
               ┌─────────────┼─────────────┐
               ▼             ▼             ▼
        ┌──────────┐  ┌──────────┐  ┌──────────┐
        │Supabase   │  │Upstash   │  │MCP       │
        │Direct:5432│  │Redis     │  │Servers   │
        │$25/mo     │  │$0-10/mo  │  │(in-proc) │
        └──────────┘  └──────────┘  └──────────┘
```

### 10.2 Fly.io 설정

```toml
# fly.toml
app = "aads-api"
primary_region = "nrt"  # Tokyo

[build]
  dockerfile = "Dockerfile"

[[vm]]
  size = "shared-cpu-2x"
  memory = "2gb"          # 2GB (FastAPI + MCP 프로세스)
  cpus = 2

[http_service]
  internal_port = 8000
  force_https = true
  auto_stop_machines = false
  auto_start_machines = true

[[http_service.checks]]
  interval = "30s"
  timeout = "5s"
  grace_period = "10s"
  method = "GET"
  path = "/health"
```

### 10.3 Fly.io 60초 유휴 타임아웃 대응
- SSE 스트림에서 15-20초마다 `:keepalive\n\n` 하트비트 전송
- 5분 이상 장기 작업: 큐 + 폴링 폴백 (POST → 202 + job_id → GET polling)

---

## 11. FastAPI 서버 설계

### 11.1 엔드포인트

| 메서드 | 경로 | 설명 |
|--------|------|------|
| POST | `/api/v1/agent/invoke` | 동기 실행 (짧은 작업) |
| POST | `/api/v1/agent/stream` | SSE 스트리밍 (주요 사용) |
| GET | `/api/v1/agent/state/{tid}` | thread 상태 조회 |
| POST | `/api/v1/agent/checkpoint/approve` | 체크포인트 승인 |
| POST | `/api/v1/agent/checkpoint/revise` | 수정 요청 |
| GET | `/health` | 헬스 체크 |
| GET | `/metrics` | Prometheus 메트릭 |

### 11.2 SSE Heartbeat

```python
from sse_starlette.sse import EventSourceResponse
import asyncio, json

async def agent_stream_generator(thread_id: str, input_data: dict):
    config = {"configurable": {"thread_id": thread_id}}
    async for event in graph.astream_events(input_data, config):
        yield {"event": "agent_update", "data": json.dumps(event)}
    yield {"event": "done", "data": ""}

@app.post("/api/v1/agent/stream")
async def stream(request: AgentRequest):
    return EventSourceResponse(
        agent_stream_generator(request.thread_id, request.input),
        media_type="text/event-stream",
        ping=15,  # sse-starlette 내장 ping (15초마다 :ping)
        headers={"Cache-Control": "no-cache"}
    )
```

### 11.3 미들웨어 스택
Request ID, structlog JSON 로깅, CORS, Supabase JWT 인증, SlowAPI rate limiting (Redis), Prometheus 메트릭, 비용 추적.

---

## 12. 안전장치 & 비용 제어

### 12.1 시스템 HITL 게이트
프로덕션 배포, DB 마이그레이션, 단일 API 호출 $10 초과, 보안 민감 파일 수정 시 자동 중단 + 관리자 승인.

### 12.2 비용 제어

| 제어 항목 | 한도 | 동작 |
|----------|------|------|
| 작업당 LLM 호출 수 | 20회 | 초과 시 중단 |
| 작업당 비용 | $10 | 초과 시 HITL |
| 월간 총 비용 | $500 | 80% 경고, 100% 중단 |
| 에이전트 루프 | 5 iteration | 무한 루프 방지 |
| 체크포인트 수정 | 3회/포인트 | 초과 시 PM 재상담 |

### 12.3 에러 핸들링 & Graceful Degradation

**재시도**: tenacity — 3회, 1→2→4초 exponential backoff.
**Circuit Breaker**: 연속 5회 실패 시 30초 비활성화.
**모델 Fallback**: 연속 3회 실패 → Fallback 모델 자동 전환.
**E2B Fallback**: 장애 시 "코드만 작성" 모드.
**Supabase Fallback**: InMemory 체크포인터 임시 전환, 복구 후 동기화.

### 12.4 동시성 제어

| 항목 | 한도 | 근거 |
|------|------|------|
| 동시 thread | 10 | Fly.io shared-cpu-2x |
| 동시 E2B sandbox | 5 | Pro 100개 중 보수적 |
| Supabase 연결 | 15 | Pro 60개 중 25% |
| 글로벌 비용 | Redis Atomic Counter | thread 간 공유 |

---

## 13. 상태 관리 & 체크포인팅

### 13.1 Supabase Direct Connection

```python
from langgraph.checkpoint.postgres.aio import AsyncPostgresSaver

# port 5432 직접 연결 필수 (6543 Supavisor 금지 — R-011)
POSTGRES_URI = "postgresql://postgres.[ref]:[pw]@db.[ref].supabase.co:5432/postgres"

async def get_checkpointer():
    saver = AsyncPostgresSaver.from_conn_string(
        POSTGRES_URI,
        pool_config={"min_size": 5, "max_size": 15}
    )
    await saver.setup()  # 마이그레이션 1회만 실행
    return saver
```

**주의사항**: Supavisor(PgBouncer) 경유 시 AsyncPipeline 충돌 → PoolClosed/OperationalError. pool_config min 5, max 15 (Supabase Pro 60 연결 한도 내). 체크포인트 테이블 마이그레이션은 앱 시작 시 1회만. AES 암호화: `LANGGRAPH_AES_KEY` 환경 변수.

### 13.2 캐싱
Upstash Redis: 세션, 중간 결과, 비용 카운터. 무료 티어(10K commands/day) → Pay-as-you-go.

### 13.3 복구
실패 시 마지막 성공 checkpoint_id로 재개. 보존 기간 7일 (설정 가능).

---

## 14. 디렉토리 구조

```
aads-server/
├── app/
│   ├── api/v1/
│   │   ├── agent.py              # 에이전트 실행
│   │   ├── checkpoint.py         # 사용자 체크포인트
│   │   ├── health.py
│   │   └── auth.py
│   ├── core/graph/
│   │   ├── supervisor.py         # Native StateGraph supervisor
│   │   ├── agents/
│   │   │   ├── pm.py, architect.py, developer.py
│   │   │   ├── qa.py, judge.py, devops.py, researcher.py
│   │   ├── state.py              # AADSState + TaskSpec
│   │   ├── checkpoints.py        # 사용자 체크포인트 로직
│   │   ├── protocols.py          # 구조화 통신
│   │   ├── tools/
│   │   │   ├── sandbox.py, mcp_tools.py, model_router.py
│   │   └── persistence.py        # AsyncPostgresSaver
│   ├── middleware/
│   │   ├── logging.py, metrics.py, heartbeat.py, request_id.py
│   ├── services/
│   │   ├── llm.py, memory.py, cost_tracker.py
│   └── main.py
├── tests/ (unit, integration, e2e)
├── Dockerfile, fly.toml, pyproject.toml
├── .github/workflows/deploy.yml
└── .env.example
```

---

## 15. CI/CD 파이프라인

```yaml
# .github/workflows/deploy.yml
name: Deploy to Fly.io
on:
  push:
    branches: [main]
    paths: ['aads-server/**']
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: '3.11' }
      - run: pip install -r requirements.txt && pytest tests/ -v
  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: superfly/flyctl-actions/setup-flyctl@master
      - run: flyctl deploy --remote-only
        env: { FLY_API_TOKEN: '${{ secrets.FLY_API_TOKEN }}' }
  health-check:
    needs: deploy
    runs-on: ubuntu-latest
    steps:
      - run: |
          sleep 30
          STATUS=$(curl -s -o /dev/null -w "%{http_code}" https://aads-api.fly.dev/health)
          [ "$STATUS" = "200" ] || (flyctl releases rollback && exit 1)
```

---

## 16. 장애 대응 시나리오

| 장애 | 탐지 | 자동 대응 | 수동 대응 |
|------|------|---------|---------|
| LLM rate limit | 429 | exponential backoff 3회 | — |
| LLM 장애 | 연속 3회 5xx | Fallback 모델 전환 | 알림 |
| E2B 다운 | 연결 실패 | "코드만 작성" 모드 | 수동 실행 |
| Supabase 끊김 | psycopg 에러 | InMemory 임시 전환 | 복구 동기화 |
| MCP 크래시 | 프로세스 종료 | 자동 재시작 3회 | MCP 제외 진행 |
| 무한 루프 | iteration 5회 | 강제 중단 | 로그 분석 |
| 비용 초과 | Redis 카운터 | 전체 중단 + 알림 | 한도 조정 |
| SSE 끊김 | 클라이언트 재연결 | 체크포인터 상태 복원 | — |

---

## 17. 점진적 자율성 게이트

**Level 1 (시작)**: 모든 체크포인트 사용자 확인, Judge 전건 검증.
**Level 2 (데이터 축적)**: Judge 통과율 90%+ → 해당 유형 자동 승인.
**Level 3 (Phase 2)**: Checkpoint ③ 선택적 건너뛰기 "빠른 모드".

기준: Judge 통과율 90%+(최근 50건), 수정 요청 10% 이하, 평균 비용 한도 50% 이하.

---

## 18. 테스트 전략

단위 테스트: 각 에이전트 노드, MCP 툴 래퍼, 모델 라우터, TaskSpec 검증. 통합 테스트: 핸드오프, 체크포인터, E2B 왕복, interrupt/resume, Judge 검증. E2E: "투두 앱 만들어줘" 전체 파이프라인. pytest, CI/CD 자동 실행. 목표 커버리지 ≥80%.

---

## 19. 보안

환경 변수(.env) API 키 관리, Git 커밋 금지(R-003). Fly.io secrets. Supabase JWT + RLS. E2B Firecracker VM 격리. 체크포인트 AES 암호화. Rate limiting(SlowAPI + Redis). MCP 네트워크 격리 + 입력 검증.

---

## 20. Phase 1 구현 로드맵 (예상 12일)

| 단계 | 작업 | 기간 | 검증 기준 |
|------|------|------|----------|
| 1.1 | 프로젝트 초기화, Fly.io 배포 | 0.5일 | health 200 |
| 1.2 | Supabase Direct + 체크포인터 | 1일 | 체크포인트 round-trip |
| 1.3 | MCP 서버 4상시 + 3온디맨드 | 1일 | 메모리 < 1.5GB |
| 1.4 | E2B 샌드박스 + fallback | 0.5일 | 코드 실행 + fallback |
| 1.5 | PM Agent + 사용자 체크포인트 | 1.5일 | interrupt→resume |
| 1.6 | Supervisor + 5 Worker Agent | 2일 | 투두앱 E2E |
| 1.7 | Judge Agent + 구조화 통신 | 1일 | pass/fail 판정 |
| 1.8 | FastAPI + SSE + heartbeat | 1일 | 5분+ SSE 유지 |
| 1.9 | 비용 제어 + 관찰성 | 1일 | fallback 자동 전환 |
| 1.10 | 통합 테스트 + 배포 | 1.5일 | E2E 전체 성공 |

---

## 21. 버전 이력

| 버전 | 날짜 | 변경 | 승인 |
|------|------|------|------|
| 1.0-draft | 2026-02-28 | 초안 | — |
| 1.1-final | 2026-02-28 | 21건 수정 전체 반영: 가격 정정, LangGraph 1.0.8, Native Supervisor, 6단계 체크포인트, PM/Judge Agent, 구조화 통신, Graceful Degradation, 인프라 실전 문제 | **CEO 승인** |

---

*본 문서는 CEO-DIRECTIVES.md v2.0에 기반한 확정 설계서이며, Phase 1 구현의 유일한 기술 근거 문서이다.*
