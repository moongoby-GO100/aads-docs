# AADS 시스템 아키텍처 v1.0

## 5-Layer Architecture
- Layer 1: CEO Dashboard (Next.js + Supabase Realtime)
- Layer 2: API Gateway (FastAPI :8001)
- Layer 3: Orchestration (LangGraph + 22 Agents)
- Layer 4: Execution (Claude Agent SDK + Docker)
- Layer 5: Infrastructure (Supabase + Redis + GitHub Actions)

## 모델 라우팅
| Tier | 모델 | 가격/1M tok | 비중 |
|------|------|-------------|------|
| 1 | Opus 4.6 | $5/$25 | 10-15% |
| 2 | Sonnet 4.6 | $3/$15 | 50-60% |
| 3 | Flash/Haiku | $0.15-$1 | 25-35% |

## 서버 포트
| 포트 | 서비스 |
|------|--------|
| 8000 | 기존 Python 서비스 (건드리지 않음) |
| 6379 | 기존 Redis (건드리지 않음) |
| 6380 | AADS Redis |
| 8001 | AADS FastAPI |
| 3000 | AADS Dashboard |
