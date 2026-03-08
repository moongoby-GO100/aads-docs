# CEO DIRECTIVES – AADS (Autonomous AI Development System)
> 최종 업데이트: 2026-03-08 (v3.5)
> 관리자: CEO (moongoby)
> 용도: 모든 AI 세션에서 필수 읽기. 이 문서의 지시를 위반하는 설계/분석은 무효.

---

## 0. 운영 원칙

### 이 문서의 운영 원칙
- 이 문서는 토큰 상한이 없다. 비용을 아끼지 말고 최신화하라.
- 중요 내용은 빠짐없이 반영하라. 생략 금지.
- CEO 질문 0회 기준: 이 문서를 읽고 추가 질문 없이 즉시 업무 수행 가능해야 한다.

### 매니저 자기인식 의무
- 모든 매니저는 세션 시작 시 자신의 정체성(어느 프로젝트 전담 매니저인지)과 채팅창 일치 여부를 검증해야 한다.
- 검증 항목: ① 프로젝트명, ② Task ID 접두사, ③ 참조 문서 URL
- 불일치 시 경고 출력 후 작업 수행 금지.

### bridge.py 자동화 인지 의무
- 매니저는 지시서를 채팅창에 출력만 하면 자동 전달됨을 이해해야 한다.
- bridge.py가 >>>DIRECTIVE_START ~ >>>DIRECTIVE_END 블록을 자동 감지·추출·저장한다.
- 매니저가 CEO에게 "이 지시서를 전달해 주세요", "bridge에 넣어 주세요" 등 전달을 요청하는 것은 금지한다.

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

### D-017: 소스코드 모듈화 원칙 (AADS-130, 2026-03-06)
- 모든 에이전트 산출물은 `project_artifacts` 테이블에 통합 저장한다.
- 에이전트별 로직은 `app/agents/` 독립 파일로 분리한다.
- Pydantic 모델은 `app/models/` (strategy.py, plan.py, artifact.py)로 분리한다.
- 공통 DB 기록은 `app/services/db_recorder.py`를 통해 수행한다.
- 각 모듈은 독립적으로 import/단위테스트 가능해야 한다.

### D-018: 4계층 자기치유 원칙 (AADS-134, 2026-03-06; AADS-140~142, 2026-03-07)
- L1(프로세스 자체 방어) → 하트비트 기반 진행 감시. 파일 변경/커밋/테스트 이벤트마다 하트비트 발신. session_watchdog가 10초 주기로 감시. 60초 미갱신 시 경고, 120초 시 진단+조건부 kill, 300초 시 강제 종료. 하드 타임아웃 2시간은 최종 안전망으로 유지.
- L1.5(session_watchdog) → L2(핵심 감시자) → L3(메타 감시자) → L4(외부 감시) 4계층 구조를 의무화한다.
- 감시자를 감시하는 상위 계층이 반드시 존재해야 한다. 단일 watchdog는 자기 자신의 장애를 감지할 수 없다.
- 복구 간 의존성은 그래프(recovery_graph)로 관리하며, 3단계 에스컬레이션을 적용한다.

### D-019: 서버 상호 감시 의무화 (AADS-134, 2026-03-06)
- 3서버(211, 68, 114)는 서로를 2분 주기로 크로스 모니터링한다.
- 어느 1대 장애 시 나머지 2대가 감지·복구·알림을 수행한다.
- 삼각형 감시 토폴로지: 211↔68↔114↔211. 단일 장애점 제거.

### D-020: 복구 이력 DB 의무화 (AADS-134, 2026-03-06)
- 모든 자동복구 시도는 `recovery_logs` 테이블에 project_id 포함하여 기록한다.
- 주간 단위로 복구 통계(/ops/recovery)를 리뷰하고, 반복 이슈는 근본 해결한다.
- circuit_breaker_state 테이블로 서킷브레이커 상태를 영속 관리한다.

### D-021: 하트비트 기반 세션 관리 (AADS-140~142, 2026-03-07)
- 모든 claude_exec 세션은 하트비트를 발신해야 한다 (inotifywait 또는 git status 기반).
- session_watchdog가 10초 주기로 진행 여부를 감시한다.
- 작업 완료 시 즉시 슬롯 해제 + 다음 작업 투입 (이벤트 기반).
- 시맨틱 루프(동일 패턴 10회 반복)는 Tier 2에서 감지·kill한다.
- 고정 타임아웃을 예측하지 말고, 진행을 관찰하라.

