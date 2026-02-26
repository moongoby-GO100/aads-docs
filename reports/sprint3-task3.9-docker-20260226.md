# Sprint 3 Task 3.9 — Docker Compose 배포 설정

**일시**: 2026-02-26 19:50 KST

## 생성 파일

| 파일 | 용도 |
|------|------|
| Dockerfile | FastAPI 서버 이미지 |
| Dockerfile.dashboard | Next.js 대시보드 이미지 |
| docker-compose.yml | 3개 서비스 오케스트레이션 |
| requirements.txt | Python 의존성 |
| .dockerignore | Docker 빌드 제외 |

## 서비스 구성

| 서비스 | 이미지 | 포트 | 헬스체크 |
|--------|--------|------|---------|
| redis | redis:7-alpine | 6380:6379 | redis-cli ping |
| api | Dockerfile | 8001:8001 | curl /health |
| dashboard | Dockerfile.dashboard | 3000:3000 | - |

## 사용법

```bash
# 전체 빌드 & 실행
docker compose up -d --build

# 개별 서비스
docker compose up -d api
docker compose logs -f api

# 중지
docker compose down
```

## 주의사항

- `.env`는 docker-compose에서 `env_file`로 주입 (이미지에 포함되지 않음).
- dashboard는 Next.js standalone 빌드 사용 (`next.config`에 `output: 'standalone'` 필요).
- 현재 **dashboard 디렉터리가 비어 있음** — Next.js 앱 추가 후 `docker compose up -d --build dashboard`로 빌드 가능.
- systemd 서비스와 병행 가능 (포트 8001/6380/3000 충돌 시 하나만 선택).

## 검증

- `docker compose config`: 문법 검증 통과 (Docker 26.1.4, Compose v2.24.0).
- Compose V2에서는 `version` 필드 생략 권장(이미 제거함).
