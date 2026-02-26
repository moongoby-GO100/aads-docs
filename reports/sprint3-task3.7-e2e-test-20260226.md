# Task 3.7: E2E 파이프라인 통합 테스트

- **일시**: 2026-02-26
- **테스트 항목**: 서비스 상태, 프로젝트 생성, 단계별 실행, 승인/거부, pytest 전체

## 1. 서비스 상태

| 항목 | 결과 |
|------|------|
| `GET /health` | ✅ `{"status":"healthy"}` |
| `GET /models` | ✅ tier1~4 모델 목록 정상 (claude-opus-4-6, sonnet-4-6, gemini-2.5-flash, haiku-4-5) |

## 2. 프로젝트 생성

- **작업**: `POST /projects` — body에 `name` 필드 필수 (태스크 스크립트에는 `idea`만 있어서 422 발생)
- **수정 후**: `{"name":"날씨 대시보드","idea":"실시간 날씨 대시보드 - ..."}` 로 생성 성공
- **결과**: `project_id: 72f09a63`, `status: created`, `current_phase: ideation`

## 3. 단계별 실행 (run)

- **요청**: `POST /projects/72f09a63/run`
- **결과**: ✅ `status: completed`, `phase_completed: ideation`, `next_phase: planning`, `waiting_approval: true`
- ideation 단계 스텁 에이전트 정상 동작

## 4. 승인 테스트

- **요청**: `POST /projects/72f09a63/approve` (body `{}` 또는 `{"approved_by":"CEO"}`)
- **결과**: ❌ `400 - "프로젝트 없음"` (동일 project_id로 `GET /projects/72f09a63` 는 정상 조회됨)
- **비고**: 동일 서버에서 GET은 성공·POST approve만 실패 — 다중 워커/인스턴스 가능성 또는 엔진 조회 경로 이슈로 추정

## 5. pytest 전체 스위트

- **실행**: `python -m pytest tests/ -v --tb=short` (pytest 사전 설치 필요)
- **요약**: **14 수집, 4 통과, 9 실패, 1 에러**

| 결과 | 테스트 |
|------|--------|
| **PASSED** | test_redis, test_supabase, test_state, test_circuit_breaker |
| **FAILED** | test_claude, test_gemini, test_openai, test_llm_client, test_base_agent, test_infra, test_claude_api, test_redis (sdk) — **async 테스트에 pytest-asyncio 미설정** |
| **FAILED** | test_model_router — **ImportError: cannot import name 'ModelRouter'** (core/model_router.py에 클래스 없음, get_all_models 등 함수만 존재) |
| **ERROR** | test_model — **fixture 'name' not found** (파라미터화 fixture 미정의) |

## 발견 이슈

1. **API 스키마**: `POST /projects` 필수 필드 `name` — 태스크 스크립트/문서에 반영 필요.
2. **목록 키**: `GET /projects` 응답은 `project_id` 사용. 스크립트에서 `projects[-1]['id']` 사용 시 KeyError — `project_id`로 통일 권장.
3. **승인**: 같은 project_id로 GET은 성공하나 approve만 "프로젝트 없음" — 원인 추적 필요 (프로세스/메모리 분리 여부).
4. **pytest**: `pytest-asyncio` 설치 및 `@pytest.mark.asyncio` 적용 시 async 테스트 실행 가능.
5. **테스트 코드**: `test_model_router`는 `ModelRouter` 클래스 대신 현재 API(get_model_for_task, get_all_models)에 맞게 수정 필요; `test_llm_models.py::test_model`은 `name` 등 파라미터화 fixture 추가 필요.

## 결과 요약

- **E2E(서비스·생성·run 단계)**: ✅ 정상
- **승인**: ❌ 재현 이슈
- **pytest**: ⚠️ 인프라/연결 테스트 일부 통과, async 및 model_router/llm_models 테스트 수정 필요
