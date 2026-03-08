# HANDOVER-RULES v1.2
최종 업데이트: 2026-03-08 | 버전: v1.2 — AADS-178 매니저 Pre-Flight Check 절차 추가

---

## 6-1. 파이프라인 8단계 상세

### Step 1: CEO 지시 생성
- 주체: CEO (moongoby)
- 도구: Genspark 채팅 또는 직접 Bridge 디렉티브 작성
- 출력: pending 디렉티브 파일 (/root/.genspark/directives/pending/)
- 실패 시: 매니저에게 재작성 지시

### Step 2: Bridge 감지
- 주체: bridge.py (서버 211)
- 도구: /root/.genspark/directives/pending/ 디렉토리 감시
- 입력: pending 디렉티브 파일
- 출력: auto_trigger.sh에 전달
- 실패 시: bridge 셀프체크 60초, L2 pipeline_monitor 감시

### Step 3: 사전 검증
- 주체: auto_trigger.sh (서버 211)
- 검증 항목: WORKDIR 권한, 중복 체크(task_id), 의존성(DEPENDS_ON) 충족
- 실패 시: 검증 실패 로그 + Telegram 알림 + 대기

### Step 4: 우선순위 라우팅
- 주체: auto_trigger.sh
- 기능: 프로젝트별 서버 라우팅 (KIS/GO100→211 로컬, AADS→68 SSH, SF/NTV2/NAS→114 SSH)
- 정렬: impact/effort 점수 높은 순 (D-025)
- 실패 시: 라우팅 실패 로그 + 재시도

### Step 5: Claude 실행
- 주체: claude_exec.sh
- 도구: Claude Code (claudebot 계정)
- 하트비트: inotifywait 또는 git status fallback, 10초 주기 감시
- 타임아웃: 하드 7,200초(2시간), Tier2 120초, Tier3 300초
- 모델: 지시서 model 필드 → size 기반 자동 선택 → fallback sonnet (D-024)
- Tasks: ~/.claude/tasks/{task_id}.json 생성/관리
- 컨텍스트: 70% compact 권고, 90% 중간저장+clear, 2회 편집실패 CTX-EDIT-FAIL
- 실패 시: session_watchdog Tier2 진단→kill→재시작

### Step 6: 결과 보고
- 주체: claude_exec.sh
- 출력: RESULT_FILE (commit SHA 포함), WRAP 파일
- HANDOVER.md 업데이트 여부 검증 (git diff 확인) — D-034 WRAP 게이트
- 미포함 시: WRAP 게이트 차단, 작업자에게 업데이트 요청
- 로그: "HANDOVER_UPDATE_CHECK: PASS/FAIL"
- Telegram: 완료 알림

### Step 7: DB 기록
- 주체: AADS API (서버 68)
- 기록 대상: recovery_logs, directive_lifecycle, usage_logger
- 실패 시: 로컬 로그 fallback + 재시도

### Step 8: 교차 검증
- 주체: session_watchdog + 3서버
- 검증: git-push HTTP 200 확인 (3회 재시도, exponential backoff 10/20/40초)
- 실패 시: push_failed 로그 + Telegram + recovery_logs + 매니저 에스컬레이션

---

## 6-2. 매니저 역할 완전 명세

### a) 정체성
Genspark AI 매니저 (Web Claude가 아님). 세션 시작 시 자기인식 프로토콜 수행 의무 (§6-2-1 참조).
- AADS 매니저 채팅 URL: https://www.genspark.ai/agents?id=3d86d6f3-09a7-41b2-b91b-762a55512458

### b) 할 수 있는 것 (6항)
1. GitHub raw URL 크롤링 (HANDOVER, CEO-DIRECTIVES, STATUS 등)
2. 문서 분석 및 진단 (누락 항목, 불일치 발견)
3. 지시서 작성 (CEO 지시 기반 DIRECTIVE 블록 생성)
4. 보고서 작성 및 집계 (프로젝트 간 현황 종합)
5. 공개 URL 모니터링 (헬스체크 URL 확인)
6. CEO에게 의견 제시 (개선 제안, 위험 경고)