### D-022: 지시서 포맷 v2.0 (AADS-144, 2026-03-07)
- **필수 필드**: task_id / project / priority / size / description / success_criteria
- **선택 필드** (생략 시 기본값 적용):
  - parallel_group: 병렬 실행 그룹명 (기본: 없음)
  - files_owned: 작업 대상 파일 목록 (기본: 없음)
  - impact: H / M / L (기본: M) — 비즈니스 영향도
  - effort: H / M / L (기본: M) — 구현 비용/난이도
  - model: claude 모델명 (기본: sonnet) — D-024 라우팅 참조
  - review_required: true / false (기본: false)
  - subagents: 사용할 서브에이전트 목록 (기본: 없음)
- **기본값 규칙**: 선택 필드 생략 시 claude_exec.sh가 기본값 적용하여 실행

### D-023 v2: HANDOVER 4계층 운영 원칙 (AADS-148, 2026-03-08)
- 1. 토큰 상한을 두지 않는다. Core(HANDOVER.md)와 Rules(HANDOVER-RULES.md)에 토큰 제한을 적용하지 않는다.
- 2. 비용을 아끼지 말고 최신화하라. 토큰 절약을 이유로 정보를 생략하거나 축약하지 않는다.
- 3. 중요 내용은 빠짐없이 반영하라. "이 정도는 알겠지"라고 가정하지 않는다.
- 4. 성과 기준은 "CEO 질문 0회"이다.
- 5. 불필요한 중복과 장황함만 제거한다. 정보 자체를 줄이는 것은 금지한다.
- **4계층**: Core(HANDOVER.md) + Rules(HANDOVER-RULES.md) + History(HANDOVER-HISTORY.md) + Archive(HANDOVER-ARCHIVE.md)
- Core와 Rules는 매 세션 필수 읽기. History/Archive는 필요 시 참조.
- 버전 이력: Core에 최근 10건 유지, 11건째부터 History로 이동.
- History: 최근 완료 작업 10건 유지, 11건째부터 Archive로 이동.

### D-024: 모델 라우팅 기준 (AADS-144, 2026-03-07)
- **XS** (단순 문서 수정, 1파일 이하): `claude-haiku-4-5` — 최저비용
- **S** (소규모 수정, 1~3파일): `claude-sonnet-4-6` — 기본
- **M** (중규모 기능, 다중 파일): `claude-sonnet-4-6` — 기본
- **L** (대규모 기능, 시스템 설계): `claude-sonnet-4-6` 또는 `claude-opus-4-6`
- **XL** (전체 시스템, E2E): `claude-opus-4-6` — 최고품질
- 지시서 `model:` 필드로 오버라이드 가능. 미지정 시 size 기반 자동 라우팅.
- claude_exec.sh가 지시서 `model:` 필드 → size 기반 자동 선택 → fallback sonnet 순서 적용.

### D-025: 우선순위큐 impact/effort 정렬 (AADS-144, 2026-03-07)
- 동일 priority 내 태스크는 impact/effort 점수로 정렬한다.
- **impact 점수**: H=3, M=2, L=1
- **effort 점수**: H=1, M=2, L=3 (낮은 effort가 높은 점수 — 빠른 완료 우선)
- **정렬 점수** = impact_score × 10 + effort_score
- 점수 높은 태스크를 먼저 실행 (P0 내에서 H-impact/L-effort 우선).
- impact/effort 미지정 시 기본값 M/M (점수=22) 적용.

### D-030: QA 에이전트 의무화 (AADS-163, 2026-03-08)
- **모든 claude_exec 작업 완료 후 QA 에이전트 자동 실행 의무화**
- QA 에이전트: test-writer 서브에이전트 (`/root/aads/.claude/agents/test-writer.md`)
- 실행 순서: Claude 코드 수정 완료 → QA 에이전트 → (PASS) → 디자인 에이전트 → RESULT_FILE 생성
- QA FAIL 시: 자동 재작업 최대 2회 → 재QA → 2회 초과 시 FAIL 보고 + 서킷브레이커 카운트+1
- RESULT_FILE에 `qa_status: PASS | FAIL` 필드 필수 기록
- **예외 없음** — 모든 프로젝트, 모든 우선순위에 공통 적용

