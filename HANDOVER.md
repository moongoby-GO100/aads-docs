# HANDOVER – AADS (Autonomous AI Development System)
> 최종 업데이트: 2026-02-28 (v1.0 — 초판, 리서치 Phase 완료, 아키텍처 설계 착수)
> 관리자: CEO (moongoby)
> 용도: 모든 AI 세션(웹 Claude, Cursor, Claude Code) 시작 시 필수 읽기

---

## 1. 프로젝트 개요

- **AADS (Autonomous AI Development System)**: 멀티 AI 에이전트가 조직처럼 협업하여 소프트웨어를 자율 개발하는 시스템
- **AADS 자체가 프로덕트** — 먼저 AADS를 완성하여 서비스 배포, 이후 첫 번째 프로젝트로 HealthMate(AI 건강기능식품 추천 앱) 진행
- 저장소: github.com/moongoby-GO100/aads-docs (문서), github.com/moongoby-GO100/aads-server (서버)
- 서버: aads.newtalk.kr (현재 로그인 리다이렉트 확인)
- 파이프라인 대시보드: https://aads.newtalk.kr/projects/2a0647ca
- GitHub PAT 'aads-server': 권한 repo + workflow, 만료 2026-05-27

### 업무 체계
- **웹 Claude (지휘 AI)**: CEO와 직접 대화, 전략 수립, 지시서 작성, 교차검증
- **Cursor (서버 작업자)**: 서버에서 코드 작성, 분석, 보고서 생성, GitHub push
- **Claude Code (서버 작업자)**: Cursor와 동일 역할, 터미널 기반
- 웹 Claude는 서버 접근 불가, 세션 끊기면 맥락 소실 → 본 HANDOVER.md로 복구

---

## 2. 완료된 작업

| Task ID | 날짜 | 커밋 | HTTP | 핵심 결과 |
|---------|------|------|------|-----------|
| RESEARCH-AI-MODELS | 02-28 | 확인필요 | 확인필요 | 프론티어 LLM 100+모델 조사, 14섹션, 가격표, AADS 에이전트별 모델 배정 전략, 비용 시뮬레이션($85/월 혼합) |
| RESEARCH-OPENSOURCE | 02-28 | 미push | — | 수학/추론 모델 7개, 코딩 모델 6개, 퀀트 도구 20+, RAG 8개, 벡터DB 6개, 크롤링/워크플로우/라벨링 전 분야 |
| RESEARCH-INFRA-MCP | 02-28 | 미push | — | 샌드박스 7개 벤치마크(E2B 권장), PaaS 3개 비교(Fly.io 권장), MCP 8,600+서버 에코시스템, 앱빌더 8개, 코딩에이전트 12개 비교 |
| PAT-VERIFY | 02-26 | — | — | GitHub PAT 'aads-server' 검증 완료: repo scope, 만료 2026-05-27 |
| PIPELINE-CHECK | 02-26 | — | — | 서버 파이프라인 실행 중 확인, 대시보드 URL 확인 |
| WORKFLOW-SCOPE | 02-28 | — | — | CEO가 PAT에 workflow scope 추가 완료 |
| PROCESS-SETUP | 02-28 | — | — | CEO 업무 프로세스 표준 지침 수립(웹Claude↔Cursor/ClaudeCode 3자 협업 체계) |
| INIT-DOCS | 02-28 | 6145a6c | 200 | HANDOVER.md + CEO-DIRECTIVES.md v1.0 생성 및 push, raw URL HTTP 200 확인 |
| DIRECTIVES-UPDATE | 2026-02-28 | — | — | CEO-DIRECTIVES v1.1 — 브라우저 URL 보고 규칙 (R-NEW-1, R-NEW-2) 추가 |
| ARCH-DESIGN-DRAFT | 2026-02-28 | — | — | Phase 1 아키텍처 설계서 v1.0-draft 작성 (design/aads-architecture-v1.md) |
| REPORT-PUSH | 2026-02-28 | — | — | 보고서 placeholder 3건 생성 (reports/) |

---

## 3. 진행 중 작업

| Task ID | 상태 | 내용 |
|---------|------|------|
| ARCH-DESIGN-FINAL | **진행** | CEO 설계서 검토 승인 후 확정판 반영 (design/aads-architecture-v1.md) |

---

## 4. 보류/미시작

| 항목 | 선행조건 | 우선순위 |
|------|----------|----------|
| 아키텍처 설계문서 확정 | ARCH-DESIGN 완료 + CEO 승인 | ★★★★★ |
| AADS 코어 구현 (LangGraph + 에이전트 체인) | 아키텍처 확정 | ★★★★★ |
| AADS 서비스 배포 | 코어 구현 완료 | ★★★★★ |
| CI/CD 파이프라인 (.github/workflows) | workflow scope 추가 완료 → 재시도 가능 | ★★★★ |
| LLM 보안/규제/개인정보 조사 | 아키텍처 확정 후 | ★★★ |
| MCP 서버 실제 구성 가이드 | 아키텍처 확정 후 | ★★★ |
| HealthMate 기획·개발 | **AADS 서비스 배포 완료 후** 첫 프로젝트로 진행 | ★★ (후순위) |
| 경쟁 서비스 분석 (Pilly, 마이루틴 등) | HealthMate 착수 시 | ★ |
| 한국 공공 API 검증 | HealthMate 착수 시 (CEO 자료 있음) | ★ |