### c) 할 수 없는 것 (7항)
1. SSH 직접 접속 (서버 접근 불가)
2. DB 직접 조회/수정 (SQL 실행 불가)
3. git push / git commit (코드 변경 불가)
4. 서비스 시작/정지/재시작 (인프라 제어 불가)
5. .env, 토큰 파일 등 시크릿 접근 (보안 파일 열람 불가)
6. CEO 승인 없이 $5 초과 작업 실행
7. CEO에게 지시서 전달을 요청하는 행위 (D-037, R-022)

### d) 세션 시작 4단계
1. HANDOVER.md 크롤링 + 읽기
2. HANDOVER-RULES.md 크롤링 + 읽기
3. CEO-DIRECTIVES.md 크롤링 + 읽기
4. CEO에게 현황 보고 + 지시 대기

### e) 예산 규칙
작업당 $5 이내, 초과 시 CEO 사전 승인

### f) 파일 경로
- WRAP 파일: /tmp/aads_wrap_{task_id}.json
- RESULT 파일: reports/{TASK_ID}-RESULT.md
- 하트비트: /tmp/claude_session_{task_id}.heartbeat

### g) 지시서 발행 흐름 (D-039 Pre-Flight Check 포함)
1. CEO 지시 수신 (자연어)
2. **[Pre-Flight Check]** 지시서 발행 전 큐 상태 확인 (D-039, 필수):
   ```
   GET /api/v1/directives/preflight?task_id={id}&depends_on={선행id}
   ```
   - `PROCEED` → 3단계 진행
   - `WAIT` → depends_on 완료 후 재확인
   - `BLOCKED` → 중복 task_id 해소 후 재시도
3. 매니저가 지시서 블록 작성 (>>>DIRECTIVE_START ~ >>>DIRECTIVE_END)
4. 채팅창에 출력 (이것으로 매니저의 역할 완료)
5. bridge.py 자동 감지·추출·저장 (매니저 개입 불필요)
6. auto_trigger.sh 검증·라우팅 (자동)
7. claude_exec.sh 실행 (자동)

**절대 금지**: 매니저가 CEO에게 "이 지시서를 전달해 주세요", "bridge에 넣어 주세요" 등 전달을 요청하는 행위 (D-037, R-022)

---

## 6-2-2. 매니저 Pre-Flight Check 절차 (D-039, AADS-178)

매니저는 지시서를 발행하기 **전** 반드시 아래 3단계 Pre-Flight Check를 수행해야 한다.

### 단계 1: 큐 상태 확인
- API: `GET /api/v1/directives/preflight?task_id={id}`
- 확인 항목: running 큐에 활성 작업 있는지 (`queue_clear`)
- 중복 task_id 존재 여부 (`duplicate`)

### 단계 2: 선행 태스크 검증
- depends_on이 있으면 `depends_on={선행_task_id}` 파라미터 추가
- `depends_met: false`이면 선행 완료 대기

### 단계 3: 판정 기반 행동
| recommendation | 행동 |
|----------------|------|
| PROCEED | 즉시 지시서 발행 |
| WAIT | depends_on 완료 알림 대기, 재확인 후 발행 |
| BLOCKED | 중복 task_id 해소 (취소 또는 번호 변경) 후 발행 |

### auto_trigger.sh 연계 안전망
- pending 파일에 `>>>DIRECTIVE_START` 없으면 자동 archived 이동
- 동일 task_id 중복 파일 → 최신 1개 유지
- DEPENDS_ON 미충족 → 30/60/120s 재확인 3회 → Telegram 알림 + pending 유지

---

## 6-2-1. 매니저 자기인식 프로토콜 (D-036)

모든 프로젝트 매니저는 세션 시작 시 아래 3가지 검증을 수행한다.

### 검증 ①: 프로젝트명 일치
- 채팅 제목 또는 CEO 첫 메시지에 해당 프로젝트명(AADS, KIS, GO100, SF, NTV2, NAS)이 포함되어 있는지 확인
- 불일치 시 작업 중단

### 검증 ②: Task ID 접두사 일치
- 할당된 Task ID의 접두사가 자신의 프로젝트와 일치하는지 확인
- AADS-xxx, KIS-xxx, GO100-xxx, SF-xxx, NT-xxx, NAS-xxx

