# AADS-148 WRAP 보고서 — /proc grep 고아 프로세스 블로킹 장애

작성: 2026-03-07 | 작성자: Claude (AADS-148) | FLOW: D-016 Wrap up

---

## 1. 장애 요약

| 항목 | 내용 |
|------|------|
| 장애 발생 | 2026-03-04 (서울 기준) |
| 장애 해소 | 2026-03-07 kill -9 20812 |
| 지속 시간 | 약 3일 (72시간+) |
| 영향 서버 | 서버 211 (211.188.51.113) |
| 원인 프로세스 | grep -r 73340093 /proc/*/fd/* (PID 20812) |
| 부모 세션 | PID 20810 (claude_exec.sh → claude --print) |
| CPU 영향 | 1코어 100% 독점 |
| 후속 영향 | postgres INSERT 작업 큐 적체 (자연 해소) |

---

## 2. 장애 타임라인

```
2026-03-04 (추정)
  - claude_exec.sh 세션(PID 20810) 실행
  - Claude Code가 /proc/*/fd/* 경로에 grep -r 73340093 실행
  - /proc/*/fd/의 소켓·파이프 fd를 read 시도 → I/O 블로킹 발생
  - PID 20812 (grep)이 블로킹 상태로 CPU 1코어 100% 점유 시작

2026-03-04 ~ 2026-03-06
  - 부모 세션(PID 20810) 어느 시점에 종료
  - grep 자식 프로세스(PID 20812)는 PPID가 1로 변경되어 고아 프로세스화
  - session_watchdog.sh가 PPID=1인 고아 프로세스를 감지하지 못함
  - 3일간 CPU 1코어 100% 독점 지속

