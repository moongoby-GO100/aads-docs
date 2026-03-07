# AADS HANDOVER HISTORY
최종 업데이트: 2026-03-07 | D-023 3계층 분리 — 최근 완료 태스크 상세 기록

---

## AADS-149 완료 사항 (2026-03-07)
- **파이프라인 전수조사 버그 5건 Wrap 보고서**: reports/AADS-149-WRAP_pipeline-audit-5bugs.md
- **BUG-1 (Critical)**: auto_trigger.sh — SCP 실패 시 seen_tasks 영구 차단
  - 수정: SCP 실패 직후 `unset seen_tasks["$task_id"]` 롤백 로직 추가
- **BUG-2 (High)**: auto_trigger.sh — RESULT 폴러 타임아웃 25분 < HARD_TIMEOUT 30분
  - 수정: `seq 1 50` → `seq 1 80` (40분, HARD_TIMEOUT + 10분 여유)
- **BUG-3 (Medium)**: auto_trigger.sh — `aads_lifecycle_queued()` 호출 시 _rt/_title 변수 미정의
  - 수정: 변수 추출 코드를 lifecycle 호출 앞으로 이동
- **BUG-4 (Critical)**: claude_exec.sh — Claude가 pending/running 경로 삭제 가능 + /proc grep 금지 없음
  - 수정: CONTEXT_HEADER에 디렉토리 조작 금지 + /proc grep -r 금지 규칙 주입
- **BUG-5 (High)**: done_watcher.sh — 서버 114 SSH 포트 7916 미지정
  - 수정: `get_project_ssh_port()` 함수 추가, SSH/SCP 명령에 `-p/-P` 옵션 적용
- **교훈 L-011 등록**: shared/lessons/infra/L-011_pipeline-audit-critical-patterns.md (5대 패턴)
- **적용 서버**: 211(전체), 68(auto_trigger+claude_exec), 114(auto_trigger+claude_exec)

---

## AADS-145 완료 사항 (2026-03-07)
- **Tasks 시스템 통합**: scripts/claude_exec.sh + claude_exec.sh(main)
  - CLAUDE_CODE_TASK_LIST_ID 환경변수 자동 생성 및 export
  - 지시서 실행 시 ~/.claude/tasks/{task_id}.json 생성 (in_progress → done/failed)
  - 세션 복구: Tasks JSON 상태 체크로 PENDING/DONE 이중관리 제거
  - auto_trigger.sh: _process_directive에 Tasks 상태 사전 조회 추가
- **투기적 실행 (AADS-141 확장)**:
  - 하트비트 "type":"final_commit" 이벤트 추가 (scripts/claude_exec.sh)
  - /tmp/aads_final_commit_{task_id}.signal 파일로 신호 전달
  - auto_trigger.sh: _speculative_preload() 함수 — final_commit 감지 시 다음 작업 git pull 병렬 실행
  - 후처리 실패시 _preload_fail 플래그로 프리로드 취소
- **컨텍스트 자동관리**:
  - claude_exec.sh(main): _ctx_monitor() 백그라운드 — JSON output-format token usage 파싱
  - 70% 도달: 로그에 /compact 권고 기록
  - 90% 도달: 중간 결과 저장 + 재시작 프롬프트로 재실행
  - scripts/claude_exec.sh: 행 수 기반 토큰 추정 모니터링 + tee로 출력 캡처
  - 2회 연속 Edit 실패 감지 → CTX-EDIT-FAIL 경고

---

## AADS-144 완료 사항 (2026-03-07)
- **CEO-DIRECTIVES v3.2**: D-022~D-025 신규 추가
  - D-022: 지시서 포맷 v2.0 (필수/선택 필드 + 기본값 체계)
  - D-023: HANDOVER 3계층 분리 (Core/HISTORY/ARCHIVE)
  - D-024: 모델 라우팅 (XS→haiku, S/M→sonnet, L/XL→sonnet/opus)
  - D-025: 우선순위큐 impact/effort 정렬
- **HANDOVER 3계층 분리**: Core(≤1500토큰) + HISTORY.md + ARCHIVE.md 구조 적용
- **claude_exec.sh**: 지시서 model 필드 동적 읽기 + D-024 size 기반 자동 라우팅
- **auto_trigger.sh**: _select_next_file에 impact/effort 정렬 추가
- **WORKFLOW-PIPELINE v3.1**: D-024 모델 라우팅 섹션 추가
- **RULE-MATRIX v1.1**: D-022~D-025 행 추가