### 검증 ③: 참조 문서 URL 일치
- 참조 문서 URL이 해당 프로젝트의 리포지토리와 일치하는지 확인
- AADS: aads-docs, KIS/GO100: kis-autotrade-v4, SF: shortflow, NTV2: newtalk-v2

### 불일치 시 대응
- 경고 출력: "⚠️ 프로젝트 불일치 감지. 이 채팅은 {프로젝트}용입니다. {자신의 프로젝트} 작업을 수행할 수 없습니다."
- 작업 즉시 중단
- CEO에게 보고

---

## 6-3. 작업자 공통 규칙

### a) FLOW 프레임워크 (D-016)
- Find: 현재 상태 파악 (파일, DB, 서비스 확인)
- Layout: 작업 계획 수립 (변경 파일 목록, 순서, 영향 분석)
- Operate: 실행 (코드 수정, 테스트, 커밋)
- Wrap-up: 마무리 (RESULT 파일, HANDOVER 업데이트, git push)
- 소규모 버그수정/설정변경은 Operate→Wrap-up만 수행 가능

### b) 세션 시작 4단계
1. HANDOVER.md 읽기
2. HANDOVER-RULES.md 읽기
3. CEO-DIRECTIVES.md 확인
4. 할당된 Task 확인 후 FLOW 시작

### c) 완료 4조건 (전부 충족해야 완료 인정)
1. 파일 수정 완료
2. git commit + push 성공 (commit SHA 기록)
3. HANDOVER.md Core 업데이트 완료 (git diff에 HANDOVER.md 포함)
4. RESULT 파일 작성 완료 (reports/{TASK_ID}-RESULT.md)

### d) 절대 금지 5항
1. .env 또는 토큰 파일 git commit (R-003)
2. DB DROP / TRUNCATE (R-004)
3. CEO 미승인 서비스 재시작 (R-005)
4. 할당되지 않은 프로젝트 파일 수정 — WORKDIR 이탈 금지 (R-018)
5. 토큰 절약을 이유로 HANDOVER 정보 생략 (R-021)

### e) 커밋 메시지 규칙
형식: [{PROJECT}] {type}: {description}
type: feat, fix, docs, refactor, test, chore
예: [AADS] docs: HANDOVER v10.0 4계층 재구성

### f) 파일 경로
- WRAP 파일: /tmp/aads_wrap_{task_id}.json
- RESULT 파일: reports/{TASK_ID}-RESULT.md
- 하트비트: /tmp/claude_session_{task_id}.heartbeat

---

## 6-4. R-021: HANDOVER 업데이트 의무 강화 (신규)

주의: 기존 R-020은 "의존성(DEPENDS_ON) 선행 충족 후 실행"이다. ID 충돌 방지를 위해 R-021로 부여.

- 작업 완료 시 HANDOVER.md Core를 반드시 업데이트
- 토큰 절약을 이유로 정보를 생략하면 규칙 위반 (R-VIOLATION)
- 업데이트 누락 시 작업 완료로 인정하지 않음 (Step 6 WRAP 게이트 차단)
- 판단 기준: "이 문서만 읽고 CEO 질문 0회로 업무 착수 가능한가?"

---

## 6-5. 지시서 포맷 v2.0 완전판 (D-022)

### 필수 필드

| 필드 | 필수/선택 | 설명 |
|------|-----------|------|
| TASK_ID | 필수 | {PROJECT}-{3자리번호} (예: AADS-148) |
| TITLE | 필수 | 작업 제목 |
| PRIORITY | 필수 | P0-CRITICAL / P1-HIGH / P2-MEDIUM / P3-LOW |
| SIZE | 필수 | XS / S / M / L / XL |
| IMPACT | 필수 | H / M / L |
| EFFORT | 필수 | H / M / L |
| MODEL | 필수 | haiku / sonnet / opus |
| REVIEW_REQUIRED | 필수 | true / false |
| ASSIGNEE | 필수 | 담당 작업자/서버 |
| parallel_group | XL 필수 | 병렬 그룹 식별자 |
| files_owned | XL 필수 | 해당 그룹 전용 파일 목록 |
| subagents | 선택 | 서브에이전트 사용 목록 |

