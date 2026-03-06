# CEO DIRECTIVES – AADS (Autonomous AI Development System)
> 최종 업데이트: 2026-03-06 (v2.8)
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
- 월 인프라 비용 $23~$63/월 목표 (프로토타이핑 단계)
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

### D-011: Sandbox 2단계 전략
- 기본: Docker 로컬 컨테이너 (python:3.12-slim, node:20-slim)
- 제한: 512MB RAM, 1 CPU, 네트워크 차단, 최대 5동시
- 대형 프로젝트: 사용자 서버 SSH/Docker API (Phase 3)
- E2B: 선택사항 (멀티테넌트 SaaS 확장 시)

### D-012: 인프라 컨설팅 서비스
- Architect Agent가 프로젝트 규모 분석 → 서버 사양 제안
- 추천 서버: Contabo, Hetzner, Vultr 등 가성비 우선

### D-013: Frontend Dual Strategy
- 2026년: Genspark AI 채팅 (무료) + AADS Dashboard 병행
- 브릿지: Genspark ↔ AADS API 연결
- Chat Endpoint: /api/v1/chat (자연어 → 액션 라우터)
- 2027년: 자체 채팅 UI 완성 후 Genspark은 선택적 채널

### D-014: 콘텐츠 품질 게이트 (2026-03-04 확정)
- **모든 자동 배포 콘텐츠는 벤치마크 85% 이상 필수** (match_percent ≥ 85%)
- 판정 기준:
  - AUTO_PUBLISH (85%+): 즉시 업로드 허용
  - CONDITIONAL (70-84%): CEO 리뷰 후 배포 결정
  - AUTO_REJECT (<70%): 자동 보정 파라미터 생성 후 재렌더링 지시 (max_retries=2)
- 적용 대상: ShortFlow 영상 (economy, finance, tech 등 채널), 향후 모든 자동 생성 콘텐츠
- 품질 게이트 API: `POST /api/v1/visual-qa/quality-gate` (AADS 68서버)
- ShortFlow 연동: `shortflow_quality_gate.sh` (211서버 cron 업로드 직전 호출)
- 벤치마크 사양: 채널별 기준 영상 프레임 등록 → system_memory 저장 (category: benchmark_specs)
- 자동 보정: CORRECTION_MAP 6항목 FFmpeg 파라미터 delta 적용 (fontsize, box_alpha, margin, resolution, color_grading 등)
- CEO 알림: CONDITIONAL 판정 시 텔레그램 + Context API qa_notifications 저장

### D-015: 68서버 프로덕션 유지 결정 (2026-03-05 확정)
- **68서버 (aads.newtalk.kr) 유지**: 옵션 A 채택 — 현 서버에서 프로덕션 운영 계속
- **도메인 aads.newtalk.kr 고정**: 변경 금지. 모든 내외부 연동에 이 도메인 사용
- **Fly.io 보류**: Phase 3 SaaS 확장 시점에 재평가. MVP/현재 단계에서는 Fly.io 사용 금지
- 근거: 추가 비용 $0, 현 인프라 안정성 확인, 68서버 docker-compose.prod.yml 기반 운영
- 프로덕션 강화 내용 (T-032):
  - docker-compose.prod.yml: restart always, json-file logging, memory limit 1.5G, pg shared_buffers=256MB
  - nginx: rate_limit 60r/m, gzip, proxy timeout 120s/10s, 보안헤더(X-Frame/X-Content-Type/X-XSS/CORS)
  - systemd: aads.service (oneshot docker compose)
  - backup.sh: 매일 03:00 cron, 7일 자동삭제

### D-016: FLOW 프레임워크
- 모든 신규 작업은 Find→Layout→Operate→Wrap up 4단계를 따른다.
- 소규모 버그수정/설정변경은 Operate→Wrap up만 수행 가능.
- 각 단계 산출물에 parent 필드로 선행 산출물을 참조한다.

---

## 2. 기술적 지시

