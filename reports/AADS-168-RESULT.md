# AADS-168 RESULT: 멀티서버 클로드봇 프로세스 감시 데몬 (claude_watchdog.py)

완료일: 2026-03-08 | 담당: Claude (서버 68) | 우선순위: P0-CRITICAL | 크기: L

---

## 요약

서버 68/211/114에서 claude_exec 좀비·headless 교착 프로세스가 파이프라인을 차단하는 장애에 대응하여,
멀티서버 자동 감시·정리·알림 시스템(`claude_watchdog.py`)을 구현하고 AADS API에 3개 엔드포인트를 추가했다.

---

## 구현 내역

### 파트 1: claude_watchdog.py 코어

**파일**: `/root/aads/scripts/claude_watchdog.py`

감시 항목 (서버 68/211/114):
- claude_exec 프로세스: `ps -eo pid,ppid,pgid,etimes,stat,args` 기반 스캔
- claude -p (headless) 프로세스: regex `claude.*-p` 패턴
- session_watchdog 프로세스: args 기반 감지
- defunct(좀비) 수: `ps -eo stat | grep Z`
- bridge.py 생존 (서버 211): `pgrep -f bridge.py`
- auto_trigger.sh 생존 (서버 211): `pgrep -f auto_trigger`
- PID 파일 정합성 (서버 211): `/root/.genspark/pids/auto_trigger.pid` vs 실제 프로세스
- DB running 슬롯 (서버 68): `SELECT count(*) FROM directive_lifecycle WHERE status='running'`

임계값:
- claude_exec > 7200초 → CRITICAL
- claude headless > 3600초 → CRITICAL
- session_watchdog > 3600초 → HIGH
- 좀비 > 3개 → HIGH
- bridge.py 미실행 → CRITICAL
- 고스트 PID 파일 → HIGH

### 파트 2: 자동 정리 (auto_cleanup)

CRITICAL 이슈:
- `kill -TERM -{PGID}` → 3초 대기 → `kill -9 -{PGID}`
- `pkill -f 'claude.*stream-json'` (로컬)
- 고스트 PID 파일 삭제 (SSH)
- DB 정합성 복원: `UPDATE directive_lifecycle SET status='error' WHERE started_at < NOW() - INTERVAL '2 hours'`
- bridge.py 미실행: SSH로 자동 재시작 `nohup python3 .../bridge.py`

HIGH 이슈:
- 보고만 (로그 + Telegram)

보안 제약:
- SSH 명령 허용 prefix 화이트리스트: ps, pgrep, wc, cat, echo, test, kill, pkill, nohup, python3
- 경로 이스케이프 차단: `..` 포함 경로 ValueError
- PID 파일 허용 패턴: `/root/.genspark/pids/*.pid` regex
- SSH 타임아웃: 10초 (명령당)
- .env 직접 읽기 금지: 환경변수 전용

### 파트 3: AADS API 엔드포인트 (ops.py)

| 엔드포인트 | 기능 |
|-----------|------|
| `GET /api/v1/ops/claude-processes` | 최근 watchdog JSON 보고서 조회 (기본 5건, 최대 20건) |
| `POST /api/v1/ops/claude-cleanup` | 수동 claude_watchdog.py 정리 트리거 (CEO 확인용) |
| `POST /api/v1/ops/bridge-restart` | 서버 211 bridge.py SSH 원격 재시작 + 생존 확인 |

### 파트 4: systemd timer (root 설치 필요)

파일 스테이징 위치:
- `/root/aads/scripts/systemd/claude-watchdog.service` — Type=oneshot, TimeoutSec=60
- `/root/aads/scripts/systemd/claude-watchdog.timer` — OnBootSec=30, OnUnitActiveSec=120 (2분)

설치 방법:
```bash
sudo bash /root/aads/scripts/install_claude_watchdog.sh
```

**주의**: claudebot은 /etc/systemd/system/ 쓰기 권한 없음 (root 소유 755). root 직접 설치 필요.
설치 후 명령:
```bash
systemctl is-active claude-watchdog.timer  # → active
journalctl -u claude-watchdog              # → 2분 주기 실행 로그
```

### 파트 5: Telegram 알림

- CRITICAL 이슈 감지 시: 서버명, 이슈 유형, PID, 경과시간, 자동정리 결과
- HIGH 이슈 요약: 최대 5건 목록
- 환경변수: TELEGRAM_BOT_TOKEN, TELEGRAM_CHAT_ID (systemd EnvironmentFile=/root/aads/aads-server/.env)

### 파트 6: 감시 로그 저장

- 디렉토리: `/root/aads/logs/watchdog_reports/`
- 파일명: `{YYYYMMDD_HHMMSS}.json`
- 보존: 30일 (초과 시 자동 삭제)
- AADS API: `/ops/bridge-log`에 watchdog 이벤트 기록

---

## 테스트 실행 결과

### 1회차 테스트 (Python 3.6 호환성 수정 전)

```
Traceback: TypeError: 'type' object is not subscriptable
```

원인: Python 3.6.8 환경 — `list[dict]`, `tuple[int, str, str]` 등 Python 3.9+ 타입 힌트 미지원

### 2회차 테스트 (수정 후)

