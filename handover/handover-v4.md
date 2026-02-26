# AADS Handover v4.0
- 작성일: 2026-02-26 21:50 KST
- 작성: Claude Opus 4.6

## 프로젝트 현황
Sprint 1: 완료 (기초 설계 + 에이전트 구조)
Sprint 2: 완료 (모델 통합 + 인프라 안정화)
Sprint 3: 진행 중 (CEO 대시보드 + 자동 프로젝트 생성)

## 인프라
- 서버: DigitalOcean centos-s-1vcpu-2gb-sgp1-01 (16GB RAM, 160GB)
- 볼륨: 50GB ext4 /mnt/volume_sgp1_01 ($5/mo)
- Spaces: newtalk1 (SGP1) 무제한 백업
- DB: Supabase PostgreSQL
- 서비스: FastAPI(8001) systemd, Redis(6380) Docker, healthcheck cron 5분

## AI 모델 (4-Tier)
| Tier | 모델 | alias | 비용(1M tokens) | 상태 |
|------|------|-------|-----------------|------|
| 1 | Claude Opus 4.6 | claude-opus-4-6 | $5/$25 | ✅ |
| 2 | Claude Sonnet 4.6 | claude-sonnet-4-6 | $3/$15 | ✅ |
| 3 | Gemini 2.5 Flash | gemini-2-5-flash | $0.15/$0.60 | ❌ 키 차단 |
| 4 | Claude Haiku 4.5 | claude-haiku-4-5 | $1/$5 | ✅ |

## 대시보드 (Sprint 3 신규)
- 프레임워크: Next.js 16, TypeScript, Tailwind 4
- 페이지: /, /projects, /projects/[id], /costs, /models, /logs
- Node: v20.20.0 (nvm), 빌드 성공
- Docker: docker-compose.yml (redis + api + dashboard)

## 비용 현황
- Anthropic 잔액: $98.95 / 월 $500
- 누적 LLM 비용: ~$0.11
- 인프라: $5/mo (볼륨)
- Task 3.6-3.7 예상: $15-45

## 보안
- .env: 암호화 + Spaces 백업, 권한 600
- PostgreSQL: localhost only
- pre-commit hook 활성

## 미해결
1. Gemini API 키 재발급 필요
2. GitHub PAT workflow 스코프 → 적용 완료 (2026-02-26)
3. Task 3.6-3.7 실행 중

## 다음 단계
- Sprint 3 완료: 3.6 자동 프로젝트 + 3.7 E2E
- Sprint 4 계획: SaaS 멀티테넌트, 사용자 인증, 과금