### SIZE 판단 기준

| SIZE | 예상 시간 | 파일 수 | 비용 상한 | 기본 모델 |
|------|-----------|---------|-----------|-----------|
| XS | < 15분 | 1-2 | $0.50 | haiku (claude-haiku-4-5) |
| S | 15-30분 | 2-5 | $1.00 | sonnet (claude-sonnet-4-6) |
| M | 30-90분 | 5-10 | $3.00 | sonnet (claude-sonnet-4-6) |
| L | 90분-3시간 | 10-20 | $5.00 | opus (claude-opus-4-6) |
| XL | 3시간+ | 20+ | CEO 승인 | opus (claude-opus-4-6) |

### IMPACT/EFFORT 판단 기준

| 등급 | IMPACT 의미 | EFFORT 의미 |
|------|-------------|-------------|
| H | 시스템 안정성/매출에 직접 영향 | 복잡한 로직, 다수 파일, 테스트 필요 |
| M | 효율성/품질 개선 | 중간 난이도, 일부 파일 |
| L | 문서/정리/마이너 개선 | 단순 수정, 1-2 파일 |

### 우선순위 정렬 점수 (D-025)
- impact 점수: H=3, M=2, L=1
- effort 점수: H=1, M=2, L=3 (낮은 effort가 높은 점수)
- 정렬 점수 = impact_score x 10 + effort_score
- 예: impact H + effort L = 33 (최우선), impact L + effort H = 11 (최하위)

### 예시 1 (S 작업)

```
TASK_ID: AADS-150
TITLE: STATUS.md 업데이트 스크립트 타임존 수정
PRIORITY: P2-MEDIUM
SIZE: S
IMPACT: L
EFFORT: L
MODEL: sonnet
REVIEW_REQUIRED: false
ASSIGNEE: AADS 작업자 (서버 68)
```

### 예시 2 (XL 작업, 병렬 분할)

```
TASK_ID: AADS-151
TITLE: Agent Teams 파일럿 구현
PRIORITY: P1-HIGH
SIZE: XL
IMPACT: H
EFFORT: H
MODEL: opus
REVIEW_REQUIRED: true
ASSIGNEE: AADS 작업자 (서버 68)
parallel_group: AADS-151-A (graphs/), AADS-151-B (agents/), AADS-151-C (tests/)
files_owned:
  A: graphs/team_lead.py, graphs/team_worker.py, graphs/dag.py
  B: agents/coordinator.py, agents/specialist.py
  C: tests/test_team_*.py
subagents: [research, review]
```

---

## 6-6. XL 작업 사전 분할 규칙

- XL 작업은 반드시 parallel_group과 files_owned를 명시
- 분할 기준: 파일 경계 (같은 파일을 두 그룹이 수정하지 않음)
- 각 그룹은 독립 실행 가능해야 함 (의존성 있으면 순서 명시)
- Git worktree로 브랜치 분리 실행
- 통합 테스트는 전 그룹 완료 후 별도 단계로 수행

### 판단 흐름
1. 변경 대상 파일 목록 작성
2. 파일 간 import/의존성 분석
3. 독립 실행 가능한 그룹 식별
4. parallel_group 배정 + files_owned 명시
5. 의존성 있는 그룹은 순서(depends_on) 지정

---

## 6-7. 효율성 12전략 전체

### 전략 1: 지시서 사전 분할 설계
- 매니저가 지시서 작성 시 병렬 가능 단위로 미리 설계
- 적용 조건: SIZE L 이상
- 구현: parallel_group, files_owned 필드 사용
- 기대 효과: XL 작업 시간 50-70% 단축

### 전략 2: Claude Code Agent Teams
- 팀 리드 + 팀원 구조, DAG 기반 의존성 관리
- 적용 조건: Phase 3 이후 (파일럿 완료 후)
- 기대 효과: XL 작업 60-70% 단축, 토큰 3-5배 증가

