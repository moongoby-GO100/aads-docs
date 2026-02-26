# AADS 시스템 아키텍처 v1.1
**최종 수정**: 2026-02-26

## 5-Layer Architecture
- Layer 1: CEO Dashboard (Next.js + Supabase Realtime)
- Layer 2: API Gateway (FastAPI :8001)
- Layer 3: Orchestration (LangGraph + 22 Agents)
- Layer 4: Execution (Claude Agent SDK + Docker)
- Layer 5: Infrastructure (Supabase + Redis + GitHub Actions)

## 모델 라우팅 (확정)
| Tier | 모델 | API ID | Input/1M | Output/1M | 비중 |
|------|------|--------|----------|-----------|------|
| 1 | Claude Opus 4.1 | claude-opus-4-1-20250805 | $15.00 | $75.00 | 10-15% |
| 2 | Claude Sonnet 4 | claude-sonnet-4-20250514 | $3.00 | $15.00 | 50-60% |
| 3 | Gemini 2.5 Flash | gemini-2.5-flash | $0.15 | $0.60 | 25-30% |
| 4 | Claude Haiku 3.5 | claude-3-5-haiku-20241022 | $0.80 | $4.00 | 5-10% |

## 서버 포트
| 포트 | 서비스 | 상태 |
|------|--------|------|
| 8000 | 기존 Python 서비스 (건드리지 않음) | ✅ 운영중 |
| 6379 | 기존 Redis (건드리지 않음) | ✅ 운영중 |
| 6380 | AADS Redis | ✅ 운영중 |
| 8001 | AADS FastAPI | ⏳ Sprint 2 |
| 3000 | AADS Dashboard | ⏳ Sprint 2 |

## DB 스키마 (Supabase)
- 9 tables: projects, pipeline_phases, agent_sessions, tasks, artifacts, cost_tracking, circuit_breakers, work_logs, daily_summary
- 4 views: v_sprint_efficiency, v_agent_efficiency, v_daily_cost_trend, v_project_overview
- Realtime enabled

## 7단계 파이프라인
```
Ideation → Planning → Design → Development → Testing → Deployment → Monitoring
    ↑         ↓         ↓           ↓            ↓           ↓          ↓
    └────── Gate ──── Gate ────── Gate ──────── Gate ────── Gate ──── Loop ──→
```
