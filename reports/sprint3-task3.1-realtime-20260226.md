# Sprint 3 Task 3.1 — Supabase Realtime 연동

**일시**: 2026-02-26 20:35 KST

## 생성 파일
| 파일 | 기능 |
|------|------|
| hooks/useRealtimeProjects.ts | 프로젝트 목록 실시간 구독 (INSERT/UPDATE/DELETE) |
| hooks/useRealtimePipeline.ts | 파이프라인 단계별 실시간 추적 |
| hooks/useRealtimeCosts.ts | 비용 로그 실시간 집계 |

## 구독 테이블
- projects (전체 CRUD 이벤트)
- pipeline_stages (프로젝트별 필터)
- cost_logs (INSERT만)

## 사용법
```tsx
const { projects, loading } = useRealtimeProjects()
const { stages } = useRealtimePipeline(projectId)
const { costs, totalCost } = useRealtimeCosts()
```
