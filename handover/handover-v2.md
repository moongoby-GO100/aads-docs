# AADS 프로젝트 인수인계서 v2.0

**작성일**: 2026-02-26 18:55 KST
**작성자**: Claude Opus 4.6
**프로젝트**: AADS (AI Autonomous Development System)

---

## 1. 프로젝트 개요
AADS는 자연어 아이디어 입력만으로 소프트웨어를 자동 생성하는 AI 자율 개발 시스템.
7단계 파이프라인(Ideation→Planning→Design→Development→Testing→Deployment→Monitoring)을 LangGraph로 오케스트레이션, 7개 전문 에이전트가 수행.
비개발자(CEO)가 대시보드에서 전체 관리.

## 2. 인프라
- 서버: DigitalOcean Droplet centos-s-1vcpu-2gb-sgp1-01 (Singapore, 16GB RAM, 160GB Disk)
- Volume: volume-sgp1-01 (50GB, ext4, /mnt/volume-sgp1-01) — $5/월
- 포트: 8000(기존서비스-금지), 6379(기존Redis-금지), 6380(AADS Redis✅), 8001(FastAPI✅), 3000(Dashboard 예정)
- 저장소: aads-core(Private) https://github.com/moongoby-GO100/aads-core
- 문서: aads-docs(Public) https://github.com/moongoby-GO100/aads-docs
- Spaces: newtalk1 (SGP1, aads-backups/), newtalk (NYC3, 기존서비스)
- DB: Supabase PostgreSQL (9 tables + 4 views + Realtime)
- 경로: /root/aads/aads-core (.venv 가상환경)

### 디스크 구성
| 위치 | 용도 | 크기 |
|------|------|------|
| / (메인) | OS + 앱 + 기존서비스 | 160GB |
| /mnt/volume-sgp1-01 | AADS 로그, 백업, webapp 백업 | 50GB |
| s3://newtalk1 | AADS 이중 백업 + 서버 백업 | 무제한 |

### 심볼릭 링크
| 원본 경로 | → Volume 경로 |
|-----------|---------------|
| /root/aads/aads-core/logs | /mnt/volume-sgp1-01/aads-logs |
| /root/webapp/backup/daily | /mnt/volume-sgp1-01/webapp-backups |

## 3. AI 모델 설정 (2026-02-26 확정, 404 해결 완료)
| Tier | 모델 | API ID (alias) | 비용 in/out per 1M | 용도 |
|------|------|----------------|-------------------|------|
| 1 | Claude Opus 4.6 | claude-opus-4-6 | $5/$25 | 기획·설계 |
| 2 | Claude Sonnet 4.6 | claude-sonnet-4-6 | $3/$15 | 코드생성·리뷰 |
| 3 | Gemini 2.5 Flash | gemini-2.5-flash | $0.15/$0.60 | 반복·대량 |
| 4 | Claude Haiku 4.5 | claude-haiku-4-5 | $1/$5 | 폴백·분류 |

- Anthropic 계정: Tier 2, 잔액 $98.95, 월한도 $500
- SDK: anthropic 0.84.0+
- alias 사용 필수 (snapshot ID 사용 금지)
- 비용관리: 총 $500, 일일 $100, 80% 경고, 100% 중단

## 4. Sprint 현황

### Sprint 1 (완료)
Task 1.0~1.9.1 전체 완료

### Sprint 2 (진행중)
| Task | 내용 | 상태 |
|------|------|------|
| 2.0~2.0.6 | 모델 ID + 404 해결 | ✅ 완료 |
| 2.1 | LangGraph 오케스트레이션 | ✅ 완료 |
| 2.2 | 에이전트 7종 | ✅ 완료 |
| 2.3 | 파이프라인 엔진 | ✅ 완료 |
| 2.4 | FastAPI 엔드포인트 | ✅ 완료 |
| 2.5 | 통합 테스트 | ✅ 완료 |
| 2.6 | Next.js 대시보드 | ⏳ 대기 |
| 2.7 | 디스크 정리 | ✅ 완료 |
| 2.7.1 | Spaces 이중 백업 | 🔄 진행중 |
| 2.7.2 | Volume 50GB 연결 | 🔄 진행중 |

### 미해결/참고
- test_pipeline.py에서 max_tokens 400 이슈 (별건, 후속 처리)

## 5. 파일 구조 (aads-core)

```
/root/aads/aads-core/
├── .env, .cursorrules, .git/hooks/pre-commit
├── core/ (state.py, model_router.py, circuit_breaker.py, llm_client.py, graph.py, pipeline.py)
├── agents/ (base_agent.py, planner/, designer/, developer/, qa/, devops/, ops/, cost/)
├── api/main.py (FastAPI :8001)
├── tests/ (test_core.py, test_llm_models.py, test_pipeline.py)
├── scripts/ (work_report.sh, commit_and_backup.sh, status.sh)
├── logs -> /mnt/volume-sgp1-01/aads-logs (심볼릭 링크)
├── reports/
```

## 6. API 엔드포인트 (FastAPI :8001)
GET /, GET /health, POST /projects, GET /projects,
GET /projects/{id}, POST /projects/{id}/run, POST /projects/{id}/run-all,
POST /projects/{id}/approve, POST /projects/{id}/reject, GET /models

## 7. 필수 작업 규칙
0. 대화 토큰 관리: 80%에서 인계서 작성
1. 커서로만 작업, 중요사항만 승인, 나머지 자체승인
2. 커서 병렬작업 활용 (대화창 여러개)
3. 커서필수규칙: 서버/DB접속, 백업, 보고서, GitHub동기화, 커밋, 배포 — 모든 지시서 반영
4. 지시서는 전체를 코드블록으로 감싸기
5. 보고서 저장시 GitHub 문서폴더 위치 명확 지정
6. 한국시간 동기화, 지시서에 현재시간 반영
7. 중요소스 검수 필수

## 8. 커서 필수 규칙 상세
- 서버: /root/aads/aads-core, source .venv/bin/activate
- 포트금지: 8000, 6379
- 커밋형식: `{이모지} {Sprint} {Task}: {설명}`
- 백업: Git push(매작업), aads-docs(주요변경), Spaces(주1회), .env(암호화)
- .env: 절대 Git 포함 금지
- 비용: Tier3우선→Tier2→Tier1, 80%경고 100%중단

## 9. 다음 작업
1. Task 2.6: Next.js 대시보드 (Sprint 2 마지막)
2. max_tokens 이슈 수정
3. Sprint 3 계획
