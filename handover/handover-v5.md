# AADS Handover v5.0
- 작성일: 2026-02-27 KST
- 작성: Claude Opus 4.6

## 현황
- Sprint 1~2: ✅ 완료
- Sprint 3: ✅ 완료
- Sprint 3.5: ✅ 완료
- Sprint 4: ✅ 완료 (10/10)

## 인프라
- DigitalOcean 1vCPU/2GB(16GB RAM)/160GB, 볼륨 50GB $5/mo
- Supabase PostgreSQL, Redis Docker(6380), FastAPI systemd(8001)
- Nginx 리버스 프록시 + certbot 준비
- CI/CD: GitHub Actions (test.yml, dashboard.yml, deploy.yml)

## AI 모델 (4-Tier) — 전부 ✅
| Tier | 모델 | 상태 |
|------|------|------|
| 1 | Claude Opus 4.6 | ✅ |
| 2 | Claude Sonnet 4.6 | ✅ |
| 3 | Gemini 2.5 Flash | ✅ |
| 4 | Claude Haiku 4.5 | ✅ |

## SaaS 기능 (Sprint 4)
- 인증: Supabase Auth (JWT)
- 멀티테넌트: Redis user_id 기반 프로젝트 분리
- 과금: Stripe 4개 플랜 (free/starter/pro/enterprise)
- 템플릿: 6개 (todo, weather, blog, chat, ecommerce, api-gateway)
- 보안: 레이트 리밋 + 보안 헤더 + CORS
- 모니터링: /metrics/system, /metrics/llm, /system 페이지
- 문서: README, 온보딩 가이드, API 레퍼런스
- 베타 테스트: 자동화 스크립트 + 결과 보고서

## 비용
- 잔액: ~$97/$500
- 누적 LLM: ~$1.5-2.0
- 인프라: $5/mo

## 다음 단계 (Sprint 5 후보)
- Stripe Price ID / Webhook Secret 실 설정
- 도메인 구매 + SSL 인증서 (certbot --nginx)
- AUTH_ENABLED=true 프로덕션 전환
- 실 사용자 베타 오픈
- 성능 최적화 + 로드 테스트
- 모바일 대응 (responsive)
