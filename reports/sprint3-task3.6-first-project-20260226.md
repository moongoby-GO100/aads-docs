# Task 3.6: 첫 자동 프로젝트 생성 테스트

- **일시:** 2026-02-26
- **아이디어:** 간단한 할일(Todo) 관리 웹앱 — 할일 추가/삭제/완료 체크, 카테고리 분류, 우선순위 설정
- **파이프라인:** 7단계 (Ideation → Planning → Design → Development → Testing → Deployment → Monitoring)

## 실행 조건 (필수)

- **API 생성 시:** `name` 필드 필수. `idea`만 보내면 422 Field required(name).
- **run-all 정상 동작:** API 서버를 **단일 워커**(`--workers 1`)로 기동해야 함. Dockerfile 기본값 `--workers 2`이면 프로젝트가 워커별 메모리에만 있어 run-all 시 "프로젝트 없음" 발생.
- **무승인 연속 실행:** `gate_mode: "auto"`로 생성. 기본값 `hybrid`면 ideation 완료 후 승인 대기에서 중단.

## 실행 명령 요약

```bash
cd /root/aads/aads-core && source .venv/bin/activate

# 단일 워커 재시작 (기존에 workers 2였을 경우)
# kill <uvicorn_pid>; nohup uvicorn api.main:app --host 0.0.0.0 --port 8001 --workers 1 >> /tmp/aads-api.log 2>&1 &

# 프로젝트 생성 (name + idea + gate_mode)
CREATED=$(curl -s -X POST http://localhost:8001/projects \
  -H "Content-Type: application/json" \
  -d '{"name": "Todo 웹앱", "idea": "간단한 할일(Todo) 관리 웹앱 - 할일 추가/삭제/완료 체크, 카테고리 분류, 우선순위 설정 기능", "gate_mode": "auto"}')
PROJECT_ID=$(echo "$CREATED" | python -c "import sys,json; print(json.load(sys.stdin)['project_id'])")

# 전체 파이프라인 실행
curl -s -X POST "http://localhost:8001/projects/${PROJECT_ID}/run-all"

# 결과 확인
curl -s "http://localhost:8001/projects/${PROJECT_ID}" | python -m json.tool
```

## 결과 (프로젝트 ID: 17262de7)

| 항목 | 내용 |
|------|------|
| **완료 단계** | Ideation, Planning, Design, Development, Testing, Deployment (6단계) |
| **중단 단계** | Monitoring (Ops) — 403 API key leaked |
| **총 비용** | 약 $0.7972 (예산 $500 대비 0.2%) |
| **소요 시간** | 약 7분 12초 |

### 단계별 에이전트·비용 요약

| 단계 | 에이전트 | 비용(USD) | 토큰(in/out) | 비고 |
|------|----------|-----------|--------------|------|
| ideation | N/A | 0 | 0/0 | 아이디어 요약만 |
| planning | Claude Opus 4.6 | 0.1352 | 369/5334 | 상세 기획서 |
| design | Claude Opus 4.6 | 0.2330 | 5634/8192 | 상세 설계서 |
| development | Claude Sonnet 4.6 | 0.1483 | 8480/8192 | 구현 코드 |
| testing | Claude Sonnet 4.6 | 0.1482 | 8426/8192 | 코드 리뷰·테스트 |
| deployment | Claude Sonnet 4.6 | 0.1325 | 3221/8192 | 배포 설정 |
| monitoring | (실패) | - | - | 403 Your API key was reported as leaked |

### 에이전트 출력 요약

- **Ideation:** 프로젝트 아이디어 문단 요약.
- **Planning:** 타깃 사용자, 핵심 가치, 기능 목록(우선순위 포함)이 포함된 상세 기획서.
- **Design:** 시스템 아키텍처, 데이터 모델, API 설계, 프론트 구조, 기술 스택이 포함된 설계서.
- **Development:** 백엔드/프론트 구조, Prisma/Node/React 기반 구현 설명 및 코드 스니펫.
- **Testing:** 코드 리뷰 및 테스트 관점(보안·JWT·bcrypt 등) 지적.
- **Deployment:** Dockerfile, nginx, docker-compose, Prometheus/Loki 등 배포·모니터링 구조.
- **Monitoring:** Ops 에이전트 호출 시 API 키 403으로 미수행.

## 결론 및 권장사항

- **파이프라인 동작:** Planner → Designer → Developer → QA → DevOps까지 자동 실행·비용 집계 정상.
- **운영 시:** API 서버는 `--workers 1` 사용하거나, 프로젝트 상태를 Redis 등으로 공유하도록 변경 권장.
- **테스트 시:** 생성 시 `name`, `gate_mode: "auto"` 사용.
- **Monitoring 403:** 사용 중인 API 키 유출로 판정된 경우이므로, 새 키 발급·교체 후 Ops 단계 재실행 필요.
