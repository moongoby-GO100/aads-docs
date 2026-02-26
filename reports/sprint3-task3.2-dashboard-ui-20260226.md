# Sprint 3 Task 3.2 — 대시보드 UI

**일시**: 2026-02-26 20:35 KST (완료)

## 생성 페이지

| 경로 | 기능 |
|------|------|
| /projects | 프로젝트 목록 (API 폴링 3초, 상태·단계 뱃지) |
| /projects/[id] | 프로젝트 상세 + 파이프라인 시각화 + 단계 테이블 |

## 기능

- API 연동: `useRealtimeProjects`(폴링 3초), `useRealtimePipeline`(폴링 2초) — Supabase 대신 인메모리 API 대응
- 7단계 파이프라인 가로 시각화 (이모지 + 색상 + 현재/완료/실행 중 애니메이션)
- 실행 / 전체실행 / 승인 / 거부 버튼 (API 연동)
- 단계별 토큰·비용·시간 테이블 (phases API 응답 기반)

## 변경 파일

| 위치 | 내용 |
|------|------|
| core/pipeline.py | list_projects: id, status, cost_total, created_at 추가; get_status: idea, created_at, phases 배열 추가 |
| dashboard/src/lib/api.ts | getProjects → res.projects 반환; approve POST body 추가 |
| dashboard/src/hooks/useRealtimeProjects.ts | Supabase → API 폴링 (3초) |
| dashboard/src/hooks/useRealtimePipeline.ts | Supabase → api.getProject(id).phases 폴링 (2초) |
| dashboard/src/app/projects/page.tsx | 프로젝트 목록 페이지 (신규) |
| dashboard/src/app/projects/[id]/page.tsx | 프로젝트 상세 + 파이프라인 뷰 (신규) |

## 검증

- Lint: 통과
- API: list_projects / get_status 응답 형식 대시보드 필드와 일치

## 적용·배포

- 적용: ✅ 소스 반영
- 배포: 로컬/스테이징에서 `npm run dev` 및 API 서버 기동 후 /projects, /projects/[id] 확인
