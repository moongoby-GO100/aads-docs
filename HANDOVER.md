# HANDOVER – AADS (Autonomous AI Development System)
> 최종 업데이트: 2026-02-28 (v2.0 — 21건 수정 반영, 설계서 v1.1-final 확정, 보고서 완성)
> 관리자: CEO (moongoby)
> 용도: 모든 AI 세션(웹 Claude, Cursor, Claude Code) 시작 시 필수 읽기

---

## 1. 프로젝트 개요

- **AADS (Autonomous AI Development System)**: 멀티 AI 에이전트가 조직처럼 협업하여 소프트웨어를 자율 개발하는 시스템
- **AADS 자체가 프로덕트** — 먼저 AADS를 완성하여 서비스 배포, 이후 첫 프로젝트 HealthMate
- 저장소: github.com/moongoby-GO100/aads-docs (문서), github.com/moongoby-GO100/aads-server (서버)
- 서버: aads.newtalk.kr
- 파이프라인 대시보드: https://aads.newtalk.kr/projects/2a0647ca
- GitHub PAT 'aads-server': 권한 repo + workflow, 만료 2026-05-27

### 업무 체계
- **웹 Claude (지휘 AI)**: CEO와 직접 대화, 전략 수립, 지시서 작성, 교차검증
- **Cursor (서버 작업자)**: 서버에서 코드 작성, 분석, 보고서 생성, GitHub push
- **Claude Code (서버 작업자)**: Cursor와 동일 역할, 터미널 기반
- 웹 Claude는 서버 접근 불가 → 본 HANDOVER.md로 맥락 복구

---

## 2. 완료된 작업

| Task ID | 날짜 | 커밋 | HTTP | 핵심 결과 |
|---------|------|------|------|-----------|
| RESEARCH-AI-MODELS | 02-28 | — | — | 프론티어 LLM 100+모델 조사, 가격표, 에이전트별 모델 배정 |
| RESEARCH-OPENSOURCE | 02-28 | — | — | 수학/추론 7개, 코딩 6개, 퀀트 20+, RAG 8개, 벡터DB 6개 |
| RESEARCH-INFRA-MCP | 02-28 | — | — | 샌드박스 7개, PaaS 3개, MCP 8,600+, 앱빌더 8개, 에이전트 12개 |
| PAT-VERIFY | 02-26 | — | — | GitHub PAT 검증: repo scope, 만료 2026-05-27 |
| PIPELINE-CHECK | 02-26 | — | — | 서버 파이프라인 실행 확인 |
| WORKFLOW-SCOPE | 02-28 | — | — | PAT에 workflow scope 추가 |
| PROCESS-SETUP | 02-28 | — | — | 웹Claude↔Cursor/ClaudeCode 3자 협업 체계 수립 |
| INIT-DOCS | 02-28 | 6145a6c | 200 | HANDOVER.md + CEO-DIRECTIVES.md v1.0 생성 |
| DIRECTIVES-v1.1 | 02-28 | 3778986 | 200 | CEO-DIRECTIVES v1.1 (브라우저 URL 보고 규칙) |
| ARCH-v1.0-DRAFT | 02-28 | 3778986 | 200 | Phase 1 설계서 v1.0-draft |
| REPORT-PLACEHOLDER | 02-28 | 3778986 | 200 | 보고서 placeholder 3건 |
| **PUSH-006** | **02-28** | **7dfd454** | **200** | **CEO-DIRECTIVES v2.0 + 설계서 v1.1-final (21건 수정) + 보고서 3건 실제 내용 + HANDOVER v2.0** |

---

## 3. 진행 중 작업

| Task ID | 상태 | 내용 |
|---------|------|------|
| PHASE1-IMPL | **착수 가능** | Phase 1 코어 구현 (12일 예상). 설계서 v1.1-final 확정 완료. |

---

## 4. 보류/미시작

| 항목 | 선행조건 | 우선순위 |
|------|----------|----------|
| AADS 코어 구현 (LangGraph Native Supervisor + 8 에이전트) | 설계서 확정 ✅ | ★★★★★ |
| AADS 서비스 배포 (Fly.io + Vercel) | 코어 완료 | ★★★★★ |
| CI/CD (.github/workflows) | 코어 완료 | ★★★★ |
| AADS 대시보드 (Phase 2) | 코어 배포 후 | ★★★ |
| MCP 서버 구성 가이드 | 코어 구현 시 동시 | ★★★ |
| HealthMate | AADS 완성 후 | ★★ |

