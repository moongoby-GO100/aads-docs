# AADS HANDOVER v9.0
최종 업데이트: 2026-03-07 | 버전: v9.0 — CEO 직접 검수: 버그수정 4건 + 모델드롭박스 + KST 동기화

## 시스템 개요
AADS (Autonomous AI Development System): 멀티 AI 에이전트 자율 개발 시스템
대시보드: https://aads.newtalk.kr/
리포: aads-docs, aads-server, aads-dashboard (moongoby-GO100)
GitHub PAT: repo+workflow, 만료 2026-05-27

## 작업 파이프라인 (8단계)
| 단계 | 이름 | 주체 |
|------|------|------|
| 1 | CEO 지시 | CEO (Genspark 에이전트 또는 직접 작성) |
| 2 | Bridge 감지 | bridge.py (서버 211, pending 저장) |
| 3 | 사전 검증 | auto_trigger.sh (WORKDIR·중복·의존성) |
| 4 | 우선순위 전송 | auto_trigger.sh (프로젝트별 서버 라우팅) |
| 5 | Claude 실행 | claude_exec.sh (claudebot, 하트비트, 2h 타임아웃) |
| 6 | 결과 보고 | claude_exec.sh (RESULT_FILE + commit_sha 기록) |
| 7 | DB 기록 | AADS API (recovery_logs, lifecycle, usage) |
| 8 | 교차 검증 | session_watchdog + 3서버 (git-push HTTP 200 확인) |

상세: shared/rules/WORKFLOW-PIPELINE.md

## 매니저 권한 한계
- 매니저(Genspark AI)는 **CEO 지시 없이 임의 태스크를 생성·변경 불가**
- HANDOVER.md 수정 권한: CEO만 (매니저 read-only)
- 서버 접근: 매니저는 SSH 직접 접근 불가 — auto_trigger.sh 통해서만
- 예산 한도: 태스크당 $5 이하 (D-004). 초과 시 CEO 승인 필수

## AI 작업자 규칙
- **R-001**: 작업 완료 후 HANDOVER.md 업데이트 의무
- **D-016**: 모든 작업 FLOW 프레임워크 준수 (Find→Layout→Operate→Wrap up)
- **D-022**: 지시서 포맷 v2.0 준수 (필수 6필드 + 선택 7필드)
- **D-023**: HANDOVER Core ≤1500토큰 유지. 상세는 HISTORY/ARCHIVE로
- **D-024**: model 필드 또는 size 기반 자동 라우팅 (XS→haiku, S/M→sonnet, L/XL→opus)
- **D-025**: 동일 priority 내 impact/effort 점수 높은 순 실행
- **R-014**: WRAP 보고서 생성 후 auto_trigger WRAP 게이트 통과 필수
- **R-016**: 서킷브레이커 준수 — 3회 연속 실패 시 5분 쿨다운
- 작업 디렉토리 이탈 금지: 지정된 WORKDIR 외부 파일 생성 불가
- git-push 의무: 작업 완료 후 commit + push + SHA RESULT_FILE 기록

## 서버 현황
| 서버 | IP | 역할 | 프로젝트 |
|------|-----|------|----------|
| 211 | 211.188.51.113 | Hub(Bridge, auto_trigger, pipeline_monitor) | KIS, GO100 |
| 68 | 68.183.183.11 | AADS Backend(FastAPI, PostgreSQL, Dashboard) | AADS |
| 114 | 116.120.58.155 | 실행 서버 | SF, NTV2 |

## 프로젝트 현황
| 프로젝트 | Phase | 최근 태스크 | 상태 |
|----------|-------|------------|------|
| AADS | Phase 2 운영 | AADS-160 (CEO 검수) | 완료 |
| KIS | V4.1 운영 | KIS-041 | 정상 |
| GO100 | 운영중 | GO100-023 | 정상 |
| NTV2 | Phase 1 | NT-001 환경구축 | 대기 |
| SF | 운영중 | SF-015 | 정상 |
| NAS | 유지보수 | NAS-010 | 정상 |

