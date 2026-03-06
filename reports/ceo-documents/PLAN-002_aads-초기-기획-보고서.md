# AADS 초기 기획 보고서
**작성일**: 2026-02-26
**작성자**: Claude (AI Architect) + CEO

## 비전
자연어 아이디어 입력 → AI가 기획 → 설계 → 개발 → 테스트 → 배포 → 운영 → 고도화까지 전체 소프트웨어 라이프사이클을 자율 수행하는 시스템

## 핵심 원칙
1. **완전 자동화**: 비개발자가 자연어로 지시하면 끝까지 완성
2. **비용 가시성**: 모든 LLM 호출의 비용을 실시간 추적·제어
3. **품질 게이트**: 각 단계 완료 시 자동/수동 승인 게이트
4. **자기 치유**: 장애 발생 시 서킷브레이커 + 자동 재시도
5. **다중 에이전트**: 22개 전문 에이전트가 역할 분담

## 기술 스택
- **오케스트레이션**: LangGraph (상태 머신 기반 파이프라인)
- **LLM**: Claude Opus 4.1 / Sonnet 4 / Gemini 2.5 Flash / Haiku 3.5 (3+1 Tier)
- **백엔드**: FastAPI (Python 3.11)
- **DB**: Supabase PostgreSQL (9 tables, 4 views)
- **캐시/상태**: Redis 6380
- **대시보드**: Next.js + Supabase Realtime
- **배포**: Docker Compose → 추후 CI/CD
- **코드 관리**: GitHub (aads-core Private, aads-docs Public)

## 예산 추정
- **개발 비용**: $1,310~$2,360 (전통 대비 40~125배 절감)
- **월 운영 비용**: $120~$250 (100 사용자 기준)
- **LLM API**: 3-Tier 라우팅으로 60~70% 비용 절감
- **프로젝트 예산**: $500 (기본), 일일 한도 $100

## 7단계 파이프라인
1. **Ideation** → 2. **Planning** → 3. **Design** → 4. **Development** → 5. **Testing** → 6. **Deployment** → 7. **Monitoring**
- 각 단계 사이에 승인 게이트 (자동/수동 선택)

## 마일스톤
| 기간 | 목표 |
|------|------|
| Week 1-2 | 인프라 + 코어 모듈 |
| Week 3-4 | 에이전트 7종 + LangGraph 파이프라인 |
| Week 5-6 | FastAPI + 대시보드 + CI/CD |
| Week 7-8 | 운영 자동화 + 통합 테스트 + 첫 프로젝트 자동 생성 |
