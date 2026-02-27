# Task 5.1: 대시보드 API URL 수정 (오프라인 해결)

- **일시**: 2026-02-27 KST
- **Sprint**: 5 — Task 5.1

## 원인

1. 대시보드가 브라우저에서 `localhost:8001`로 API 호출 → 사용자 PC에서 서버 접근 불가 → "시스템 상태: ❌ 오프라인" 표시  
2. **(추가)** 일부 환경에서 `/api/health` 요청이 캐시된 HTML(다른 사이트)을 받아 `res.json()` 실패 → health=null → 오프라인 표시

## 변경

| 파일 | 내용 |
|------|------|
| dashboard/.env.production | `NEXT_PUBLIC_API_URL=https://aads.newtalk.kr/api` 추가 |
| dashboard/.env.local | `NEXT_PUBLIC_API_URL` 제거 (프로덕션 빌드 시 .env.production 적용되도록) |
| dashboard/src/lib/api.ts | `fetch` 옵션에 `cache: 'no-store'` 추가 (캐시로 인한 잘못된 응답 방지) |
| nginx aads.conf | `/api/` location에 `add_header Cache-Control "no-store, no-cache, must-revalidate"` 추가 |

## 검증

| 항목 | 결과 |
|------|------|
| npm run build | ✓ Compiled successfully (Next.js 16.1.6) |
| systemctl status aads-dashboard | active (running), next-server (v16.1.6) |
| https://aads.newtalk.kr/api/health | 200 `{"status":"healthy","time":"..."}` |
| 브라우저 직접 확인 | https://aads.newtalk.kr 접속 후 **시스템 상태: ✅ 정상** 표시 확인 (2026-02-27) |

## 비고

- `NEXT_PUBLIC_` 접두사 변수는 빌드 시 클라이언트 번들에 인라인됨.
- `.env.local`에 동일 키가 있으면 `.env.production`을 덮어쓰므로, 프로덕션 빌드 시 `.env.local`에서 `NEXT_PUBLIC_API_URL` 제거함.
- 개발 모드(`next dev`)에서는 api.ts 폴백 `http://localhost:8001` 유지.
- 루트(/) 접속 시 캐시 때문에 이전 앱이 보일 수 있음 → 쿼리 파라미터(예: `?v=1`)로 새로고침 권장.
