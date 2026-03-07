# AADS HANDOVER v10.1
최종 업데이트: 2026-03-08 | 버전: v10.1 — 이번 세션 전체 작업 반영 + CEO 문서 등록 + 대시보드 문서편집 기능

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

### 서버 114 (116.120.58.155)
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
- 최근: AADS-148 세션 (4계층 문서 재구성 + 대시보드 문서편집 기능 + project-docs API) 진행 중
- 서버: 68
- 도구: Claude
- 헬스체크: https://aads.newtalk.kr/
- 특수: LLM 15회/작업 이하, langgraph-supervisor 사용 금지

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

## 진행 중 작업 상세

없음 (이번 세션 작업 모두 완료)

---

## 대기 작업 큐

없음 (CEO 지시 대기)

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

## CEO-DIRECTIVES 전문 요약 (v3.3)

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
- classify_intent(): 5분류 (dashboard/diagnosis/research/execute/strategy)
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
| CEO-DIRECTIVES.md (v3.3) | https://github.com/moongoby-GO100/aads-docs/blob/main/CEO-DIRECTIVES.md | CEO 지시 전문 (D-001~D-034, R-001~R-021, T-001~T-011) |
| WORKFLOW-PIPELINE.md (v3.2) | https://github.com/moongoby-GO100/aads-docs/blob/main/shared/rules/WORKFLOW-PIPELINE.md | 8단계 파이프라인 상세 + 라우팅 + 모델 라우팅 |
| RULE-MATRIX.md (v1.2) | https://github.com/moongoby-GO100/aads-docs/blob/main/shared/rules/RULE-MATRIX.md | 23규칙 x 8단계 매핑 매트릭스 |
| STATUS.md | https://github.com/moongoby-GO100/aads-docs/blob/main/STATUS.md | 실시간 작업 상태 (last_completed, next_pending) |
| AADS-KNOWLEDGE.md | /root/aads/aads-server/docs/knowledge/AADS-KNOWLEDGE.md | AADS 전용 지식 |
| AADS_ARCHITECTURE_v1.0 | https://github.com/moongoby-GO100/aads-docs/blob/main/architecture/AADS_ARCHITECTURE_v1.0.md | 시스템 아키텍처 (인프라, API 50+, DB 13테이블) |
| LANGGRAPH_PIPELINE_AUDIT | https://github.com/moongoby-GO100/aads-docs/blob/main/architecture/LANGGRAPH_PIPELINE_AUDIT_v1.0.md | 8-에이전트 파이프라인 기술 감사 |
| project-docs.json | https://github.com/moongoby-GO100/aads-docs/blob/main/shared/project-docs.json | 대시보드 문서 링크 설정 (자동 push) |

---

## 버전 이력 (최근 10건)

| 버전 | 날짜 | Task ID | 변경 요약 |
|------|------|---------|-----------|
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
