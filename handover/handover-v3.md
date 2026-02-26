# AADS 프로젝트 인수인계서 v3.0

**작성일**: 2026-02-26 20:20 KST
**작성자**: Claude Opus 4.6
**프로젝트**: AADS (AI Autonomous Development System)

---

## 1. 프로젝트 개요
AADS는 자연어 아이디어 입력만으로 소프트웨어를 자동 생성하는 AI 자율 개발 시스템.
7단계 파이프라인을 LangGraph로 오케스트레이션, 7개 전문 에이전트가 수행.

## 2. 인프라

### 서버
| 항목 | 내용 |
|------|------|
| Droplet | centos-s-1vcpu-2gb-sgp1-01 (SGP1, 16GB RAM, 160GB) |
| Volume | volume-sgp1-01 (50GB, ext4, /mnt/volume_sgp1_01) $5/월 |
| 메인 디스크 | 82% 사용 |
| Volume | 28% 사용 |

### 포트
| 포트 | 용도 | 상태 |
|------|------|------|
| 8000 | 기존 서비스 | ⛔ 접근 금지 |
| 6379 | 기존 Redis | ⛔ 접근 금지 |
| 6380 | AADS Redis | ✅ 사용중 (Docker) |
| 8001 | AADS FastAPI | ✅ systemd (aads-api) |
| 3000 | AADS Dashboard | 🔧 Sprint 3 구현중 |
| 5432 | PostgreSQL | ✅ localhost only (보안 제한 완료) |

### 저장소
| 레포 | URL | 비고 |
|------|-----|------|
| aads-core | https://github.com/moongoby-GO100/aads-core | Private |
| aads-docs | https://github.com/moongoby-GO100/aads-docs | Public |
| Spaces | s3://newtalk1/aads-backups/ | 이중 백업 |

### 심볼릭 링크
| 원본 | → Volume |
|------|---------|
| /root/aads/aads-core/logs | /mnt/volume_sgp1_01/aads-logs |
| /root/webapp/backup/daily | /mnt/volume_sgp1_01/webapp-backups |

### 서비스 (systemd)
| 서비스 | 포트 | 상태 |
|--------|------|------|
| aads-api | 8001 | enabled, active |
| aads-redis | 6380 | Docker 운영 |

### 모니터링
- healthcheck.sh: cron 5분 간격, FastAPI/Redis/디스크/메모리
- server-status.sh: 수동 상태 대시보드
- 자동복구: FastAPI·Redis 다운 시 systemctl restart

## 3. AI 모델 설정

| Tier | 모델 | API ID (alias) | 비용 in/out | 상태 |
|------|------|----------------|------------|------|
| 1 | Claude Opus 4.6 | claude-opus-4-6 | $5/$25 | ✅ 정상 |
| 2 | Claude Sonnet 4.6 | claude-sonnet-4-6 | $3/$15 | ✅ 정상 |
| 3 | Gemini 2.5 Flash | gemini-2.5-flash | $0.15/$0.60 | ❌ 키 차단 (재발급 필요) |
| 4 | Claude Haiku 4.5 | claude-haiku-4-5 | $1/$5 | ✅ 정상 |

- Anthropic: Tier 2, 잔액 $98.95
- SDK: anthropic 0.84.0
- **Gemini**: GOOGLE_AI_API_KEY 유출 차단 → https://aistudio.google.com/apikey 에서 재발급 필요
- alias 사용 필수, snapshot ID 사용 금지

## 4. Sprint 현황

### Sprint 1 ✅ 완료
Tasks 1.0~1.9.1 전체 완료

### Sprint 2 ✅ 완료
| Task | 내용 | 비용 |
|------|------|------|
| 2.0~2.0.6 | 모델 ID + 404 해결 | ~$0.06 |
| 2.1~2.5 | LangGraph + 에이전트 + 파이프라인 + FastAPI + 테스트 | ~$0.05 |
| 2.5.1 | max_tokens Tier별 수정 | $0 |
| 2.7~2.7.2 | 디스크 정리 + Spaces + Volume | $0 |
| 2.8 | systemd 서비스 등록 | $0 |
| 2.9 | 헬스체크 + 모니터링 | $0 |
| 2.10 | .env 암호화 + 보안 | $0 |
| 총 비용 | | ~$0.11 |

