# AADS 초기 기획 보고서
**작성일**: 2026-02-26
**버전**: v1.0
**상태**: CEO 승인 완료

## 1. 비전
아이디어 입력만으로 AI가 소프트웨어의 전체 생명주기를 자동 수행

## 2. 3대 원칙
1. 최소 인간 개입 (승인/방향 지시만)
2. 실시간 가시성 (대시보드)
3. 비용·성능 최적화 (3-Tier 모델 라우팅)

## 3. 기술 스택
- Orchestration: LangGraph
- AI: Claude Opus 4.6 / Sonnet 4.6 / Gemini Flash
- Code Gen: Claude Agent SDK
- DB: Supabase PostgreSQL + Realtime
- State: Redis
- Dashboard: Next.js
- CI/CD: GitHub Actions

## 4. 비용 추정
- AADS 구축: $3,000-$5,000
- 프로젝트당 MVP: $650-$1,500
- 월 운영: $120-$250

## 5. 에이전트 구성 (22개)
- Orchestrator: 1 (Opus 4.6)
- Planning: 3 (Sonnet 4.6)
- Design: 4 (Opus + Sonnet)
- Development: 6 (Opus + Sonnet + Gemini)
- QA: 3 (Sonnet)
- DevOps: 2 (Sonnet)
- Operations: 3 (Haiku + Sonnet)
