# Sprint 2 Task 2.8 — FastAPI systemd 서비스 등록

**일시**: 2026-02-26 19:41 KST

## 등록 서비스

| 서비스   | 포트 | 설명                 | 비고                    |
|----------|------|----------------------|-------------------------|
| aads-api | 8001 | FastAPI 메인 서버    | enabled, 재부팅 시 자동 시작 |
| aads-redis | 6380 | AADS 전용 Redis   | 6380은 Docker 사용 중, systemd 미등록 |

## 로그 경로

- stdout: `/mnt/volume-sgp1-01/aads-logs/api-stdout.log`
- stderr: `/mnt/volume-sgp1-01/aads-logs/api-stderr.log`

## 관리 명령

- 상태: `systemctl status aads-api`
- 재시작: `systemctl restart aads-api`
- 로그: `journalctl -u aads-api -f`

## 검증

- `curl -s http://localhost:8001/health` → `{"status":"healthy",...}` 정상 응답
- 서비스: `systemctl is-enabled aads-api` → enabled

## 적용·배포

- 적용: ✅ `/etc/systemd/system/aads-api.service` 반영
- 배포: N/A (동일 서버 systemd 설정)
