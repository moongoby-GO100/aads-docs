# Task 5.1: 대시보드 API URL 수정 (오프라인 해결)

- **일시**: 2026-02-27 KST
- **Sprint**: 5 — Task 5.1

## 원인

대시보드가 브라우저에서 `localhost:8001`로 API 호출 → 사용자 PC에서 서버 접근 불가 → "시스템 상태: ❌ 오프라인" 표시

## 변경

| 파일 | 내용 |
|------|------|
| dashboard/.env.production | `NEXT_PUBLIC_API_URL=https://aads.newtalk.kr/api` 추가 |
| dashboard/.env.local | `NEXT_PUBLIC_API_URL` 제거 (프로덕션 빌드 시 .env.production 적용되도록) |
| dashboard/src/lib/api.ts | 변경 없음 (이미 `process.env.NEXT_PUBLIC_API_URL` 사용 중) |

## 검증

| 항목 | 결과 |
|------|------|
| npm run build | ✓ Compiled successfully (Next.js 16.1.6, .env.local + .env.production 로드) |
| systemctl status aads-dashboard | active (running), next-server (v16.1.6) |
| https://aads.newtalk.kr/api/health | 200 `{"status":"healthy","time":"..."}` |
| 브라우저 /system | `https://aads.newtalk.kr/api/metrics/system` 호출 확인, CPU/메모리/디스크·Services(api: running, redis: running) 표시 |
| 시스템 상태 | API 호출이 도메인 기반으로 전환되어 ✅ 온라인 동작 |

## 비고

- `NEXT_PUBLIC_` 접두사 변수는 빌드 시 클라이언트 번들에 인라인됨.
- `.env.local`에 동일 키가 있으면 `.env.production`을 덮어쓰므로, 프로덕션 빌드 시 `.env.local`에서 `NEXT_PUBLIC_API_URL` 제거함.
- 개발 모드(`next dev`)에서는 api.ts 폴백 `http://localhost:8001` 유지.