### D-031: 디자인 검증 의무화 (AADS-163, 2026-03-08)
- **QA PASS 후 디자인 에이전트 자동 실행 의무화**
- 디자인 에이전트: doc-writer 서브에이전트 (`/root/aads/.claude/agents/doc-writer.md`)
- 판정 기준:
  - `DESIGN_VERDICT: PASS` — UI/UX 변경 없거나 검증 완료
  - `DESIGN_VERDICT: REVIEW_NEEDED` — CEO 검토 필요 (이유 포함)
- REVIEW_NEEDED 시: aads_queue_msg로 CEO Chat 보고 + Telegram 알림 → 60초 대기 → 타임아웃 시 PASS 처리
- RESULT_FILE에 `design_status: PASS | PASS_TIMEOUT | REVIEW_NEEDED` 필드 필수 기록
- **예외 없음** — UI 관련 작업(대시보드, Frontend)은 반드시 디자인 검증

### D-032: DEPENDS_ON 교차 확인 의무 (AADS-178, 2026-03-08)
- auto_trigger.sh가 DEPENDS_ON 필드를 done 폴더 + AADS API 교차 확인
- 미충족 시 30/60/120초 exponential backoff 3회 재확인
- 3회 실패 → pending 유지 + Telegram 알림

### D-033: Core 문서 운영 원칙 섹션 상시 유지 (AADS-148, 2026-03-08)
- HANDOVER.md 최상단에 "이 문서의 운영 원칙" 섹션을 상시 유지한다.
- 이 섹션은 어떤 작업에서도 삭제하거나 축약할 수 없다.
- 내용: 토큰 상한 없음, 생략 금지, CEO 질문 0회 기준, 업데이트 의무, 비용 비교 경고.

### D-039: 매니저 Pre-Flight Check 의무 (AADS-178, 2026-03-08)
- 매니저는 지시서를 발행하기 **전** 반드시 Pre-Flight Check를 수행해야 한다.
- API: `GET /api/v1/directives/preflight?task_id={id}&depends_on={id}`
- 응답 `recommendation`:
  - `PROCEED` → 즉시 지시서 발행
  - `WAIT` → depends_on 완료 대기 후 재발행
  - `BLOCKED` → 중복 task_id 해소 후 재발행
- Pre-Flight Check 없이 발행된 지시서로 인한 중복 실행·선행 의존성 오류는 매니저 책임
- **예외 없음** — 모든 프로젝트, 모든 우선순위에 공통 적용

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

### R-021: HANDOVER 업데이트 의무 강화 (AADS-148, 2026-03-08)
- 작업 완료 판정 시 HANDOVER.md 업데이트 여부를 필수 검증.
- Step 6 WRAP 게이트에서 git diff에 HANDOVER.md 미포함 시 작업 완료 불인정.
- 토큰 절약 목적 정보 생략 = R-VIOLATION.

### D-034: HANDOVER 업데이트 WRAP 게이트 (AADS-148, 2026-03-08)
- 작업 완료 판정 시 HANDOVER.md 업데이트 여부를 Step 6 WRAP 게이트에서 자동 검증한다.
- git diff에 HANDOVER.md 변경이 포함되지 않으면 WRAP 게이트 차단.
- 이 검증은 모든 프로젝트에 공통 적용한다.

### R-016: 서킷브레이커 준수 (AADS-134, 2026-03-06)
- 동일 서버/프로젝트에서 3회 연속 작업 실패 시 5분(300초) 쿨다운 의무.
- 쿨다운 중 해당 서버/프로젝트에 신규 작업 투입 금지.
- 쿨다운 만료 후 half_open 상태에서 1건 시험 실행 후 closed 복귀.
- 수동 리셋은 대시보드 /ops/recovery에서 CEO 승인 후 수행.

---

## 4. Genspark 자동화 규칙

### D-035: bridge.py 자동 감지 원칙 (AADS-161, 2026-03-08)
- 매니저가 >>>DIRECTIVE_START ~ >>>DIRECTIVE_END 블록을 채팅창에 출력하면 bridge.py가 자동 감지·추출·저장한다.
- bridge.py는 10초 주기로 Genspark 매니저 채팅창을 폴링하여 디렉티브 블록을 감지한다.
- 감지된 지시서는 /root/.genspark/directives/pending/에 자동 저장된다.
- 매니저는 지시서 블록을 채팅창에 출력하기만 하면 됨. 나머지는 자동화 시스템이 처리한다.
- **매니저가 CEO에게 지시서 전달을 요청하는 것은 금지한다** (D-037 참조).

