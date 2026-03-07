---
id: L-010
title: /proc grep 블로킹 + 고아 프로세스 CPU 독점 방지
category: infra
severity: critical
task_ref: AADS-148
created_at: 2026-03-07
---

## 핵심 교훈

Claude Code에서 프로세스 탐색 시 `/proc`, `/sys` 경로에 `grep -r`을 실행하면
소켓·파이프 fd가 무한 블로킹되어 CPU 100%를 수일간 독점할 수 있다.
부모 세션 종료 후 자식이 고아 프로세스로 잔존하는 것을 방지하려면
프로세스 그룹 단위 kill과 session_watchdog 고아 탐지가 필수다.

## 문제

- 서버 211에서 Claude Code 세션이 `grep -r 73340093 /proc/*/fd/*`를 실행
- `/proc/*/fd/`의 소켓(socket), 파이프(pipe) 파일을 read() 시도 → 커널 I/O 블로킹
- 부모 세션(PID 20810) 종료 후 grep 자식(PID 20812)이 PPID=1 고아로 잔존
- 3일간(2026-03-04~07) CPU 1코어 100% 독점, postgres INSERT 큐 적체 발생
- session_watchdog이 PPID=1 고아 프로세스를 탐지하지 못해 자동 해소 실패

## 해결책

### 규칙 1: Claude Code /proc, /sys grep 금지

Claude Code 세션 컨텍스트(CONTEXT_HEADER)에 다음 규칙을 주입한다:

```
[보안 규칙] 절대로 /proc, /sys 경로에 grep -r을 실행하지 마라.
프로세스 탐색은 반드시 pgrep, ps, lsof를 사용하라.
```

프로세스 탐색 대체 명령:
```bash
# 금지 (블로킹 위험)
grep -r <pattern> /proc/*/fd/*
grep -r <pattern> /sys/

# 권장
pgrep -f "process_name"
ps aux | grep "process_name"
lsof -p <pid>
lsof -c <command>
```

### 규칙 2: 프로세스 그룹 단위 kill

`claude_exec.sh` 시작 시 PGID를 기록하고, cleanup trap에서 그룹 전체를 kill한다:

```bash
# 세션 시작 시 PGID 기록
PGID=$(ps -o pgid= -p $$ 2>/dev/null | tr -d ' ' || echo $$)

# cleanup 함수
cleanup_session() {
    # 프로세스 그룹 전체 kill (고아 프로세스 방지)
    if [ -n "${PGID:-}" ] && [ "$PGID" -gt 1 ]; then
        kill -- -"$PGID" 2>/dev/null || true
    fi
    # ... 기타 정리
}
trap cleanup_session EXIT INT TERM
```

### 규칙 3: session_watchdog 고아 프로세스 탐지

session_watchdog 메인 루프에 고아 프로세스 탐지를 추가한다:

```bash
check_orphan_processes() {
    # PPID=1 AND user=claudebot AND elapsed > 3600초
    local orphan_pids
    orphan_pids=$(ps -u claudebot -o pid,ppid,etimes,comm --no-headers 2>/dev/null \
        | awk '$2==1 && $3>3600 {print $1}')
    for opid in $orphan_pids; do
        log_msg "[ORPHAN] 고아 프로세스 감지: pid=${opid} → kill -9"
        send_tg "⚠️ 고아 프로세스 감지: pid=${opid} → kill -9"
        kill -9 "$opid" 2>/dev/null || true
    done
}
```

## 결과

- Before: grep /proc 블로킹 → CPU 100% × 3일, 자동 해소 불가
- After: 컨텍스트 규칙 주입 → Claude Code가 /proc grep 실행하지 않음
         PGID kill → 부모 종료 시 자식 프로세스 그룹 전체 즉시 제거
         session_watchdog 고아 탐지 → 1시간 이내 자동 kill

## 적용 방법

1. `scripts/claude_exec.sh` — CONTEXT_HEADER에 /proc grep 금지 규칙 주입 (이미 적용)
2. `scripts/claude_exec.sh` — cleanup_session() 함수에 PGID kill 추가 (이미 적용)
3. `scripts/session_watchdog.sh` — check_orphan_processes() 추가 (이미 적용)
4. 전체 프로젝트(AADS, KIS, GO100, SF, NTV2, NAS) claude_exec.sh 동일 패턴 적용

## 관련 파일

- `/root/aads/scripts/claude_exec.sh` — PGID kill + /proc 금지 주입
- `/root/aads/scripts/session_watchdog.sh` — 고아 프로세스 탐지+kill
- `/root/aads/aads-docs/reports/AADS-148-WRAP_proc-grep-blocking-incident.md` — 장애 Wrap 보고서
