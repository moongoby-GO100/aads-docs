# Sprint 3 Task 3.8 — CI/CD GitHub Actions

**일시**: 2026-02-26 19:50 KST

## 워크플로우

| 파일 | 트리거 | 내용 |
|------|--------|------|
| test.yml | push main/develop, PR | Python 테스트 + lint |
| deploy.yml | push main (core/agents/api 변경) | SSH로 서버 배포 + 재시작 |
| dashboard.yml | push main (dashboard 변경) | Next.js 빌드 체크 |

## 필요 Secrets

- **DO_HOST**: DigitalOcean 서버 IP
- **DO_SSH_KEY**: SSH 프라이빗 키

설정 경로: GitHub 레포 (aads-core) → Settings → Secrets and variables → Actions

## 서비스

- Redis 6380 (GitHub Actions 서비스 컨테이너, test job)

## 적용·배포

- 적용: ✅ `.github/workflows/*.yml` 3개 반영
- 배포: 푸시 후 GitHub Actions 자동 실행 (Secrets 설정 후 유효)

## 체크사항

- [ ] GitHub Secrets(DO_HOST, DO_SSH_KEY) 등록
- [ ] main 푸시 시 test → deploy 순서 확인
- [ ] dashboard 변경 시 dashboard 빌드 워크플로우 동작 확인
