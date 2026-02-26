# AADS 프로젝트 인수인계서 v1.0

**작성일**: 2026-02-26 13:55 KST
**작성자**: Claude Opus 4.6 (대화창 #1)
**인계 대상**: 새 대화창 (Claude Opus 4.6)
**프로젝트**: AADS (AI Autonomous Development System)

---

## 1. 프로젝트 개요
AADS는 자연어 아이디어 입력만으로 소프트웨어를 자동 생성하는 AI 자율 개발 시스템.
7단계 파이프라인(Ideation→Planning→Design→Development→Testing→Deployment→Monitoring)을 LangGraph로 오케스트레이션, 7개 전문 에이전트가 수행.
비개발자(CEO)가 대시보드에서 전체 관리.

## 2. 인프라
- 서버: DigitalOcean Droplet centos-s-1vcpu-2gb-sgp1-01 (Singapore, 160GB, 96% 사용중)
- 포트: 8000(기존서비스-금지), 6379(기존Redis-금지), 6380(AADS Redis✅), 8001(FastAPI✅), 3000(Dashboard 예정)
- 저장소: aads-core(Private) https://github.com/moongoby-GO100/aads-core
- 문서: aads-docs(Public) https://github.com/moongoby-GO100/aads-docs
- Spaces: s3://newtalk1/aads-backups/
- DB: Supabase PostgreSQL (9 tables + 4 views + Realtime)
- 경로: /root/aads/aads-core (.venv 가상환경)

## 3. AI 모델 설정 (2026-02-26 최신)
| Tier | 모델 | API ID (alias!) | 비용 in/out per 1M | 용도 |
|------|------|----------------|-------------------|------|
| 1 | Claude Opus 4.6 | claude-opus-4-6 | $5/$25 | 기획·설계 |
| 2 | Claude Sonnet 4.6 | claude-sonnet-4-6 | $3/$15 | 코드생성·리뷰 |
| 3 | Gemini 2.5 Flash | gemini-2.5-flash | $0.15/$0.60 | 반복·대량 |
| 4 | Claude Haiku 4.5 | claude-haiku-4-5 | $1/$5 | 폴백·분류 |

중요: snapshot ID(-20260205 등)가 아닌 alias 사용 필수! SDK 0.84.0 이상 필요.
비용관리: 총 $500, 일일 $100, 80% 경고, 100% 중단. 누적 ~$0.07

## 4. Sprint 현황

### Sprint 1 (완료)
Task 1.0~1.9.1 전체 완료
- 서버 정리, GitHub 레포, 프로젝트 초기화, .env, SSL 재빌드
- Redis 6380, Supabase 9T+4V, 코어모듈 5개, .cursorrules, 관리스크립트 3종
- 연동: Redis✅ Supabase✅ Sonnet✅ Gemini✅ Opus✅ Haiku✅

### Sprint 2 (진행중)
Task 2.0~2.5 완료, Task 2.0.1 완료
- 모델 ID 수정, LangGraph(graph.py), 에이전트 7종, pipeline.py, FastAPI, 통합테스트
- Task 2.6 대기: Next.js 대시보드

### ★ 미해결 이슈 (최우선)
Opus 4.6, Sonnet 4.6 → 404 에러
원인: SDK 버전 오래됨 + snapshot ID 사용
해결: pip install --upgrade anthropic + .env에서 alias 사용
해결 지시서 아래 8장 참조

## 5. 파일 구조 (aads-core)
```
/root/aads/aads-core/
├── .env, .cursorrules, .git/hooks/pre-commit
├── core/ (state.py, model_router.py, circuit_breaker.py, llm_client.py, graph.py, pipeline.py)
├── agents/ (base_agent.py, planner/, designer/, developer/, qa/, devops/, ops/, cost/)
├── api/main.py (FastAPI :8001)
├── tests/ (test_core.py, test_llm_models.py, test_pipeline.py)
├── scripts/ (work_report.sh, commit_and_backup.sh, status.sh)
├── logs/, reports/
```

## 6. 문서 현황 (aads-docs) — 전체 최신
- README.md, reports/initial-planning.md, reports/cost-analysis.md
- reports/saas-expansion.md, reports/project-rules.md
- phase-reports/sprint-1-progress.md, sprint-2-progress.md
- architecture/system-architecture.md
- handover/handover-v1.md (본 문서)

## 7. API 엔드포인트 (FastAPI :8001)
GET / (서비스정보), GET /health, POST /projects, GET /projects,
GET /projects/{id}, POST /projects/{id}/run, POST /projects/{id}/run-all,
POST /projects/{id}/approve, POST /projects/{id}/reject, GET /models

## 8. 즉시 처리 작업 (인계 후 최우선)

### 작업1 — Opus/Sonnet 4.6 404 해결 (5분)
```bash
cd /root/aads/aads-core && source .venv/bin/activate
pip install --upgrade anthropic
sed -i 's|DEFAULT_TIER1_MODEL=claude-opus-4-6|DEFAULT_TIER1_MODEL=claude-opus-4-6|' .env
sed -i 's|DEFAULT_TIER2_MODEL=claude-sonnet-4-6|DEFAULT_TIER2_MODEL=claude-sonnet-4-6|' .env
sed -i 's|DEFAULT_TIER4_MODEL=claude-haiku-4-5|DEFAULT_TIER4_MODEL=claude-haiku-4-5|' .env
# model_router.py에서도 동일하게 alias로 변경
python tests/test_llm_models.py
git add -A && git commit -m "🔧 Fix: SDK + alias (404 해결)" && git push origin main
```

### 작업2 — Sprint 2 Task 2.6 대시보드 (Next.js)

### 작업3 — 디스크 96% 추가 정리 또는 Volume 추가

## 9. 필수 작업 규칙
- 대화 토큰 관리: 80%에서 인계서 작성
- 커서로만 작업, 중요사항만 승인, 나머지 자체승인
- 커서 병렬작업 활용 (대화창 여러개)
- 커서필수규칙: 서버/DB접속, 백업, 보고서, GitHub동기화, 커밋, 배포 — 모든 지시서 반영
- 지시서는 전체를 코드블록으로 감싸기
- 보고서 저장시 GitHub 문서폴더 위치 명확 지정
- 한국시간 동기화, 지시서에 현재시간 반영
- 중요소스 검수 필수

## 10. 커서 필수 규칙 상세
- 서버: /root/aads/aads-core, source .venv/bin/activate
- 포트금지: 8000, 6379
- 커밋형식: `{이모지} {Sprint} {Task}: {설명}`
- 이모지: 🏗️인프라 🧠오케스트레이션 🤖에이전트 🔧수정 📊대시보드 🧪테스트 📝문서 🚀배포 💾백업 ⚡성능
- 백업: Git push(매작업), aads-docs(주요변경), Spaces(주1회), .env(암호화)
- .env: 절대 Git 포함 금지
- 보고서: scripts/work_report.sh, Supabase work_logs
- 비용: Tier3우선→Tier2→Tier1, 80%경고 100%중단

## 11. 새 대화창 시작 메시지 (복사해서 사용)
```
AADS 프로젝트 인수인계 받습니다.
인계서 확인: https://raw.githubusercontent.com/moongoby-GO100/aads-docs/main/handover/handover-v1.md
아키텍처: https://raw.githubusercontent.com/moongoby-GO100/aads-docs/main/architecture/system-architecture.md
Sprint 2 진행: https://raw.githubusercontent.com/moongoby-GO100/aads-docs/main/phase-reports/sprint-2-progress.md
프로젝트 규칙: https://raw.githubusercontent.com/moongoby-GO100/aads-docs/main/reports/project-rules.md

위 문서를 모두 읽고, 미해결 이슈(Opus/Sonnet 4.6 404)부터 처리해주세요.
작업규칙 9개 항목 반드시 준수.
```