---

## 5. 핵심 발견 (누적)

### AI 모델 시장 (2026-02-28)
- 가격 98% 폭락(2023 대비), GPT-5 nano $0.05/1M 토큰
- 2026년 = 멀티 에이전트 아키텍처 전환 원년
- 오픈소스(Qwen 3.5, Llama 4, DeepSeek-R1)가 상용 모델과 경쟁 수준
- 한국어 모델(HyperCLOVA X, Gauss 2) API 접근성/성능 제한적
- AADS 최적 혼합: Claude Opus(아키텍처) + Sonnet(개발) + GPT-5 nano(분류) + Gemini Flash(데이터) → 월 ~$85

### MCP 에코시스템
- 8,600+ 서버 등록, Google/OpenAI/GitHub/Atlassian 공식 지원
- AADS 필수 7개 서버: Filesystem, Git/GitHub, PostgreSQL, Brave Search, Memory, Supabase, Fetch
- MCP = AADS 에이전트 도구 접근의 표준 레이어

### 샌드박스/배포 인프라
- E2B: AI 에이전트 코드 실행 최적(~150ms, $100 무료 크레딧, 오픈소스)
- Modal: GPU 필요 시 유일한 선택지
- Fly.io(API) + Vercel(프론트) + Supabase(DB+Auth) 조합 최적
- 월 인프라: ~$320-350 (프로토타이핑 $0 가능)

### AADS vs AI 앱 빌더 차별화
- 앱 빌더(Lovable, Bolt 등): 15-20 컴포넌트에서 AI 컨텍스트 손실, GUI 전용
- Cursor/Windsurf: 사람이 쓰는 IDE, AADS 시스템 내부에서 AI로 활용 불가
- AADS: API + MCP + 샌드박스로 동일 기능을 프로그래밍 방식 자동화
- 핵심 차이: 멀티 에이전트 조직 시뮬레이션 (CEO→PM→Dev→QA→Ops)

### 아키텍처 패턴
- LangGraph Supervisor 패턴: 2단계 계층이 프로덕션 최적 (3단계는 오버헤드)
- 상태 머신 모델링: 예측·테스트 가능한 방향 그래프
- 체크포인팅: PostgreSQL/Redis, 실패 시 마지막 성공 단계에서 재개
- Human-in-the-Loop: 고위험 작업에 승인 게이트
- 비용 제어: 모델 라우팅 + 토큰 예산 + 캐싱 + 조기 종료

---

## 6. 웹 Claude 인수인계 사항

### 6-1. 최신 상태
- CEO-DIRECTIVES v1.1 반영 (R-NEW-1 브라우저 URL 보고, R-NEW-2 완료 보고 형식)
- Phase 1 아키텍처 설계서 v1.0-draft 작성 완료, 보고서 placeholder 3건 push
- 다음: CEO 설계서 검토 승인 → 확정판 반영 → Phase 1.1 구현 지시 발행

### 6-2. 웹 Claude가 해야 할 일
1. CEO 설계서 검토 승인 → 확정판 반영
2. Phase 1.1 구현 지시서 발행 (에이전트 체인·MCP·샌드박스 연동 순서)

### 6-3. 대표님 확인 필요 사항
- [x] INIT-DOCS push 완료 (2026-02-28, 6145a6c, HTTP 200)
- [ ] 아키텍처 설계문서 방향 최종 승인
- [ ] AADS가 생성할 프로젝트의 범위 정의 (웹앱? 모바일? API?)

### 6-4. 주의사항
- PAT 토큰 값은 웹 Claude 세션 간 유지 안 됨 → Cursor/Claude Code가 직접 push
- 웹 Claude는 서버 접근 불가 → 모든 서버 작업은 Cursor/Claude Code 경유
- Sprint 4 보고서 2건(sprint4-task4.3 등) 접근 불가 상태 지속

---

## 7. 업데이트 규칙
- 모든 Task 완료 시 이 문서 업데이트 필수 (미수행 시 작업 미완료 간주)
- 섹션 2에 Task 추가, 섹션 3 갱신, 섹션 5 핵심 발견 추가, 섹션 6 웹 Claude 인수인계 갱신
- push 후 raw URL HTTP 200 확인:
  curl -s -o /dev/null -w "%{http_code}" https://raw.githubusercontent.com/moongoby-GO100/aads-docs/main/HANDOVER.md
- 보고서 마지막 줄: `HANDOVER.md 업데이트 완료: {커밋해시}`

---

## 8. 버전 이력

| 버전 | 날짜 | 변경 |
|------|------|------|
| v1.0 | 2026-02-28 | 초판 — 리서치 Phase 완료, 인계서 시스템 구축, 아키텍처 설계 착수 |