---

## 5. 핵심 발견 (누적)

### AI 모델 (2026-02-28, 검증 가격)
- Claude Opus 4.6: $5/$25, Sonnet 4.6: $3/$15, Haiku 4.5: $1/$5
- GPT-5.3 Codex: $1.75/$14, GPT-5 mini: $0.25/$2, GPT-5 nano: $0.05/$0.40
- Gemini 3.1 Pro: $2/$12, Gemini 2.5 Flash: $0.30/$2.50
- GPT-5.2 Pro: $21/$168 (극고가, 사용 비추천)
- AADS 최적 혼합 캐싱 후 월 ~$55

### MCP 에코시스템
- 8,600+ 서버, Phase 1 필수 7개
- MultiServerMCPClient lifespan 초기화, 상시 4개 + 온디맨드 3개
- langgraph-supervisor MCP 루프 버그 #249 → Native StateGraph 사용

### 인프라
- E2B $150-250/mo, Fly.io 2GB RAM $20-50/mo, Supabase Direct:5432 $25-40/mo
- Fly.io 60초 idle timeout → 15초 SSE heartbeat
- 월 예상 $320 (D-004 $500 내)

### 아키텍처 핵심 결정 (21건 수정 완료)
- LangGraph >= 1.0.8, Native Tool-Based Supervisor
- 8 에이전트: Supervisor, PM, Architect, Developer, QA, Judge, DevOps, Researcher
- 6단계 사용자 체크포인트 (경쟁사 차별점)
- 구조화 JSON TaskSpec 통신 (자유 텍스트 금지)
- Judge Agent 독립 검증 (정확도 ~10%→~70%)
- 점진적 자율성 게이트 (90% 자동승인, 70% 재활성화)
- Graceful Degradation: LLM fallback, E2B "코드만" 모드, Supabase InMemory 임시
- 동시성: thread 10, E2B 5, DB 15, Redis 글로벌 비용 카운터

---

## 6. 웹 Claude 인수인계 사항

### 6-1. 최신 상태
- CEO-DIRECTIVES v2.0 확정
- 설계서 v1.1-final 확정 (21건 수정 반영)
- 보고서 3건 실제 내용 작성
- **Phase 1 구현 즉시 착수 가능**

### 6-2. 웹 Claude가 해야 할 일
1. Phase 1.1 구현 지시서 발행 (12일 로드맵)
2. 구현 진행 중 교차검증 및 이슈 대응

### 6-3. 대표님 확인 사항
- [x] INIT-DOCS push 완료
- [x] 설계서 v1.1-final 확정
- [x] 21건 수정사항 반영
- [ ] Phase 1 구현 착수 최종 승인
- [ ] AADS 생성 프로젝트 범위 (웹앱? 모바일? API?)

### 6-4. 주의사항
- PAT 토큰 웹 Claude 세션 간 비유지 → Cursor/Claude Code가 push
- 웹 Claude 서버 접근 불가 → Cursor/Claude Code 경유
- `langgraph-supervisor` 프로덕션 사용 금지 (R-010)
- Supabase Supavisor 경유 금지 (R-011)

---

## 7. 업데이트 규칙
- 모든 Task 완료 시 이 문서 업데이트 필수
- push 후 raw URL HTTP 200 확인:
  `curl -s -o /dev/null -w "%{http_code}" https://raw.githubusercontent.com/moongoby-GO100/aads-docs/main/HANDOVER.md`
- 보고서 마지막 줄: `HANDOVER.md 업데이트 완료: {커밋해시}`

---

## 8. 버전 이력

| 버전 | 날짜 | 변경 |
|------|------|------|
| v1.0 | 2026-02-28 | 초판 — 리서치 완료, 인계서 구축 |
| v2.0 | 2026-02-28 | 대규모 개정 — CEO-DIRECTIVES v2.0, 설계서 v1.1-final, 보고서 3건 완성, 21건 수정 반영 |
