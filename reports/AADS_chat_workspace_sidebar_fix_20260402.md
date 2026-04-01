# AADS 채팅 — 프로젝트·세션 목록 미표시 수정 (2026-04-02 KST)

## 원인 요약

1. **토큰 불일치**: 미들웨어는 `aads_token` **쿠키**로 `/chat` 통과를 허용하지만, `chatApi`는 **localStorage**만 사용. LS가 비면 API 호출에 `Authorization`이 없어 목록이 비거나 실패.
2. **세션 제목 null**: 백엔드 `SessionOut.title`이 Optional인데, 필터에서 `s.title.toLowerCase()` 호출 시 런타임 예외로 목록 렌더가 깨질 수 있음.
3. **태블릿/모바일 UX**: `mobileOverlay` 기본이 닫혀 있어 사이드바(프로젝트·세션)가 화면 밖에 있음. 헤더 ☰ 없이는 목록이 “없는 것처럼” 보일 수 있음.
4. **빈 목록 메시지**: API 실패 후에도 “워크스페이스 로딩 중...”만 표시됨.

## 변경 (aads-dashboard)

- `src/app/chat/api.ts`: `getToken()` — cookie 폴백 후 LS 동기화 (kakaobot 페이지와 동일 패턴).
- `src/app/chat/page.tsx`: `workspaceListStatus`, `filteredSessions`의 안전한 `title`, 비데스크톱 최초 1회 사이드바 오버레이 자동 오픈.
- `src/app/chat/ChatSidebar.tsx`: 로딩/오류/빈 목록 구분 문구.

## 배포

- `docker compose -f docker-compose.prod.yml build aads-dashboard` 후 `up -d aads-dashboard`.

## 검증

- `npm run build` 통과.