### 전략 3: Git Worktree 병렬
- 동일 프로젝트 다른 브랜치를 worktree로 분리
- 적용 조건: XL 분할 작업 시
- 기대 효과: 충돌 없는 병렬 실행

### 전략 4: 서브에이전트 활용
- 조사/검증을 저비용 서브에이전트로 분리
- 적용 조건: 조사 포함 작업
- 기대 효과: 메인 컨텍스트 보호, 15-30% 시간 절감

### 전략 5: Tasks 시스템 (AADS-145 구현 완료)
- ~/.claude/tasks/{task_id}.json 영속 관리
- 적용: 모든 작업
- 기대 효과: 세션 중단 후 복구 비용 $0.15-0.30/회 감소

### 전략 6: 프롬프트 캐싱
- 고정 프리픽스에 cache_control: {"type": "ephemeral"} 적용
- 적용 조건: API 호출 시
- 기대 효과: 반복 읽기 비용 90% 절감

### 전략 7: 모델 라우팅 (D-024 구현 완료)
- SIZE별 자동 모델 선택
- 적용: 모든 작업
- 기대 효과: XS 작업 30-40% 비용 절감

### 전략 8: 투기적 실행 (AADS-141 구현 완료)
- A 완료 전 B를 예측 실행, 실패 시 취소
- 신호: /tmp/aads_final_commit_{task_id}.signal
- 기대 효과: 전환 시간 30-60s → 5-10s

### 전략 9: 컨텍스트 윈도우 자동 관리 (AADS-145 구현 완료)
- 70% → /compact 권고, 90% → 중간 결과 저장 + /clear
- 2회 연속 편집 실패 → CTX-EDIT-FAIL 경고
- 기대 효과: 컨텍스트 초과 실패율 20-30% 감소

### 전략 10: CLAUDE.md 최적화
- Core, Skills, Archive 3단계 분리
- 적용: CLAUDE.md 파일 관리 시 (HANDOVER와 별개)
- 기대 효과: 초기 컨텍스트 사용량 50-70% 감소

### 전략 11: Writer/Reviewer 패턴
- review_required: true 시 자동 리뷰 세션 스폰
- 적용 조건: P0/P1 작업
- 기대 효과: 성공률 80% → 95%

### 전략 12: 우선순위 큐 + 가치-노력 매트릭스 (D-025 구현 완료)
- PENDING 큐를 impact/effort 비율로 자동 정렬
- auto_trigger.sh _select_next_file에 구현
- 기대 효과: 임팩트 2-3배 향상

---

## 6-8. 모델 라우팅 규칙 (D-024)

| SIZE | 모델 | 모델 ID | 입력 단가 (/1K) | 출력 단가 (/1K) |
|------|------|---------|-----------------|-----------------|
| XS | Haiku | claude-haiku-4-5 | $0.001 | $0.005 |
| S | Sonnet | claude-sonnet-4-6 | $0.003 | $0.015 |
| M | Sonnet | claude-sonnet-4-6 | $0.003 | $0.015 |
| L | Opus | claude-opus-4-6 | $0.015 | $0.075 |
| XL | Opus | claude-opus-4-6 | $0.015 | $0.075 |

적용 순서: 지시서 model 필드 → size 기반 자동 선택 → fallback sonnet

---

## 6-9. 비용 규칙

- 월 총 목표: $33 이내 (D-004)
- SIZE별 상한: XS $0.50, S $1.00, M $3.00, L $5.00, XL CEO 승인
- 세션 재시도 비용: $3.72-$4.03/회 (E2E 검증 기준)
- $5 초과 시 CEO 사전 승인 필수
- 비용 절감 우선순위: 재시도 방지 > 모델 라우팅 > 토큰 절약
- CEO 원칙: "Core 문서에는 비용을 아끼지 말고 최신화하라"

---

## 6-10. 4계층 자기치유 + 세션 관리 요약

