# AADS 대시보드 Next.js 청크 404 조치 보고 (2026-04-02 KST)

## 요약

- **현상**: 채팅(`/chat`) 등에서 `Failed to load chunk /_next/static/chunks/....js` 및 해당 URL 404.
- **원인**: 컨테이너 내 정적 파일이 `.next/static/static/chunks/`에만 있고, 런타임이 찾는 `.next/static/chunks/`와 불일치.
- **조치**: `Dockerfile`에 `RUN rm -rf ./.next/static` 후 static 복사 추가, `docker compose … build`로 이미지 갱신, `aads-dashboard` 컨테이너 재생성(3100).

## 변경 파일

| 경로 | 내용 |
|------|------|
| `aads-dashboard/Dockerfile` | runner 단계에서 standalone 복사 후 `.next/static` 삭제 뒤 static 단일 복사 |

## 검증

- 컨테이너: `/app/.next/static/static/chunks` **없음** (`NO_NESTED_OK`), `chunks/*.js` 54개.
- 포트: `3100:3100` (nginx `proxy_pass` 127.0.0.1:3100 과 일치).
- 프로덕션: `https://aads.newtalk.kr/login` HTML에서 추출한 `/_next/static/chunks/c0645d6fea057384.js` 요청 **HTTP 200**.

## 적용·배포 상태

- **적용**: Dockerfile 수정 반영, 이미지 `aads-server-aads-dashboard` 재빌드(캐시 활용 구간 포함, 최종 export 성공).
- **배포**: `docker compose -f docker-compose.prod.yml up -d aads-dashboard` 로 컨테이너 재기동 완료.

## 후속 체크

- [ ] 브라우저에서 `/chat` 강력 새로고침(캐시 비우기) 후 청크 오류 재현 여부 확인.
- [ ] 향후 배포 시 `docker compose build --no-cache aads-dashboard` 로 이상 레이어 방지(필요 시).

## 참고

- 이전 빌드 해시의 청크 URL(예: `46fc618ffbc0c079.js`)은 신규 빌드 후 **404가 정상**일 수 있음. 반드시 최신 HTML이 로드되는지 확인.