## AADS-143 완료 사항 (2026-03-07)
- **WORKFLOW-PIPELINE v3.0**: shared/rules/WORKFLOW-PIPELINE.md 생성 (8단계 파이프라인 + 라우팅 구조)
- **git-push 감시**: claude_exec.sh commit SHA 기록 + auto_trigger.sh verify_git_push() (3회 backoff + Telegram + recovery_logs)
- **HANDOVER 표준화**: AADS HANDOVER 3섹션 삽입 (작업 파이프라인/매니저 권한 한계/AI 작업자 규칙) + 5개 프로젝트 staged
- **NTV2/NAS CEO-DIRECTIVES**: NTV2-CEO-DIRECTIVES.md + NAS-CEO-DIRECTIVES.md 신규 배포
- **RULE-MATRIX.md**: 15개 규칙 × 8단계 매핑 매트릭스 생성
- **대시보드 트리거 통합**: /channels 트리거 전송 버튼 + /managers HANDOVER 링크

## AADS-142 완료 사항 (2026-03-07)
- deploy_heartbeat_3servers.sh: 3서버 일괄 배포 스크립트 (inotify-tools/jq 설치 + systemd 등록)
- CEO-DIRECTIVES v3.1: D-018 개정(하트비트 기반 L1) + D-021 신규(세션 관리 원칙) 배포
- E2E 검증: 파일럿 B-1(완료 흐름 4초), B-2(Tier2 시뮬레이션 150초), B-3(3건 즉시투입 2초) 완료
- 서버 68: session_watchdog RUNNING(PID 6166), meta_watchdog 감시 포함, HC=200 ✅

## AADS-141 완료 사항 (2026-03-07)
- auto_trigger.sh: /tmp/aads_trigger_next.signal 감지 → 즉시 투입 (크론 대기 없음), 크론 fallback 유지
- session_watchdog: trigger_post_processing()에서 signal 생성 + auto_trigger 즉시 호출
- 시맨틱 루프 에스컬레이션: PARTIAL 보고서 자동 생성 (10개 패턴+토큰+권고)
- recovery_logs DB: issue_type='semantic_loop' 기록
- CLAUDE.md 주입: 재시작 시 이전 실패 패턴 + 다른 접근법 지시
- 텔레그램 4종: Tier2(⚠️), Tier3(🔴), Tier4(🚨), 완료(✅)
- 슬롯 관리: check_global_slots() 글로벌 ≤4 + trigger_decisions.log 기록

## AADS-140 완료 사항 (2026-03-07)
- 기존: 고정 30분 하드 타임아웃 → 좀비 30분 방치 문제
- 신규: 하트비트 이벤트 기반 (120초 무반응 Tier2, 300초 Tier3)
- session_watchdog.sh: Tier2 CPU+시맨틱루프 통합판별, Tier3 강제종료+복구
- 하드 타임아웃은 7200초(2시간)로 연장, 안전망 역할만

## AADS-130 최종 완료 사항 (2026-03-06 Wrap-up)
- E2E 3시나리오 검증 완료: AI 퍼포먼스 마케팅 SaaS / K-12 교육 플랫폼 / 이커머스 셀러 도구
  - 각 시나리오: strategy_reports ✅ + project_plans ✅ + debate_logs ✅ + artifacts 6건 ✅
  - 건당 비용: $3.72~$4.03 (전체 $5 이하 기준 통과)
  - DB 총합: strategy_reports 4건, project_plans 4건, debate_logs 6건, artifacts 18건
- 소스코드 모듈화 (D-017 준수):
  - agents/: 10파일 (BaseAgent + strategist/planner/pm/supervisor/architect/developer/qa/judge/devops/researcher)
  - graphs/: 3파일 (ideation_subgraph/execution_chain/full_cycle_graph) + __init__
  - models/: 4파일 (strategy/plan/task/artifact) + __init__
  - services/: 4파일 (mcp_client/model_router/db_recorder/cost_tracker) + __init__
- 신규 DB 테이블: debate_logs (토론 이력) + projects (모드 컬럼 포함)
- 신규 API: GET /api/v1/debate-logs?project_id=...
- 테스트: strategist 7/7, planner 7/7, ideation_subgraph 9/9, full_cycle 10/10, unit 114/118

## AADS-128~129 완료 사항 (2026-03-06)
- AADS-128: Full-Cycle Graph (ideation+execution 서브그래프 통합), project_artifacts DB, artifacts API
- AADS-129: CEO 체크포인트 UI 4페이지 (select-item, approve-plan, full-cycle, reports)

---

## 참조
- Core: HANDOVER.md
- Archive: HANDOVER-ARCHIVE.md
- CEO-DIRECTIVES: CEO-DIRECTIVES.md (v3.2)
