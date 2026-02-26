# Sprint 2 Task 2.7.1 — Spaces 이중 백업

**일시**: 2026-02-26 19:24 KST
**대상**: newtalk1.sgp1.digitaloceanspaces.com

## 백업 항목
| 항목 | Spaces 경로 |
|------|-------------|
| .env 백업 | s3://newtalk1/aads-backups/env/ |
| AADS 보고서 | s3://newtalk1/aads-backups/reports/ |
| pg_migration | s3://newtalk1/server-backups/pg_migration_20260209/ |
| urgent_fix | s3://newtalk1/server-backups/urgent_fix_20260210/ |

## 이번 실행 결과
- .env 백업: 업로드 완료 (.env.backup.20260226_152310)
- AADS 보고서: 6개 파일 동기화 완료
- pg_migration / urgent_fix: 로컬 디렉터리 없음 → 스킵 (이미 이전 작업에서 Spaces 이동됨)

## 복원 방법
```
s3cmd get --recursive s3://newtalk1/aads-backups/ /복원경로/
s3cmd get --recursive s3://newtalk1/server-backups/ /복원경로/
```