### Sprint 3 🔄 진행중
| Task | 내용 | 상태 |
|------|------|------|
| 3.0 | Next.js 대시보드 초기화 | 🔄 확인중 |
| 3.1 | Supabase Realtime 연동 | ⏳ 대기 (3.0 의존) |
| 3.2 | 대시보드 UI | ⏳ 대기 |
| 3.3 | 게이트 승인/거부 UI | ⏳ 대기 |
| 3.4 | 비용 모니터링 차트 | ⏳ 대기 |
| 3.5 | 모델 상태/사용량 뷰 | ⏳ 대기 |
| 3.6 | 첫 자동 프로젝트 생성 | ⏳ 대기 |
| 3.7 | E2E 테스트 | ⏳ 대기 |
| 3.8 | CI/CD GitHub Actions | ✅ 생성 (PAT workflow 스코프 필요) |
| 3.9 | Docker Compose | 🔄 확인중 |
| 보안 | Git .env 정리 + PG 바인드 | ✅ 완료 |
| 보완 | Gemini 403 진단 | ✅ (키 재발급 필요) |

## 5. 파일 구조 (aads-core)

```
/root/aads/aads-core/
├── .env, .cursorrules, .gitignore
├── .github/workflows/ (test.yml, deploy.yml, dashboard.yml)
├── core/ (state, model_router, circuit_breaker, llm_client, graph, pipeline)
├── agents/ (base, planner, designer, developer, qa, devops, ops, cost)
├── api/main.py (FastAPI :8001)
├── dashboard/ (Next.js — Sprint 3)
├── tests/ (test_core, test_llm_models, test_pipeline)
├── scripts/ (work_report, commit_and_backup, status, healthcheck, server-status)
├── logs -> /mnt/volume_sgp1_01/aads-logs
├── reports/
├── Dockerfile, Dockerfile.dashboard, docker-compose.yml
├── requirements.txt
```

## 6. API 엔드포인트 (FastAPI :8001)
GET /, GET /health, POST /projects, GET /projects,
GET /projects/{id}, POST /projects/{id}/run, POST /projects/{id}/run-all,
POST /projects/{id}/approve, POST /projects/{id}/reject, GET /models

## 7. 필수 작업 규칙
0. 대화 토큰 80%에서 인계서 작성
1. 커서로만 작업, 중요사항만 승인, 나머지 자체승인
2. 커서 병렬작업 (대화창 최대 5개)
3. 커서필수규칙: 서버접속, 백업, 보고서, GitHub동기화, 커밋, 배포 반영
4. 지시서 전체 코드블록
5. 보고서 GitHub 문서폴더 위치 지정
6. 한국시간 동기화
7. 중요소스 검수 필수

## 8. 즉시 조치 필요
1. **Gemini API 키 재발급**: https://aistudio.google.com/apikey → .env 교체
2. **GitHub PAT workflow 스코프**: Settings → Tokens → workflow 활성화
3. **dashboard .env.local**: Supabase URL/KEY 설정

## 9. 비용 현황
- Anthropic 잔액: $98.95
- 누적 LLM 비용: ~$3.41
- 프로젝트 예산: $500 (일일 $100)
- 인프라: Volume $5/월
- 예산 소진율: 0.7%

## 10. 새 대화창 시작 메시지
AADS 프로젝트 인수인계 받습니다. 인계서: https://raw.githubusercontent.com/moongoby-GO100/aads-docs/main/handover/handover-v3.md 아키텍처: https://raw.githubusercontent.com/moongoby-GO100/aads-docs/main/architecture/system-architecture.md Sprint 2: https://raw.githubusercontent.com/moongoby-GO100/aads-docs/main/phase-reports/sprint-2-progress.md Sprint 3: https://raw.githubusercontent.com/moongoby-GO100/aads-docs/main/phase-reports/sprint-3-plan.md 프로젝트 규칙: https://raw.githubusercontent.com/moongoby-GO100/aads-docs/main/reports/project-rules.md

위 문서 모두 읽고, Sprint 3 대시보드 작업부터 진행해주세요. 작업규칙 반드시 준수.
