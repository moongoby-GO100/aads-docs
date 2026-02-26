# Sprint 2 Task 2.7.2 — Volume 50GB 연결 + 데이터 이전

**일시**: 2026-02-26 19:05 KST
**Volume**: volume-sgp1-01 (50GB, SGP1, ext4)
**마운트**: /mnt/volume_sgp1_01
**비용**: $5/월

## 구성
| 경로 | 용도 |
|------|------|
| /mnt/volume_sgp1_01/aads-logs | AADS 로그 (심볼릭 링크) |
| /mnt/volume_sgp1_01/aads-backups | AADS 백업 |
| /mnt/volume_sgp1_01/webapp-backups | webapp 백업 (심볼릭 링크 + 대용량) |

## 이전 파일
- pg_migration_20260209 (~762MB)
- urgent_fix_20260210 (~715MB)
- kisautotrade.db.bak (~100MB+)
- webapp daily 백업
- AADS 로그

## 디스크 현황
- 메인: 82% 사용
- Volume: 28% 사용
