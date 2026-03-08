# AADS HANDOVER v12.3
최종 업데이트: 2026-03-08 | 버전: v12.3 — AADS-182 Chat SSE 스트리밍 렌더링 긴급 수정 완료

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
- 주요 서비스: FastAPI, PostgreSQL, Dashboard (Docker Compose)
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
- 최근: AADS-172 완료 (Chat-First 프론트엔드 UI — 3-Panel 레이아웃 + 다크/라이트 + SSE 스트리밍 + 아티팩트 패널 3단계 + 반응형)
- 이전: AADS-172-A 완료 (Chat-First 3-Column UI + 다크/라이트 테마 + 사이드바 Hub 7개 워크스페이스 + /chat 라우트 독립 레이아웃)
- 이전: AADS-170 완료 (CEO Chat-First DB 6테이블 + 채팅 API 22개 엔드포인트 + 인텐트 12개 확장 + 7 워크스페이스 시딩)
- 이전: AADS-171 완료 (LiteLLM Proxy Docker + Gemini/Claude 듀얼 모델 라우팅 + 인텐트→모델 매핑 + 일 $5 비용상한)
- 서버: 68
- 도구: Claude
- 헬스체크: https://aads.newtalk.kr/
- 특수: LLM 15회/작업 이하, langgraph-supervisor 사용 금지
- LiteLLM: http://litellm:4000 (Docker 내부), Gemini 3종 + Claude 3종, 일 $5 / 월 $150 상한

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

없음

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

- **GitHub PAT 만료**: 2026-05-27 (잔여 약 80일)
- **디스크 사용률**: 서버 114 = 79% (649GB/875GB) — 70% 초과 시 정리 권장
- **현재 장애**: 없음
- **절대 금지**: .env 커밋 금지, DB DROP 금지, 미승인 서비스 재시작 금지
- **OAuth 토큰**: 전체 만료 상태 — 재인증 필요 시 브라우저 통해 수행
- **Cloudflare**: OAuth 후 proxied DNS 유지

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
| v12.3 | 2026-03-08 | AADS-182 | Chat SSE 렌더링 긴급 수정: 버퍼 파싱+done 필드 매핑+StreamMeta stale closure+reverse() 제거+30초 타임아웃+폴링 fallback |
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
