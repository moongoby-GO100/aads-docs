# AADS HANDOVER v15.1
최종 업데이트: 2026-04-04 | 버전: v15.1 — P0 401 인증 장애 복구, Gemini 폴백 수정, Background 10종 qwen-turbo 100% 전환 (AADS-204)

## 이 문서의 운영 원칙
- 이 문서는 토큰 상한이 없다. 비용을 아끼지 말고 최신화하라.
- 중요 내용은 빠짐없이 반영하라. 생략 금지.
- 이 문서를 읽고 CEO에게 추가 질문 없이 즉시 업무 수행이 가능해야 한다.
- 모든 작업 완료 시 반드시 이 문서를 업데이트하라.
- CEO 재설명 1회 비용 > 문서 토큰 비용 100회분임을 명심하라.

---

## 시스템 개요

- **AADS 정의**: Autonomous AI Development System — 멀티 AI 에이전트 자율 개발 시스템
- **아키텍처**: CEO→PM→개발자→QA→운영 역할 분리 멀티 에이전트
- **대시보드**: https://aads.newtalk.kr/
- **GitHub 리포**:
  - aads-docs: https://github.com/moongoby-GO100/aads-docs (문서/지시서/보고서)
  - aads-server: https://github.com/moongoby-GO100/aads-server (백엔드 FastAPI)
  - aads-dashboard: https://github.com/moongoby-GO100/aads-dashboard (프론트 Next.js)
- **GitHub PAT**: repo+workflow 권한, 만료 2026-05-27
- **기술 스택**: LangGraph >= 1.0.10, FastAPI, Next.js, PostgreSQL, Docker
- **E2E 검증 완료**: 3시나리오, 건당 $3.72~$4.03

---

## 매니저 자기인식 프로토콜

### 1-1. 나는 누구인가
- **정체성**: AADS 전담 AI 매니저 (Genspark AI)
- **역할**: CEO 지시 수신 → 지시서 발행 → 결과 검증 → CEO 보고
- **권한 제한**: CEO 승인 없이 작업 생성·변경·삭제 불가, 코드 작성·SSH 접속 금지
- **관할**: AADS 프로젝트 전체

### 1-2. 이 채팅창이 내 채팅창인가
- **확인 ①**: 채팅 제목 또는 CEO 첫 메시지에 "AADS" 포함
- **확인 ②**: Task ID 접두사 AADS-xxx 일치
- **확인 ③**: 참조 문서 URL이 aads-docs 리포지토리
- **불일치 시**: "⚠️ 프로젝트 불일치 감지" 경고 출력 후 작업 수행 금지

### 1-3. 세션 시작 프로토콜
1. HANDOVER.md 읽기
2. HANDOVER-RULES.md 읽기
3. CEO-DIRECTIVES.md 읽기
4. 마지막 완료/다음 대기 작업 확인
5. CEO에 세션 시작 보고

### AADS 매니저 채팅 URL
- https://www.genspark.ai/agents?id=3d86d6f3-09a7-41b2-b91b-762a55512458

---

## 지시서 자동화 파이프라인 (bridge.py)

### 전체 흐름
1. 매니저가 채팅창에 >>>DIRECTIVE_START ~ >>>DIRECTIVE_END 출력
2. bridge.py 자동 감지 (10초 폴링, 서버 211)
3. /root/.genspark/directives/pending/ 자동 저장
4. auto_trigger.sh 검증 (WORKDIR 권한, 중복 체크, 의존성)
5. claude_exec.sh 실행 (모델 자동 라우팅)

### 절대 금지 (D-037, R-022)
- 매니저가 CEO에게 "이 지시서를 전달해 주세요" 금지
- 매니저가 CEO에게 "bridge에 넣어 주세요" 금지
- 매니저가 CEO에게 "파일을 저장해 주세요" 금지
- 기타 지시서 전달을 CEO에게 요청하는 모든 행위 금지

### 올바른 흐름
- 매니저는 지시서 블록(>>>DIRECTIVE_START ~ >>>DIRECTIVE_END)을 채팅창에 출력만 하면 됨
- 나머지는 bridge.py → auto_trigger.sh → claude_exec.sh 자동화 시스템이 처리
- CEO 개입 불필요

### 지시서 작성 예시

**S 작업 예시:**
```
>>>DIRECTIVE_START
TASK_ID: AADS-170
TITLE: STATUS.md 타임존 수정
PRIORITY: P2-MEDIUM
SIZE: S
IMPACT: L
EFFORT: L
MODEL: sonnet
REVIEW_REQUIRED: false
ASSIGNEE: Claude (서버 68, /root/aads)
DESCRIPTION: STATUS.md의 타임스탬프를 KST로 통일
SUCCESS_CRITERIA: 모든 timestamp이 +09:00 형식
>>>DIRECTIVE_END
```

**M 작업 예시:**
```
>>>DIRECTIVE_START
TASK_ID: AADS-171
TITLE: 대시보드 Watchdog 페이지 UI 개선
PRIORITY: P1-HIGH
SIZE: M
IMPACT: M
EFFORT: M
MODEL: sonnet
REVIEW_REQUIRED: false
ASSIGNEE: Claude (서버 68, /root/aads)
DESCRIPTION: /watchdog 페이지에 서비스 상태 카드 + 에러 히스토리 테이블 추가
SUCCESS_CRITERIA: 1) 서비스 카드 6개 표시, 2) 에러 테이블 최근 20건 표시, 3) 30초 자동 갱신
>>>DIRECTIVE_END
```

**XL 작업 예시:**
```
>>>DIRECTIVE_START
TASK_ID: AADS-172
TITLE: Agent Teams 파일럿 구현
PRIORITY: P1-HIGH
SIZE: XL
IMPACT: H
EFFORT: H
MODEL: opus
REVIEW_REQUIRED: true
ASSIGNEE: Claude (서버 68, /root/aads)
parallel_group: AADS-172-A (graphs/), AADS-172-B (agents/), AADS-172-C (tests/)
files_owned:
  A: graphs/team_lead.py, graphs/team_worker.py
  B: agents/coordinator.py, agents/specialist.py
  C: tests/test_team_*.py
DESCRIPTION: Agent Teams 아키텍처 구현 - 팀 리드 + 팀원 구조
SUCCESS_CRITERIA: 1) 팀 리드 DAG 실행, 2) 팀원 병렬 처리, 3) E2E 테스트 통과
>>>DIRECTIVE_END
```


---

## Pipeline Runner 시스템 (AADS-190+, 2026-03-10~)

### Pipeline C → Pipeline Runner 전환
- **Pipeline C** (pipeline_c.py, 104KB): 레거시 보존. 컨테이너 내 asyncio 실행. 도구명(pipeline_c_start 등) 폐기, 시스템 프롬프트에 "사용 금지" 명시.
- **Pipeline Runner** (pipeline-runner.sh): 활성 실행 시스템. 호스트 systemd 독립 프로세스.

### Pipeline Runner 실행 흐름
```
CEO 채팅 → intent_router(pipeline_runner) → chat_service(AutonomousExecutor)
    → tool_executor → POST /api/v1/pipeline/jobs (submit)
    → pipeline-runner.sh (호스트 systemd, 5초 폴링)
    → Claude Code CLI 실행 (6단계 모델+계정 폴백)
    → AI Reviewer 자동 검수 → awaiting_approval
    → CEO approve → git push → 프로젝트별 배포 → done
```

### Pipeline C vs Runner 비교

| 항목 | Pipeline C (보존) | Pipeline Runner (활성) |
|------|------------------|----------------------|
| 실행 위치 | 컨테이너 내 asyncio | 호스트 systemd 독립 프로세스 |
| 서버 재시작 영향 | 작업 소멸 | 무영향 |
| 계정 폴백 | 없음 | 6단계 (Sonnet/Opus/Haiku × 2계정) |
| AI 검수 | 없음 | AI Reviewer 자동 실행 |
| CEO 승인 | 없음 | approve/reject 필수 |
| 프로젝트 | AADS만 | AADS/KIS/GO100/SF/NTV2 전체 |

### 관련 파일
- `scripts/pipeline-runner.sh`: 메인 실행 스크립트 (systemd)
- `app/api/pipeline_runner_api.py`: REST API (submit/status/approve/reject)
- `app/services/pipeline_runner_service.py`: 비즈니스 로직
- `app/api/ceo_chat_tools.py`: pipeline_runner_submit/status/approve 도구

---

## 서버 현황 (3대 전체)

### 서버 211 (211.188.51.113)
- 역할: Hub 서버 (Bridge, auto_trigger, pipeline_monitor)
- OS: Ubuntu
- 주요 서비스: bridge.py, auto_trigger.sh, pipeline_monitor, session_watchdog, meta_watchdog
- 담당 프로젝트: KIS, GO100
- WORKDIR: /root/kis-autotrade-v4

### 서버 68 (68.183.183.11)
- 역할: AADS Backend
- OS: Ubuntu
- 주요 서비스: FastAPI, PostgreSQL, Dashboard, LiteLLM, Redis, SearXNG (Docker Compose 7컨테이너)
- 담당 프로젝트: AADS
- WORKDIR: /root/aads
- session_watchdog: PID 6166, HC=200

### 서버 114 (114.207.244.86 (port 7916))
- 역할: 실행 서버
- OS: Ubuntu
- 주요 서비스: ShortFlow, NTV2
- 담당 프로젝트: SF, NTV2, NAS
- WORKDIR: SF=/data/shortflow, NTV2=/srv/newtalk-v2

### 서버 간 모니터링
211↔68↔114 삼각 구조, 2분 주기 크로스 모니터링

---

## 6개 프로젝트 상태

### a) AADS — Phase 2 운영
- **최근: AADS-195+ (2026-03-22~30) — Pipeline Runner 실전 운용, Blue-Green 배포, PC Agent v1.0.13, Prompt Caching BP 구조**
- AADS-190 완료 (2026-03-10): 원격 쓰기/실행 9개 도구 + 서브에이전트 (spawn_subagent, spawn_parallel_subagents)
- AADS-191 완료: OAuth 전환 (ANTHROPIC_AUTH_TOKEN), R-AUTH 체계 확립, anthropic_client.py 중앙화
- Pipeline Runner 전면 도입: pipeline_c_start 도구 폐기 → pipeline_runner_submit으로 대체. Pipeline C 소스는 보존(레거시)
- Blue-Green 무중단 배포: deploy.sh, unified_healer.py bluegreen 복구, aads-server-green 컨테이너 대기
- PC Agent v1.0.13: 뮤텍스/단일인스턴스, EXE 빌드 GitHub Actions, 에러 리질리언스
- Prompt Caching 3-breakpoint: BP1(시스템)/BP2(도구)/BP3(히스토리), model_selector.py 적용
- 메모리 진화: Self-Evaluator LLM-free 재구현, Meta-Evaluator 트랜잭션 통합, visual memory
- LiteLLM 확장: OpenRouter 5종(grok-4-fast, deepseek-v3 등), Groq 7종, Gemini 3.0/3.1
- Watchdog 강화: 연쇄 재시작 방지, watchdog-host.sh systemd 등록
- SearXNG 검색 우선순위 삽입 (Gemini Grounding 앞)
- 대시보드: Tool 접기/펼치기, recovered 메시지 tool UI, SSE 끊김 복구
- 이전: AADS-188E 완료 (2026-03-09) — 전체 통합 검증 + E2E 테스트 66개 PASS
  - E2E 4개 시나리오: 코드 수정 풀플로우 / Deep Research / Agent SDK 자율 실행 / 시맨틱 코드 검색
  - test_e2e_code_modify.py(24), test_e2e_deep_research.py(19), test_e2e_agent_sdk.py(23) 신규
  - chat_service.py: 시맨틱 코드 검색 컨텍스트 자동 주입 (섹션 4.5, `<codebase_knowledge_inline>`)
  - 기존 핵심 테스트 178건 전부 PASS (회귀 없음)
  - 보고서: reports/AADS-188E-INTEGRATION-REPORT.md
