# Sprint 2 Task 2.9 — 헬스체크 + 모니터링

**일시**: 2026-02-26 19:41 KST

## 구성
| 항목 | 내용 |
|------|------|
| healthcheck.sh | 5분마다 cron 실행, FastAPI/Redis/디스크/메모리 체크 |
| server-status.sh | 수동 실행용 서버 상태 대시보드 |
| 로그 | /mnt/volume_sgp1_01/aads-logs/healthcheck-YYYYMMDD.log |

## 자동 복구
- FastAPI 다운 시 `systemctl restart aads-api` 자동 실행
- Redis 다운 시 `systemctl restart aads-redis` 자동 실행
- 디스크 90% 초과 시 경고 로그 기록

## cron
```
*/5 * * * * /root/aads/aads-core/scripts/healthcheck.sh
```

## 검증 (2026-02-26 실행 결과)
- FastAPI :8001 → 200
- Redis :6380 → PONG
- 디스크: 메인 82%, Volume 28%
- 메모리: 18%
