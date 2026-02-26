# Sprint 3 Task 3.0 — Next.js 대시보드 초기화

**일시**: 2026-02-26 20:xx KST

## 구성

| 항목 | 내용 |
|------|------|
| 경로 | /root/aads/aads-core/dashboard |
| 프레임워크 | Next.js 16 (App Router, TypeScript) |
| UI | Tailwind CSS 4 + Lucide 아이콘 |
| 차트 | Recharts |
| DB | Supabase (Realtime) |
| API | FastAPI :8001 연동 |
| 포트 | 3000 (기본) |
| Node | 20.18.0 (nvm, glibc-217 빌드) |

## 생성·수정 파일

- `dashboard/.env.local` — 환경 변수 (Supabase URL/KEY는 메인 .env에서 복사 필요)
- `dashboard/.nvmrc` — Node 20 고정
- `dashboard/src/lib/supabase.ts` — Supabase 클라이언트
- `dashboard/src/lib/api.ts` — FastAPI 래퍼 (health, projects, run, approve, reject, models 등)
- `dashboard/src/app/layout.tsx` — 사이드바 레이아웃 (대시보드/프로젝트/모델/비용/로그)
- `dashboard/src/app/page.tsx` — 메인 대시보드 (상태 카드 + 프로젝트 테이블)

## 빌드 결과

- `npm run build`: ✅ 성공 (Next.js 16.1.6, Turbopack, 정적 페이지 4개)

## 환경 참고

- CentOS 7 glibc 2.17 이슈로 Node 18/20은 **unofficial-builds (x64-glibc-217)** 사용
- nvm 설치 후: `nvm_get_arch() { echo x64-glibc-217; }` 후 `nvm install 20.18.0`

## 다음 작업

- Task 3.1: Supabase Realtime 연동
- Task 3.2: 프로젝트 상세/파이프라인 뷰
- Task 3.3: 게이트 승인/거부 UI