- 이전: AADS-188D 완료 (Monaco DiffEditor 프론트: CodeDiffViewer+CodePanel+useDiffApproval+4패널+diff_preview SSE+300초 카운트다운) commit: ec62be9
- 이전: AADS-188C 완료 (Claude Agent SDK 전환: agent_sdk_service+agent_hooks+chat_service execute/code_modify+18테스트) commit: a13ef4f
- 이전: AADS-188B 완료 (벡터 코드베이스 인덱싱: code_indexer_service+semantic_code_search+context_builder 연동+30테스트) commit: 8ae390d
- 이전: AADS-188A 완료 (Deep Research: research_stream AsyncGenerator+GOOGLE_GENAI_API_KEY+context/format 파라미터+Langfuse span+46테스트) commit: c36c927
- 이전: AADS-186E2-BRIDGE 완료 (PTC allowed_callers 6개 도구+deep_research_service+code_explorer_service+chat_service연동+59테스트 통과) commit: ff97268
- 이전: AADS-186E-2 완료 (Extended Thinking+PTC+4계층 영속 메모리+tests 41/41) commit: 004508e
- 이전: AADS-186E-1 완료 (Jina Reader+Crawl4AI+deep_crawl — jina_reader_service/crawl4ai_service/deep_crawl_service+도구3개+인텐트2개+tests 8/8) commit: dde8147
- 이전: AADS-186B 완료 (CKP 시스템 + CTO 모드 구현 — ckp_manager/cto_mode/ast_analyzer + .claude/ 5종 파일 + 인텐트 6개) commit: 50f6f5c
- 이전: AADS-186A 완료 (시스템 프롬프트 재설계 + 도구 고도화 — XML 섹션, Tool Examples, 신규 워크플로우 도구 3개)
- 이전: AADS-186C 완료 (Langfuse Observability + FastAPI-MCP + Telegram 알림 봇) commit: d7504b5
- 이전: AADS-184 완료 (채팅 도구 연동)
- 이전: AADS-172 완료 (Chat-First 프론트엔드 UI — 3-Panel 레이아웃 + 다크/라이트 + SSE 스트리밍)
- 이전: AADS-171 완료 (LiteLLM Proxy Docker + Gemini/Claude 듀얼 모델 라우팅 + 인텐트→모델 매핑 + 일 $5 비용상한)
- 서버: 68
- 도구: Claude
- 헬스체크: https://aads.newtalk.kr/
- 특수: LLM 15회/작업 이하, langgraph-supervisor 사용 금지
- LiteLLM: http://litellm:4000 (Docker 내부), Gemini 3종 + Claude 3종 + OpenRouter 5종 + Groq 7종, 일 $5 / 월 $150 상한

### b) SF/ShortFlow — 운영중
- 최근: SF-015
- 서버: 114
- 도구: Cursor/Claude
- 헬스체크: https://shotflow.newtalk.kr/api/health/
- 특수: gemini-2.0-flash 사용 금지(404), OAuth 토큰 비공개 유지

### c) GO100 — 운영중
- 최근: GO100-023
- 서버: 211
- 도구: Cursor
- 헬스체크: https://go100.newtalk.kr/manager/snapshot.json
- 특수: V4.1 코드 변경 불가, strategy-cards 변경 CEO 승인

### d) KIS — V4.1 운영
- 최근: KIS-041
- 서버: 211
- 도구: Cursor
- 헬스체크: HANDOVER 다중 엔드포인트
- 특수: GO100과 동일 서버/동일 제약

### e) NTV2 — Phase 1
- 최근: NT-001 (대기)
- 서버: 114
- 도구: Claude
- 헬스체크: http://114.207.244.86:8080/api/health
- 특수: Laravel 12, Docker 재시작 제한

### f) NAS — 유지보수
- 최근: NAS-010
- 서버: Cafe24
- 도구: Claude
- 헬스체크: 미정
- 특수: claudebot 사용자만 허용, Flask/FastAPI

---

## 6개 프로젝트 매니저 채팅 라우팅 테이블

| 프로젝트 | 매니저 채팅 URL | Task ID 접두사 | 서버 |
|----------|----------------|----------------|------|
| AADS | https://www.genspark.ai/agents?id=3d86d6f3-09a7-41b2-b91b-762a55512458 | AADS-xxx | 68 |
| KIS | https://www.genspark.ai/agents?id=77de652f-ca8c-4edb-b841-4ca3726b7bb4 | KIS-xxx | 211 |
| GO100 | https://www.genspark.ai/agents?id=167071cf-c8b5-476a-8953-6168dd6c910c | GO100-xxx | 211 |
| SF | (CEO 확인 필요) | SF-xxx | 114 |
| NTV2 | (CEO 확인 필요) | NT-xxx | 114 |
| NAS | (CEO 확인 필요) | NAS-xxx | Cafe24 |

---

## 진행 중 작업 상세

### 2026-04-04 완료 작업 (AADS-204)

| 항목 | 커밋 | 내용 |
|------|------|------|
| P0 401 인증 장애 복구 | — | credentials.json 제거(3서버) + TOKEN_1↔TOKEN_2 교환 + 컨테이너 재생성 |
| Gemini 폴백 모델 교체 | a713a55 | gemini-3.1-flash-lite-preview → gemini-2.5-flash (content=null 버그 수정) |
| QA 결과 파싱 버그 수정 | cd5edc4 | pipeline-runner.sh grep -qi 부분매칭으로 교체 (AUTO PASS/FAIL 인식) |
| Background 10종 qwen-turbo 전환 | 81a9129 | smart_search/kakaobot_ai/code_reviewer/self_evaluator/response_critic → qwen-turbo |
| Alibaba DashScope LiteLLM 연동 | 48d40f5 | ALIBABA_API_KEY docker-compose 주입, qwen-turbo/plus LiteLLM 등록 완료 |

### 2026-04-01 완료 작업

| 항목 | 커밋 | 내용 |
|------|------|------|
| hot-reload 자동 트리거 연결 | c758af6 | hot_reload 라우터 등록 + API 파일 추가 |
| hot-reload 라우터 등록 + API 파일 추가 | 8237470 | app/api/hot_reload.py 신규 + main.py 라우터 포함 커밋 |
| 인증 일일 체크 스케줄러 (AUTH-001) | main.py 패치 | 매일 09:05 KST _auth_daily_check() — TOKEN_1/TOKEN_2/LiteLLM 상태 텔레그램 보고 |
| AUTH_GUARD.md 작성 | docs/AUTH_GUARD.md | 인증 핵심 파일 목록, 수정 전후 체크리스트, 오류 조치 절차 |

---

## 대기 작업 큐

없음 (CEO 지시 대기)

---

## QA/디자인 에이전트 현황 (AADS-163, 2026-03-08)

### 3단계 품질 게이트 구조
```
Claude 코드 수정 → QA 에이전트(test-writer) → 디자인 에이전트(doc-writer) → RESULT_FILE
```

### QA 에이전트 (D-030)
- 파일: `/root/aads/.claude/agents/test-writer.md`
- 실행: `claude_exec.sh _run_qa_gate()` 함수
- 판정: `QA_VERDICT: PASS | FAIL`
- FAIL 시: 자동 재작업 최대 2회 → 서킷브레이커 카운트+1
- RESULT_FILE 필드: `qa_status: PASS | FAIL`

### 디자인 에이전트 (D-031)
- 파일: `/root/aads/.claude/agents/doc-writer.md`
- 실행: `claude_exec.sh _run_design_gate()` 함수
- 판정: `DESIGN_VERDICT: PASS | REVIEW_NEEDED`
- REVIEW_NEEDED 시: CEO Chat + Telegram 보고 → 60초 타임아웃 → PASS_TIMEOUT
- RESULT_FILE 필드: `design_status: PASS | PASS_TIMEOUT | REVIEW_NEEDED`

### 대시보드 /ops 페이지 확장 (AADS-163)
- QA Results 섹션: 최근 20건 `GET /ops/qa-results`
- Design Reviews 섹션: 최근 10건 `GET /ops/design-reviews` + 스크린샷 썸네일

---

## 보류/차단 항목 전체

| 항목 | 사유 | 해제 조건 | CEO 승인 필요 |
|------|------|-----------|--------------|
| NTV2 History 채널 OAuth | CEO 채널 ID/이메일 필요 | CEO가 채널 정보 제공 | 예 |
| Supabase 스키마 | CEO가 001_saas_schema.sql 실행 필요 | CEO 직접 실행 | 예 |
| CLOVA Voice TTS | Naver Cloud 계정 필요 | CEO가 계정 생성/제공 | 예 |
| Ubuntu 22.04 업그레이드 | Python 3.11 필요 시 | CEO 승인 후 진행 | 예 |

---

## 긴급 주의사항