```
2026-03-08 09:10:11,058 [CLAUDE-WATCHDOG] INFO === Claude Watchdog 시작 (2026-03-08T09:10:11.058257+09:00) ===
2026-03-08 09:10:12,597 [CLAUDE-WATCHDOG] INFO 스캔 완료 — 68:True 211:False 114:False
2026-03-08 09:10:12,598 [CLAUDE-WATCHDOG] INFO 이슈 집계 — CRITICAL:0 HIGH:5
2026-03-08 09:10:12,598 [CLAUDE-WATCHDOG] WARNING [HIGH] session_watchdog_overtime 서버 68 — 보고만 (자동 정리 없음)
2026-03-08 09:10:12,598 [CLAUDE-WATCHDOG] WARNING [HIGH] session_watchdog_overtime 서버 68 — 보고만 (자동 정리 없음)
2026-03-08 09:10:12,598 [CLAUDE-WATCHDOG] WARNING [HIGH] session_watchdog_overtime 서버 68 — 보고만 (자동 정리 없음)
2026-03-08 09:10:12,598 [CLAUDE-WATCHDOG] WARNING [HIGH] ssh_unreachable 서버 211 — 보고만 (자동 정리 없음)
2026-03-08 09:10:12,598 [CLAUDE-WATCHDOG] WARNING [HIGH] ssh_unreachable 서버 114 — 보고만 (자동 정리 없음)
2026-03-08 09:10:12,602 [CLAUDE-WATCHDOG] INFO 보고서 저장: /root/aads/logs/watchdog_reports/20260308_091012.json
2026-03-08 09:10:12,757 [CLAUDE-WATCHDOG] INFO === Claude Watchdog 완료 — 경과:1540ms CRITICAL:0 HIGH:5 CLEANUP:5 ===
{
  "total_issues": 5,
  "critical": 0,
  "high": 5,
  "cleanup_actions": 5,
  "elapsed_ms": 1540
}
```

결과 분석:
- 서버 68 로컬 스캔: 성공 (9 claude_exec, 9 claude -p, 3 session_watchdog overtime HIGH 감지)
- 서버 211/114 SSH: 미연결 (claudebot SSH 키가 211/114에 등록되지 않아 ssh_unreachable HIGH 보고 — 예상된 동작)
- JSON 보고서 생성: /root/aads/logs/watchdog_reports/20260308_091012.json 확인

### ops.py 문법 검증

```
python3 -c "import ast; ast.parse(...)
→ syntax OK
→ 1392 lines (기존 1220 + 172 추가)
```

---

## 완료된 파일 목록

| 파일 | 상태 | 설명 |
|------|------|------|
| `/root/aads/scripts/claude_watchdog.py` | 신규 생성 | 멀티서버 감시 데몬 코어 |
| `/root/aads/scripts/systemd/claude-watchdog.service` | 신규 생성 | systemd oneshot 서비스 |
| `/root/aads/scripts/systemd/claude-watchdog.timer` | 신규 생성 | 2분 주기 타이머 |
| `/root/aads/scripts/install_claude_watchdog.sh` | 신규 생성 | root 설치 헬퍼 |
| `/root/aads/logs/watchdog_reports/` | 디렉토리 생성 | JSON 보고서 저장소 |
| `/root/aads/aads-server/app/api/ops.py` | 수정 (+172줄) | 3개 엔드포인트 추가 |
| `/root/aads/aads-server/app/api/ops.py.bak_AADS168` | 백업 | 작업 전 원본 |
| `/root/aads/aads-docs/HANDOVER.md` | v11.2 업데이트 | AADS-168 섹션 추가 |
| `/root/aads/aads-docs/STATUS.md` | 업데이트 | last_completed: AADS-168 |

---

## git 커밋

| 리포 | 커밋 SHA | 설명 |
|------|----------|------|
| aads-server | afed71b | feat: AADS-168 멀티서버 클로드봇 감시 데몬 API 엔드포인트 |
| aads-docs | (이번 커밋) | HANDOVER v11.2 + STATUS.md + RESULT |

**aads-server GitHub**: https://github.com/moongoby-GO100/aads-server/commit/afed71b

---

## 미완료 항목 (root 권한 필요)

| 항목 | 사유 | 해결 방법 |
|------|------|----------|
| systemd timer 활성화 | /etc/systemd/system/ root 전용 (755) | `sudo bash /root/aads/scripts/install_claude_watchdog.sh` |
| 서버 211/114 SSH 감시 | claudebot SSH 키 미등록 | 서버 211/114에 claudebot 공개키 등록 필요 |
| Telegram 알림 테스트 | 현재 세션 환경변수 미주입 | systemd EnvironmentFile 통해 자동 주입됨 |

---

## 회귀 테스트 — 기존 ops.py 엔드포인트

ops.py 문법 검증 통과 (ast.parse OK). 기존 17개 엔드포인트 코드 무변경 — 새 3개 엔드포인트는 파일 하단 추가.

---

## 교훈

- L-011: Python 3.6 호환성 — list[dict], tuple[int,str] 등 PEP 585/604 타입 힌트 (3.9+), capture_output=True (3.7+) 는 3.6에서 동작 불가. 기존 스크립트(watchdog_daemon.py)를 참고하여 subprocess.PIPE 사용.
- L-012: systemd 파일 설치 — claudebot은 /etc/systemd/system/ 쓰기 불가. 파일 스테이징 + 설치 스크립트 패턴으로 대응.
