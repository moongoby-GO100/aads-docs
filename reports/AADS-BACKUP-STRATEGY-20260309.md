# AADS 백업·복원 정책

- **최종 업데이트**: 2026-03-09
- **목적**: GitHub만으로는 복원에 부족한 항목 정리 + PostgreSQL 자동 백업 우선 적용

## 1. 복원 시 필요한 것 요약

| 대상 | 백업 방법 | 주기 | 비고 |
|------|------------|------|------|
| **소스코드** | GitHub (커밋/푸시) | 매 커밋 | 별도 백업 불필요 |
| **PostgreSQL DB** | pg_dump → 로컬/S3 또는 별도 스토리지 | **일 1회** | **가장 시급** — 메모리 레이어·세션 노트·관찰 데이터 전부 DB에 있음 |
| **.env / 시크릿** | 암호화 저장 (Vault, 1Password, 또는 암호화된 별도 레포) | 변경 시 | 커밋 금지 |
| **Nginx / systemd 설정** | 코드 레포의 infra/ 또는 Ansible/Docker IaC | 변경 시 | 현재는 수동 관리 가능 |
| **Docker 볼륨 / 업로드** | 스냅샷 또는 S3 동기화 | 주 1회 | 필요 시 |

## 2. PostgreSQL 자동 백업 (시급)

- **스크립트**: aads-server/scripts/backup_postgres.sh (docker exec aads-postgres pg_dump)
- **DB/유저**: aads / aads (docker-compose.prod.yml 기준)
- **출력**: /root/aads/backups/aads_YYYYMMDD_HHMMSS.sql, 보관 7일
- **cron**: 0 3 * * * /root/aads/aads-server/scripts/backup_postgres.sh >> /root/aads/logs/backup.log 2>&1
- 기존 backup.sh가 aads_db/aads_user로 실패 시 cron을 backup_postgres.sh로 교체.

## 3. 복원

docker exec -i aads-postgres psql -U aads -d aads < /root/aads/backups/aads_YYYYMMDD_HHMMSS.sql

**요약**: 소스는 GitHub로 충분. PostgreSQL 일일 백업을 먼저 적용하고, .env는 암호화 저장, Nginx/systemd는 infra/IaC로 관리 권장.
