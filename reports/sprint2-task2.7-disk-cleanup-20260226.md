# Sprint 2 Task 2.7 — 서버 디스크 정리 보고

**일시**: 2026-02-26 18:12 KST  
**작업**: 디스크 사용률 96% → 목표 80% 이하

## 정리 항목

| 항목 | 내용 |
|------|------|
| 시스템 로그 | journalctl --vacuum-size=50M |
| yum 캐시 | yum clean all |
| pip 캐시 | pip cache purge (1.2 MB) |
| /tmp | 7일 이상 오래된 파일 삭제 |
| AADS 로그 | 30일+ 대상 없음 (유지) |
| .env 백업 | 최신 1개만 유지 |
| Docker | system prune (0B reclaimed) |
| /var/log/btmp | truncate (약 185 MB 확보) |
| webapp backup/daily | **7일 초과 분 삭제** (최근 7일만 유지, 약 8 GB 확보) |

## 결과

- **정리 전**: 97% 사용 (154G / 160G, 가용 6.2G)
- **정리 후**: 91% 사용 (146G / 160G, 가용 15G)

## 대용량 파일 (100MB+)

- `/root/webapp/backend/kisautotrade.db` — 실서비스 DB (유지)
- `/root/webapp/backup/daily/*.sql` — 최근 7일만 유지 (정리 완료)
- `/root/webapp/backup/pg_migration_20260209/`, `urgent_fix_20260210/` — 필요 시 검토 후 삭제
- `/root/webapp/backend/kisautotrade.db.bak_20260208_223903` — 필요 시 삭제 검토

## 후속

- **80% 미달**: 현재 91%. 80% 이하를 위해 DigitalOcean Volume 추가 또는 추가 백업 정리 검토.
- **선택 정리**: `pg_migration_20260209`(762M), `urgent_fix_20260210`(715M) 디렉터리 미사용 시 삭제로 약 1.5G 추가 확보 가능.
- **체크**: cron 일일 백업이 최근 7일만 유지되도록 정책 반영 여부 확인 권장.