### T-001 멀티 에이전트 아키텍처
- **프레임워크**: LangGraph >= 1.0.10 (최신 1.0.10, 2026-02-27 릴리스)
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
- **프로세스 관리**: supervisord로 MCP 서버 프로세스 관리, FastAPI lifespan에서 `MultiServerMCPClient` 초기화
- **상시 가동 4개**: Filesystem, Git, Memory, PostgreSQL (~200-400MB)
- **온디맨드 3개**: GitHub, Brave Search, Fetch (필요 시만 스폰, 메모리 절약)
- **전송 방식**: supervisord 환경에서는 SSE transport (localhost), 직접 호출 시 stdio. 원격 MCP 서버는 SSE/HTTP
- **인증**: 환경 변수로 토큰 주입

### T-004 샌드박스 전략 (2단계 하이브리드, CEO 확정 2026-03-03)
- **소형 프로젝트**: 자체 서버 Docker 컨테이너 실행 (비용 $0)
  - Docker SDK (`docker` Python 패키지)로 컨테이너 생성→실행→결과수집→삭제
  - 보안: --network=none, --memory=512m, --cpus=1, --read-only, tmpfs /tmp 100M
  - 동시 샌드박스 최대 5개
- **대형 프로젝트**: 사용자 서버 원격 실행 (AADS가 최적 서버+비용 자동 제안, 사용자 승인 후 SSH/Docker API 접속)
- **E2B**: SaaS 확장 시(Phase 3+) 옵션. MVP에서는 사용 안 함.

인프라 비용표:
| 용도 | 도구 | 비용 |
|------|------|------|
| 소형 코드 실행 | Docker 로컬 샌드박스 | $0 |
| 대형 코드 실행 | 사용자 서버 SSH | $0 (사용자 부담) |
| API 서버 | Contabo Docker Compose | $12.99/월 |
| DB | 로컬 PostgreSQL (Docker) | $0 |
| 캐시 | 로컬 Redis (Docker) | $0 |

### T-005 AADS 기술 스택
- **오케스트레이션**: LangGraph >= 1.0.10 (Native StateGraph, `langgraph-supervisor` 사용 금지)
- **AI API**: Anthropic Claude API (메인), OpenAI API (보조), Google Gemini API (데이터)
- **MCP**: `langchain-mcp-adapters` + `MultiServerMCPClient`
- **샌드박스**: Docker 로컬 (D-011, 소형 기본) / SSH 원격 (대형, Phase 3) / E2B (Phase 3+ 옵션)
- **상태 저장**: PostgreSQL + `langgraph-checkpoint-postgres` (AsyncPostgresSaver, Supabase 직접 연결)
- **캐싱**: Upstash Redis
- **관측성**: LangSmith Free Tier (5K traces/월) — 초과 시 Langfuse OSS 전환 검토
- **프론트엔드**: Next.js (latest stable) + React + Tailwind CSS
- **백엔드 API**: FastAPI (Python 3.12+) + Uvicorn
- **배포**: 68서버 Docker Compose (API, MVP) + Vercel (프론트) → SaaS 확장 시 Fly.io 이전
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
- TaskSpec 필수 필드: `task_id`, `parent_task_id`, `description`, `assigned_agent`, `success_criteria`, `constraints`, `input_artifacts`, `output_artifacts`, `max_iterations`, `max_llm_calls`, `budget_limit_usd`, `status`
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

### T-010 수익 모델 (신규, 2026-03-03)
| 수익원 | 방식 | 예상 |
|--------|------|------|
| 개발비 | 프로젝트당 과금/월 구독 | $19~$199/월 |
| 인프라 추천 | 제휴 커미션 | 서버 비용 5~15% |
| 유지보수 | 배포 후 모니터링 구독 | $9~$29/월 |

SaaS 티어:
| 티어 | 내용 | 가격 |
|------|------|------|
| Free | Docker 소형 월3건 | $0 |
| Starter | Docker 소형 월10건 | $19/월 |
| Pro | 소형+사용자서버 연결 | $49/월 |
| Enterprise | 전용 서버, 전담 지원 | 커스텀 |

### T-011: 5-Layer Memory Architecture
- L1: Working Memory (AADSState + AsyncPostgresSaver)
- L2: Project Memory (project_memory 테이블)
- L3: Experience Memory (experience_memory + pgvector)
- L4: System Memory (system_memory, HANDOVER 대체)
- L5: Procedural Memory (procedural_memory)

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
푸시 완료했습니다.

