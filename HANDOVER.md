# AADS HANDOVER v8.5
최종 업데이트: 2026-03-07 | 버전: v8.5 — AADS-149 파이프라인 전수조사 버그 5건 수정 Wrap

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
| AADS | Phase 2 운영 | AADS-149 | 완료 |
| KIS | V4.1 운영 | KIS-041 | 정상 |
| GO100 | 운영중 | GO100-023 | 정상 |
| NTV2 | Phase 1 | NT-001 환경구축 | 대기 |
| SF | 운영중 | SF-015 | 정상 |
| NAS | 유지보수 | NAS-010 | 정상 |

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