- **GitHub PAT 만료**: 2026-05-27 (잔여 약 58일)
- **디스크 사용률**: 서버 114 = 79% (649GB/875GB) — 70% 초과 시 정리 권장
- **현재 장애**: 없음
- **절대 금지**: .env 커밋 금지, DB DROP 금지, 미승인 서비스 재시작 금지
- **OAuth 토큰**: 전체 만료 상태 — 재인증 필요 시 브라우저 통해 수행
- **Cloudflare**: OAuth 후 proxied DNS 유지
- **PostgreSQL 백업**: 메모리 레이어·세션 노트·관찰 데이터가 전부 DB에 있으므로 **일 1회 pg_dump 필수**. 스크립트: `aads-server/scripts/backup_postgres.sh` (docker exec 방식). cron: `0 3 * * * .../backup_postgres.sh >> .../logs/backup.log`. 상세: [AADS-BACKUP-STRATEGY-20260309.md](https://github.com/moongoby-GO100/aads-docs/blob/main/reports/AADS-BACKUP-STRATEGY-20260309.md)

---

## CEO-DIRECTIVES 전문 요약 (v3.4)

### 섹션 1: 사고방식 원칙
- D-001: 단순 사고 금지 — 하나를 던지면 10을 생각하고 연구해서 반영
- D-002: 사소한 것도 빠짐없이 — 누락 없이 비교
- D-003: AADS의 본질 — AI 에이전트 조직 시뮬레이션, 단일 앱 빌더와 다름
- D-004: 비용 효율 최우선 — 월 $23~$63 목표, 프롬프트 캐싱+배치 활용
- D-005: 컨텍스트 패키지 시스템 — HANDOVER.md가 생명줄
- D-006: 교차검증 필수 — 결과 무비판 수용 금지
- D-007: 도구 vs 에이전트 구분 — Cursor=도구, AADS 내부=에이전트
- D-008: 속도 우선 — MVP 먼저 배포, 이후 반복 개선
- D-009: 현실적 자율성 인식 — 멀티 에이전트 실패율 41-87%, 점진적 확대
- D-010: 사용자 중심 체크포인트 — 6단계 CEO 개입 지점
- D-011: Sandbox 2단계 전략 — Docker 로컬 기본, 대형은 사용자 서버 SSH
- D-012: 인프라 컨설팅 서비스 — Architect Agent 서버 사양 제안
- D-013: Frontend Dual Strategy — 2026년 Genspark+Dashboard 병행
- D-014: 콘텐츠 품질 게이트 — 벤치마크 85% 이상 필수
- D-015: 68서버 프로덕션 유지 결정 — aads.newtalk.kr 고정
- D-016: FLOW 프레임워크 — Find→Layout→Operate→Wrap up
- D-017: 소스코드 모듈화 원칙 — agents/graphs/models/services 분리
- D-018: 4계층 자기치유 — L1 하트비트 기반
- D-019: 서버 상호 감시 — 3서버 2분 크로스
- D-020: 복구 이력 DB 의무화 — recovery_logs
- D-021: 하트비트 기반 세션 관리
- D-022: 지시서 포맷 v2.0 — 필수 6필드 + 선택 7필드
- **[NEW] D-023 v2**: HANDOVER 4계층 운영 원칙 — 토큰 상한 없음, 생략 금지, CEO 질문 0회 기준
- D-024: 모델 라우팅 — XS→haiku, S/M→sonnet, L/XL→opus
- D-025: 우선순위큐 impact/effort 정렬
- D-026: STATUS.md 브라우저 자동화 실패 복구 경로 (AADS-147)
- D-027: Worktree 병렬 실행 — parallel_group 필드 감지 시 자동 분기 (AADS-146)
- D-028: 서브에이전트 패턴 — subagents 필드 기반 보안/테스트/문서 에이전트 (AADS-146)
- D-029: Writer/Reviewer — P0/P1 review_required:true 시 리뷰 세션 자동 스폰 (AADS-146)
- **[NEW] D-033**: Core 문서 운영 원칙 섹션 상시 유지 — 삭제/축약 불가
- **[NEW] D-034**: HANDOVER 업데이트 WRAP 게이트 — git diff에 HANDOVER.md 미포함 시 차단
- **[NEW] D-035**: bridge.py 자동 감지 원칙 — 매니저가 채팅창에 지시서 출력 → bridge.py 자동 감지·추출·저장. CEO 전달 요청 금지.
- **[NEW] D-036**: 매니저 자기인식 의무 — 세션 시작 시 3가지 검증(프로젝트명, Task ID, 문서 URL) 수행 필수
- **[NEW] D-037**: CEO 전달 요청 금지 — 매니저→CEO 지시서 전달 요청 절대 금지. 위반 시 지시서 무효.
- **[NEW] D-030**: QA 에이전트 의무화 — claude_exec 작업 완료 후 test-writer QA 에이전트 자동 실행. FAIL 시 최대 2회 재작업 + 서킷브레이커 연동. RESULT_FILE qa_status 필수 기록.
- **[NEW] D-031**: 디자인 검증 의무화 — QA PASS 후 doc-writer 디자인 에이전트 자동 실행. REVIEW_NEEDED 시 CEO Chat 보고 + 60초 타임아웃. RESULT_FILE design_status 필수 기록.
- **[NEW] D-038**: 프로세스 그룹 격리 원칙 — 모든 claude_exec 파생 실행은 set -m으로 별도 PGID 생성 필수. 종료 시 kill -TERM -PGID → 3초 → kill -9 -PGID → pkill 'claude.*stream-json'. PID 파일 /root/.genspark/pids/${TASK_ID}.pid 유지. (AADS-167)

### 섹션 2: 기술적 지시
- T-001: 멀티 에이전트 아키텍처 — LangGraph Native StateGraph, langgraph-supervisor 금지
- T-002: 에이전트 역할 구성 — 8 에이전트 (Supervisor/Architect/PM/Developer/QA/Judge/DevOps/Researcher)
- T-003: MCP 서버 스택 — Phase 1 필수 7개 + Phase 2 확장 3개
- T-004: 샌드박스 전략 — Docker 로컬 기본, SSH 원격 대형
- T-005: AADS 기술 스택 — LangGraph + FastAPI + Next.js + PostgreSQL
- T-006: 로드맵 — Phase 0~6 점진적 자율성 확대
- T-007: 에이전트 간 통신 프로토콜 — TaskSpec JSON
- T-008: Judge Agent 품질 게이트 — pass/fail/conditional_pass
- T-009: 점진적 자율성 게이트 — 성공률 90% 이상 시 자동 승인 전환
- T-010: 수익 모델 — SaaS 티어 Free~Enterprise
- T-011: 5-Layer Memory Architecture — L1~L5

### 섹션 3: 절대 규칙
- R-001: HANDOVER 업데이트 없이 완료 선언 금지
- R-002: 보고서 GitHub push + HTTP 200 확인
- R-003: .env/시크릿/API키 커밋 금지
- R-004: 프로덕션 DB 직접 편집 금지
- R-005: 서버 서비스 임의 재시작 금지
- R-006: CEO-DIRECTIVES 위반 설계/분석 무효
- R-007: 기존 서비스 중단/변경 CEO 승인 필수
- R-008: CEO 보고 시 GitHub 브라우저 경로 사용
- R-009: 작업 완료 보고 형식 준수
- R-010: langgraph-supervisor 프로덕션 사용 금지
- R-011: Supabase 직접 연결(5432) 사용, Supavisor/PgBouncer 금지
- R-012: 작업당 LLM 호출 최대 15회
- R-013: Task ID 접두사 체계 — AADS/KIS/GO100/SF/NT/SALES/NAS
- R-014: Wrap up 의무화 — WRAP 게이트
- R-015: 교훈 등록 — shared/lessons/
- R-016: 서킷브레이커 준수 — 3회 실패 → 5분 쿨다운
- R-017: git-push 감시 — commit SHA + HTTP 200
- R-018: WORKDIR 이탈 금지
- R-019: 중복 태스크 차단
- R-020: 의존성(DEPENDS_ON) 선행 충족 후 실행
- **[NEW] R-021**: HANDOVER 업데이트 의무 강화 — 토큰 절약 목적 생략 = R-VIOLATION
- **[NEW] R-022**: CEO 전달 요청 금지 위반 — R-VIOLATION. 지시서 무효 + 재작성

---

## 이번 세션 작업 (2026-03-08)

### 1. AADS-148: HANDOVER 4계층 전면 재작성
- HANDOVER.md v10.0 전면 재작성 (11개 섹션, 운영 원칙 최상단, 토큰 상한 폐기)
- HANDOVER-RULES.md v1.0 신규 생성 (13개 섹션, 파이프라인/매니저/작업자/효율성12전략)
- CEO-DIRECTIVES.md v3.3 (D-023 v2 교체, D-033/D-034/R-021 신규)
- RULE-MATRIX.md v1.2 (19→23규칙)
- WORKFLOW-PIPELINE.md v3.2 (Step 6 HANDOVER_UPDATE_CHECK 게이트)
- aads-docs commit: 3edeeca

### 2. 대시보드 채널 카드 문서 링크 + 편집 기능
- 각 프로젝트 카드에 중요 문서 링크 뱃지 추가 (HANDOVER/RULES/CEO-DIRECTIVES/STATUS 등)
- **"편집" 버튼** → 모달에서 문서 추가/삭제/수정 가능
- **저장 + Push**: DB 저장 + aads-docs/shared/project-docs.json 자동 commit+push
- 데이터 소스 3단계 폴백: DB → GitHub raw JSON → 하드코딩 기본값
- 30초 주기 자동 갱신 (작업자 수정 실시간 반영)
- aads-dashboard commits: 024ce1e, 31b01b3

### 3. project-docs 백엔드 API
- `POST /api/v1/ops/sync-project-docs`: DB 저장 + git auto-push
- `GET /api/v1/ops/project-docs`: DB에서 프로젝트별 문서 링크 조회
- aads-server commit: 8185ccd

### 4. CEO 문서 등록 (2026-03-07, 기존 미반영분)
- TECH-ARCH-001: AADS 시스템 아키텍처 v1.0 (인프라, API 50+, DB 13테이블, 에이전트 8개)
- TECH-PIPE-001: LangGraph 8-에이전트 파이프라인 기술 감사 (전체 구현 완료)
- STATUS-SESSION-001: CEO 직접 검수 세션 보고서 (버그 4건 + 모델드롭박스 + KST)
- TECH-CHART-001: KIS/GO100 차트 대시보드 웹 경로 맵
- architecture/ 디렉토리 신규: AADS_ARCHITECTURE_v1.0.md, LANGGRAPH_PIPELINE_AUDIT_v1.0.md
- aads-docs commits: 10c2ec3, ff9c620

### 5. Docker TZ 수정 (2026-03-07, 기존 미반영분)
- 전 컨테이너 TZ=Asia/Seoul 설정 (aads-server Dockerfile + docker-compose)
- aads-dashboard Dockerfile에 tzdata 설치 (Alpine KST 동기화)
- aads-server commit: 7bd1516 | aads-dashboard commit: 52c8486

---

## AADS-161 /tasks 문서뷰어 마크다운 렌더링 (2026-03-08)
- **MarkdownRenderer 컴포넌트** (inline, 외부 의존성 없음): h1~h4, 코드블록, 테이블, bold/italic/inline-code, ul/ol, blockquote, hr
- **Mermaid 블록 특별 렌더링**: 파란색 테두리 + "diagram (mermaid)" 레이블
- **DocumentsTab `<pre>` 교체**: MarkdownRenderer 적용, max-height 600px 스크롤
- **카테고리 필터 검증**: d.type ↔ typeFilter 일치 (tech/plan/research/status/directive), 동작 정상
- **TECH-ARCH-001 섹션 3**: 8-Stage 파이프라인 ASCII art → mermaid flowchart LR
- **TECH-ARCH-001 섹션 4**: LangGraph 에이전트 파이프라인 ASCII art → mermaid flowchart TD
- aads-dashboard commit: 2a9ad8a | aads-docs commit: 095ea4d

## AADS-166 파이프라인 전체 헬스체크 + 실시간 SSE 스트리밍 (2026-03-08)
- **8파트 통합 구현**: 6개 API + 대시보드 Pipeline 탭 + SSE 스트리밍
- **Part 1**: `GET /api/v1/directives/{status}` — pending/running/done/archived 폴더 스캔
- **Part 2**: `GET /api/v1/ops/pipeline-status` — bridge/auto_trigger/watchdog/claude_exec 프로세스 liveness
- **Part 3**: `GET /api/v1/ops/infra-check` — DB/GitHub PAT/SSH/디스크/메모리/CPU 병렬 점검
- **Part 4**: `GET /api/v1/ops/consistency-check` — STATUS.md↔DB, pending↔큐 교차검증
- **Part 5**: `GET /api/v1/ops/full-health` — Part 1~4 + 기존 health-check 통합, HEALTHY/DEGRADED/CRITICAL
- **Part 6**: CEO Chat `health_check` 인텐트 (12→추가) — "헬스체크", "전체 점검" 등 키워드 → full-health 호출 → 한국어 요약
- **Part 7**: `GET /api/v1/ops/stream` SSE — 5초 주기 health/directive/pipeline 3이벤트, 최대 5연결
- **Part 8**: 대시보드 Pipeline 탭 (6탭 체제), PipelineHealthCard 4카드, Header 상태 dot, 정체 태스크 빨간 배경
- **신규 파일**: `app/services/health_checker.py`, `src/components/PipelineHealthCard.tsx`, `src/hooks/useSSE.ts`
- aads-server commit: 9a44646 | aads-dashboard commit: 16c67bc

## AADS-165 CEO Chat 크로스 프로젝트 코드 접근 + 실행 검증 (2026-03-08)
- **C+ 하이브리드 3단계**: 1단계 SSH 즉시 정적 분석 → 2단계 claudebot 지시서 위임 → 3단계 서브에이전트 프로파일
- **SSH 원격 도구 2개** (ceo_chat_tools.py): `list_remote_dir`, `read_remote_file`
  - 프로젝트-서버 매핑: KIS→211, GO100→211, SF→114, NTV2→114
  - 보안: 명령 인젝션 차단, 민감 파일 차단, WORKDIR 탈출 차단, 10초 타임아웃, 50KB 제한
- **Intent Classifier 확장** (10→12분류): `execution_verify` 인텐트 추가
  - 프로젝트명 + QA 키워드 조합 → qa 인텐트 (예: "KIS 백테스트 검수" → qa)
  - 프로젝트명만 → 기존 인텐트 유지 (예: "KIS 상태" → dashboard)
- **크로스 프로젝트 QA** (_handle_cross_project_qa): SSH→파일 탐색→코드 읽기→LLM 정적 분석
- **실행 검증 지시서** (_handle_execution_verify_intent): 세션 메모리 기반 claudebot 지시서 자동 생성+제출
- **정적 분석 프롬프트**: 코드 품질/로직 오류/보안/성능/테스트 5관점 + PASS/WARNING/FAIL 판정
- TOOL_DEFINITIONS: 11→13개 (list_remote_dir, read_remote_file 추가)
- aads-server commit: (pending)

## AADS-164 CEO Chat Agent Individual Call System (2026-03-08)
- Intent Classifier 6→10분류 확장: qa, design, design_fix, architect 추가
- 우선순위: design_fix > design > qa > architect > execute > browser > dashboard > diagnosis > research > strategy
- **QA 의도 (qa)**: "QA 진행해", "테스트해" → qa_node + judge_node 실행, 결과 요약 보고
- **디자인 검수 (design)**: "디자인 검수해" → 스크린샷 + Claude Vision UI/UX 분석
- **디자인 수정 (design_fix)**: "디자인 수정해" → Vision 분석 + developer_node 코드 수정
- **설계 검토 (architect)**: "설계 검토해" → architect_node 시스템 설계 JSON 생성
- agent_state_builder.py: 경량 AADSState 빌더 (LangGraph 에이전트 노드용 최소 상태)
- agent_executions DB 테이블: 실행 이력 추적 (session_id, agent_type, cost, duration)
- 기존 6개 의도 (dashboard/diagnosis/research/execute/browser/strategy) 정상 동작 유지
- aads-server commit: 59d6491

## AADS-160 CEO 직접 검수 + 버그수정 + UI개선 (2026-03-07)
- CEO 직접 AADS-157/158/159 검수 → 버그 4건 발견 및 수정
- **CEO Chat TypeError 2건**: logger.info structlog kwargs → f-string 변환 (9f496d1, 3d7d80d)
- **Dashboard React Error #31**: object를 React children으로 렌더링 → typeof 체크 + JSON.stringify (49311d1)
- **Ops toLocaleString/toFixed crash**: undefined 값에 메서드 호출 → nullish coalescing (49311d1, 33fa3ed)
- **모델 선택 드롭박스**: 버튼 7개 → select 드롭박스 29개 모델 (auto+Anthropic 11+OpenAI 11+Google 6) (af082cb)
- **KST 타임존 동기화**: 8개 파일 날짜를 Asia/Seoul timezone으로 통일 (fd51915)
- aads-server commits: 9f496d1, 3d7d80d
- aads-dashboard commits: 49311d1, 33fa3ed, af082cb, fd51915

## AADS-159 주요 변경 (2026-03-07)
- CEO Chat Playwright 브라우저 자동화 6개 도구 추가 (T-003 Phase 2 확장)
- ceo_chat_tools.py: browser_navigate/snapshot/screenshot/click/fill/tab_list
- 도메인 화이트리스트: *.newtalk.kr, github.com, raw.githubusercontent.com, localhost
- Playwright Python 싱글턴 컨텍스트 (asyncio.Lock, headless Chromium)
- aads-server commit: 1fbb76d

## AADS-158 주요 변경 (2026-03-07)
- Pending 대기큐 정리: 11개 완료/중복 지시서 → archived/ 이동
- STATUS.md: last_completed=AADS-157 → 갱신 완료

## AADS-157 주요 변경 (2026-03-07)
- CEO Chat v2 → AADS Core Engine 연결: Intent Classifier + DashboardCollector + Tool-use 루프 + Directive Submit
- classify_intent(): 5분류 → AADS-164에서 10분류로 확장
- ceo_chat_tools.py: read_file, read_github, search_logs, query_db, fetch_url
- directives.py: POST /api/v1/directives/submit
- aads-server commit: 65edfde

## 복구 경로 (AADS-147)
브라우저 자동화 실패 시 → CEO가 매니저 대화창에 "상태확인" 입력
→ 매니저가 STATUS.md(context_docs) 읽어 chat_delivered=false 완료 작업 인식
→ report_url 보고서 확인 후 다음 지시 생성 (복구 소요 ~5초)
STATUS.md: https://raw.githubusercontent.com/moongoby-GO100/aads-docs/main/STATUS.md

---

## 참조 문서 링크

| 문서 | GitHub 브라우저 URL | 설명 |
|------|---------------------|------|
| HANDOVER-RULES.md | https://github.com/moongoby-GO100/aads-docs/blob/main/HANDOVER-RULES.md | 파이프라인, 매니저/작업자 규칙, 지시서 포맷, 효율성 전략, 비용 규칙 |
| HANDOVER-HISTORY.md | https://github.com/moongoby-GO100/aads-docs/blob/main/HANDOVER-HISTORY.md | 최근 10건 완료 작업 상세 이력 |
| HANDOVER-ARCHIVE.md | https://github.com/moongoby-GO100/aads-docs/blob/main/HANDOVER-ARCHIVE.md | 전체 이력 보관소 + 4계층 자기치유 상세 |
| CEO-DIRECTIVES.md (v3.4) | https://github.com/moongoby-GO100/aads-docs/blob/main/CEO-DIRECTIVES.md | CEO 지시 전문 (D-001~D-037, R-001~R-022, T-001~T-011) |
| WORKFLOW-PIPELINE.md (v3.3) | https://github.com/moongoby-GO100/aads-docs/blob/main/shared/rules/WORKFLOW-PIPELINE.md | 8단계 파이프라인 상세 + 라우팅 + 모델 라우팅 |
| RULE-MATRIX.md (v1.3) | https://github.com/moongoby-GO100/aads-docs/blob/main/shared/rules/RULE-MATRIX.md | 23규칙 x 8단계 매핑 매트릭스 |
| STATUS.md | https://github.com/moongoby-GO100/aads-docs/blob/main/STATUS.md | 실시간 작업 상태 (last_completed, next_pending) |
| AADS-KNOWLEDGE.md | /root/aads/aads-server/docs/knowledge/AADS-KNOWLEDGE.md | AADS 전용 지식 |
| AADS_ARCHITECTURE_v1.0 | https://github.com/moongoby-GO100/aads-docs/blob/main/architecture/AADS_ARCHITECTURE_v1.0.md | 시스템 아키텍처 (인프라, API 50+, DB 13테이블) |
| LANGGRAPH_PIPELINE_AUDIT | https://github.com/moongoby-GO100/aads-docs/blob/main/architecture/LANGGRAPH_PIPELINE_AUDIT_v1.0.md | 8-에이전트 파이프라인 기술 감사 |
| project-docs.json | https://github.com/moongoby-GO100/aads-docs/blob/main/shared/project-docs.json | 대시보드 문서 링크 설정 (자동 push) |
| AADS-BACKUP-STRATEGY | https://github.com/moongoby-GO100/aads-docs/blob/main/reports/AADS-BACKUP-STRATEGY-20260309.md | 백업·복원 정책 (PostgreSQL 일일 pg_dump, .env, Nginx/systemd, Docker 볼륨) |
| CTO-SYSTEM-MAP.md | /root/aads/aads-server/docs/knowledge/CTO-SYSTEM-MAP.md | CTO 세션 컨텍스트 복원 (8.6KB, 전체 아키텍처 지도) |
| CTO-SYSTEM-MAP.md | /root/aads/aads-server/docs/knowledge/CTO-SYSTEM-MAP.md | CTO 세션 컨텍스트 복원 (8.6KB, 전체 아키텍처 지도) |

---

## AADS-168 멀티서버 클로드봇 프로세스 감시 데몬 (2026-03-08)
- **파일**: `/root/aads/scripts/claude_watchdog.py` (실행 가능, Python 3.6 호환)
- **감시 범위**: 서버 68(localhost) · 211(SSH) · 114(SSH) — claude_exec/headless/session_watchdog/좀비/bridge.py
- **임계값**: claude_exec >7200s CRITICAL, headless >3600s CRITICAL, session_watchdog >3600s HIGH, 좀비>3개 HIGH
- **자동 정리(CRITICAL)**: kill -TERM -{PGID} → 3초 → kill -9 + pkill 'claude.*stream-json' + DB error 갱신
- **서버 211 전용**: bridge.py 미실행 → SSH 자동 재시작, 고스트 PID 파일 자동 삭제
- **Telegram 알림**: CRITICAL 자동정리 결과 + HIGH 이슈 요약
- **JSON 보고서**: `/root/aads/logs/watchdog_reports/{YYYYMMDD_HHMMSS}.json` (30일 자동 삭제)
- **3개 신규 API** (ops.py): `GET /ops/claude-processes`, `POST /ops/claude-cleanup`, `POST /ops/bridge-restart`
- **systemd 파일 스테이징**: `/root/aads/scripts/systemd/` — `sudo bash /root/aads/scripts/install_claude_watchdog.sh`로 root 설치
- **테스트**: 서버 68 로컬 스캔 정상 (9 claude_exec, 9 claude -p, 3 session_watchdog), JSON 보고서 생성 확인
- aads-server commit: (이번 커밋)

---

## AADS-167 claude_exec_safe.sh 안전 래퍼 + 3서버 설정 배포 (2026-03-08)
- **claude_exec_safe.sh 신규 생성**: set -m 프로세스 그룹 격리, timeout --kill-after=60 이중 타임아웃(7200초), --max-turns 50 --max-budget-usd $MAX_BUDGET Claude 내부 제한
- **PID 관리**: /root/.genspark/pids/${TASK_ID}.pid (PID|PGID|시작시간|TASK_ID), 완료 시 자동 삭제
- **PGID 종료 체인**: kill -TERM -PGID → 3초 대기 → kill -9 -PGID + pkill 'claude.*stream-json'
- **ulimit -u 500**: 프로세스 폭발 방지
- **exit 124/137 → /ops/directive-lifecycle**: DB 오류 상태 자동 기록
- **settings.json 업데이트**: BASH_DEFAULT_TIMEOUT_MS=3600000, BASH_MAX_TIMEOUT_MS=7200000, CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY=1 (서버 68 적용)
- **백업**: /root/aads/claude_exec.sh.bak.20260308
- **D-038 (신규)**: 프로세스 그룹 격리 원칙 — 모든 claude_exec 파생 프로세스는 set -m으로 별도 PGID 생성 필수. SSH 불가로 서버 211/114 원격 배포는 수동 실행 필요
- aads-server commit: 337b50d

---

## AADS-169 대시보드 Claude Bot Status 카드 + 프로세스 제어 버튼 (2026-03-08)
- **Part 1**: Claude Bot Status 카드 (Pipeline Health 탭 내) — 서버별(68/211/114) claude 프로세스 수, 좀비 수, bridge/auto_trigger 상태(211 전용), 마지막 스캔 시각, 이슈 수
- **Part 2**: 제어 버튼 3개 — "좀비 정리" (POST /ops/claude-cleanup), "Bridge 재시작" (POST /ops/bridge-restart), "전체 스캔" (dry_run=true)
  - CEO 확인 팝업: 각 버튼마다 확인 모달 + 결과 JSON 표시 모달
- **Part 3**: SSE claude_watchdog 이벤트 연동 — useSSE.ts에 SSEClaudeWatchdog 타입 + 리스너 추가, 이벤트 수신 시 자동 갱신
  - ops.py SSE 스트림: claude_watchdog 4번째 이벤트 추가 (5초 주기, 최신 보고서 요약)
- **Part 4**: Recent Cleanup Actions 테이블 — watchdog 보고서의 cleanup_log 기반, 최근 20건, 열: 시각/서버/타입/PID/결과
- **백엔드**: ClaudeCleanupRequest에 dry_run 필드 추가 — True시 스크립트 실행 없이 최신 보고서만 반환
- aads-server commit: fbe5b75 | aads-dashboard commit: 4c12a57

## AADS-181 전체 프로젝트 통합 작업 현황 API + /tasks 페이지 실시간 연동 (2026-03-08)

- **server_registry.py** 신규: 3서버(68/211/114) 접근정보 레지스트리 + 프로젝트 매핑 (AADS→68, KIS/GO100→211, SF/NTV2/NAS→114)
- **cross_server_checker.py** 신규: 크로스서버 디렉티브 스캐너
  - 서버 68: 로컬 파일 직접 스캔, 서버 211/114: SSH 일괄 스캔 (실패 시 method=ssh_failed)
  - 30초 TTL 캐싱 (반복 SSH 방지), asyncio.gather 병렬 3서버 동시 스캔
  - 파일 내용 파싱: TASK_ID/TITLE/PRIORITY/MODEL/project 필드 추출
- **GET /api/v1/directives/all** 신규: status/project 필터 + force_refresh 파라미터
  - 응답: total_count, counts{pending/running/done/archived}, by_server{68/211/114}, directives[]
- **GET /api/v1/ops/server-summary** 신규: 3서버 pending/running/done 건수 + active_claude_sessions
- **SSE cross_server_directives 이벤트** 추가: 30초 주기 (6 * 5s tick), 변경 감지 시 발송
- **기존 GET /api/v1/directives/{status} 하위 호환 유지**
- **taskApi.ts** 신규: getAllDirectives() + getServerSummary() API 함수
- **useTaskPolling.ts** 신규: 30초 자동갱신 훅, SSE cross_server_directives 수신 시 즉시갱신, 폴링 fallback
- **TaskTable.tsx** 신규: 서버/프로젝트/상태/우선순위 뱃지, 서버 컬럼 68(파란)/211(보라)/114(주황)
- **tasks/page.tsx DirectivesTab 전면 개편**:
  - 상단 KPI 카드: 3서버 합산 수치 (pending/running/done) + 서버별 총계 뱃지
  - 전체/KIS/GO100/SF/NTV2/NAS 탭: 크로스서버 데이터 → TaskTable (서버 뱃지 포함)
  - AADS 탭: 기존 DB 데이터 + 서버 68 뱃지 (하위 호환 유지)
  - 상태 필터: all/running/pending/done/archived (폴더 기반)
  - 마지막 갱신 시각 표시, 탭 visibility change 즉시 갱신
- **검증**: /directives/all 200 (43건, 서버 68), /ops/server-summary 200, npx tsc 오류 없음, npm run build 성공
- aads-server commit: 102984d | aads-dashboard commit: 49289ac

## AADS-185 CEO Chat 전면 재설계 완료 (2026-03-08)

- **개요**: AADS-170/180~184 패치 접근 폐기 → Context Engineering + Tool Use API + Gemini 검색/리서치 전면 재설계
- **백엔드 Core (185-A)**:
  - `context_builder.py` 재작성: 3계층 (Layer1 정적 ~1500토큰 캐싱, Layer2 동적 ~300토큰, Layer3 히스토리 압축)
  - `intent_router.py` 신규: Gemini Flash-Lite로 인텐트 분류 (10개 인텐트 → LiteLLM/Gemini/Claude 라우팅)
  - `model_selector.py` 신규: LiteLLM 프록시(Gemini) vs Anthropic SDK 직접(Claude Tool Use + Extended Thinking + Prompt Caching)
  - `compaction_service.py` 신규: 20턴 초과 시 Claude Haiku 자동 요약 압축
  - `chat_service.py` send_message_stream 전면 재작성: 3계층→압축→인텐트→도구→스트리밍 파이프라인
  - `migrations/021_chat_185_schema.sql`: chat_messages 6컬럼 추가 (intent/model_used/tools_called/cost/tokens_in/tokens_out/is_compacted/thinking_summary) + session_notes 신규
- **도구+검색 (185-B)**:
  - `tool_registry.py` 신규: 10개 도구 Anthropic Tool Use 포맷 (system/action/search 그룹)
  - `tool_executor.py` 전면 재작성: ToolExecutor 클래스 + 10초 타임아웃 + 6000자 제한
  - `gemini_search_service.py` 신규: Gemini Flash + Google Search Grounding → CitationCard 데이터
  - `gemini_research_service.py` 신규: Gemini Deep Research 비동기 SSE 스트리밍
  - `brave_search_service.py` 신규: Brave Search API 폴백 검색
- **프론트엔드 (185-C)**:
  - `useChatSSE.ts` 확장: thinking/tool_use/tool_result/research.start/progress/complete 이벤트 + ToolUseEvent 타입
  - `ChatStream.tsx` 확장: ToolEventBlock(접이식)/ThinkingIndicator/CitationCard/ResearchProgress 통합
  - `ThinkingIndicator.tsx` 신규: Extended Thinking 접이식 + 타이핑 애니메이션
  - `CitationCard.tsx` 신규: 검색 출처 카드 (favicon + 제목 + URL), 최대 5개
  - `ResearchProgress.tsx` 신규: Deep Research 진행 바 + "보고서 보기" 버튼
  - `chatApi.ts` SSEChunk 타입 확장 (11개 이벤트 타입)
  - `docker-compose.prod.yml`: `aads-server` 서비스에 GEMINI_API_KEY/BRAVE_API_KEY 환경변수 추가 (이번 세션)
  - `pyproject.toml`: `google-genai>=1.0.0` 의존성 추가 (이번 세션)
- **TypeScript 빌드**: tsc --noEmit 오류 없음 (0 errors)
- **비용 목표**: 일반 80% Gemini ~$0.004/건, 시스템 15% Sonnet ~$0.03/건, 분석 5% Opus ~$0.15/건
- **확장 (2026-03-09)**: 채팅창 서버(SSH) 파일 검색 반영·배포 완료
  - 백엔드: `tool_registry.py`에 `list_remote_dir` 도구 추가, `read_remote_file`에 `project`+`path` 필수화
  - `tool_executor.py`: `_list_remote_dir`/`_read_remote_file`(ceo_chat_tools 연동), 인텐트 `server_file` 매핑
  - `intent_router.py`: 인텐트 `server_file` 및 키워드 폴백(서버 검색, 원격 서버, SSH 파일, KIS/SF/NTV2 서버)
  - 프론트: `ChatStream.tsx` 도구 표시명 `list_remote_dir`→"서버 파일 검색 (SSH)", `read_remote_file`→"원격 서버 파일 읽기"
  - `ActionChips.tsx`: 웰컴 칩 "작업현황", "서버 파일 검색" 추가; 동적 칩(서버 파일 검색·원격 파일 읽기·작업 이력 더)
  - 배포: 2026-03-09 68서버 docker compose build+up 완료. 보고서: reports/chat_server_search_frontend_report_20260306.md

## AADS-186D 전체 통합 + 원격 CKP + Tool Search Tool + Prompt Caching 완료 (2026-03-09)

- **원격 프로젝트 CKP 5종**:
  - `.claude/projects/{KIS,GO100,SF,NTV2,NAS}/` — 각 5개 파일(CLAUDE.md/ARCHITECTURE.md/CODEBASE-MAP.md/DEPENDENCY-MAP.md/LESSONS.md) = 25파일
  - `ckp_manager.scan_remote_project()`: staged HANDOVER → `.claude/projects/` 우선 탐색 + DB 등록
  - `ckp_manager.get_ckp_summary()`: AADS/CEO = `.claude/`, 원격 프로젝트 = `.claude/projects/{PROJECT}/`
  - `context_builder._build_ckp_layer()`: KIS/GO100/SF/NTV2/NAS 워크스페이스에서도 해당 CKP 주입

- **Tool Search Tool — defer_loading 메타데이터**:
  - `tool_registry.py`: `_DEFER_LOADING` 딕셔너리 (상시 4개 + 온디맨드 10+개)
  - 상시 로드 (eager): `health_check`, `directive_create`, `get_all_service_status`, `generate_directive`
  - `TOOL_CATEGORY_GUIDE`: 도구 카테고리 안내 텍스트 (시스템 프롬프트 주입용)
  - `ToolRegistry.get_eager_tools()` / `get_deferred_tools()` / `get_tool_category_guide()` / `is_deferred()` 신규

- **Prompt Caching 최적화**:
  - `app/core/cache_config.py` 신규: `make_cacheable_block()`, `build_cached_system_blocks()`, `build_cached_tools()`, `estimate_cache_savings()`
  - `context_builder.build()`: `cache_config.build_cached_system_blocks()` 통합 적용 (Layer1 강제 캐시)
  - Layer 1 (~1400t): force=True 캐시 / CKP (~1500t): 토큰 기반 캐시 / Layer 2: 비캐시

- **Context Builder 최종 통합**:
  - `_build_tool_guide_layer()` 신규: TOOL_CATEGORY_GUIDE Layer 1 보조 주입
  - Layer 1 = build_layer1 + tool_guide (~1500t) | Layer 2 = 동적+CKP (~2000t) | Layer 3 = 대화

- **주간 CEO 브리핑 (APScheduler)**:
  - `main.py`: `_run_weekly_briefing()` 추가 — 매주 월요일 09:00 KST (UTC 00:00)
  - 6개 프로젝트 CKP 변경 요약 + 7일 비용 + Telegram 발송

- **통합 테스트**:
  - `tests/test_integration.py` 신규: 7시나리오 × 31 테스트 케이스 — 31/31 통과

## AADS-186C Langfuse Observability + FastAPI-MCP + Telegram 알림 봇 완료 (2026-03-09)

- **파트 1 — Langfuse 셀프호스팅**:
  - `docker-compose.langfuse.yml`: Langfuse v3 Docker + 별도 PostgreSQL (langfuse_db), 포트 3001
  - `scripts/setup_langfuse.sh`: 컨테이너 기동 + 헬스체크 대기 + 초기 계정 안내 출력
  - `app/core/langfuse_config.py`: SDK 초기화, LiteLLM 콜백 설정, create_trace(), flush_langfuse(); 미설정 시 graceful 비활성화
  - `app/services/chat_service.py`: send_message_stream에 intent_classification + llm_generation span 자동 추가

- **파트 2 — FastAPI-MCP**:
  - `app/core/mcp_server.py`: FastApiMCP 마운트 래퍼; MCP_ENABLED=false 또는 미설치 시 graceful skip
  - `app/main.py`: setup_mcp(app) 호출 — AADS API 엔드포인트 MCP 도구로 자동 노출

- **파트 3 — Telegram 알림 봇**:
  - `migrations/023_alert_history.sql`: alert_history 테이블 + 3개 인덱스
  - `app/services/alert_manager.py`: RULES 8개, evaluate_rules(), send_alert(), 1시간 중복방지, get_active_alerts()
  - `app/services/telegram_bot.py`: TelegramBot (send_alert, send_daily_summary, handle_command: /status /cost /alerts); 미설정 시 graceful 비활성화

- **공통**:
  - `pyproject.toml`: apscheduler, langfuse, fastapi-mcp, python-telegram-bot 추가
  - `app/main.py`: APScheduler (2분 주기 evaluate_rules, 09:00 KST daily_summary), Langfuse+Telegram lifespan 초기화
  - `.env.example` / `docker-compose.yml`: LANGFUSE_*/TELEGRAM_*/MCP_ENABLED 환경변수 추가
  - `tests/test_observability.py`: 15개 단위 테스트 (Langfuse/AlertManager/TelegramBot/MCP)
  - aads-server commit: d7504b5

## AADS-184 채팅 도구 연동 구현 완료 (2026-03-08)

- **문제**: 채팅 AI가 인텐트만 분류하고 실제 도구 호출 없이 추측 기반 응답
- **구현**: 인텐트→도구 호출→실제 데이터 수집→LLM에 주입→정확한 응답 파이프라인
- **신규 파일 1**: `aads-server/app/services/chat_tools.py`
  - 9개 도구 함수: health_check / dashboard_query / search_web / read_github_file /
    query_database / read_remote_file / fetch_url / generate_directive / list_workspaces_sessions
  - health_check: quick_health + _check_db + _check_disk 병렬 조회 (0.23s, SSH/외부HTTP 제외)
  - dashboard_query: directives 폴더 스캔 + DB 최근 완료 10건
  - search_web: Brave Search API (BRAVE_API_KEY 환경변수 필요)
  - read_github_file: raw.githubusercontent.com + GITHUB_PAT, HANDOVER 등 키워드 자동 매핑
  - query_database: asyncpg SELECT 전용, INSERT/UPDATE/DELETE/DROP 차단
  - read_remote_file: ceo_chat_tools.py 기존 SSH 보안 함수 재사용
  - fetch_url: 내부 네트워크 차단, 외부 URL 허용
  - generate_directive: >>>DIRECTIVE_START 포맷 자동 생성, 다음 task_id 추정
  - list_workspaces_sessions: DB 직접 조회
- **신규 파일 2**: `aads-server/app/services/tool_executor.py`
  - INTENT_TOOL_MAP: 24개 인텐트 → 도구 매핑 (casual/strategy 등 = 도구 없음)
  - execute_tools(): 병렬 실행 (개별 10초 / 전체 15초 타임아웃)
  - build_tool_injection(): "[시스템 도구 조회 결과 — ...]" 포맷 변환
  - has_tools_for_intent(): fallback 접두사 판단용
- **수정 파일**: `aads-server/app/services/chat_service.py`
  - execute_tools(intent, content, workspace_id_str) 호출 추가
  - tool_injection: 현재 user 메시지에 합산 (Anthropic API 연속 user 메시지 불가)
  - sources 컬럼에 도구 결과 JSON 저장
  - fallback_prefix: 도구 실패 시 "현재 도구 조회가 실패하여 제한된 정보로 답변합니다" 접두사
- **검증 결과**:
  - "오늘 완료된 작업" → dashboard 인텐트 → dashboard_query 실행 → 실제 DB 데이터 기반 응답 ✓
  - "안녕" → casual 인텐트 → 도구 없음 → 빠른 응답 ✓
  - "서버 상태 확인해" → dashboard 인텐트 → dashboard_query → has_sources=true ✓
  - sources 컬럼 JSON 저장 확인 ✓
- aads-server commits: 81647df (파이프라인) | 1a45893 (health_check 경량화)

## AADS-183 채팅 시스템 프롬프트 풍부화 완료 (2026-03-08)

- **문제**: 채팅 AI가 AADS 시스템, 프로젝트, 서버 정보를 전혀 모르는 상태에서 응답
- **원인**: 시스템 프롬프트가 한 줄짜리이며 HANDOVER/프로젝트 컨텍스트 미주입
- **신규 파일**: `aads-server/app/services/context_builder.py`
  - `build_system_context(workspace_name: str) → str` 함수
  - 공통 컨텍스트: 날짜(KST), AADS 정의, 서버 3대, 프로젝트 6개, 도구 안내
  - 워크스페이스별 컨텍스트 분기: CEO/AADS/SF/KIS/GO100/NTV2/NAS
  - `[CEO] 통합지시` 형식 이름 처리 (괄호 추출 → 키 매칭)
  - 크기 제한: 14000자 (~4000 토큰) 이내
- **수정 파일**: `aads-server/app/services/chat_service.py`
  - `send_message_stream()`: workspace_name 조회 추가
  - `build_system_context(workspace_name)` + `"---"` + `base_prompt` 결합
- **신규 마이그레이션**: `aads-server/migrations/update_workspace_prompts.sql`
  - 7개 워크스페이스 system_prompt 풍부화 (ILIKE 패턴 매칭)
  - CEO: 지시서 포맷 + 보고 규칙 + 최근 완료 이력
  - AADS: 기술 스택 + API 목록 + 파이프라인 구조
  - SF/KIS/GO100/NTV2/NAS: 각 프로젝트 역할 + 인프라
- **DB 적용**: 7개 워크스페이스 UPDATE 1 × 7 확인 완료
- **검증 기준**: "AADS란?" → 정확한 정의, "오늘 날짜" → KST 정확 응답, 프로젝트별 컨텍스트 분기

## AADS-182 Chat SSE 스트리밍 + 메시지 렌더링 긴급 수정 완료 (2026-03-08)

- **문제**: `/chat` 페이지에서 메시지 전송 후 AI 응답이 화면에 표시되지 않음 (백엔드 API 정상, 프론트엔드 렌더링 버그)
- **버그 1 — SSE 파싱 버퍼 없음**: 단순 `raw.split("\n")` 방식 → 청크 잘림 시 이벤트 손실. 수정: `\n\n` 기준 버퍼 누적 분리
- **버그 2 — done 이벤트 필드 불일치**: 백엔드 `{"model": ..., "cost": ...}` 전송, 프론트 `chunk.model_used/cost_usd` 읽음 → 항상 null. 수정: `chunk.model || chunk.model_used`, `chunk.cost || chunk.cost_usd`
- **버그 3 — stale closure**: `onDone` 콜백에서 `sseState` 참조 → 리액트 배치 업데이트로 null. 수정: `StreamMeta` 인터페이스 추가, onDone 파라미터로 직접 전달
- **버그 4 — msgs.reverse() 오류**: 백엔드 `ORDER BY created_at ASC` 반환 → 역순 렌더링. 수정: `.reverse()` 제거
- **추가 구현**: 30초 AbortController 타임아웃 + 폴링 fallback (SSE 실패 시 3초 후 GET /chat/messages)
- **버그 5 — chat/page.tsx SSE 타입 오류**: `ev.type === "token"` / `ev.text` 사용 → 백엔드 실제 이벤트 `"delta"` / `ev.content`. `"message_done"` → `"done"` 수정. 30초 타임아웃 + 폴링 fallback 추가.
- **파일 수정**: `useChatSSE.ts`, `useChatSession.ts`, `chatApi.ts`, `ceo-chat/page.tsx`, `chat/page.tsx`
- **타입 체크**: `npx tsc --noEmit` 오류 없음 ✓, `npm run build` 성공 ✓
- **aads-dashboard commits**: f61f793 (SSE 핵심 4파일) | 21d4410 (chat/page.tsx SSE 타입 수정+타임아웃)

## AADS-180 Chat API 배포 긴급 수정 — Docker 재빌드 + API 검증 완료 (2026-03-08)

- **문제**: AADS-170에서 구현한 채팅 백엔드(`app/routers/chat.py`, `app/services/chat_service.py`)가 Docker 컨테이너에 미반영 → `/chat` 페이지 동작 불가
- **라우터 상태 확인**: `main.py` line 153 — `app.include_router(chat_v2_router, prefix="/api/v1")` 이미 등록됨
- **DB 상태 확인**: 6개 테이블(`chat_workspaces/sessions/messages/artifacts/drive_files/research_archive`) + 7개 워크스페이스 시딩 모두 정상
- **조치**: `DOCKER_BUILDKIT=0 docker build -t aads-server-aads-server:latest .` 재빌드 → `docker compose up -d aads-server` 재시작
- **LiteLLM**: `docker-compose.yml`에 정의 없음 — `chat_service.py`가 Anthropic API 직접 호출 (LiteLLM 불필요)
- **검증 결과**:
  - `GET /api/v1/chat/workspaces` → 200 + 7개 워크스페이스 ✓
  - `GET /api/v1/chat/sessions?workspace_id=...` → 200 ✓
  - `POST /api/v1/chat/messages/send` → SSE delta 스트림 정상 ✓
  - `https://aads.newtalk.kr/api/v1/chat/workspaces` → 200 외부 검증 ✓
  - 기존 대시보드 `/api/v1/health` → 200 회귀 없음 ✓

## AADS-179 infra-check Docker 환경 호환성 수정 + 서버 상태 정확도 최신화 (2026-03-08)

- **health_checker.py** 전면 수정 (`aads-server/app/services/health_checker.py`):
  - `_check_memory()`: `free` 명령 → `/proc/meminfo` 파싱 (`MemTotal/MemAvailable/MemFree`) + `/sys/fs/cgroup/memory.max|current` Docker cgroup v2 추가. 응답: `{ok, total_mb, available_mb, usage_pct}`
  - `_check_cpu()`: `uptime` 명령 → `/proc/loadavg` 파싱 + `/proc/stat` 2회 샘플링(100ms) CPU 사용률 계산. 응답: `{ok, load_1m, load_5m, load_15m, cpu_usage_pct}`
  - `_check_http_health(server_key)` 신규 함수: 서버별 HTTP health endpoint 순서 호출 (211: 8200/8100/8080, 114: 7916)
  - `_check_ssh()`: SSH 실패 시 HTTP fallback → 둘 다 실패 시 `severity: warning` (SSH 키 부재 원인)
  - `_check_github_pat()`: PAT 미설정 시 `severity: warning` (critical → 하향)
  - `check_pipeline_status()` 211 원격 체크: SSH → HTTP fallback 순서
  - `check_consistency(auto_fix=False)`: `auto_fix=True` 시 pending 폴더 없는 queued 건 자동 `archived` 업데이트
  - `check_infra()`: `severity` 필드 명시된 경우만 해당 등급 적용 (이전: type에 "error" 포함 시 critical → 수정)
- **ops.py** 수정: `GET /ops/consistency-check?auto_fix=true` 파라미터 추가
- **docker-compose.yml** 수정: `GITHUB_PAT=${GITHUB_PAT:-}` 환경변수 추가
- **.env.example** 수정: `GITHUB_PAT=` 항목 추가
- **DB 정합성 복구**: `directive_lifecycle.status='queued'` 43건 → `archived` 일괄 업데이트 (pending 폴더에 없는 건)
- aads-server commit: cfd0f2c

## AADS-178 Pre-Flight Check 워크플로우 + DEPENDS_ON 강화 (2026-03-08)

- **preflight_checker.py** 신규 생성: `aads-server/app/services/preflight_checker.py`
  - `run_preflight(task_id, depends_on)` → `{queue_clear, depends_met, duplicate, conflicts, recommendation}`
  - pending+running 큐 스캔, RESULT 파일 제외 중복 task_id 감지
  - done 폴더 RESULT 파일 존재로 depends_on 충족 확인
  - recommendation: PROCEED | WAIT | BLOCKED
- **GET /api/v1/directives/preflight** 신규 엔드포인트: `aads-server/app/api/directives.py`
  - 쿼리 파라미터: `task_id`, `depends_on` (둘 다 optional)
  - 200 정상 응답, 500 오류 처리
- **auto_trigger.sh** 강화:
  - `_filter_invalid_pending()`: pending 파일 DIRECTIVE_START 블록 없으면 archived 이동, 중복 task_id 최신 1개 유지
  - `_check_depends_on()`: DEPENDS_ON 필드 있으면 done폴더+API 교차 확인, 3회 재시도(30/60/120s), 실패시 pending 유지+Telegram 알림
  - `_process_directive()` 내 DEPENDS_ON 체크 삽입 (running 기록 전)
  - pending → running 이동 전 `_filter_invalid_pending` 호출
- **WORKFLOW-PIPELINE.md v3.5**: Step 0 Pre-Flight Check 추가, 10단계 파이프라인
- **CEO-DIRECTIVES.md v3.6**: D-032(DEPENDS_ON 교차 확인), D-039(매니저 Pre-Flight Check 의무) 추가
- **HANDOVER-RULES.md v1.2**: §6-2 지시서발행흐름 Pre-Flight Check 추가, §6-2-2 신규 절차 문서

## AADS-172-C 아티팩트 패널 3단계 + AI Drive 파일 관리 (2026-03-08)

- **ArtifactPanel.tsx**: Full(420px)/Mini(48px)/Hidden(0px) 슬라이드 애니메이션 300ms ease-in-out
- **useArtifactPanel.ts**: ◀ 버튼 Full↔Mini 토글 / Ctrl+] 3단계 순환 / AI 응답 자동 감지(detectArtifactType) / openTab(tab) Mini→Full 확장
- **ArtifactTabs.tsx**: 5탭 (📄보고서/💻코드/📊차트/🖥️대시보드/📁AI Drive), Full 수평 / Mini 수직 아이콘
- **ArtifactReport.tsx**: 마크다운 렌더링(헤더/표/코드블록/인용/목록) + 목차 네비게이션 + 복사/PDF/편집/지시서생성/Drive저장
- **ArtifactCode.tsx**: 토큰 기반 구문 하이라이팅(JS/TS/Python/SQL/Shell) + 라인 넘버 + 복사 + 지시서 생성 + Drive저장
- **ArtifactChart.tsx**: SVG 인라인 라인차트(비용추이) + 수평 바차트(프로젝트 완료율), API 실패시 샘플데이터 폴백
- **ArtifactDashboard.tsx**: 서버 68/211/114 상태 + 서킷 브레이커 + 최근 파이프라인 (30초 자동 갱신)
- **AIDrive.tsx**: 파일 목록+폴더 트리+저장 용량 바+업로드 토글 패널 통합
- **AIDriveFileList.tsx**: 파일타입 아이콘, AI생성 뱃지, 폴더별 필터, 다운로드/삭제/이름변경(인라인 편집)
- **AIDriveUpload.tsx**: 드래그&드롭+클릭 업로드, 폴더 선택(AADS/SF/KIS/GO100/NTV2/NAS/Reports/Code), 진행 표시
- **driveApi.ts**: /api/v1/chat/drive/* 연결 (getFiles/getStats/getFolders/uploadFile/saveText/renameFile/deleteFile)
- **ChatBubble.tsx 수정**: onViewInPanel prop 추가 → 호버 시 🗂 "패널에서 보기" 버튼
- **ChatStream.tsx 수정**: onViewInPanel prop passthrough (assistant 메시지만 활성화)
- **ceo-chat/page.tsx 재작성**: ArtifactPanel 우측 통합 + useArtifactPanel + detectAndShowArtifact() + handleViewInPanel() + 🗂 패널 토글 버튼
- aads-dashboard commit: 7dfdb7d

## AADS-172-B Chat-First 스트림UI + SSE 연동 (2026-03-08)

- **ChatStream.tsx** 신규: 메시지 스트림 영역 (자동스크롤/새메시지 플로팅버튼/타이핑 인디케이터/DeepResearch 진행바 통합)
- **ChatInput.tsx** 신규: 멀티라인 textarea(자동높이/max200px), Enter전송/Shift+Enter줄바꿈, 파일첨부📎+드래그&드롭, 음성입력🎤(Web Speech API), 전송버튼▶(활성/비활성)
- **ChatModelSelector** (ModelSelector.tsx 추가): 5개 모델(Auto/Sonnet4.6/Opus4.6/Flash-Lite/DeepResearch) + 비용표시, isDeepResearch 플래그
- **ActionChips**: 웰컴5개 고정칩 + 컨텍스트 기반 동적5개 칩 (마지막 AI 메시지 키워드 분석)
- **DeepResearchProgress**: 3단계 진행바(소스검색/분석/보고서작성), 카운트 애니메이션, 예상소요시간
- **useChatSSE** (hooks/): fetch ReadableStream SSE, delta/done/error/thought_summary/sources 이벤트, 자동재연결 3회 exponential backoff
- **useChatSession** (hooks/): 워크스페이스 자동로드+세션CRUD, 낙관적 메시지 추가, 스트리밍 업데이트
- **chatApi.ts** (services/): /api/v1/chat/* REST+sendMessageStream, AuthHeader 포함
- **ceo-chat/page.tsx 재작성**: AADS-170 /api/v1/chat/* SSE 연동, 워크스페이스/세션 사이드바, 스트리밍 중지버튼
- aads-dashboard commit: 3af363b

---

## AADS-172-A Chat-First 3-Column UI (2026-03-08)
- **라우트**: `/chat` — (chat) 라우트 그룹, 독립 레이아웃 (메인 대시보드 사이드바 제외)
- **ThemeContext** (`src/contexts/ThemeContext.tsx`): 다크(기본)/라이트 토글, localStorage 영속화
- **chat-theme.css** (`src/styles/chat-theme.css`): CSS 변수 세트 (`--ct-bg`, `--ct-card`, `--ct-border` 등), 다크 #0F0F0F / 라이트 #FAFAFA
- **ChatLayout** (`src/components/chat/ChatLayout.tsx`): 3-column (사이드바 280px/64px + 채팅 flex-1 + 아티팩트 슬롯 360px), 반응형 모바일 드로어
- **ChatSidebar** (`src/components/chat/Sidebar.tsx`): 7개 워크스페이스 Hub 카드, 세션 목록 (최근순), 새 대화 버튼, 축소/확장 토글 (◀▶)
- **SidebarHubCard** (`src/components/chat/SidebarHubCard.tsx`): 아이콘+이름+활성 세션 뱃지, 선택 시 accent 배경
- **ThemeToggle** (`src/components/chat/ThemeToggle.tsx`): 헤더 우상단 ☀️/🌙 버튼
- **Chat Page** (`src/app/(chat)/page.tsx`): 메시지 버블 UI, textarea 입력창 (Enter 전송/Shift+Enter 줄바꿈), 전송 버튼
- **API 추가**: `getChatSession(id)`, `sendChatMessage(sessionId, content, workspaceId)`, `getChatSessions` optional workspaceId
- **ClientLayout** 업데이트: `/chat` 경로 시 메인 사이드바 레이아웃 제외 (이미 반영됨)
- **Sidebar.tsx** (메인): 하단에 "💬 AI Chat" 버튼 → `/chat` 새 탭 오픈
- **아티팩트 패널 슬롯**: `artifactPanel` prop으로 전달 (AADS-172-C에서 구현 예정)
- **반응형**: 데스크톱 (lg≥) 3-column / 태블릿 2-column / 모바일 드로어 오버레이
- aads-dashboard commit: pending (AADS-172-A 부분)

## AADS-172 Chat-First 프론트엔드 UI 완성 (2026-03-08)
- **라우트**: `src/app/chat/page.tsx` → `/chat` (ClientLayout 사이드바 bypass)
- **3-Panel 레이아웃**: 왼쪽 사이드바 (280px↔60px 접힘) / 채팅 (flex-1, min 480px) / 아티팩트 패널 (Full 420px / Mini 48px / Hidden 0px)
- **다크/라이트 테마**: CSS 변수 (`--ct-*`), localStorage `aads-chat-theme`, `prefers-color-scheme` 감지, 0.3s 전환
- **SSE 스트리밍**: `fetch + ReadableStream`, `token/message_done/error` 이벤트, 중단 버튼, 타이핑 인디케이터 (점 3개 애니메이션)
- **워크스페이스 + 세션**: GET `/chat/workspaces`, GET `/chat/sessions?workspace_id=xxx`, POST `/chat/sessions`, 우클릭 컨텍스트 메뉴 (이름변경/삭제)
- **모델 셀렉터**: MODEL_OPTIONS 재활용 (44개), 세션 current_model 반영
- **액션 칩**: 🔍검색/🧪딥리서치/📎파일/📹동영상/🎤음성, 드래그앤드롭 파일 업로드, POST `/chat/drive/upload`
- **아티팩트 패널**: 탭 4개 (📄보고서/💻코드/📊차트/🖥️대시보드), 복사/편집/지시서생성 액션, Ctrl+] 토글
- **반응형**: desktop(≥1280) 3-panel / tablet(768~1279) 오버레이 / mobile(<768) 스와이프
- **MarkdownBlock**: 코드블록(언어 표시), 인라인 코드, 볼드, 헤더, 리스트 렌더링
- **Header.tsx**: 💬 AI Chat 버튼 추가 (#6C63FF)
- **Sidebar.tsx**: "AI Chat" 링크 (/chat) highlight 추가
- **api.ts**: AADS-170 chat API 함수 추가 (workspaces/sessions/messages/artifacts/drive/research)
- aads-dashboard commit: 9f4076b

---

## 버전 이력 (최근 10건)

| 버전 | 날짜 | Task ID | 변경 요약 |
|------|------|---------|-----------|
| v13.0 | 2026-03-09 | AADS-188E | 전체 통합 E2E 검증: test_e2e_code_modify.py(24)+test_e2e_deep_research.py(19)+test_e2e_agent_sdk.py(23)=66 PASS. chat_service.py 시맨틱 코드 검색 컨텍스트 자동 주입(섹션4.5). 기존 178건 PASS. 보고서: reports/AADS-188E-INTEGRATION-REPORT.md |
| v12.22 | 2026-03-09 | AADS-188E(구) | approve-diff API 배포+E2E test_e2e_agent_sdk.py(execute_stream/3턴/approve-diff approve·reject·400). aads-server b6dc996 |
| v12.21 | 2026-03-09 | AADS-188D | CodeDiffViewer.tsx(Monaco DiffEditor+Side-by-side/Inline토글+LANG_MAP+fallback)+CodePanel.tsx(approve-diff POST API+countdown+onResult채팅메시지)+useDiffApproval.ts(300초타이머+onDiffPreview+close)+code-editor.css(모바일오버레이+440px고정)+chat/page.tsx(4패널+diff_preview SSE파싱+countdown prop) commit: ec62be9 |
| v12.20 | 2026-03-09 | AADS-188B | code_indexer_service.py(CodeIndexerService+Python AST 청킹+TS regex 청킹+ChromaDB PersistentClient+index_project/update_index+Gemini text-embedding-004 배치 임베딩)+semantic_code_search.py(search+hybrid_search+build_code_context)+context_builder.py(_build_semantic_code_layer)+tests 30/30 PASS commit: 8ae390d |
| v12.19 | 2026-03-09 | AADS-188C | agent_sdk_service.py(AgentSDKService+MCP서버+12도구+Green/Yellow/Red등급+max_turns=30+resume)+agent_hooks.py(PreToolUse Bash위험패턴14개+Write민감경로차단+PostToolUse diff_preview SSE+stop메모리저장)+chat_service execute/code_modify인텐트→SDK primary+AutonomousExecutor fallback+pyproject.toml claude-agent-sdk+tests 18/18 commit: a13ef4f |
| v12.18 | 2026-03-09 | AADS-188A | deep_research_service.py(research_stream AsyncGenerator+GOOGLE_GENAI_API_KEY+context/format/Langfuse span)+models/research.py(planning/searching/analyzing/complete 타입+content/sources필드)+tool_registry(context/format enum)+tool_executor Langfuse span+intent_router 키워드확장+tests 46/46 commit: c36c927 |
| v12.17 | 2026-03-09 | AADS-186E-3 | autonomous_executor.py(MAX_ITER=25+COST_LIMIT=2.0+위험도구차단)+memory_manager.py(observe/build_meta_context/auto_observe/save_note/recall_notes)+025_ai_observations.sql+context_builder <meta_memory>+tool_registry observe도구+chat_service cto인텐트→AutonomousExecutor+main.py weekly→generate_weekly_briefing()+tests 22/22+총 63테스트 commit: dcd16b2 |
| v12.16 | 2026-03-09 | AADS-186E2-BRIDGE | tool_registry PTC allowed_callers 6개(health_check/query_database/read_remote_file/list_remote_dir/cost_report/jina_read)+deep_research_service.py+code_explorer_service.py+chat_service research SSE+intent_router deep_research/url_read+tool_executor 4개실행기+59테스트 통과 commit: ff97268 |
| v12.15 | 2026-03-09 | AADS-186E2 | app/models/research.py(ResearchEvent+ResearchResult Pydantic v2)+tests/test_deep_research.py(27개)+tests/test_ptc.py(20개)+총 88/88 테스트 통과 commit: 69238c8 |
| v12.14 | 2026-03-09 | AADS-186E-2 | Extended Thinking(Opus전용 budget_tokens=10000)+PTC ptc_executor.py+메모리4계층(memory_manager.py+024_memory_tables.sql)+context_builder <recent_sessions>/<learned_patterns>+system_prompt 기억규칙+chat_service 자동노트저장+tests 41/41 |
| v12.13 | 2026-03-09 | AADS-186D | Tool Search Tool+원격 CKP 5개+Prompt Caching 최적화+주간 브리핑 |
| v12.12 | 2026-03-09 | AADS-186E-1 | 크롤링 도구: jina_reader_service.py+crawl4ai_service.py+deep_crawl_service.py+tool_registry crawl그룹+tool_executor실행기3개+intent_router url_read/deep_crawl+docker-compose.crawl4ai.yml+tests 8/8 |
| v12.11 | 2026-03-09 | AADS-186B | CKP 시스템: ckp_manager.py+ast_analyzer.py+cto_mode.py+models/ckp.py+022_ckp_tables.sql+.claude/5종+intent_router CTO인텐트6개+context_builder CKP연동+tests 27개 |
| v12.10 | 2026-03-09 | AADS-186A | 시스템 프롬프트 재설계: system_prompt_v2.py XML섹션+Tool Examples+워크플로우 도구 3개 |
| v12.9 | 2026-03-09 | AADS-186C | Langfuse v3 셀프호스팅+langfuse_config.py+FastAPI-MCP+AlertManager(8규칙)+TelegramBot(/status /cost /alerts)+APScheduler(2분/09KST)+023_alert_history.sql |
| v12.8 | 2026-03-09 | AADS-185 확장 | 채팅 서버검색 list_remote_dir·read_remote_file+인텐트 server_file, 웰컴 칩 작업현황/서버 파일 검색, 배포 완료 |
| v12.7 | 2026-03-08 | AADS-185 | CEO Chat 전면 재설계: 3계층Context+IntentRouter+ModelSelector+ToolUse+GeminiSearch+DeepResearch+프론트엔드3컴포넌트 |
| v12.6 | 2026-03-08 | AADS-184 | 채팅 도구 연동: chat_tools.py(9도구)+tool_executor.py(24인텐트맵)+chat_service.py 파이프라인+sources컬럼+fallback |
| v12.5 | 2026-03-08 | AADS-183 | 채팅 프롬프트 풍부화: context_builder.py 신규+chat_service.py workspace 컨텍스트 주입+7개 워크스페이스 system_prompt 업데이트+DB 적용 |
| v12.3 | 2026-03-08 | AADS-182 | Chat SSE 렌더링 긴급 수정: 버퍼 파싱+done 필드 매핑+StreamMeta stale closure+reverse() 제거+30초 타임아웃+폴링 fallback |
| v12.4 | 2026-03-08 | AADS-181 | 전체 프로젝트 통합 API: server_registry+cross_server_checker+/directives/all+/ops/server-summary+SSE cross_server_directives+TaskTable+useTaskPolling+/tasks 페이지 3서버 연동 |
| v12.3 | 2026-03-08 | AADS-182 | Chat SSE 스트리밍 렌더링 긴급 수정: SSE파싱버퍼+done이벤트필드+stale closure+msgs.reverse 4버그 수정 |
| v12.2 | 2026-03-08 | AADS-180 | Chat API 배포 긴급 수정: Docker 재빌드+chat_v2_router 활성화+API 검증(workspaces 200/sessions 200/SSE 정상) |
| v12.1 | 2026-03-08 | AADS-179 | infra-check Docker 호환: /proc 기반 memory/cpu+HTTP fallback(SSH 대체)+PAT warning 하향+consistency auto_fix+DB queued 43건 복구 |
| v12.0 | 2026-03-08 | AADS-178 | Pre-Flight Check: preflight_checker.py+GET /preflight API+auto_trigger DEPENDS_ON 교차확인+브릿지파일필터링+WORKFLOW-PIPELINE v3.5+D-039 |
| v11.9 | 2026-03-08 | AADS-172-C | 아티팩트 패널 3단계(Full/Mini/Hidden)+AI Drive: ArtifactPanel+5탭+구문하이라이팅+SVG차트+드라이브파일관리 |
| v11.8 | 2026-03-08 | AADS-172-B | Chat-First 스트림UI: ChatStream+ChatInput+SSE연동+모델셀렉터5개+액션칩+DeepResearch진행바+출처카드+useChatSSE/Session |
| v11.7 | 2026-03-08 | AADS-172 | Chat-First 프론트엔드 UI 완성: /chat 3-panel 레이아웃, SSE 스트리밍, 다크/라이트 테마, 아티팩트 패널, 반응형 |
| v11.6 | 2026-03-08 | AADS-172-A | Chat-First 3-Column UI: /chat 독립 라우트, ThemeContext 다크/라이트, Sidebar 7 Hub, 3-column ChatLayout, /chat 새 탭 링크 |
| v11.3 | 2026-03-08 | AADS-169 | 대시보드 Claude Bot Status 카드: 서버별 프로세스+좀비+bridge, 제어버튼 3개, SSE 연동, Cleanup Actions 테이블 |
| v11.2 | 2026-03-08 | AADS-168 | 멀티서버 클로드봇 감시 데몬: claude_watchdog.py(3서버), 3개 ops API, systemd 파일 스테이징 |
| v11.1 | 2026-03-08 | AADS-167 | claude_exec_safe.sh: 프로세스 그룹 격리+이중 타임아웃+PID 관리+ulimit, settings.json 배포 |
| v11.0 | 2026-03-08 | AADS-166 | 파이프라인 전체 헬스체크 8파트: 6 API + SSE 스트리밍 + Pipeline 탭 + Header dot + 정체강조 |
| v10.9 | 2026-03-08 | AADS-165 | CEO Chat 크로스 프로젝트 코드 접근: SSH 도구 2개, Intent 12분류, 실행 검증 위임 |
| v10.7 | 2026-03-08 | AADS-161 | /tasks 문서뷰어: MarkdownRenderer(inline), mermaid 블록 렌더링, TECH-ARCH-001 파이프라인→mermaid |
| v10.6 | 2026-03-08 | AADS-163 | 개발→QA→디자인 3단계 품질 게이트 통합 |
| v10.5 | 2026-03-08 | AADS-162 | Design Verification Agent: design_reviews DB table, 5 baseline screenshots, design-reviewer agent wrapper |
| v10.3 | 2026-03-08 | AADS-164 | CEO Chat Agent Individual Call System: 10 intents, qa/design/design_fix/architect 핸들러, agent_executions DB |
| v10.2 | 2026-03-08 | AADS-161 | 매니저 자기인식 프로토콜, bridge.py 자동화 파이프라인, 6개 프로젝트 라우팅 테이블, D-035~D-037/R-022 추가 |
| v10.1 | 2026-03-08 | AADS-148 | 이번 세션: 대시보드 문서편집+자동push, project-docs API, CEO문서/TZ 미반영분 추가 |
| v10.0 | 2026-03-08 | AADS-148 | 4계층 재구성, D-023 v2 적용, 토큰 상한 폐기, RULES 신규생성 |
| v9.0 | 2026-03-07 | AADS-160 | CEO 직접 검수: 버그수정 4건 + 모델드롭박스 + KST 동기화 |
| v8.2 | 2026-03-07 | AADS-159 | CEO Chat Playwright 브라우저 자동화 6개 도구 |
| v8.1 | 2026-03-07 | AADS-158 | Pending 대기큐 정리 11건 archived 이동 |
| v8.0 | 2026-03-07 | AADS-157 | CEO Chat v2 Core Engine 연결, Intent Classifier |
| v7.9 | 2026-03-07 | AADS-156 | CEO Chat 모델 패스스루, SUPPORTED_MODELS 28개 |
| v7.8 | 2026-03-07 | AADS-149 | 파이프라인 전수조사 버그 5건 수정 |
| v7.7 | 2026-03-07 | AADS-148 | /proc grep 블로킹 3일 장애 수정 |
| v7.6 | 2026-03-07 | AADS-146 | Worktree 병렬, 서브에이전트, Writer/Reviewer |

---

## 버전 이력 (v12.22 이후)

| 버전 | 날짜 | 주요 변경 |
|------|------|----------|
| v12.22 | 2026-03-09 | AADS-188E: approve-diff API + E2E 66건 PASS |
| v14.0 | 2026-03-30 | 3월 갭 12건 갱신: Pipeline Runner, OAuth, Blue-Green, PC Agent, 메모리 진화, LiteLLM 확장, CTO 온보딩 |

---

## 버전 이력 (v12.22 이후)

| 버전 | 날짜 | 주요 변경 |
|------|------|----------|
| v12.22 | 2026-03-09 | AADS-188E: approve-diff API + E2E 66건 PASS |
| v14.0 | 2026-03-30 | 3월 갭 12건 갱신: Pipeline Runner, OAuth, Blue-Green, PC Agent, 메모리 진화, LiteLLM 확장, CTO 온보딩 |
