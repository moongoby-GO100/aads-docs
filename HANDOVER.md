# AADS HANDOVER v8.1
최종 업데이트: 2026-03-07 | 버전: v8.1 — AADS-145 EFFICIENCY Phase1 파이프라인 고도화

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
| AADS | Phase 2 운영 | AADS-145 | 완료 |
| KIS | V4.1 운영 | KIS-041 | 정상 |
| GO100 | 운영중 | GO100-023 | 정상 |
| NTV2 | Phase 1 | NT-001 환경구축 | 대기 |
| SF | 운영중 | SF-015 | 정상 |
| NAS | 유지보수 | NAS-010 | 정상 |

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

## 상세 참조
- 완료 이력: HANDOVER-HISTORY.md (최근 10건)
- 구버전 이력: HANDOVER-ARCHIVE.md
- AADS 전용 지식: /root/aads/aads-server/docs/knowledge/AADS-KNOWLEDGE.md
- CEO 지침: CEO-DIRECTIVES.md (v3.2)
- 워크플로우: shared/rules/WORKFLOW-PIPELINE.md (v3.1)
- 규칙 매트릭스: shared/rules/RULE-MATRIX.md (v1.1)
