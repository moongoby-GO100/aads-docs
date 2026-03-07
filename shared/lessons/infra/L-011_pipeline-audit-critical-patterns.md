---
id: L-011
title: 파이프라인 감사 — 5대 Critical 패턴 (상태롤백/폴러마진/변수순서/AI격리/멀티포트)
category: infra
severity: critical
task_ref: AADS-149
created_at: 2026-03-07
---

## 핵심 교훈

파이프라인 전수조사(2026-03-07)에서 Critical 2건, High 2건, Medium 1건이 발견됐다.
공통 원인: 낙관적 설계(실패를 상정하지 않은 코드), AI 격리 미비, 멀티서버 환경 차이 미반영.

---

## ① 상태 선등록 후 실행 실패 시 반드시 롤백 (BUG-1, Critical)

### 패턴

```bash
# 잘못된 패턴 (롤백 없음)
seen_tasks["$task_id"]=1
scp "$file" "$remote:$dir/" || { log_error "실패"; continue; }  # seen_tasks 미제거

# 올바른 패턴 (롤백 포함)
seen_tasks["$task_id"]=1
if ! scp "$file" "$remote:$dir/"; then
    log_error "SCP 실패 — seen_tasks 롤백: $task_id"
    unset seen_tasks["$task_id"]
    continue
fi
```

### 원칙

- **상태를 먼저 등록하는 모든 코드**는 후속 작업 실패 시 상태 롤백 로직을 필수 포함
- 특히 네트워크 전송(SCP, curl, API 호출) 직전 등록된 상태는 전송 실패 시 즉시 unset/delete
- 이 패턴을 따르지 않으면 파이프라인 큐가 영구 차단됨

---

## ② 폴러 타임아웃은 반드시 HARD_TIMEOUT 이상 (BUG-2, High)

### 패턴

```bash
# 잘못된 설정 (폴러 < HARD_TIMEOUT)
HARD_TIMEOUT=1800   # 30분
for i in $(seq 1 50); do sleep 30; done  # 25분 — 30분보다 짧음!

# 올바른 설정 (폴러 > HARD_TIMEOUT + 여유)
HARD_TIMEOUT=1800   # 30분
for i in $(seq 1 80); do sleep 30; done  # 40분 = HARD_TIMEOUT + 10분 여유
```

### 원칙

- **폴러 최대 대기 시간 ≥ HARD_TIMEOUT + 여유 마진 (최소 10분)**
- HARD_TIMEOUT 변경 시 모든 연관 폴러의 루프 횟수도 함께 재계산할 것
- 폴러가 먼저 종료되면 성공한 작업이 실패로 처리되어 중복 실행 발생

---

## ③ 변수 정의-사용 순서는 함수 호출 전 반드시 검증 (BUG-3, Medium)

### 패턴

```bash
# 잘못된 순서 (변수 미정의 상태로 함수 호출)
send_to_lifecycle "$task_id" "$_rt" "$_title"  # _rt, _title 아직 없음
_rt=$(grep "^runtime:" "$file" | ...)
_title=$(grep "^description:" "$file" | ...)

# 올바른 순서 (선정의 후 호출)
_rt=$(grep "^runtime:" "$file" | ...)
_title=$(grep "^description:" "$file" | ...)
send_to_lifecycle "$task_id" "$_rt" "$_title"  # 정상
```

### 원칙

- 함수가 받는 인자 변수는 **함수 호출 코드 위에** 정의되어야 함
- Bash는 런타임에 빈 문자열을 조용히 전달하므로 컴파일 오류 없이 DB에 빈 값 삽입
- 코드 리뷰 시 함수 호출 직전 모든 인자 변수의 할당 여부 확인 필수

---

## ④ AI 작업자에게 파이프라인 제어 디렉토리 접근 권한을 절대 부여하지 말 것 (BUG-4, Critical)

### 원칙

AI 작업자(Claude Code 등)의 실행 컨텍스트에 반드시 다음 규칙을 주입:

```
[파이프라인 보호 규칙]
- /root/.genspark/directives/pending/ 및 running/ 경로의 파일을 절대 삭제·이동·수정하지 마라
- 작업 디렉토리(WORKDIR) 외부에 파일을 생성하지 마라

[프로세스 탐색 규칙]
- /proc, /sys 경로에 grep -r을 절대 실행하지 마라 (커널 I/O 블로킹 위험)
- 프로세스 탐색은 반드시 pgrep, ps, lsof를 사용하라
```

### 왜 중요한가

- AI는 "파일 정리", "불필요한 파일 삭제" 등의 추론으로 파이프라인 큐를 삭제할 수 있음
- 파이프라인 제어 파일이 삭제되면 모든 진행 중 태스크가 고아 상태로 전락
- 컨텍스트 주입은 가장 저비용이면서 가장 효과적인 예방책

### 관련

- L-010: /proc grep 블로킹 + 고아 프로세스 (AADS-148) — /proc 금지 규칙의 원점

---

## ⑤ 멀티서버 환경에서 SSH 포트는 프로젝트별 매핑 함수로 관리 (BUG-5, High)

### 패턴

```bash
# 잘못된 방법 (하드코딩 또는 기본값 가정)
ssh user@116.120.58.155 "..."  # 포트 7916이지만 22로 연결 → 실패

# 올바른 방법 (매핑 함수)
get_project_ssh_port() {
    case "$1" in
        SF|NTV2|NAS) echo "7916" ;;  # 서버 114
        *)           echo "22"   ;;  # 서버 68, 211
    esac
}

SSH_PORT=$(get_project_ssh_port "$PROJECT")
ssh  -p "$SSH_PORT" user@"$HOST" "..."
scp  -P "$SSH_PORT" user@"$HOST":"$src" "$dst"   # scp는 대문자 -P
```

### 원칙

- **멀티서버 환경에서 SSH 포트를 직접 하드코딩하지 마라**
- 서버 추가·포트 변경 시 매핑 함수 1곳만 수정하면 전체 반영됨
- `ssh -p` (소문자), `scp -P` (대문자) 포트 옵션 차이에 주의

### 현재 서버 포트 현황

| 서버 | IP | SSH 포트 | 프로젝트 |
|------|----|----------|----------|
| 211 | 211.188.51.113 | 22 | KIS, GO100 |
| 68 | 68.183.183.11 | 22 | AADS |
| 114 | 116.120.58.155 | **7916** | SF, NTV2, NAS |

---

## 요약 체크리스트

파이프라인 스크립트 작성/리뷰 시 확인할 사항:

- [ ] 상태 등록(seen_tasks, DB insert) 후 실패 경로에 롤백 코드 있는가?
- [ ] 폴러 최대 대기 시간 ≥ HARD_TIMEOUT + 10분 인가?
- [ ] 함수 호출 인자 변수가 호출 코드 위에서 모두 정의되어 있는가?
- [ ] AI 작업자 CONTEXT_HEADER에 파이프라인 경로 보호 규칙이 주입되어 있는가?
- [ ] SSH/SCP 명령에 `get_project_ssh_port()` 또는 동등한 매핑이 적용되어 있는가?

## 관련 파일

- `scripts/auto_trigger.sh` — BUG-1,2,3 수정
- `claude_exec.sh` — BUG-4 CONTEXT_HEADER 주입
- `scripts/done_watcher.sh` — BUG-5 get_project_ssh_port() 추가
- `reports/AADS-149-WRAP_pipeline-audit-5bugs.md` — 전체 버그 Wrap 보고서
- `shared/lessons/infra/L-010_proc-grep-orphan-process.md` — /proc grep 관련 L-010
