# GO100 HANDOVER v1.1
최종 업데이트: 2026-03-07 | 버전: v1.1 — AADS-146 D-022~D-025 추가
> 배포 대상: GO100 repo (서버 211, /root/kis-autotrade-v4/HANDOVER.md)

## 시스템 개요
GO100: AI 주식 자동매매 시스템 (KIS API 기반)
서버: 211.188.51.113 (서버 211)
WORKDIR: /root/kis-autotrade-v4

## 작업 파이프라인
8단계 파이프라인 (WORKFLOW-PIPELINE v3.0 기준):

| 단계 | 이름 | 주체 |
|------|------|------|
| 1 | CEO 지시 | CEO (Genspark 에이전트 또는 직접 작성) |
| 2 | Bridge 감지 | bridge.py (서버 211, pending 저장) |
| 3 | 사전 검증 | auto_trigger.sh (WORKDIR·중복·의존성) |
| 4 | 우선순위 전송 | auto_trigger.sh (로컬 실행 — 서버 211) |
| 5 | Claude 실행 | claude_exec.sh (claudebot, 하트비트, 2h 타임아웃) |
| 6 | 결과 보고 | claude_exec.sh (RESULT_FILE + commit_sha 기록) |
| 7 | DB 기록 | AADS API (recovery_logs, lifecycle, usage) |
| 8 | 교차 검증 | session_watchdog + 3서버 (git-push HTTP 200 확인) |

상세: /root/aads/shared/rules/WORKFLOW-PIPELINE.md

## 매니저 권한 한계
- 매니저(Genspark AI)는 **CEO 지시 없이 임의 태스크를 생성·변경 불가**
- HANDOVER.md 수정 권한: CEO만 (매니저 read-only)
- 서버 접근: 매니저는 SSH 직접 접근 불가 — auto_trigger.sh 통해서만
- 예산 한도: 태스크당 $5 이하 (D-004). 초과 시 CEO 승인 필수
- 계정 스위칭: claude_exec.sh 자동 처리 — 매니저 개입 불가
- 에스컬레이션: L2→L3→L4 순서. 매니저가 직접 L4(외부) 호출 불가

## AI 작업자 규칙
- **R-001**: 작업 완료 후 HANDOVER.md 업데이트 의무
- **D-016**: 모든 작업 FLOW 프레임워크 준수 (Find→Layout→Operate→Wrap up)
- **D-018**: L1 하트비트 기반 세션 관리 (60/120/300초 임계값)
- **D-021**: 하트비트 발신 의무 — inotifywait 또는 git status fallback
- **D-022**: 지시서 포맷 v2.0 준수 (필수 6필드 + 선택 7필드)
- **D-023**: HANDOVER Core ≤1500토큰 유지. 상세는 HISTORY/ARCHIVE로
- **D-024**: model 필드 또는 size 기반 자동 라우팅 (XS→haiku, S/M→sonnet, L/XL→opus)
- **D-025**: 동일 priority 내 impact/effort 점수 높은 순 실행
- **R-014**: WRAP 보고서 생성 후 auto_trigger WRAP 게이트 통과 필수
- **R-016**: 서킷브레이커 준수 — 3회 연속 실패 시 5분 쿨다운
- 작업 디렉토리 이탈 금지: /root/kis-autotrade-v4 외부 파일 생성 불가
- git-push 의무: 작업 완료 후 commit + push + SHA RESULT_FILE 기록

## 프로젝트 현황
| 항목 | 값 |
|------|-----|
| 최근 태스크 | GO100-023 |
| 상태 | 정상 운영 중 |
| 서버 | 211 (로컬 실행) |

## 참조
- AADS HANDOVER: /root/aads/aads-docs/HANDOVER.md
- 워크플로우: /root/aads/shared/rules/WORKFLOW-PIPELINE.md
- 규칙 매트릭스: /root/aads/shared/rules/RULE-MATRIX.md
