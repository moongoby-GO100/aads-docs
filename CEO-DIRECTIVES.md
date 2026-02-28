# CEO DIRECTIVES – AADS (Autonomous AI Development System)
> 최종 업데이트: 2026-02-28 (v1.1)
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
- 모델 라우팅: 단순 태스크→nano/flash($0.05~0.10/1M), 복잡 태스크→Opus/Pro($5~25/1M)
- 오픈소스 우선: 상용 SaaS 대체 가능하면 오픈소스 적극 활용
- 월 인프라 비용 $500 이하 유지 목표 (프로토타이핑 단계)

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

---

## 2. 기술적 지시

### T-001 멀티 에이전트 아키텍처
- **프레임워크**: LangGraph (상태 기반 방향 그래프, 체크포인팅, HITL 지원)
- **패턴**: Supervisor 패턴 (2단계 계층 — 최상위 Supervisor + 전문 에이전트)
- **상태 관리**: PostgreSQL 기반 체크포인팅 (실패 시 마지막 성공 단계 재개)
- **Human-in-the-Loop**: 배포, DB 변경, 외부 API 호출 등 고위험 작업에 승인 게이트
- 3단계 이상 계층은 오버헤드 → 2단계로 제한

### T-002 에이전트 역할 구성
| 에이전트 | 역할 | 추천 모델 | 대안 모델 |
|----------|------|-----------|-----------|
| Supervisor | 전체 오케스트레이션, 태스크 분배·합성 | Claude Opus 4.6 | Gemini 3.1 Pro |
| Architect | 시스템 설계, 기술 의사결정 | Claude Opus 4.6 | GPT-5.2 Pro |
| PM | 태스크 분해, 우선순위, 일정 | Claude Sonnet 4.6 | GPT-5.2 Chat |
| Developer | 코드 생성·수정·리팩터 | Claude Sonnet 4.6 | GPT-5.3 Codex |
| QA | 테스트 생성·실행·버그 리포트 | Claude Sonnet 4.6 | GPT-5 mini |
| DevOps | CI/CD, 배포, 모니터링 | GPT-5 mini | Claude Haiku 4.5 |
| Researcher | 데이터 수집·분석·웹 검색 | Gemini 2.5 Flash | GPT-5 nano |

### T-003 MCP 서버 스택
- **Phase 1 필수 (7개)**: Filesystem, Git/GitHub, PostgreSQL, Brave Search, Memory, Supabase, Fetch
- **Phase 2 확장 (3개)**: Puppeteer(브라우저 자동화), Sentry(에러 추적), Slack(팀 알림)

### T-004 샌드박스 & 배포 인프라
| 용도 | 도구 | 비용 |
|------|------|------|
| 에이전트 코드 실행 | E2B Pro | $150/월 (초기 $100 무료 크레딧) |
| GPU 태스크 | Modal | 종량제 |
| API 서버 배포 | Fly.io | ~$20-50/월 |
| 프론트엔드 배포 | Vercel Pro | $20/월 |
| DB + Auth | Supabase Pro | $25/월 |
| MCP 서버 | 자체 호스팅(오픈소스) | $0 (Fly.io에 포함) |

### T-005 AADS 기술 스택
- **오케스트레이션**: LangGraph + langgraph-supervisor
- **AI API**: Anthropic Claude API (메인), OpenAI API (보조), Google Gemini API (데이터)
- **MCP**: @modelcontextprotocol/* 공식 서버 + 커뮤니티 서버
- **샌드박스**: E2B (에이전트 코드 실행 격리 환경)
- **상태 저장**: PostgreSQL (체크포인팅) + Redis (캐싱)
- **관측성**: LangSmith (에이전트 트레이싱) 또는 오픈소스 대안
- **프론트엔드 (AADS 대시보드)**: Next.js + React + Tailwind CSS
- **백엔드 API**: FastAPI (Python) 또는 Supabase Edge Functions (TypeScript)
- **배포**: Fly.io (API) + Vercel (프론트)

### T-006 로드맵
| Phase | 내용 | 상태 |
|-------|------|------|
| Phase 0 | 리서치 + 아키텍처 설계 + 인계서 시스템 | **진행중** |
| Phase 1 | AADS 코어 — LangGraph + 에이전트 체인 + MCP + 샌드박스 첫 동작 | 대기 |
| Phase 2 | AADS 대시보드 — 에이전트 상태 모니터링, 태스크 관리 UI | 대기 |
| Phase 3 | AADS 서비스 배포 — 외부 사용자가 프로젝트를 생성하고 에이전트에게 개발 위임 | 대기 |
| Phase 4 | 자율 개선 — 에이전트가 자체 코드를 개선하는 피드백 루프 | 대기 |
| Phase 5 | 첫 번째 프로젝트: HealthMate — AADS를 사용하여 AI 건강기능식품 추천 앱 개발 | 대기 |
| Phase 6 | 범용화 — 다양한 프로젝트에 AADS 적용, SaaS 모델 확장 | 대기 |

---

## 3. 절대 규칙 (위반 시 작업 무효)
- HANDOVER.md 업데이트 없이 작업 완료 선언 금지
- 보고서는 반드시 GitHub push + HTTP 200 확인
- .env / 시크릿 / API 키 커밋 금지
- 프로덕션 DB 직접 편집 금지 (마이그레이션 스크립트로만)
- 서버 서비스 임의 재시작 금지
- CEO-DIRECTIVES.md의 지시를 무시하는 설계/분석은 전부 무효
- 기존 서비스(aads-server 파이프라인) 중단/변경 시 반드시 CEO 승인

### R-NEW-1: 보고 시 브라우저 URL 사용 필수
- CEO에게 파일을 보고할 때는 반드시 GitHub **브라우저 경로**를 사용한다.
- 형식: `https://github.com/moongoby-GO100/aads-docs/blob/main/{파일경로}`
- `raw.githubusercontent.com` URL은 HTTP 200 검증 용도로만 내부적으로 사용하며, CEO 보고에는 절대 포함하지 않는다.
- 예시:
  - ✅ `https://github.com/moongoby-GO100/aads-docs/blob/main/design/aads-architecture-v1.md`
  - ❌ `https://raw.githubusercontent.com/moongoby-GO100/aads-docs/main/design/aads-architecture-v1.md`

### R-NEW-2: 완료 보고 형식
- 작업 완료 후 CEO에게 보고할 때 아래 형식을 따른다:
```
푸시 완료했습니다.
- [파일명]: https://github.com/moongoby-GO100/aads-docs/blob/main/{경로}
- HANDOVER: https://github.com/moongoby-GO100/aads-docs/blob/main/HANDOVER.md
```
- 모든 생성·수정된 파일의 브라우저 URL을 빠짐없이 나열한다.

---

## 4. 버전 이력

| 버전 | 날짜 | 변경 |
|------|------|------|
| v1.0 | 2026-02-28 | 초판 — D-001~D-008, T-001~T-006, 절대 규칙 |
| v1.1 | 2026-02-28 | 절대 규칙 추가: R-NEW-1 브라우저 URL 보고 필수, R-NEW-2 완료 보고 형식 |
