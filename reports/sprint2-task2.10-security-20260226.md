# Sprint 2 Task 2.10 — .env 암호화 백업 + 보안 점검

**일시**: 2026-02-26 19:41 KST

## 점검 항목

| 항목 | 상태 |
|------|------|
| .env Git 이력 포함 | ⚠️ 과거 커밋 77d499c(Initial Setup)에 .env 포함 이력 있음. 현재는 .gitignore로 차단 |
| .gitignore .env 차단 | ✅ `.env`, `.env.backup.*` 포함 |
| pre-commit hook 동작 | ✅ .env 파일 및 API 키 패턴(sk-ant-, sk-proj- 등) 차단 |
| .env 암호화 Spaces 백업 | ✅ `.env.encrypted.20260226_194106` 업로드 완료 |
| .env 파일 권한 600 | ✅ 수정 완료 (기존 644 → 600) |
| Git 내 비밀값 노출 | ✅ .git 내 실제 API 키 문자열 없음 (hook 스크립트 내 패턴 문자열만 존재) |
| 불필요 포트 노출 | ⚠️ 5432(PostgreSQL) 전역 리스닝 — 필요 시 bind 주소 제한 권장 |

## 백업 파일

- **Spaces 경로**: `s3://newtalk1/aads-backups/env/.env.encrypted.20260226_194106`
- **암호화**: OpenSSL aes-256-cbc -salt (CentOS 7 OpenSSL 1.0.2, -pbkdf2 미지원)

## 복원 방법

```bash
s3cmd get s3://newtalk1/aads-backups/env/.env.encrypted.20260226_194106 /tmp/
openssl enc -aes-256-cbc -d -salt -in /tmp/.env.encrypted.20260226_194106 -out .env -pass "pass:aads-backup-2026"
chmod 600 .env
rm /tmp/.env.encrypted.20260226_194106
```

※ 신규 OpenSSL(pbkdf2 지원) 환경에서는 `-pbkdf2` 옵션 추가 가능.

## 권고사항

- **ENV_BACKUP_PASSWORD**를 별도 비밀관리 도구에 저장 후 `ENV_BACKUP_PASSWORD` 환경변수로 사용 권장 (현재 기본값 사용).
- 주 1회 자동 암호화 백업 cron 추가 권장.
- .env가 과거에 커밋된 저장소이므로, 필요 시 `git filter-branch` 또는 BFG 등으로 이력에서 .env 제거 검토(이미 .env가 삭제되어 있으면 커밋 해시만 남음).
