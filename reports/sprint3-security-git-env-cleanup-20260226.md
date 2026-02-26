# 보안 태스크 — Git .env 이력 정리 + PostgreSQL 바인드

**일시**: 2026-02-26 20:xx KST

## Git .env 이력 정리

| 항목 | 내용 |
|------|------|
| 확인 | `git log --all --full-history --oneline -- .env` → **0건** |
| 방법 | 정리 불필요 (이력 없음) |
| 결과 | .env가 커밋된 이력 없음 — BFG/filter-branch 미실행 |
| force push | ❌ 불필요 |

## PostgreSQL 5432 바인드

| 항목 | 내용 |
|------|------|
| 설정 파일 | `/var/lib/pgsql/9.6/data/postgresql.conf` |
| 이전 | `listen_addresses = '*'` → *:5432, [::]:5432 |
| 이후 | `listen_addresses = 'localhost'` → 127.0.0.1:5432, [::1]:5432 |
| 백업 | `postgresql.conf.bak.20260226` |
| 재시작 | systemctl restart postgresql-9.6 ✅ |
| 영향 | 웹앱 등 로컬 접속만 허용, 외부에서 5432 접속 불가 |

## 주의사항

- **force push**: 미실행. 다른 로컬 클론 조치 불필요.
- **webapp**: 동일 서버 내에서만 PostgreSQL 접속 시 정상. 외부에서 5432로 접속하던 구성이었다면 연결 불가 → 로컬/유닉스 소켓 또는 SSH 터널 사용 필요.
