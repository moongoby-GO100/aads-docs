# AADS Phase 1 Architecture Design Document v1.1

**문서 ID**: `design/aads-architecture-v1.1.md`
**버전**: 1.1 (통합본)
**작성일**: 2026-02-28
**작성자**: AI Architect (Claude Opus 4.6)
**승인 대기**: CEO

---

## 1. Purpose & Scope

AADS(AI‑Autonomous Development System)는 사용자의 자연어 지시를 받아 멀티에이전트 AI가 앱을 설계·개발·테스트·배포하는 SaaS 플랫폼이다. Phase 1은 핵심 멀티에이전트 파이프라인, 6단계 사용자 체크포인트, 크레딧 기반 과금 시스템, 인프라 기반을 구축한다. Phase 2(UI/UX 고도화, 마케팅 자동화)는 범위에서 제외한다.

---

## 2. Reference Documents

`aads-docs/HANDOVER.md`, `aads-docs/CEO-DIRECTIVES.md` (D-001~D-008), `aads-docs/reports/aads-ai-models-survey-20260228.md`, `aads-docs/reports/aads-opensource-tools-survey-20260228.md`, `aads-docs/reports/aads-infra-mcp-appbuilder-survey-20260228.md`

---

## 3. Core Design Principles

**Sandbox-as-Tool**: 모든 코드 실행은 E2B 격리 샌드박스 내에서만 수행한다. 호스트 시스템 접근을 원천 차단한다.

**Checkpointed State Machine**: LangGraph 1.0의 `interrupt()` 기반 체크포인트로 그래프 상태를 PostgreSQL에 영속화한다. 사용자 체크포인트 6단계와 내부 HITL 게이트를 모두 이 메커니즘으로 처리한다.

**MCP Bounded Context**: 각 MCP 서버는 단일 책임을 가진다. Filesystem은 파일 접근만, Git은 버전 관리만, PostgreSQL은 DB 스키마 관리만 담당한다. 에이전트는 할당된 MCP 도구만 사용할 수 있다.

**크레딧 기반 과금**: 사용자에게 "프로젝트" 단위로 과금한다. PM Agent가 요구사항 분석 시 복잡도를 판정하고 소요 크레딧을 사전 고지한다. 내부적으로는 토큰 사용량과 샌드박스 시간을 추적하여 실비용을 관리한다.

**월 인프라 예산**: 초기 $500, 성장기 $800으로 단계적 확장한다. 사용자 과금 매출이 인프라 비용의 2배를 초과하면 자동 스케일업 기준으로 삼는다.

---

## 4. Overall Architecture Pattern

