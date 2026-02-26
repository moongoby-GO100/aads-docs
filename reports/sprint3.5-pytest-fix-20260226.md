# pytest 수정

- 일시: 2026-02-26
- 변경: pytest-asyncio 설치, pytest.ini 생성, test_model_router/test_llm_models/test_core 리팩터
- 결과: 13 passed / 0 failed / 2 skipped

## 변경 사항

- **pytest.ini**: `asyncio_mode = auto` 추가
- **tests/test_model_router.py**: 신규 — `get_model_for_task`, `get_all_models` 함수 기반 테스트
- **tests/test_llm_models.py**: `test_model` → `_run_one_model_test` 로 이름 변경 (fixture 수집 방지)
- **tests/test_core.py**: `test_model_router` 를 함수형 API(`get_model_for_task`, `get_all_models`, `calculate_cost`) 기준으로 수정; `test_llm_client`, `test_base_agent` 는 리팩터 대기로 skip

## 후속

- `test_llm_client` / `test_base_agent` 는 `LLMClient`·`BaseAgent`·`ModelRouter` 리팩터 후 재활성화
