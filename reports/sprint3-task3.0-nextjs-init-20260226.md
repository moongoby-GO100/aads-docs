# Sprint 3 Task 3.0 — Next.js 대시보드 초기화

**일시**: 2026-02-26 20:20 KST (상태 확인·보고서 동기화)

## 상태 요약

| 항목 | 상태 |
|------|------|
| Next.js 초기화 | ✅ 완료 |
| dashboard/package.json | ✅ 존재 |
| src/lib/supabase.ts, api.ts | ✅ 존재 |
| docker-compose.yml / Dockerfile | ✅ 존재 (Task 3.9) |
| .gitignore dashboard 항목 | ✅ 반영 |
| npm run build | ⚠️ Node 20 필요 (현재 18.19.1) |

## 구성

| 항목 | 내용 |
|------|------|
| 경로 | /root/aads/aads-core/dashboard |
| 프레임워크 | Next.js 16 (App Router, TypeScript) |
| UI | Tailwind CSS 4 + Lucide React |
| 차트 | Recharts |
| DB | Supabase Realtime (@supabase/supabase-js, @supabase/ssr) |
| API | FastAPI :8001 |
| 포트 | 3000 |

## 생성·확인된 파일

- `src/lib/supabase.ts` — Supabase 클라이언트 (Realtime eventsPerSecond: 10)
- `src/lib/api.ts` — 10개 엔드포인트 래퍼 (health, projects, run, approve, reject, models 등)
- `.env.local` — NEXT_PUBLIC_SUPABASE_URL/ANON_KEY, NEXT_PUBLIC_API_URL (수동 설정 필요)

## 빌드

- `npm run build`: **Node.js >= 20.9.0 필요**. 현재 18.19.1 → 서버/CI에서 Node 20 업그레이드 후 실행.
- env 미설정 시 런타임 오류 가능 — `.env.local`에 Supabase/API URL 설정 후 빌드 권장.

## 다음

- Task 3.1~3.5: Realtime + UI 페이지 구현
- 환경: Node 20 설치 후 `cd dashboard && npm run build` 로 빌드 검증