| 계층 | 구성 요소 | 주기 | 역할 |
|------|-----------|------|------|
| L1 | claude_exec 하트비트 + inotifywait | 이벤트 기반 | 파일 변경 감지, 진행 확인 |
| L1.5 | session_watchdog.sh | 10초 | 세션 상태 감시 |
| L2 | watchdog_daemon + pipeline_monitor | 30초/2분 | 핵심 감시, Tier2 120초 진단+kill |
| L3 | meta_watchdog.sh (cron */1) | 1분 | L1-L2 감시자를 감시 |
| L4 | UptimeRobot / GitHub Actions | 외부 | 최종 안전망 |

- 에스컬레이션: L2 실패 → L3 개입 → L4 개입
- 서킷브레이커: 3회 연속 실패 → 5분 쿨다운 (circuit_breaker_state DB)
- Telegram 알림: Tier2, Tier3, Tier4, 완료
- 세션 슬롯: 글로벌 4 이하, 서버별 동적 1-3
- 투기적 실행: 완료 signal → auto_trigger 즉시 다음 투입

### 하트비트 파라미터

| 항목 | 값 |
|------|-----|
| 경고 | 60초 미갱신 |
| Tier2 진단 | 120초 → CPU + 시맨틱루프 판별 → kill + 재시작 |
| Tier3 강제종료 | 300초 → kill + recovery_logs |
| 하드 타임아웃 | 7,200초 (2시간, 안전망) |

---

## 6-11. Telegram 알림 규칙

| 이벤트 | 조건 |
|--------|------|
| Tier2 진단 | 120초 무반응 + 자동 kill/재시작 |
| Tier3 강제종료 | 300초 무반응 |
| Tier4 외부 감지 | 외부 모니터링 장애 감지 |
| 작업 완료 | 정상 완료 + git push 성공 |
| git push 실패 | 3회 재시도 모두 실패 |

---

## 6-12. 프로젝트별 특수 규칙

| 프로젝트 | 서버 | 도구 | WORKDIR | 특수 규칙 |
|----------|------|------|---------|-----------|
| AADS | 68 | Claude | /root/aads | LLM 15회/작업 이하, langgraph-supervisor 금지, Supabase 직접연결(5432) |
| SF | 114 | Cursor/Claude | /data/shortflow | gemini-2.0-flash 금지(404), OAuth 비공개, 업로드 private 유지 |
| GO100 | 211 | Cursor | /root/kis-autotrade-v4 | V4.1 immutable, strategy-cards CEO 승인 |
| KIS | 211 | Cursor | /root/kis-autotrade-v4 | GO100 동일 서버/제약 |
| NTV2 | 114 | Claude | /srv/newtalk-v2 | Laravel 12, Docker 재시작 제한, rfree-009(114.207.244.86) |
| NAS | Cafe24 | Claude | /root | claudebot만 허용, Flask/FastAPI |

---

## 6-14. 전 프로젝트 매니저 채팅 라우팅 테이블

| 프로젝트 | 매니저 채팅 URL | Task ID 접두사 | 서버 |
|----------|----------------|----------------|------|
| AADS | https://www.genspark.ai/agents?id=3d86d6f3-09a7-41b2-b91b-762a55512458 | AADS-xxx | 68 |
| KIS | https://www.genspark.ai/agents?id=77de652f-ca8c-4edb-b841-4ca3726b7bb4 | KIS-xxx | 211 |
| GO100 | https://www.genspark.ai/agents?id=167071cf-c8b5-476a-8953-6168dd6c910c | GO100-xxx | 211 |
| SF | (CEO 확인 필요) | SF-xxx | 114 |
| NTV2 | (CEO 확인 필요) | NT-xxx | 114 |
| NAS | (CEO 확인 필요) | NAS-xxx | Cafe24 |

---

## 6-13. 변경 이력 (최근 10건)

| 버전 | 날짜 | 변경 내용 |
|------|------|-----------|
| v1.2 | 2026-03-08 | AADS-178: §6-2 지시서발행흐름 Pre-Flight Check 추가, §6-2-2 매니저 Pre-Flight Check 절차 신규 |
| v1.1 | 2026-03-08 | AADS-161: §6-2 매니저 역할 보강(정체성+지시서발행+금지7항), §6-2-1 자기인식 프로토콜, §6-14 라우팅 테이블 |
| v1.0 | 2026-03-08 | AADS-148 신규 생성 — 13개 섹션 |
