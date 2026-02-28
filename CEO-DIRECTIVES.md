# CEO DIRECTIVES – AADS (Autonomous AI Development System)
> 최종 업데이트: 2026-02-28 (v2.0)
> 관리자: CEO (moongoby)
> 용도: 모든 AI 세션에서 필수 읽기. 이 문서의 지시를 위반하는 설계/분석은 무효.

---

## 1. 사고방식 원칙

### D-001 단순 사고 금지
- "하나를 던지면 10을 생각하고 연구해서 반영하라"
- 단일 변수, 단일 시점, 단일 관점 분석은 불충분
- 복합계, 다층 구조, 다시점 분석이 기본

### D-002 사소한 것도 빠짐없이
- 조사·분석 시 "정말 사소한 것도 빠짐없이" 정리
- 모든 모델, 도구, 옵션을 누락 없이 비교
- 나중에 필요할 수 있는 정보는 미리 확보

### D-003 AADS의 본질
- AADS는 "코드를 생성하는 AI"가 아니다
- AADS는 "AI 에이전트들이 조직처럼 협업하여 소프트웨어를 자율 개발하는 시스템"
- CEO→PM→개발자→QA→운영의 역할 분리가 핵심
- 단일 AI 앱 빌더(Lovable, Bolt 등)와의 근본적 차이: 멀티 에이전트 조직 시뮬레이션
- **AADS 자체가 프로덕트** — 먼저 완성·배포, 이후 HealthMate 등 프로젝트 생성

### D-004 비용 효율 최우선
- 최저 비용으로 최대 품질 달성이 설계 원칙
- 모델 라우팅: 단순 태스크→nano/flash($0.05~0.40/1M), 복잡 태스크→Opus/Pro($5~25/1M)
- 오픈소스 우선: 상용 SaaS 대체 가능하면 오픈소스 적극 활용
- 월 인프라 비용 $500 이하 유지 목표 (프로토타이핑 단계)
- 프롬프트 캐싱(최대 90% 절감) + 배치 API(50% 절감) 적극 활용

### D-005 컨텍스트 패키지 시스템
- 모든 AI 세션에 HANDOVER.md + CEO-DIRECTIVES.md + 설계문서 필수 읽기
- 이전 맥락 없이 작업하면 같은 실수 반복
- 매 작업 완료 시 HANDOVER.md 업데이트 의무
- 세션이 끊겨도 맥락이 이어져야 함 — HANDOVER.md가 생명줄

### D-006 교차검증 필수
- 작업 결과를 무비판적으로 수용하지 말 것
- 수치와 결론의 논리적 일관성 확인
- CEO-DIRECTIVES의 지시가 반영되었는지 검증
- 누락된 분석이나 요청했으나 빠진 항목 지적

### D-007 도구 vs 에이전트 구분
- Cursor, Windsurf = 사람(CEO/개발자)이 직접 쓰는 생산성 도구 (GUI, 시스템 자동화 불가)
- AADS 내부 에이전트 = Claude API + MCP 서버 + 샌드박스로 프로그래밍 방식 자동화
- 이 구분을 혼동하지 말 것

### D-008 속도 우선
- AADS 완성 → 서비스 배포까지 빠르게 진행
- 완벽보다 동작하는 것이 먼저
- MVP 먼저 배포, 이후 반복 개선

### D-009 현실적 자율성 인식
- 멀티 에이전트 LLM 시스템 프로덕션 실패율 41–87% (arXiv 2503.13657, 2026-02)
- 79%의 실패는 스펙 모호성 + 에이전트 간 조정 실패 (인프라 문제 아님)
- 완전 자율은 환상 — 점진적 자율성 확대가 현실적 전략
- 단계별 접근: Level 1(소규모 코딩 보조, ≥80%) → Level 2(중규모 기능 + HITL, 60-70%) → Level 3(앱 생성, 장기 목표)

