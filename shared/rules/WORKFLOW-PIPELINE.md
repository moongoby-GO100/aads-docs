# WORKFLOW-PIPELINE v3.1
최종 업데이트: 2026-03-07 | 버전: v3.1 — AADS-144 D-024 모델 라우팅 추가

## 개요
AADS 자율 개발 시스템의 8단계 파이프라인 정의.
6개 프로젝트(AADS, GO100, KIS, SF, NTV2, NAS) 공통 적용.

---

## 8단계 파이프라인

| 단계 | 이름 | 주체 | 설명 |
|------|------|------|------|
| 1 | CEO 지시 | CEO (moongoby) | Genspark 에이전트 채팅 또는 직접 Bridge 디렉티브 작성 |
| 2 | Bridge 감지 | bridge.py (서버 211) | pending 디렉토리 파일 감지 → auto_trigger 라우팅 |
| 3 | 사전 검증 | auto_trigger.sh | WORKDIR 권한, 중복 체크, 의존성(DEPENDS_ON) 충족 확인 |
| 4 | 우선순위 전송 | auto_trigger.sh | 프로젝트별 서버 라우팅, SCP 전송, SSH claude_exec.sh 실행 |
| 5 | Claude 실행 | claude_exec.sh | claudebot 계정에서 Claude Code 실행, 하트비트, 타임아웃 관리 |
| 6 | 결과 보고 | claude_exec.sh | RESULT_FILE 생성, commit SHA 기록, Telegram 알림 |
| 7 | DB 기록 | AADS API | recovery_logs, directive_lifecycle, usage_logger 기록 |
| 8 | 교차 검증 | session_watchdog + 3서버 | git-push HTTP 200 확인, 교차 모니터링, 에스컬레이션 |

---

## Bridge → auto_trigger 라우팅 구조

```
CEO 지시 (Genspark 채팅)
     ↓
bridge.py (서버 211)
     ├── pending 디렉토리 파일 저장: /root/.genspark/directives/pending/
     └── auto_trigger.sh 폴링 루프 (10초 주기)
              ↓
     프로젝트별 라우팅
     ├── KIS / GO100  → 서버 211 로컬 실행
     ├── AADS         → 서버 68 (SSH) → claude_exec.sh
     ├── SF / NTV2 / NAS → 서버 114 (SSH) → claude_exec.sh
     └── NTV2 라우팅: rfree-009 (114.207.244.86) 확인 후 반영
```

### 서버 라우팅 매핑
| 프로젝트 | 서버 | IP | WORKDIR |
|----------|------|-----|---------|
| KIS | 211 (로컬) | 211.188.51.113 | /root/kis-autotrade-v4 |
| GO100 | 211 (로컬) | 211.188.51.113 | /root/kis-autotrade-v4 |
| AADS | 68 (원격) | 68.183.183.11 | /root/aads |
| SF | 114 (원격) | 116.120.58.155 | /data/shortflow |
| NTV2 | 114 (원격) | 116.120.58.155 / rfree-009 114.207.244.86 | /srv/newtalk-v2 |
| NAS | 114 (원격) | 116.120.58.155 | /root |

---

## 하드 타임아웃

| 항목 | 값 | 설명 |
|------|-----|------|
| 하드 타임아웃 | **7200초 (2시간)** | L1 안전망. 초과 시 자체종료 + Telegram 알림 |
| 소프트 경고 | 6600초 (110분) | 경고 Telegram 발송 |
| 하트비트 감시 | 10초 주기 | session_watchdog.sh |
| Tier2 대기 기준 | 120초 무반응 | CPU + 시맨틱루프 판별 |
| Tier3 강제종료 | 300초 무반응 | kill + recovery_logs 기록 |

---

## git-push 책임 체계

### 1차 책임: Claude Code 작업자
- 작업 완료 후 git commit + push 수행
- commit SHA를 RESULT_FILE YAML 헤더에 기록
- 형식: `commit_sha: <40자 SHA>`

### 2차 확인: auto_trigger.sh 후처리
- RESULT_FILE에서 commit_sha 추출
- GitHub raw URL curl → HTTP 200 확인
- 최대 3회 재시도 (exponential backoff: 10초, 20초, 40초)
- 실패 시: push_failed 로그 + Telegram 알림 + recovery_logs DB 기록
- 매니저 에스컬레이션 트리거

---

## D-024 모델 라우팅 (AADS-144)

지시서 `model:` 필드 → size 기반 자동 선택 → fallback sonnet 순서로 적용.

| size | 기본 모델 | 모델 ID | 비고 |
|------|----------|---------|------|
| XS | haiku | claude-haiku-4-5 | 단순 문서 수정, 1파일 이하 |
| S | sonnet | claude-sonnet-4-6 | 소규모 수정, 1~3파일 |
| M | sonnet | claude-sonnet-4-6 | 중규모 기능, 다중 파일 |
| L | sonnet | claude-sonnet-4-6 | 대규모 기능 (opus 선택 가능) |
| XL | opus | claude-opus-4-6 | 전체 시스템, E2E |

`model:` 필드로 오버라이드 가능. claude_exec.sh가 지시서에서 동적 읽기 (D-022 참조).

---

## NTV2 라우팅 확인

- 기본 경로: server-114 (116.120.58.155), 포트 7916
- rfree-009 확인: 114.207.244.86 (별도 설정 시 적용)
- 현재: auto_trigger.sh REMOTE_HOST_MAP["NTV2"]="server-114" 사용

---

## 참조
- RULE-MATRIX.md: 규칙 × 8단계 매핑 (v1.1)
- HANDOVER.md: 프로젝트별 최신 상태 (Core)
- HANDOVER-HISTORY.md: 최근 완료 태스크 상세
- CEO-DIRECTIVES.md: D-016~D-025, R-001~R-016