## AADS-160 CEO 직접 검수 + 버그수정 + UI개선 (2026-03-07)
- CEO 직접 AADS-157/158/159 검수 → 버그 4건 발견 및 수정
- **CEO Chat TypeError 2건**: logger.info structlog kwargs → f-string 변환 (9f496d1, 3d7d80d)
- **Dashboard React Error #31**: object를 React children으로 렌더링 → typeof 체크 + JSON.stringify (49311d1)
- **Ops toLocaleString/toFixed crash**: undefined 값에 메서드 호출 → nullish coalescing (49311d1, 33fa3ed)
- **모델 선택 드롭박스**: 버튼 7개 → select 드롭박스 29개 모델 (auto+Anthropic 11+OpenAI 11+Google 6) (af082cb)
- **KST 타임존 동기화**: 8개 파일 날짜를 Asia/Seoul timezone으로 통일 (fd51915)
  - ceo-chat, conversations, reports, ProjectCard, CheckpointList, select-item, managers, SSEMonitor
  - conversations: 수동 KST_OFFSET 계산 → native toLocaleString timezone 교체
- aads-server commits: 9f496d1, 3d7d80d
- aads-dashboard commits: 49311d1, 33fa3ed, af082cb, fd51915

## AADS-159 주요 변경 (2026-03-07)
- CEO Chat Playwright 브라우저 자동화 6개 도구 추가 (T-003 Phase 2 확장)
- ceo_chat_tools.py: browser_navigate/snapshot/screenshot/click/fill/tab_list
  - 도메인 화이트리스트 하드코딩: *.newtalk.kr, github.com, raw.githubusercontent.com, localhost
  - 차단 시 "[접근 차단] 허용되지 않은 도메인입니다" 반환
  - Playwright Python 싱글턴 컨텍스트 (asyncio.Lock, headless Chromium)
  - graceful degradation: playwright 미설치/초기화 실패 시 "[브라우저 도구 사용 불가]" 반환
  - 메모리 제한 512MB (--memory-pressure-off), 동시 탭 최대 3개, 세션 타임아웃 60초
- ceo_chat.py: Intent Classifier 6분류 (browser 추가)
  - browser 키워드: 스크린샷/페이지/열어/화면/브라우저/사이트/접속
  - 우선순위: execute > browser > dashboard > diagnosis > research > strategy
  - send_ceo_message: browser 의도 → _call_anthropic_with_tools 자동 분기
- supervisord.conf: playwright-mcp 엔트리 추가 (autostart=false, Node.js 옵션)
- aads-server commit: 1fbb76d

## AADS-158 주요 변경 (2026-03-07)
- Pending 대기큐 정리: 11개 완료·중복 지시서 → /root/.genspark/directives/archived/ 이동 (백업)
- 이동 파일: 065101/065506/121540/141313/141315/141317/142512/142514/142516/142518/142520 (모두 BRIDGE.md)
- T-AADS-150, T-AADS-148-A/B/C: pending에 부재 (이미 처리됨)
- STATUS.md: last_completed=AADS-157 → 갱신 완료

## AADS-157 주요 변경 (2026-03-07)
- CEO Chat v2 → AADS Core Engine 연결: Intent Classifier + DashboardCollector + Tool-use 루프 + Directive Submit
- classify_intent(): 5분류 (dashboard/diagnosis/research/execute/strategy) — 의도별 자동 라우팅
- DashboardCollector: 6소스 병렬 수집 (health, STATUS.md, projects, 세션비용, 태스크현황) + system_prompt 주입
- _call_anthropic_with_tools(): tool-use while 루프 (max 5 iter), tool_use block → execute_tool → 결과 재전달
- ceo_chat_tools.py 신규: read_file(화이트리스트), read_github, search_logs(100줄/10KB), query_db(SELECT전용), fetch_url(20KB)
- directives.py 신규: POST /api/v1/directives/submit → D-022 포맷 파일 생성 → bridge.py 파이프라인 투입
- _handle_execute_intent(): LLM 지시서 JSON 생성 → parse → submit_directive_sync() 직접 호출
- 응답에 intent 필드 추가 (dashboard/diagnosis/research/execute/strategy)
- aads-server commit: 65edfde

## AADS-156 주요 변경 (2026-03-07)
- CEO Chat 모델 패스스루: 프론트 model 선택값 → 백엔드 직접 사용 (MODEL_ID_MAP 제거)
- 하드코딩 제거: claude-*-4-5 → claude-opus-4-6/claude-sonnet-4-6 직접 사용
- SUPPORTED_MODELS 28개: Claude 11 + GPT 11 + Gemini 6 + GET /ceo-chat/models 엔드포인트
- 402 fallback: ANTHROPIC_API_KEY_2 자동 전환 (credit_balance_too_low)
- OpenAI 직접 호출: _call_openai() 추가 (GPT-5/gpt-5-mini 등 직접 라우팅)
- ModelSelector.tsx: 5개 → 7개 최신 모델 (Haiku 4.5, GPT-5 추가, Gemini 2.5 Flash)
- aads-server commit: 0576e79 | aads-dashboard commit: 5cd39d6