### D-010 사용자 중심 체크포인트
- AADS는 단일 프롬프트 빌더가 아니라 다단계 협업 시스템
- 사용자(CEO)가 핵심 의사결정 지점에서 개입할 수 있어야 함
- 6단계 사용자 체크포인트: 요구사항 대화 → 기획서 승인 → 설계 확인 → 자율 개발 → 중간 확인 → 최종 승인·배포
- 이 체크포인트가 경쟁사(Bolt, Lovable)와의 핵심 차별점

---

## 2. 기술적 지시

### T-001 멀티 에이전트 아키텍처
- **프레임워크**: LangGraph >= 1.0.8 (최신 1.0.8, 2026-02-19 릴리스)
- **패턴**: **Native Tool-Based Supervisor** (LangGraph StateGraph 직접 구현)
- `langgraph-supervisor` 라이브러리는 MCP 루프 버그(GitHub #249, 미해결)로 프로덕션 사용 금지. 프로토타입 참고만 허용
- **상태 관리**: PostgreSQL AsyncPostgresSaver + Supabase 직접 연결(port 5432)
- Supavisor/PgBouncer 경유 금지 (AsyncPipeline 충돌)
- **Human-in-the-Loop**: 배포, DB 변경, 외부 API 호출 등 고위험 작업에 승인 게이트
- 3단계 이상 계층은 오버헤드 → 2단계로 제한

### T-002 에이전트 역할 구성 (2026-02-28 검증 가격)

| 에이전트 | 역할 | 추천 모델 (model ID) | 입력/출력 $/1M | 대안 모델 (model ID) | 입력/출력 $/1M |
|----------|------|---------------------|---------------|---------------------|---------------|
| Supervisor | 오케스트레이션, 태스크 분배·합성 | Claude Opus 4.6 (`claude-opus-4-6`) | $5 / $25 | Gemini 3.1 Pro (`gemini-3.1-pro-preview`) | $2 / $12 |
| Architect | 시스템 설계, 기술 의사결정 | Claude Opus 4.6 (`claude-opus-4-6`) | $5 / $25 | Gemini 3.1 Pro (`gemini-3.1-pro-preview`) | $2 / $12 |
| PM | 사용자 대화, 구조화 JSON 스펙 생성, 체크포인트 관리 | Claude Sonnet 4.6 (`claude-sonnet-4-6`) | $3 / $15 | GPT-5.2 Chat (`gpt-5.2-chat-latest`) | $1.75 / $14 |
| Developer | 코드 생성·수정·리팩터 | Claude Sonnet 4.6 (`claude-sonnet-4-6`) | $3 / $15 | GPT-5.3 Codex (`gpt-5.3-codex`) | $1.75 / $14 |
| QA | 테스트 생성·실행·버그 리포트 | Claude Sonnet 4.6 (`claude-sonnet-4-6`) | $3 / $15 | GPT-5 mini (`gpt-5-mini`) | $0.25 / $2 |
| Judge | 독립 출력 검증, 스펙 대비 코드 정합성 평가 | Claude Sonnet 4.6 (`claude-sonnet-4-6`) | $3 / $15 | Gemini 3.1 Pro (`gemini-3.1-pro-preview`) | $2 / $12 |
| DevOps | CI/CD, 배포, 모니터링 | GPT-5 mini (`gpt-5-mini`) | $0.25 / $2 | Claude Haiku 4.5 (`claude-haiku-4-5`) | $1 / $5 |
| Researcher | 데이터 수집·분석·웹 검색 | Gemini 2.5 Flash (`gemini-2.5-flash`) | $0.30 / $2.50 | GPT-5 nano (`gpt-5-nano`) | $0.05 / $0.40 |

**주의**: GPT-5.2 Pro (`gpt-5.2-pro`, $21/$168)는 극고가이므로 사용 금지. Architect 대안은 Gemini 3.1 Pro ($2/$12) 사용.

### T-003 MCP 서버 스택
- **Phase 1 필수 (7개)**: Filesystem, Git/GitHub, PostgreSQL, Brave Search, Memory, Supabase, Fetch
- **Phase 2 확장 (3개)**: Puppeteer(브라우저 자동화), Sentry(에러 추적), Slack(팀 알림)
- **프로세스 관리**: FastAPI lifespan에서 `MultiServerMCPClient` 1회 초기화, 앱 수명 내내 유지
- **상시 가동 4개**: Filesystem, Git, Memory, PostgreSQL (~200-400MB)
- **온디맨드 3개**: GitHub, Brave Search, Fetch (필요 시만 스폰, 메모리 절약)
- **전송 방식**: stdio 기본, SSE/HTTP는 원격 MCP 서버에만 사용
- **인증**: 환경 변수로 토큰 주입

### T-004 샌드박스 & 배포 인프라
| 용도 | 도구 | 비용 |
|------|------|------|
| 에이전트 코드 실행 | E2B Pro | $150–250/월 (사용량 기반, 초기 $100 무료 크레딧) |
| GPU 태스크 | Modal | 종량제 (Phase 2+) |
| API 서버 배포 | Fly.io (shared-cpu-2x, **2GB RAM**) | ~$20–50/월 |
| 프론트엔드 배포 | Vercel Pro | $20/월 |
| DB + Auth | Supabase Pro (**직접 연결 port 5432**) | $25–40/월 |
| 캐시 | Upstash Redis | $0–10/월 |
| MCP 서버 | Fly.io 동일 컨테이너 내 | 포함 |

### T-005 AADS 기술 스택
- **오케스트레이션**: LangGraph >= 1.0.8 (Native StateGraph, `langgraph-supervisor` 사용 금지)
- **AI API**: Anthropic Claude API (메인), OpenAI API (보조), Google Gemini API (데이터)
- **MCP**: `langchain-mcp-adapters` + `MultiServerMCPClient`
- **샌드박스**: E2B (Sandbox-as-Tool 패턴)
- **상태 저장**: PostgreSQL + `langgraph-checkpoint-postgres` (AsyncPostgresSaver, Supabase 직접 연결)
- **캐싱**: Upstash Redis
- **관측성**: LangSmith Free Tier (5K traces/월) — 초과 시 Langfuse OSS 전환 검토
- **프론트엔드**: Next.js (latest stable) + React + Tailwind CSS
- **백엔드 API**: FastAPI (Python 3.11+) + uvloop
- **배포**: Fly.io (API) + Vercel (프론트)
- **CI/CD**: GitHub Actions → Fly.io 자동 배포 + health-check rollback

### T-006 로드맵 (점진적 자율성 확대 전략)
| Phase | 내용 | 자율성 Level | 예상 성공률 | 상태 |
|-------|------|-------------|-----------|------|
| Phase 0 | 리서치 + 아키텍처 설계 + 인계서 시스템 | — | — | **완료** |
| Phase 1 | AADS 코어 — LangGraph Native Supervisor + 8 에이전트 + MCP + E2B + HITL | Level 1 | ≥80% | **착수 가능** |
| Phase 1.5 | 중규모 기능 + 빈번한 HITL 체크포인트 | Level 2 | 60–70% | 대기 |
| Phase 2 | AADS 대시보드 + SSE 스트리밍 + 사용자 체크포인트 UI | Level 2 | — | 대기 |
| Phase 3 | AADS 서비스 배포 — 외부 사용자, 멀티테넌시 | Level 2→3 | — | 대기 |
| Phase 4 | 자율 개선 루프 | Level 3 | — | 대기 |
| Phase 5 | 첫 번째 프로젝트: HealthMate | Level 2–3 | — | 대기 |
| Phase 6 | 범용화 — SaaS 확장 | Level 3 | — | 대기 |

### T-007 에이전트 간 통신 프로토콜
- 에이전트 간 작업 위임은 **구조화 JSON 태스크 스펙(TaskSpec)**으로 수행
- PM Agent가 CEO 고수준 명령을 구조화 JSON으로 변환
- TaskSpec 필수 필드: `task_id`, `description`, `assigned_agent`, `dependencies`, `success_criteria`, `constraints`, `max_iterations`
- 자연어 메시지는 히스토리에 보존하되, 핵심 지시는 TaskSpec으로 전달

### T-008 Judge Agent 및 품질 게이트
- Judge Agent는 Developer/QA 산출물을 독립 검증 (별도 컨텍스트)
- TaskSpec의 `success_criteria`와 최종 산출물만 비교하여 판정
- 판정: pass / fail / conditional_pass
- fail 시 Supervisor가 Developer에게 구체적 피드백과 함께 재작업 지시
- 교차 검증을 위해 Judge는 Developer/QA와 다른 모델 사용 권장

### T-009 점진적 자율성 게이트
- 에이전트/태스크 유형별 성공률 추적 (PostgreSQL 누적 저장)
- Judge 통과율 90% 이상 (최근 50건) → 해당 유형 자동 승인 전환
- 사용자 수정 요청 비율 10% 이하 → 체크포인트 간소화 가능
- 성공률 70% 미만 하락 시 HITL 게이트 재활성화
- 최소 20건 데이터 필요, 그 전에는 항상 HITL 유지

---

## 3. 절대 규칙 (위반 시 작업 무효)

- R-001: HANDOVER.md 업데이트 없이 작업 완료 선언 금지
- R-002: 보고서는 반드시 GitHub push + HTTP 200 확인
- R-003: .env / 시크릿 / API 키 커밋 금지
- R-004: 프로덕션 DB 직접 편집 금지 (마이그레이션 스크립트로만)
- R-005: 서버 서비스 임의 재시작 금지
- R-006: CEO-DIRECTIVES.md의 지시를 무시하는 설계/분석은 전부 무효
- R-007: 기존 서비스(aads-server 파이프라인) 중단/변경 시 반드시 CEO 승인
- R-008: CEO에게 파일을 보고할 때는 반드시 GitHub **브라우저 경로** 사용. 형식: `https://github.com/moongoby-GO100/aads-docs/blob/main/{파일경로}`. `raw.githubusercontent.com` URL은 HTTP 200 검증 용도로만 내부 사용, CEO 보고에는 절대 포함 금지.
- R-009: 작업 완료 후 CEO 보고 형식:
```
푸시 완료했습니다.
- [파일명]: https://github.com/moongoby-GO100/aads-docs/blob/main/{경로}
- HANDOVER: https://github.com/moongoby-GO100/aads-docs/blob/main/HANDOVER.md
```
- R-010: `langgraph-supervisor` 라이브러리는 MCP 루프 버그(GitHub #249)로 프로덕션 코드에 사용 금지. 프로토타입 참고만 허용.
- R-011: Supabase 연결 시 반드시 직접 연결(port 5432) 사용. Supavisor/PgBouncer(port 6543) 경유 금지 (AsyncPostgresSaver pipeline 충돌).

---

## 4. 버전 이력

| 버전 | 날짜 | 변경 |
|------|------|------|
| v1.0 | 2026-02-28 | 초판 — D-001~D-008, T-001~T-006, 절대 규칙 |
| v1.1 | 2026-02-28 | 절대 규칙 추가: R-NEW-1 브라우저 URL 보고, R-NEW-2 완료 보고 형식 |
| v2.0 | 2026-02-28 | 대규모 개정 — 21건 수정사항 반영. D-009/D-010 추가, T-001~T-006 전면 수정(가격 정정, LangGraph 1.0.8, Native Supervisor, Judge Agent, 구조화 JSON, 점진적 자율성), T-007~T-009 신규, R-001~R-011 정리 |