2026-03-07
  - CEO가 서버 부하 이상 감지
  - ps/top으로 PID 20812 확인 — grep -r /proc/*/fd/* 고아 프로세스 발견
  - kill -9 20812 실행 → CPU 정상화
  - 후속: postgres INSERT 큐 적체 확인 → 자연 해소 대기 후 완전 정상화
```

---

## 3. 근본 원인 분석

### 3.1 직접 원인

Claude Code 세션이 프로세스 탐색 목적으로 `/proc/*/fd/*` 경로에 `grep -r`을 실행한 것이 직접 원인이다.

`/proc/*/fd/*`는 소켓(`socket:[12345]`), 파이프(`pipe:[67890]`), 블록디바이스 등 특수 파일 디스크립터를 포함한다. `grep`이 이런 파일을 read()로 열면 데이터가 올 때까지 무한 대기(block)한다. 가상 파일시스템인 `/proc`에서 소켓 fd 읽기는 커널 내부 상태를 직접 접근하므로 응답이 오지 않거나 매우 늦게 온다.

### 3.2 구조적 원인

| # | 문제 | 설명 |
|---|------|------|
| 1 | /proc grep 방지 규칙 부재 | Claude Code 프롬프트에 `/proc`, `/sys` grep -r 금지 규칙이 없었음 |
| 2 | 프로세스 그룹 단위 kill 미적용 | `claude_exec.sh` 종료 시 자식 프로세스 그룹 전체를 kill하지 않음. 개별 PID만 kill함 |
| 3 | session_watchdog 고아 감지 없음 | PPID=1이 된 claudebot 소유 고아 프로세스를 session_watchdog이 탐지하지 못함 |
| 4 | 고아 프로세스 생존 기간 제한 없음 | 장기 실행 중인 고아 프로세스를 자동으로 kill하는 로직이 없었음 |

### 3.3 기여 인자

- `setsid` 미사용: claude_exec.sh가 새 세션/프로세스 그룹으로 실행되지 않아 PGID를 활용한 일괄 kill이 어려웠음
- 하드 타임아웃(HARD_TIMEOUT=7200)이 부모 프로세스 기준이었고 자식 프로세스에는 영향 없음
- cleanup_inotify() trap이 inotify PID만 kill하고 claude 서브프로세스 그룹 전체를 kill하지 않음

---

## 4. 영향 범위

| 항목 | 영향 |
|------|------|
| CPU | 서버 211 CPU 1코어 100% × 72시간+ |
| 정상 서비스 | KIS, GO100 작업 처리 지연 (서버 211 슬롯 부족) |
| PostgreSQL | INSERT 큐 적체 → 자연 해소 |
| 재정 | 서버 211 CPU 오버헤드 3일치 (약 $0.5~$1 추정) |
| 사용자 영향 | 대외 서비스 직접 영향 없음 (내부 파이프라인 처리 지연) |

---

## 5. 조치 내역 (AADS-148 수행)

### 5.1 즉시 조치 (사전 실행)
- `kill -9 20812` — 고아 grep 프로세스 즉시 제거

### 5.2 재발 방지 조치 (본 태스크)

| # | 조치 | 파일 | 상태 |
|---|------|------|------|
| 1 | `/proc`, `/sys` grep -r 금지 규칙 — CONTEXT_HEADER 주입 | scripts/claude_exec.sh | ✅ 완료 |
| 2 | 세션 종료 시 프로세스 그룹 전체 kill (`kill -- -$PGID`) | scripts/claude_exec.sh | ✅ 완료 |
| 3 | cleanup에 고아 프로세스 정리 로직 추가 | scripts/claude_exec.sh | ✅ 완료 |
| 4 | session_watchdog에 고아 프로세스 탐지+kill 로직 추가 | scripts/session_watchdog.sh | ✅ 완료 |
| 5 | 교훈 L-010 등록 (R-015) | shared/lessons/infra/L-010_proc-grep-orphan-process.md | ✅ 완료 |

---

## 6. 재발 방지 — 세부 구현

### 6.1 claude_exec.sh — /proc grep 금지 규칙 주입

CONTEXT_HEADER에 다음 규칙을 추가했다:

```
프로세스 탐색 시 /proc, /sys 경로에 grep -r을 절대 실행하지 마라.
프로세스 탐색은 반드시 pgrep, ps, lsof를 사용하라.
```

이 규칙은 모든 Claude Code 세션 시작 시 컨텍스트 헤더로 주입되므로 어떤 지시서가 들어와도 동일하게 적용된다.

### 6.2 claude_exec.sh — 프로세스 그룹 kill

세션 시작 시 현재 PGID를 기록하고, cleanup 함수에서 `kill -- -$PGID`를 실행한다:

```bash
PGID=$(ps -o pgid= -p $$ 2>/dev/null | tr -d ' ' || echo $$)

cleanup_session() {
    # 프로세스 그룹 전체 kill
    if [ -n "${PGID:-}" ] && [ "$PGID" -ne 1 ]; then
        kill -- -"$PGID" 2>/dev/null || true
    fi
    ...
}
trap cleanup_session EXIT
```

### 6.3 session_watchdog.sh — 고아 프로세스 탐지

메인 루프에 `check_orphan_processes()` 호출을 추가했다. 탐지 조건:
- PPID = 1 (부모 없이 init에 붙은 고아)
- 소유자 = claudebot
- 실행 시간 > 3600초 (1시간)

탐지 시: 경고 로그 + 텔레그램 알림 + kill -9

---

## 7. 후속 권고

1. **정기 고아 프로세스 점검**: 주 1회 `ps -u claudebot --ppid 1 -o pid,etime,cmd` 확인
2. **Claude Code 사용 가이드 갱신**: 프로세스 탐색 시 반드시 `pgrep` / `ps` / `lsof` 사용
3. **서버 211 CPU 모니터링 강화**: CPU > 80% 지속 5분 시 텔레그램 알림 (현재 없음)
4. **/proc 접근 감사**: 향후 Claude Code 세션에서 /proc 파일 접근 패턴을 inotifywait으로 추적 고려

---

## 8. Wrap up 체크리스트

- [x] 장애 타임라인 문서화
- [x] 근본 원인 분석 완료
- [x] 재발 방지 코드 변경 완료
- [x] 교훈 L-010 등록 완료
- [x] HANDOVER.md 업데이트 (AADS-148 완료 섹션)
- [x] aads-docs git commit + push

---

*보고서 작성: AADS-148 Claude Sonnet 4.6 | 2026-03-07 KST*