## AADS-149 주요 변경 (2026-03-07)
- 파이프라인 전수조사 버그 5건 Wrap 보고서: reports/AADS-149-WRAP_pipeline-audit-5bugs.md
- 교훈 L-011 등록: shared/lessons/infra/L-011_pipeline-audit-critical-patterns.md
- BUG-1(Critical): auto_trigger.sh seen_tasks 롤백 + BUG-2(High): 폴러 타임아웃 40분 확장
- BUG-3(Medium): lifecycle 변수 순서 수정 + BUG-4(Critical): CONTEXT_HEADER 파이프라인 보호 규칙
- BUG-5(High): done_watcher.sh get_project_ssh_port() 서버 114 포트 7916 적용

## AADS-148 주요 변경 (2026-03-07)
- /proc grep 블로킹 3일 장애 Wrap 보고서: reports/AADS-148-WRAP_proc-grep-blocking-incident.md
- 교훈 L-010 등록: shared/lessons/infra/L-010_proc-grep-orphan-process.md
- claude_exec.sh: PGID kill (`kill -- -$PGID`) + /proc grep 금지 CONTEXT_HEADER 주입
- session_watchdog.sh: check_orphan_processes() — ppid=1 AND elapsed>3600s 자동 kill
- aads-docs commit: b96e6f7 | aads-server commit: 2b7b16d

## AADS-146 주요 변경 (2026-03-07)
- Worktree 병렬: auto_trigger.sh `_parallel_worktree()` + merge_worktree.sh
- 서브에이전트: .claude/agents/ security-reviewer/test-writer/doc-writer + claude_exec.sh subagents 파싱
- Writer/Reviewer: `_spawn_review_session()` — P0/P1 review_required:true 시 자동 리뷰
- 5프로젝트 HANDOVER v1.1: D-022~D-025 추가 (GO100/KIS/SF/NTV2/NAS)
- 대시보드: /api/v1/managers 엔드포인트 + 3단계 fallback

## 복구 경로 (AADS-147)
브라우저 자동화 실패 시 → CEO가 매니저 대화창에 "상태확인" 입력
→ 매니저가 STATUS.md(context_docs) 읽어 chat_delivered=false 완료 작업 인식
→ report_url 보고서 확인 후 다음 지시 생성 (복구 소요 ~5초)
STATUS.md: https://raw.githubusercontent.com/moongoby-GO100/aads-docs/main/STATUS.md

## 긴급 이슈
없음

## CEO-DIRECTIVES 현행 원칙 (v3.2)
- D-016: FLOW 프레임워크 (Find→Layout→Operate→Wrap up)
- D-017: 소스코드 모듈화 (agents/graphs/models/services)
- D-018: 4계층 자기치유 (L1 하트비트 기반)
- D-019: 서버 상호 감시 (3서버 2분 크로스)
- D-020: 복구 이력 DB 의무화
- D-021: 하트비트 기반 세션 관리
- D-022: 지시서 포맷 v2.0
- D-023: HANDOVER 3계층 분리
- D-024: 모델 라우팅 (size 기반)
- D-025: 우선순위큐 impact/effort 정렬
- D-026: STATUS.md 브라우저 자동화 실패 복구 경로 (AADS-147)
- D-027: Worktree 병렬 실행 — parallel_group 필드 감지 시 자동 분기 (AADS-146)
- D-028: 서브에이전트 패턴 — subagents 필드 기반 보안/테스트/문서 에이전트 활성화 (AADS-146)
- D-029: Writer/Reviewer — P0/P1 review_required:true 시 리뷰 세션 자동 스폰 (AADS-146)

## 상세 참조
- 완료 이력: HANDOVER-HISTORY.md (최근 10건)
- 구버전 이력: HANDOVER-ARCHIVE.md
- AADS 전용 지식: /root/aads/aads-server/docs/knowledge/AADS-KNOWLEDGE.md
- CEO 지침: CEO-DIRECTIVES.md (v3.2)
- 워크플로우: shared/rules/WORKFLOW-PIPELINE.md (v3.1)
- 규칙 매트릭스: shared/rules/RULE-MATRIX.md (v1.1)
