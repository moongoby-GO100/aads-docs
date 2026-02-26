# AADS 시스템 아키텍처 v2.0
**최종 수정**: 2026-02-26

## 5-Layer Architecture
- Layer 1: CEO Dashboard (Next.js + Supabase Realtime)
- Layer 2: API Gateway (FastAPI :8001)
- Layer 3: Orchestration (LangGraph + 7 Agents)
- Layer 4: Execution (Claude API + Gemini API + Docker)
- Layer 5: Infrastructure (Supabase + Redis + GitHub)

## 모델 라우팅 v2.0 (2026-02-26 확정)
| Tier | 모델 | API ID | Input/1M | Output/1M | 상태 | 비중 |
|------|------|--------|----------|-----------|------|------|
| 1 | Claude Opus 4.5 | claude-opus-4-5-20251101 | $5.00 | $25.00 | GA | 10-15% |
| 2 | Claude Sonnet 4 | claude-sonnet-4-20250514 | $3.00 | $15.00 | GA | 50-60% |
| 3 | Gemini 2.5 Flash | gemini-2.5-flash | $0.15 | $0.60 | Stable | 25-30% |
| 4 | Claude Haiku 4.5 | claude-haiku-4-5-20251001 | $1.00 | $5.00 | GA | 5-10% |

## Deprecated/Retired (사용 금지)
| 모델 | 상태 | 대체 |
|------|------|------|
| Claude 3.5 Haiku | ❌ Retired 2026-02 | → Haiku 4.5 |
| Claude 3.7 Sonnet | ❌ Retired 2026-02 | → Sonnet 4 |
| Claude 3 Opus | ❌ Retired 2026-01 | → Opus 4.5 |

## 참고: 최신 모델 (사용 가능, 현재 미적용)
| 모델 | API ID | 비고 |
|------|--------|------|
| Claude Opus 4.6 | claude-opus-4-6-20250205 | 2026-02-05 출시, MAX 플랜 |
| Claude Sonnet 4.6 | claude-sonnet-4-6-20250514 | 최신 Sonnet |
| Claude Sonnet 4.5 | claude-sonnet-4-5-20250929 | GA 안정 |

## 서버 포트
| 포트 | 서비스 | 상태 |
|------|--------|------|
| 8000 | 기존 Python 서비스 (건드리지 않음) | ✅ 운영중 |
| 6379 | 기존 Redis (건드리지 않음) | ✅ 운영중 |
| 6380 | AADS Redis | ✅ 운영중 |
| 8001 | AADS FastAPI | ✅ 테스트 완료 |
| 3000 | AADS Dashboard | ⏳ Sprint 3 |

## DB 스키마 (Supabase)
- 9 tables + 4 views + Realtime enabled

## 7단계 파이프라인
```
Ideation → Planning → Design → Development → Testing → Deployment → Monitoring
    ↑         ↓         ↓           ↓            ↓           ↓          ↓
    └────── Gate ──── Gate ────── Gate ──────── Gate ────── Gate ──── Loop ──→
```