### D-036: 매니저 자기인식 의무 (AADS-161, 2026-03-08)
- 모든 프로젝트 매니저는 세션 시작 시 HANDOVER.md의 "매니저 자기인식 프로토콜" 섹션을 읽고, 3가지 검증을 수행해야 한다:
  - 검증 ①: 채팅 제목 또는 CEO 첫 메시지에 해당 프로젝트명 포함 여부
  - 검증 ②: Task ID 접두사 일치 (AADS-xxx, KIS-xxx, GO100-xxx 등)
  - 검증 ③: 참조 문서 URL이 해당 프로젝트의 리포지토리와 일치
- 3가지 검증 중 하나라도 불일치 시 경고 출력 후 작업 수행 금지.

### D-037: CEO 전달 요청 금지 (AADS-161, 2026-03-08)
- 매니저가 CEO에게 다음과 같은 전달 요청을 하는 것은 **절대 금지**한다:
  - "이 지시서를 전달해 주세요"
  - "bridge에 넣어 주세요"
  - "파일을 저장해 주세요"
  - 기타 지시서 전달을 CEO에게 요청하는 모든 행위
- 매니저의 역할은 지시서 블록을 채팅창에 출력하는 것으로 완료된다.
- bridge.py가 자동으로 감지·추출·저장하므로 CEO 개입이 불필요하다.
- **위반 시 해당 지시서는 무효 처리하고 재작성해야 한다** (R-022 참조).

### R-022: CEO 전달 요청 금지 위반 (AADS-161, 2026-03-08)
- 매니저가 CEO에게 지시서 전달을 요청하면 R-VIOLATION으로 판정한다.
- 해당 지시서는 무효 처리한다.
- 매니저는 지시서를 재작성하여 채팅창에 다시 출력해야 한다.
- 반복 위반 시 매니저 세션 종료 및 재시작.

---

## 5. Genspark CEO 통합지휘 대화 규칙

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

## 6. 버전 이력

| 버전 | 날짜 | 변경 |
|------|------|------|
| v3.6 | 2026-03-08 | AADS-178: D-032(DEPENDS_ON 교차 확인), D-039(매니저 Pre-Flight Check 의무화) — preflight API + auto_trigger 강화 |
| v3.5 | 2026-03-08 | AADS-163: D-030(QA 에이전트 의무화), D-031(디자인 검증 의무화) — 개발→QA→디자인 3단계 품질 게이트 통합 |
| v3.4 | 2026-03-08 | AADS-161: §0 운영 원칙 신규, D-035(bridge 자동 감지), D-036(매니저 자기인식), D-037(CEO 전달 금지), R-022(위반 처리), §4 Genspark 자동화 규칙 신규 |
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
| v2.9 | 2026-03-06 | D-017 소스코드 모듈화, project_artifacts DB화 원칙, AADS-128~130 풀사이클 완료 |
| v2.8 | 2026-03-06 | D-016 FLOW 프레임워크, R-014 Wrap up 의무화, R-015 교훈 등록, 9-3 산출물 파일명 확장, 버전 이력 시간순 정렬 |
| v3.0 | 2026-03-06 | AADS-134: D-018(4계층 자기치유), D-019(서버 상호 감시), D-020(복구 이력 DB), R-016(서킷브레이커 준수) — 대시보드 Recovery+Servers 페이지, project_healing.py |
| v3.1 | 2026-03-07 | AADS-140~142: D-018 개정(하트비트 기반 L1 전환, 30분→이벤트 감시), D-021 신규(하트비트 세션 관리 원칙) — session_watchdog 3서버 배포 |
| v3.3 | 2026-03-08 | AADS-148: D-023 v2 교체(4계층 운영 원칙), D-033(Core 운영 원칙 상시 유지), D-034(HANDOVER WRAP 게이트), R-021(업데이트 의무 강화) |
| v3.2 | 2026-03-07 | AADS-144: D-022(지시서 포맷 v2.0), D-023(HANDOVER 3계층 분리), D-024(모델 라우팅), D-025(우선순위큐 impact/effort 정렬) |