HANDOVER: https://github.com/moongoby-GO100/aads-docs/blob/main/HANDOVER.md
- R-010: `langgraph-supervisor` 라이브러리는 MCP 루프 버그(GitHub #249)로 프로덕션 코드에 사용 금지. 프로토타입 참고만 허용.
- R-011: Supabase 연결 시 반드시 직접 연결(port 5432) 사용. Supavisor/PgBouncer(port 6543) 경유 금지 (AsyncPostgresSaver pipeline 충돌).
- R-012: 작업당 LLM 호출 최대 15회. 초과 시 자동 중단 후 Supervisor에 에스컬레이션.

### R-013: Task ID 접두사 체계
- 신규 지시서는 반드시 프로젝트 접두사 사용
- AADS-xxx, KIS-xxx, GO100-xxx, SF-xxx, NT-xxx, SALES-xxx, NAS-xxx
- 번호는 프로젝트별 마지막 번호+1 (AADS: 107~, KIS: 168~, GO100: 038~)
- T-xxx 레거시 ID는 읽기 전용으로 유효, 신규 발행 금지

### R-014: Wrap up 의무화
- P0/P1: WRAP 결과 파일 필수. 미완료 시 다음 작업 진입 차단.
- P2(15분 초과): 최소 5분 모니터링 + HTTP 200 확인.
- P2(15분 이하)/P3: claude_exec.sh 자동 health-check. 실패 시 WRAP 자동 생성.

### R-015: 교훈 등록
- Wrap up 시 다른 프로젝트에도 적용 가능한 교훈이 있으면 shared/lessons/에 등록.
- 결과 파일에 ## 교훈 섹션 작성 시 API가 자동 등록.

---

## 4. Genspark CEO 통합지휘 대화 규칙

### 9-1. 작업 완료의 정의
- "완료"란 아래 4가지가 모두 충족된 상태만을 의미한다:
  1. 로컬 파일 수정 완료
  2. `git add -A && git commit && git push origin main` 성공
  3. `curl` HTTP 200 확인
  4. HANDOVER.md 업데이트 완료
- 위 4가지 중 하나라도 미충족이면 "작업 진행중"으로 보고한다
- push 전에 "완료"라고 보고하는 것은 금지

### 9-2. 매니저 대화창 보고 형식 (필수)
매니저 대화창에 보고할 때 반드시 아래 형식을 사용한다:

```
[CURSOR-AADS] {상태}
작업: {작업 내용 1줄 요약}
보고서: https://github.com/moongoby-GO100/aads-docs/blob/main/reports/{파일명}
커밋: https://github.com/moongoby-GO100/aads-docs/commit/{SHA} 또는 https://github.com/moongoby-GO100/aads-server/commit/{SHA}
HTTP: {200|실패}
HANDOVER: {업데이트 완료|미완료}
다음: {다음 작업 또는 "지시 대기"}
```

`{상태}` 값: `push 완료` | `작업 진행중` | `문제 발생` | `지시 대기`

### 9-3. 파일명 규칙
- 보고서: `{TASK_NAME}-{SEQ}.md` (예: CAPABILITY-MAP-001.md, INFRA-STRATEGY-001.md)
- 보고서 저장 경로: `/root/aads/aads-docs/reports/`
- 연구: `{PROJECT}-FIND-{SEQ}_{제목}.md`
- 설계: `{PROJECT}-LAYOUT-{SEQ}_{제목}.md`
- 실행: `{PROJECT}-{SEQ}_{제목}.md` (기존 유지)
- 검증: `{PROJECT}-WRAP-{SEQ}_{제목}.md`
- 교훈: `L-{SEQ}_{제목}.md`

### 9-4. push 전 필수 체크리스트
모든 commit/push 전에 아래 순서를 반드시 실행한다:
```bash
cd /root/aads/aads-docs  # 또는 /root/aads/aads-server
git add -A && git diff --cached --stat
git commit -m "[AADS] {type}: {설명}"
git push origin main
sleep 5
curl -s -o /dev/null -w "%{http_code}" https://raw.githubusercontent.com/moongoby-GO100/aads-docs/main/{파일경로}
```

### 9-5. Directive 블록 파싱
매니저 대화창의 AI가 `>>>DIRECTIVE_START ~ >>>DIRECTIVE_END` 블록으로 지시를 내린다.
- 이 블록 안의 내용을 순서대로 실행한다
- 실행 불가능한 항목이 있으면 실행하지 않고 사유를 보고한다
- 블록 외부의 내용은 맥락 정보이며 실행 대상이 아니다

### 9-6. 세션 시작/복원 프로토콜
매니저 대화창에 첫 메시지를 보낼 때 아래 형식을 사용한다:
```
[CURSOR-AADS] 세션 시작
- HANDOVER: https://raw.githubusercontent.com/moongoby-GO100/aads-docs/main/HANDOVER.md
- CEO-DIRECTIVES: https://raw.githubusercontent.com/moongoby-GO100/aads-docs/main/CEO-DIRECTIVES.md
- 최근 완료 3건: {최근 3건}
- 진행중: {있으면 기재}
- 착수 가능: {다음 작업}
- 보류: {보류 항목}
- 작업 지시 대기
```

### 9-7. 승인 권한 위임
- **CEO 직접 승인 필요**: 유료 결제/과금, 프로덕션 서비스 중단, 외부 서비스 계약, CEO-DIRECTIVES 절대 규칙 변경
- 위 항목 외 모든 승인은 매니저 대화창 AI의 DIRECTIVE 블록 = CEO 승인으로 간주하고 즉시 실행
- 매니저 AI가 "CEO 승인 필요"라고 명시한 경우에만 CEO에게 직접 확인

### 9-8. 매니저 대화창 라우팅
| 작업 대상 | 보고 대화창 |
|----------|------------|
| AADS | [AADS] 프로젝트 매니저 `https://www.genspark.ai/agents?id=3d86d6f3-09a7-41b2-b91b-762a55512458` |
| CEO 통합지휘소 | bridge.py 자동 발송 전용 (Cursor 수동 보고 금지) |

---

## 5. 버전 이력

| 버전 | 날짜 | 변경 |
|------|------|------|
| v1.0 | 2026-02-28 | 초판 — D-001~D-008, T-001~T-006, 절대 규칙 |
| v1.1 | 2026-02-28 | 절대 규칙 추가: R-NEW-1 브라우저 URL 보고, R-NEW-2 완료 보고 형식 |
| v2.0 | 2026-02-28 | 대규모 개정 — 21건 수정사항 반영. D-009/D-010 추가, T-001~T-006 전면 수정(가격 정정, LangGraph 1.0.8, Native Supervisor, Judge Agent, 구조화 JSON, 점진적 자율성), T-007~T-009 신규, R-001~R-011 정리 |
| v2.1 | 2026-03-01 | LangGraph 1.0.10 상향, MCP SSE transport 반영, LLM 호출 한도 15회 명시(R-012), T-007 TaskSpec 필드 12개로 확장, 6건 불일치 해소 |
| v2.4 | 2026-03-04 | D-011~D-013 추가(Sandbox 2단계, 인프라 컨설팅, Frontend Dual Strategy), T-010~T-011 추가(수익 모델, 5-Layer Memory), D-004 비용 $23~$63 변경 |
| v2.3 | 2026-03-02 | T-004 배포 전략 변경 (Fly.io → 68서버 Docker Compose, $0 추가, MVP 단계) |
| v2.2 | 2026-03-02 | Genspark 통합지휘 규칙 섹션 4 추가 (9-1~9-8), Directive 블록 파싱, 보고 형식 표준화 |
| v2.6 | 2026-03-05 | D-015 추가 — 68서버 프로덕션 유지 결정: 옵션 A (68서버 유지), 도메인 aads.newtalk.kr 고정, Fly.io Phase 3 SaaS 확장 시 재평가. T-032 프로덕션 강화 완료. |
| v2.5 | 2026-03-04 | D-014 추가 — 콘텐츠 품질 게이트: 자동 배포 콘텐츠 벤치마크 85% 이상 필수, ShortFlow 연동(shortflow_quality_gate.sh), AUTO_PUBLISH/CONDITIONAL/AUTO_REJECT 판정 기준 |
| v2.7 | 2026-03-06 | AADS-107: R-013 Task ID 접두사 체계 등록 — 프로젝트별 독립 넘버링(AADS/KIS/GO100/SF/NT/SALES/NAS), T-xxx 레거시 신규 발행 금지 |
| v2.8 | 2026-03-06 | D-016 FLOW 프레임워크, R-014 Wrap up 의무화, R-015 교훈 등록, 9-3 산출물 파일명 확장, 버전 이력 시간순 정렬 |