LangGraph 1.0 네이티브 2단계 계층 Supervisor 패턴을 사용한다. `langgraph-supervisor` 라이브러리는 MCP 도구 연동 시 반복 루프 버그(GitHub #249)가 존재하므로, 프로토타이핑 참조용으로만 두고 프로덕션에서는 네이티브 `StateGraph`로 직접 구현한다.

Top-Level Supervisor가 사용자 요청을 받아 PM Agent에게 전달하고, PM Agent는 요구사항 대화·체크포인트를 관리하며, 실행 단계에서 Supervisor가 Architect → Developer → QA → Judge → DevOps 체인으로 작업을 라우팅한다. Researcher는 필요 시 호출되는 유틸리티 에이전트이다.

---

## 5. Agent Roles & Model Allocation

**PM Agent** — Claude Sonnet 4.6 (`claude-sonnet-4-6`, $3/$15 per 1M tokens). 사용자와 직접 대화하는 유일한 에이전트. 요구사항 수집, 복잡도 판정, 크레딧 산정, 6단계 체크포인트 관리, 진행 상황 스트리밍을 담당한다. Fallback: `gpt-5.2-chat-latest` ($1.75/$14).

**Top Supervisor** — Claude Opus 4.6 (`claude-opus-4-6`, $5/$25 per 1M tokens). 에이전트 간 라우팅, 태스크 분해, 전체 워크플로우 오케스트레이션을 담당한다. 호출 횟수가 가장 많으므로 비용 민감. 안정화 후 Sonnet으로 다운그레이드 검토 대상. Fallback: `gemini-3.1-pro-preview` ($2/$12).

**Architect Agent** — Claude Opus 4.6. 시스템 설계, DB 스키마, API 구조, 화면 구성을 산출한다. Fallback: `gpt-5.2-chat-latest`.

**Developer Agent** — Claude Sonnet 4.6. 실제 코드 생성, E2B 샌드박스 실행, 파일 시스템 조작을 수행한다. Fallback: `gpt-5.3-codex` ($1.75/$14).

**QA Agent** — Claude Sonnet 4.6. 테스트 코드 작성, 샌드박스 내 테스트 실행, 결과 분석을 수행한다. Fallback: `gpt-5-mini` ($0.25/$2).

**Judge Agent** — Claude Sonnet 4.6. 다른 에이전트 산출물을 TaskSpec 기준으로 평가한다. Pass/Fail/Revision을 판정한다. 독립적 평가를 위해 이전 에이전트의 reasoning은 전달하지 않고 산출물과 스펙만 전달한다. Fallback: `gpt-5-mini`.

**DevOps Agent** — GPT-5 Mini (`gpt-5-mini`, $0.25/$2). 배포 스크립트 생성, 환경 설정, 헬스체크를 담당한다. Fallback: `claude-haiku-4-5` ($1/$5).

**Researcher Agent** — Gemini 2.5 Flash (`gemini-2.5-flash`, $0.30/$2.50). 기술 조사, 라이브러리 검색, 문서 참조를 담당한다. Fallback: `gpt-5-nano` ($0.05/$0.40).

---

## 6. Model Routing Pseudocode

```python
from enum import Enum
from dataclasses import dataclass

class AgentRole(Enum):
    PM = "pm"
    SUPERVISOR = "supervisor"
    ARCHITECT = "architect"
    DEVELOPER = "developer"
    QA = "qa"
    JUDGE = "judge"
    DEVOPS = "devops"
    RESEARCHER = "researcher"

@dataclass
class ModelConfig:
    primary: str
    fallback: str
    max_tokens: int
    temperature: float

MODEL_ROUTING: dict[AgentRole, ModelConfig] = {
    AgentRole.PM: ModelConfig("claude-sonnet-4-6", "gpt-5.2-chat-latest", 4096, 0.7),
    AgentRole.SUPERVISOR: ModelConfig("claude-opus-4-6", "gemini-3.1-pro-preview", 2048, 0.3),
    AgentRole.ARCHITECT: ModelConfig("claude-opus-4-6", "gpt-5.2-chat-latest", 8192, 0.4),
    AgentRole.DEVELOPER: ModelConfig("claude-sonnet-4-6", "gpt-5.3-codex", 16384, 0.2),
    AgentRole.QA: ModelConfig("claude-sonnet-4-6", "gpt-5-mini", 8192, 0.2),
    AgentRole.JUDGE: ModelConfig("claude-sonnet-4-6", "gpt-5-mini", 4096, 0.1),
    AgentRole.DEVOPS: ModelConfig("gpt-5-mini", "claude-haiku-4-5", 4096, 0.2),
    AgentRole.RESEARCHER: ModelConfig("gemini-2.5-flash", "gpt-5-nano", 8192, 0.3),
}

async def route_to_model(role: AgentRole, prompt: str, state: "AADSState") -> str:
    config = MODEL_ROUTING[role]
    if state["cost_accumulated"] >= state["cost_limit"]:
        raise BudgetExhaustedError(f"Task budget ${state['cost_limit']} exceeded")
    try:
        response = await call_llm(config.primary, prompt, config.max_tokens, config.temperature)
    except (RateLimitError, ServiceUnavailableError):
        response = await call_llm(config.fallback, prompt, config.max_tokens, config.temperature)
    state["cost_accumulated"] += calculate_cost(response, config)
    return response
```

---

## 7. Structured Communication Protocol — TaskSpec

에이전트 간 통신은 자연어가 아닌 JSON 스키마 기반 TaskSpec으로 수행한다. 이는 멀티에이전트 실패 원인 79%(스펙 모호성+조율 실패)를 완화하기 위한 핵심 설계이다.

```python
from pydantic import BaseModel, Field
from typing import Optional
from enum import Enum

class TaskStatus(str, Enum):
    PENDING = "pending"
    IN_PROGRESS = "in_progress"
    REVIEW = "review"
    APPROVED = "approved"
    REVISION_NEEDED = "revision_needed"
    COMPLETED = "completed"
    FAILED = "failed"

class TaskSpec(BaseModel):
    task_id: str = Field(description="Unique task identifier")
    parent_task_id: Optional[str] = Field(default=None, description="Parent task if subtask")
    description: str = Field(description="Concrete task description")
    assigned_agent: str = Field(description="Target agent role")
    success_criteria: list[str] = Field(description="Measurable criteria for completion")
    constraints: list[str] = Field(default_factory=list, description="Technical constraints")
    input_artifacts: list[str] = Field(default_factory=list, description="File paths or references")
    output_artifacts: list[str] = Field(default_factory=list, description="Expected output paths")
    max_iterations: int = Field(default=3, description="Max retry attempts")
    max_llm_calls: int = Field(default=15, description="Max LLM calls for this task")
    budget_limit_usd: float = Field(default=10.0, description="Max cost in USD")
    status: TaskStatus = TaskStatus.PENDING
```

---

## 8. State Schema

```python
from typing import TypedDict, Annotated, Optional
from langgraph.graph import add_messages

class UserCheckpointStage(str, Enum):
    REQUIREMENTS = "requirements"
    PLAN_REVIEW = "plan_review"
    DESIGN_REVIEW = "design_review"
    DEVELOPMENT = "development"
    MIDPOINT_REVIEW = "midpoint_review"
    FINAL_REVIEW = "final_review"
    DEPLOYED = "deployed"

class AADSState(TypedDict):
    messages: Annotated[list, add_messages]
    current_task: Optional[TaskSpec]
    task_queue: list[TaskSpec]
    completed_tasks: list[TaskSpec]
    next_agent: str
    previous_agent: str
    user_checkpoint_stage: UserCheckpointStage
    user_feedback: Optional[str]
    approved_stages: list[str]
    revision_count: int
    project_id: str
    project_spec: dict
    generated_files: list[str]
    test_results: dict
    cost_accumulated: float
    cost_limit: float
    llm_calls_count: int
    user_id: str
    user_plan_tier: str
    project_credits_consumed: float
    project_complexity_estimate: float
    sandbox_id: Optional[str]
    sandbox_url: Optional[str]
```

---

## 9. User Checkpoint Flow — 6단계

이것이 AADS의 핵심 차별점이다. PM Agent가 사용자와 대화하며 6단계 체크포인트에서 승인을 받는다. 각 체크포인트는 LangGraph interrupt() 기반으로 구현되어 그래프 실행이 일시 중지되고, 사용자 응답 시 Command(resume=...) 로 재개된다.

**Stage 1 — Requirements Dialogue (요구사항 대화)** PM Agent가 사용자에게 핵심 질문을 던진다: "어떤 앱을 만드시겠습니까?", "주요 기능은?", "대상 사용자는?", "참고하는 앱이 있나요?". 대화형으로 2~5회 교환 후 요구사항을 정리하고, 복잡도를 판정하며, 소요 크레딧을 사전 고지한다.

```python
def pm_requirements_node(state: AADSState):
    requirements_summary = pm_agent.analyze(state["messages"])
    complexity = pm_agent.estimate_complexity(requirements_summary)
    credits_needed = calculate_credits(complexity)
    user_credits = get_user_credits(state["user_id"])
    if user_credits < credits_needed:
        return interrupt({
            "stage": "requirements",
            "type": "insufficient_credits",
            "message": f"이 프로젝트는 약 {credits_needed} 크레딧이 필요합니다. 현재 {user_credits} 크레딧 보유 중입니다.",
            "upgrade_options": get_upgrade_options(state["user_plan_tier"])
        })
    approval = interrupt({
        "stage": "requirements",
        "type": "approval_needed",
        "summary": requirements_summary,
        "estimated_credits": credits_needed,
        "estimated_time": f"{complexity.time_estimate_minutes}분",
        "message": "요구사항을 확인해 주세요. 수정할 부분이 있으면 말씀해 주세요."
    })
    if approval.get("approved"):
        return {
            "user_checkpoint_stage": "plan_review",
            "project_complexity_estimate": complexity.score,
            "project_credits_consumed": 0,
            "approved_stages": ["requirements"]
        }
    else:
        return {"user_feedback": approval.get("feedback"), "revision_count": state["revision_count"] + 1}
```

**Stage 2 — Plan Review (계획 검토)** PM Agent가 Architect Agent의 출력을 정리하여 사용자에게 제시한다: 기능 목록, 화면 구성, 기술 스택, DB 구조 개요, 예상 소요 시간. 사용자는 승인하거나 수정을 요청할 수 있다.

**Stage 3 — Design Review (설계 검토)** Architect Agent가 생성한 구체적 화면 와이어프레임(텍스트 기반), DB 스키마, API 엔드포인트 목록을 PM Agent가 사용자에게 전달한다. "대시보드에 차트를 추가해 주세요" 같은 피드백을 받으면 Architect에게 재작업을 요청한다.

**Stage 4 — Development (개발 진행)** 이 단계는 자동 진행이다. SSE(Server-Sent Events)로 실시간 진행 상황을 스트리밍한다: "프로젝트 초기화 완료", "인증 모듈 구현 중 (3/7)", "대시보드 UI 렌더링 완료". 사용자가 별도 승인하지 않아도 되지만, 진행 상황을 확인할 수 있다.

**Stage 5 — Midpoint Review (중간 검토)** 핵심 화면이 완성되면 PM Agent가 E2B 샌드박스 프리뷰 URL을 사용자에게 전달한다. 사용자가 직접 확인하고 피드백을 준다. "로그인 화면 디자인을 바꿔주세요", "이 기능을 추가해 주세요" 등. 최대 3회 수정 가능.

**Stage 6 — Final Review & Deployment (최종 검토 및 배포)** 완성된 앱의 프리뷰, 테스트 통과율, 기능 체크리스트를 PM Agent가 제시한다. 사용자가 "배포"를 승인하면 DevOps Agent가 실행하고, 배포 URL을 전달한다.

```python
def user_checkpoint_node(state: AADSState):
    stage = state["user_checkpoint_stage"]
    max_revisions = 3
    if state["revision_count"] >= max_revisions:
        return interrupt({
            "stage": stage,
            "type": "max_revisions_reached",
            "message": f"최대 수정 횟수({max_revisions}회)에 도달했습니다. 현재 상태로 진행할까요?"
        })
    checkpoint_data = build_checkpoint_payload(stage, state)
    response = interrupt({
        "stage": stage,
        "type": "approval_needed",
        **checkpoint_data
    })
    if response.get("approved"):
        return {
            "approved_stages": state["approved_stages"] + [stage],
            "revision_count": 0,
            "user_checkpoint_stage": get_next_stage(stage)
        }
    else:
        return {
            "user_feedback": response.get("feedback"),
            "revision_count": state["revision_count"] + 1
        }
```

---

## 10. LangGraph Graph Design

```python
from langgraph.graph import StateGraph, START, END
from langgraph.checkpoint.postgres.aio import AsyncPostgresSaver

builder = StateGraph(AADSState)
builder.add_node("pm_requirements", pm_requirements_node)
builder.add_node("supervisor", supervisor_node)
builder.add_node("architect", architect_node)
builder.add_node("developer", developer_node)
builder.add_node("qa", qa_node)
builder.add_node("judge", judge_node)
builder.add_node("devops", devops_node)
builder.add_node("researcher", researcher_node)
builder.add_node("user_checkpoint", user_checkpoint_node)

builder.add_edge(START, "pm_requirements")
builder.add_edge("pm_requirements", "user_checkpoint")
builder.add_conditional_edges("user_checkpoint", route_after_checkpoint)
builder.add_conditional_edges("supervisor", route_to_agent)
builder.add_edge("architect", "judge")
builder.add_edge("developer", "judge")
builder.add_edge("qa", "judge")
builder.add_conditional_edges("judge", route_after_judge)
builder.add_edge("devops", END)

checkpointer = AsyncPostgresSaver.from_conn_string(
    conn_string=SUPABASE_DIRECT_URL,  # port 5432, not 6543
    min_size=5,
    max_size=15
)
await checkpointer.setup()
graph = builder.compile(checkpointer=checkpointer)
```

---

## 11. MCP Server Configuration

상시 실행(always-on): Filesystem MCP, Git MCP, Memory MCP, PostgreSQL MCP. 온디맨드(on-demand): GitHub MCP, Brave Search MCP, Fetch MCP. langchain-mcp-adapters의 MultiServerMCPClient로 관리하며 SSE transport를 사용한다.

```python
from langchain_mcp_adapters.client import MultiServerMCPClient

mcp_config = {
    "filesystem": {"url": "http://localhost:3001/sse", "transport": "sse"},
    "git": {"url": "http://localhost:3002/sse", "transport": "sse"},
    "memory": {"url": "http://localhost:3003/sse", "transport": "sse"},
    "postgres": {"url": "http://localhost:3004/sse", "transport": "sse"},
}

on_demand_config = {
    "github": {"url": "http://localhost:3005/sse", "transport": "sse"},
    "brave_search": {"url": "http://localhost:3006/sse", "transport": "sse"},
    "fetch": {"url": "http://localhost:3007/sse", "transport": "sse"},
}
```

MCP 프로세스 관리는 supervisord로 수행한다. Fly.io 컨테이너 시작 시 supervisord가 FastAPI, 상시 MCP 서버 4개, Redis를 기동한다.

---

## 12. Sandbox-as-Tool Implementation

```python
from e2b_code_interpreter import AsyncSandbox
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(stop=stop_after_attempt(3), wait=wait_exponential(min=2, max=10))
async def execute_in_sandbox(code: str, language: str = "python") -> dict:
    try:
        sandbox = await AsyncSandbox.create(timeout=300)
        execution = await sandbox.run_code(code, language=language)
        return {
            "stdout": execution.logs.stdout,
            "stderr": execution.logs.stderr,
            "exit_code": 0 if not execution.error else 1,
            "error": str(execution.error) if execution.error else None
        }
    except Exception as e:
        return {
            "stdout": "",
            "stderr": str(e),
            "exit_code": -1,
            "error": f"Sandbox unavailable: {str(e)}",
            "fallback": True,
            "code": code
        }
    finally:
        if sandbox:
            await sandbox.kill()
```

---

## 13. SaaS Pricing & Billing System

### 13.1 Pricing Tiers

- **Starter (무료)**: 월 2 프로젝트(간단한 앱만), 체크포인트 3단계(요구사항·계획·최종), 배포 불가(프리뷰만), AADS 워터마크.
- **Pro ($49/월)**: 월 5 프로젝트 포함, 6단계 풀 체크포인트, Supabase 배포 포함, 커스텀 도메인, 추가 프로젝트 $15/건.
- **Business ($149/월)**: 월 20 프로젝트 포함, 우선 처리 큐, 팀 워크스페이스(5명), GitHub 연동+코드 소유권, API 연동 지원, 추가 프로젝트 $10/건.
- **Enterprise (커스텀)**: 무제한 프로젝트, 전용 인프라, SLA 보장, SSO, 감사 로그.

### 13.2 Project Credit System

| 프로젝트 유형 | 크레딧 소모 | 설명 |
|--------------|------------|------|
| 간단한 앱 (랜딩, 투두) | 1 프로젝트 | 3화면 이하, DB 1~2 테이블 |
| 중간 규모 앱 (인증, CRUD) | 1 프로젝트 | 5~7화면, 인증 포함 |
| 복잡한 앱 (결제, 대시보드) | 2~3 프로젝트 | 10+화면, 외부 API 연동 |
| 기능 추가/수정 | 0.5 프로젝트 | 기존 프로젝트 수정 |
| 버그 수정 | 0.2 프로젝트 | 단일 이슈 해결 |

PM Agent가 Stage 1(요구사항 대화)에서 복잡도를 자동 판정하고 소요 크레딧을 사용자에게 사전 고지한다. 사용자 승인 후에만 크레딧이 차감된다.

### 13.3 Competitive Positioning

| | Bolt/Lovable | AADS | 프리랜서 |
|---|-------------|-----|---------|
| 월 비용 | $25~100 | $49~149 | $500~3,000/건 |
| 결과물 | 코드 조각/프로토타입 | 완성된 앱+배포 | 완성된 앱 |
| 대화형 요구사항 수집 | 없음 | PM Agent 6단계 | 미팅 |
| 중간 확인/수정 | 수동 프롬프트 반복 | 자동 체크포인트 | 주간 미팅 |

AADS는 Bolt/Lovable 대비 2배 비싸지만, 프리랜서 대비 10~60배 저렴하다. 포지셔닝: "프리랜서 품질을 AI 가격에".

### 13.4 Billing Infrastructure

Stripe를 사용한 구독+사용량 하이브리드 과금이다. Next.js + Supabase + Stripe 스타터 킷(vercel/subscription-starter)을 기반으로 구현한다.

```sql
CREATE TABLE user_subscriptions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES auth.users(id),
    stripe_customer_id TEXT NOT NULL,
    stripe_subscription_id TEXT,
    plan_tier TEXT NOT NULL DEFAULT 'starter',
    monthly_credits_total FLOAT NOT NULL DEFAULT 2,
    monthly_credits_used FLOAT NOT NULL DEFAULT 0,
    billing_cycle_start TIMESTAMPTZ NOT NULL,
    billing_cycle_end TIMESTAMPTZ NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE credit_transactions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES auth.users(id),
    project_id UUID,
    credits_amount FLOAT NOT NULL,
    transaction_type TEXT NOT NULL,
    description TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE project_billing (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID NOT NULL,
    user_id UUID REFERENCES auth.users(id),
    credits_estimated FLOAT NOT NULL,
    credits_actual FLOAT,
    model_cost_usd FLOAT DEFAULT 0,
    sandbox_cost_usd FLOAT DEFAULT 0,
    total_internal_cost_usd FLOAT DEFAULT 0,
    complexity_score FLOAT,
    status TEXT DEFAULT 'in_progress',
    started_at TIMESTAMPTZ DEFAULT NOW(),
    completed_at TIMESTAMPTZ
);

ALTER TABLE user_subscriptions ENABLE ROW LEVEL SECURITY;
ALTER TABLE credit_transactions ENABLE ROW LEVEL SECURITY;
ALTER TABLE project_billing ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can read own subscription" ON user_subscriptions
    FOR SELECT USING (auth.uid() = user_id);
CREATE POLICY "Users can read own transactions" ON credit_transactions
    FOR SELECT USING (auth.uid() = user_id);
CREATE POLICY "Users can read own billing" ON project_billing
    FOR SELECT USING (auth.uid() = user_id);
```

---

## 14. Tech Stack Summary

- **Core Orchestration**: LangGraph >= 1.0.10, langgraph-checkpoint-postgres >= 2.0, langchain-core >= 1.0, langchain-mcp-adapters
- **Backend**: FastAPI 0.115+, Python 3.12, Uvicorn, Pydantic v2
- **Frontend**: Next.js (latest stable), React, Tailwind CSS, shadcn/ui, Supabase Auth
- **Database**: Supabase PostgreSQL 16 + pgvector (direct connection port 5432), Upstash Redis (세션, 큐, 캐시)
- **Billing**: Stripe (구독 + 사용량 미터링)
- **Sandbox**: E2B Pro (코드 실행, 프리뷰)
- **MCP**: Filesystem, Git, Memory, PostgreSQL (상시), GitHub, Brave Search, Fetch (온디맨드)
- **Observability**: LangSmith (free tier → $39/mo), structlog JSON logging, Prometheus metrics
- **Hosting**: Fly.io (FastAPI + MCP, shared-cpu-2x 2GB), Vercel (Next.js), E2B Cloud (sandbox), Supabase (DB + Auth)

---

## 15. Infrastructure & Deployment

### 15.1 Fly.io Configuration

```toml
app = "aads-api"
primary_region = "nrt"

[build]
  dockerfile = "Dockerfile"

[env]
  PYTHON_ENV = "production"
  MCP_TRANSPORT = "sse"

[http_service]
  internal_port = 8000
  force_https = true
  auto_stop_machines = false
  auto_start_machines = true
  min_machines_running = 1

[[vm]]
  cpu_kind = "shared"
  cpus = 2
  memory_mb = 2048
```

### 15.2 SSE Keep-Alive (Fly.io 60s timeout 대응)

```python
async def sse_stream_with_keepalive(project_id: str):
    async def event_generator():
        last_event_time = time.time()
        async for event in project_event_stream(project_id):
            yield {"event": event.type, "data": json.dumps(event.data)}
            last_event_time = time.time()
            if time.time() - last_event_time > 20:
                yield {"event": "keepalive", "data": ""}
    return EventSourceResponse(event_generator())
```

### 15.3 Process Management (supervisord)

```ini
[supervisord]
nodaemon=true

[program:fastapi]
command=uvicorn app.main:app --host 0.0.0.0 --port 8000
autorestart=true

[program:mcp-filesystem]
command=npx -y @modelcontextprotocol/server-filesystem /workspace --transport sse --port 3001
autorestart=true

[program:mcp-git]
command=npx -y @modelcontextprotocol/server-git --transport sse --port 3002
autorestart=true

[program:mcp-memory]
command=npx -y @modelcontextprotocol/server-memory --transport sse --port 3003
autorestart=true

[program:mcp-postgres]
command=npx -y @modelcontextprotocol/server-postgres --transport sse --port 3004
autorestart=true
```

---

## 16. Monthly Cost Breakdown

### 16.1 Infrastructure Cost (Fixed)

| Item | Min | Expected | Max |
|------|-----|----------|-----|
| E2B Pro | $150 | $180 | $250 |
| Fly.io (shared-cpu-2x, 2GB) | $15 | $25 | $50 |
| Vercel Pro | $20 | $20 | $25 |
| Supabase Pro | $25 | $30 | $40 |
| Upstash Redis | $0 | $5 | $10 |
| LangSmith | $0 | $0 | $39 |
| Stripe fees (2.9% + 30¢) | $5 | $15 | $50 |
| **인프라 소계** | **$215** | **$275** | **$464** |

### 16.2 AI API Cost (Variable, 100 tasks/month)

| Scenario | Tasks/mo | Cost/task | API total |
|----------|----------|-----------|-----------|
| 보수적 (간단 위주) | 100 | $3.00 | $300 |
| 기대치 (혼합) | 100 | $5.50 | $550 |
| 최대 (복잡 위주) | 100 | $7.00 | $700 |

### 16.3 Total Monthly Cost vs Revenue

| Phase | Users | Monthly Revenue | Infra + API Cost | Margin |
|-------|-------|-----------------|------------------|--------|
| 초기 (월 1~3) | 10~20 | $490~$1,500 | $515~$825 | 0~45% |
| 성장기 (월 4~6) | 50~100 | $2,450~$8,000 | $825~$1,500 | 65~82% |
| 안정기 (월 7~12) | 200~500 | $9,800~$40,000 | $1,500~$5,000 | 85~90% |

---

## 17. Cost Controls & Safety

- **Per-task**: LLM 호출 최대 15회, 비용 상한 $10, 연속 실패 3회 시 circuit-break.
- **Per-user/day**: $20 상한.
- **Monthly**: 인프라 예산 80% 도달 시 Slack/Discord 경고, 100% 도달 시 새 프로젝트 접수 중단 (진행 중 프로젝트는 완료).
- **Graceful degradation**: LLM API 장애 → fallback 모델 자동 전환. E2B 장애 → 코드 생성만 수행(실행 불가 명시). Supabase 장애 → InMemory checkpoint로 임시 전환 후 복구 시 sync.
- **Concurrency caps**: 동시 사용자 스레드 10개, 동시 E2B sandbox 5개, Supabase 커넥션 풀 15개.

---

## 18. Graceful Degradation Strategy

```python
class DegradationLevel(str, Enum):
    FULL = "full"
    REDUCED = "reduced"
    MINIMAL = "minimal"
    MAINTENANCE = "maintenance"

async def health_check() -> DegradationLevel:
    checks = await asyncio.gather(
        check_llm_apis(),
        check_e2b(),
        check_supabase(),
        check_mcp_servers(),
        return_exceptions=True
    )
    failures = sum(1 for c in checks if isinstance(c, Exception))
    if failures == 0: return DegradationLevel.FULL
    if failures <= 1: return DegradationLevel.REDUCED
    if failures <= 2: return DegradationLevel.MINIMAL
    return DegradationLevel.MAINTENANCE
```

---

## 19. FastAPI Endpoint Layout

- `POST   /api/v1/projects` — Create new project
- `GET    /api/v1/projects/{id}` — Get project status
- `POST   /api/v1/projects/{id}/checkpoint` — Respond to checkpoint
- `GET    /api/v1/projects/{id}/stream` — SSE real-time progress
- `DELETE /api/v1/projects/{id}` — Cancel project
- `GET    /api/v1/users/me` — Current user info + credits
- `GET    /api/v1/users/me/credits` — Credit balance & history
- `POST   /api/v1/users/me/credits/topup` — Purchase additional credits
- `POST   /api/v1/billing/webhook` — Stripe webhook handler
- `GET    /api/v1/billing/plans` — Available pricing plans
- `POST   /api/v1/billing/subscribe` — Create/update subscription
- `POST   /api/v1/billing/portal` — Stripe customer portal redirect
- `GET    /api/v1/health` — Health check + degradation level
- `GET    /api/v1/metrics` — Prometheus metrics

**Middleware Stack**: Request ID injection, structlog JSON logging, CORS, JWT verification via Supabase Auth, SlowAPI rate limiting (60 req/min per user), Prometheus request metrics, credit balance check middleware.

---

## 20. Data Flow Diagram

User (Next.js) → POST /projects (natural language request) → FastAPI validates JWT, checks credits → LangGraph graph.invoke() with thread_id → PM Agent: requirements dialogue → interrupt() → SSE sends checkpoint to user → User approves → Command(resume=True) → Supervisor routes to Architect → Architect produces design → Judge validates → interrupt() → Design Review checkpoint → User approves → Supervisor routes to Developer → Developer generates code in E2B sandbox → QA writes + runs tests → Judge validates → SSE streams progress → interrupt() → Midpoint Review (preview URL) → User approves → Developer completes → interrupt() → Final Review → User approves deploy → DevOps deploys → SSE sends deployment URL → Project complete → credit_transactions recorded

---

## 21. Directory Layout

```
aads/
├── app/
│   ├── main.py
│   ├── config.py
│   ├── middleware/
│   │   ├── auth.py
│   │   ├── credits.py
│   │   ├── logging.py
│   │   └── rate_limit.py
│   ├── api/
│   │   ├── projects.py
│   │   ├── users.py
│   │   ├── billing.py
│   │   ├── stream.py
│   │   └── health.py
│   ├── agents/
│   │   ├── supervisor.py
│   │   ├── pm.py
│   │   ├── architect.py
│   │   ├── developer.py
│   │   ├── qa.py
│   │   ├── judge.py
│   │   ├── devops.py
│   │   └── researcher.py
│   ├── graph/
│   │   ├── state.py
│   │   ├── builder.py
│   │   ├── checkpoints.py
│   │   └── routing.py
│   ├── services/
│   │   ├── billing_service.py
│   │   ├── credit_service.py
│   │   ├── sandbox_service.py
│   │   ├── mcp_service.py
│   │   └── model_router.py
│   └── models/
│       ├── database.py
│       └── schemas.py
├── mcp/
│   ├── supervisord.conf
│   └── start.sh
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
│       └── test_todo_app_flow.py
├── Dockerfile
├── fly.toml
├── pyproject.toml
└── README.md
```

---

## 22. Implementation Roadmap (Phase 1, ~15 days)

- **Week 1 (Day 1~5)**: PM Agent + Supervisor + Developer (3-agent chain). Supabase 직접 연결 + checkpoint 설정. 간단한 체크포인트 플로우(Stage 1, 2, 6). 투두앱 E2E 테스트.
- **Week 2 (Day 6~10)**: QA Agent + Judge Agent 추가. 6단계 풀 체크포인트 구현. SSE 실시간 스트리밍. Stripe 구독 + 크레딧 시스템. MCP 서버 연동(4 상시 + 3 온디맨드).
- **Week 3 (Day 11~15)**: Architect Agent + DevOps Agent + Researcher Agent 추가. 중간 규모 앱(건강관리 앱) 풀 E2E 테스트. 비용 모니터링 + 경고 시스템. 프로덕션 배포(Fly.io + Vercel). 부하 테스트 및 안정화.

---

## 23. Testing Strategy

- **Unit tests**: 각 에이전트 노드의 입출력 검증, TaskSpec 직렬화, 크레딧 계산 로직, 모델 라우팅 fallback.
- **Integration tests**: LangGraph graph 전체 흐름 (mock LLM 사용), Supabase checkpoint 저장/복원, Stripe webhook 처리, MCP 도구 호출.
- **E2E tests**: 투두앱 생성 플로우 (Stage 1~6), 크레딧 부족 시나리오, 체크포인트 수정 요청 3회 시나리오, SSE 스트리밍 안정성, Fly.io 60s timeout 대응.

```python
async def test_todo_app_full_flow():
    response = await client.post("/api/v1/projects", json={"message": "투두 앱 만들어줘"})
    project_id = response.json()["project_id"]
    checkpoint = await wait_for_checkpoint(project_id, "requirements")
    assert checkpoint["estimated_credits"] <= 1.0
    await client.post(f"/api/v1/projects/{project_id}/checkpoint", json={"approved": True})
    checkpoint = await wait_for_checkpoint(project_id, "plan_review")
    assert "features" in checkpoint
    await client.post(f"/api/v1/projects/{project_id}/checkpoint", json={"approved": True})
    # ... continue through all 6 stages
    final = await wait_for_completion(project_id)
    assert final["status"] == "deployed"
    assert final["deployment_url"] is not None
```

---

## 24. Security Measures

- **Secrets management**: 모든 API 키는 Fly.io secrets으로 관리. .env 파일은 .gitignore에 포함.
- **Authentication**: Supabase Auth (OAuth 2.0 — Google, GitHub). JWT 검증은 FastAPI 미들웨어에서 매 요청마다 수행.
- **Database**: Supabase RLS(Row Level Security) 활성화. 사용자는 자신의 프로젝트·구독·트랜잭션만 접근 가능.
- **Checkpoint encryption**: LANGGRAPH_AES_KEY 환경 변수로 AES-256 암호화.
- **Sandbox isolation**: E2B 컨테이너 격리. 호스트 파일시스템 접근 불가.
- **Rate limiting**: SlowAPI — 60 req/min per user, 1000 req/min global.
- **Input validation**: Pydantic v2로 모든 API 입력 검증. SQL injection, XSS 방지.

---

## 25. Approval Log Table

```sql
CREATE TABLE approval_log (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID NOT NULL,
    user_id UUID REFERENCES auth.users(id),
    checkpoint_stage TEXT NOT NULL,
    action TEXT NOT NULL,
    feedback TEXT,
    agent_output JSONB,
    thread_id TEXT NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_approval_log_project ON approval_log(project_id);
CREATE INDEX idx_approval_log_user ON approval_log(user_id);
```

---

## 26. Revenue Projection Summary

| | 초기 (3개월) | 성장기 (6개월) | 안정기 (12개월) |
|---|-------------|---------------|----------------|
| 사용자 | 10~20 | 50~100 | 200~500 |
| 월 매출 | $490~$1,500 | $2,450~$8,000 | $9,800~$40,000 |
| 인프라 비용 | $500~$825 | $825~$1,500 | $1,500~$5,000 |
| 마진 | 0~45% | 65~82% | 85~90% |
| **BEP (손익분기)** | 사용자 약 15~20명 시점 | | |

---

## 27. Key Risks & Mitigations

- **Risk 1 — 멀티에이전트 체인 실패 (41~86.7% 산업 실패율)**: Judge Agent 도입, TaskSpec 구조화 통신, 최대 3회 재시도로 완화. 초기 3-agent 구성으로 시작하여 점진적 확장.
- **Risk 2 — LangGraph interrupt 재개 시 노드 재실행**: interrupt() 이전 side-effect는 idempotent하게 구현. DB write는 interrupt 이후에 배치.
- **Risk 3 — Supabase AsyncPostgresSaver 호환성**: 직접 연결(port 5432) 사용, Supavisor/PgBouncer 우회. 커넥션 풀 min 5, max 15.
- **Risk 4 — Fly.io 60s SSE timeout**: 20초 간격 keepalive 전송. 5분 초과 작업은 task-queue + polling fallback.
- **Risk 5 — 월 비용 초과**: 실시간 비용 추적, 80% 경고, 100% 차단. Prompt Caching + Batch API로 최적화.

---

## Appendix A — 변경 이력

| 버전 | 날짜 | 변경 내용 |
|------|------|----------|
| v1.0 | 2026-02-28 | 초안 작성 |
| v1.1 | 2026-02-28 | 21개 수정사항 반영 (가격 정정, LangGraph 1.0 마이그레이션, 네이티브 supervisor 전환, Judge Agent 추가, 구조화 통신 프로토콜, 6단계 사용자 체크포인트, SaaS 과금 모델 통합, Stripe 빌링, 크레딧 시스템, Supabase 직접 연결, SSE keepalive, MCP 프로세스 관리, RAM 업그레이드, graceful degradation, PM Agent 재정의) |
