# AADS 대시보드 Git/Docker 롤백 (2026-04-02 KST)

## 조치

- **저장소**: `aads-dashboard` `main` → 커밋 **`e0b896d`** (`git reset --hard` + `git push --force-with-lease origin main`).
- **런타임**: `docker compose -f docker-compose.prod.yml` 로 `aads-dashboard` 이미지 재빌드 후 컨테이너 재기동 (3100).

## 제외된 커밋 (롤백으로 제거됨)

- `6231348` — Dockerfile `.next/static` 선삭제
- `974c925` — 채팅 토큰/필터/사이드바 UX

## 배포 검증 (빌드 직후 컨테이너)

- `/app/.next/static/static/chunks` **없음** (`CHUNKS_OK`), `chunks/*.js` 54개.

## 참고

- 원격 히스토리가 `974c925` 이전으로 되돌아갔음. 필요 시 해당 커밋은 `git reflog` / GitHub 에서 복구 가능.
