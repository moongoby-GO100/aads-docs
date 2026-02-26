# Sprint 2 Task 2.7.1 — Spaces 활용 디스크 확보

**일시**: 2026-02-26 19:00 KST  
**작업**: 대용량 백업 파일 → newtalk1 Spaces 이동 + 로컬 삭제

## 이동 파일

| 파일 | 크기 | Spaces 경로 |
|------|------|-------------|
| pg_migration_20260209/ | 762MB | s3://newtalk1/server-backups/pg_migration_20260209/ |
| urgent_fix_20260210/ | 715MB | s3://newtalk1/server-backups/urgent_fix_20260210/ |
| kisautotrade.db.bak_20260208_223903 | 395MB | s3://newtalk1/server-backups/kisautotrade.db.bak_20260208_223903 |

**총 이동**: 약 1.9GB

## 결과

- **정리 전**: 146G 사용, 15G 가용, 91%
- **정리 후**: 145G 사용, 16G 가용, 91%
- **추가 확보**: 약 1GB (로컬 삭제 ~1.9GB, df 반영 약 1GB)

## 검증

- s3cmd ls로 server-backups 내 3개 대상 존재 확인 완료
- 로컬 삭제 후 디렉터리/파일 부재 확인

## Spaces 접근 방법

```bash
s3cmd ls s3://newtalk1/server-backups/
s3cmd get s3://newtalk1/server-backups/파일명 /복원경로/
# 디렉터리 복원 예:
s3cmd get --recursive s3://newtalk1/server-backups/pg_migration_20260209/ /복원경로/
```

## 체크사항

- [ ] 주기적 백업 정리 시 동일 절차 재사용 가능
- ⚠️ .s3cfg 유지 관리 (Spaces 접근 필수)
