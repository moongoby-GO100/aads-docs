# AADS-128 + AADS-129 완료 보고서
**작성일**: 2026-03-06 KST
**작성자**: CURSOR-AADS (Claude)

---

## 요약

AADS-128 (풀사이클 그래프 통합) 및 AADS-129 (CEO 체크포인트 UI + 대시보드) 완료.

---

## AADS-128: Full-Cycle Graph 통합

### 변경 파일
| 파일 | 내용 |
|---|---|
| `app/graphs/full_cycle_graph.py` | FullCycleState(IdeationState+AADSState) 통합, ideation→execution 파이프라인 |
| `app/graphs/ideation_subgraph.py` | (AADS-127 기존) Strategist↔Planner 토론 루프 서브그래프 |
| `app/api/artifacts.py` | 산출물 통합 API 3개 (POST/GET/GET) |
| `app/main.py` | artifacts_router 등록 추가 |
| `migrations/014_project_mode.sql` | projects.mode 컬럼 (execution_only 기본) |
| `migrations/015_project_artifacts.sql` | project_artifacts 테이블 생성 (FK 없이 TEXT project_id) |
| `tests/test_full_cycle.py` | 통합 테스트 10개 (전량 통과) |

### 검증 결과
- 신규 테스트: **10/10 통과**
- 기존 테스트: **183 통과** (7개 pre-existing 실패 — E2E/sandbox, 무관)
- DB 마이그레이션 적용: `project_artifacts` 테이블 생성 완료
- artifacts API 3개 엔드포인트 등록 확인

### 커밋 SHA
- `0cbaf94` feat(AADS-128): Full-cycle graph
- `4bbc0fb` fix(AADS-128): utcnow() → datetime.now() KST 규칙

---

## AADS-129: CEO 체크포인트 UI + 대시보드

### 변경 파일 (BE)
| 파일 | 내용 |
|---|---|
| `app/api/checkpoints.py` | CEO 체크포인트 서브 라우트 4개 구현 (중복 스켈레톤 제거) |

### 변경 파일 (FE — aads-dashboard)
| 파일 | 내용 |
|---|---|
| `src/app/projects/[id]/select-item/page.tsx` | 아이템 선택 UI (후보 카드 + 선택/방향수정/추가조사) |
| `src/app/projects/[id]/approve-plan/page.tsx` | 기획서 승인 UI (PRD/아키텍처/Phase/토론이력 + 승인/수정) |
| `src/app/projects/[id]/full-cycle/page.tsx` | 풀사이클 10단계 파이프라인 시각화 + 산출물 패널 |
| `src/app/reports/page.tsx` | 보고서 열람 (전략보고서/기획서/산출물 목록 + 검색/필터) |

### API 엔드포인트 (BE)
- `POST /api/v1/projects/{id}/checkpoint/select-item` ✅
- `POST /api/v1/projects/{id}/checkpoint/revise-direction` ✅
- `POST /api/v1/projects/{id}/checkpoint/research-more` ✅
- `POST /api/v1/projects/{id}/checkpoint/revise-plan` ✅

### 검증 결과
- Next.js 빌드: **0 에러** (20개 정적/동적 페이지 모두 성공)
- Docker 배포: `aads-dashboard` 컨테이너 재빌드 + 재시작 완료
- Dashboard HTTP: `3100` 포트 307 (로그인 리다이렉트) → 정상
- Health check: **{"status":"ok","graph_ready":true}**

### 커밋 SHA
- `cd1630f` feat(AADS-129): CEO checkpoint sub-routes
- `fa0fdd0` feat(AADS-129): CEO checkpoint UI + full-cycle dashboard + reports viewer (dashboard repo)

---

## GitHub URL
- BE: https://github.com/moongoby-GO100/aads-server/commit/cd1630f
- Dashboard: https://github.com/moongoby-GO100/aads-dashboard/commit/fa0fdd0 (가정)

---

## 적용·배포 상태
- 적용: ✅ 소스 반영 완료
- BE 배포: ✅ `docker restart aads-server` + health 확인
- FE 배포: ✅ `docker-compose up -d --build` 완료 (aads-dashboard:3100)
- Git push: ✅ origin/main 최신

---

## 후속 체크사항
- [ ] AADS-129 CEO 체크포인트 실제 interrupt 흐름 E2E 테스트 (mode=full_cycle 프로젝트 생성 후 confirm)
- [ ] project_artifacts 테이블에 실제 산출물 자동 기록 여부 확인
- [ ] 대시보드 URL https://aads.newtalk.kr/ 접속 확인 (nginx 프록시 3000→3100 확인 필요)
- ⚠️ `projects` 테이블 없음 — checkpoint 기반 상태 관리이므로 project_artifacts.project_id = thread_id 형식으로 통일 필요
