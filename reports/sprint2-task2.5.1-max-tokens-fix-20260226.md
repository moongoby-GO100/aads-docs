# Sprint 2 Task 2.5.1 — max_tokens 수정

**일시**: 2026-02-26 19:30 KST
**작업**: max_tokens Tier별 적정값 변경 (기존 4096/2048/1024 → 8192/4096/2048)

## 변경 내역

| 대상 | 이전 | 이후 | 사유 |
|------|------|------|------|
| core/model_router.py tier1 (Opus) | 4096 | 8192 | 기획/설계 긴 출력 |
| core/model_router.py tier2 (Sonnet) | 4096 | 8192 | 코드 생성 |
| core/model_router.py tier3 (Gemini) | 2048 | 4096 | 반복/모니터링 |
| core/model_router.py tier4 (Haiku) | 1024 | 2048 | 분류/폴백 |
| core/llm_client.py | 4096 | 8192 | 기본값 |
| agents/base_agent.py | 4096 | 8192 | execute 기본값 |
| Planner, Designer, Developer, QA, DevOps, Ops, Cost | model["max_tokens"] | (model_router 반영) | 에이전트는 라우터 값 사용 |

**참고**: 코드베이스에 `max_tokens=400`은 없었으며, 기존 값(4096/2048/1024)을 Tier별 목표값으로 수정함. pipeline.py는 에이전트를 호출만 하므로 별도 수정 없음.

## 수정 파일 (3건)

- `core/model_router.py` — MODEL_CONFIG tier1~4 max_tokens
- `core/llm_client.py` — call_anthropic, call_google, call_llm 기본 인자
- `agents/base_agent.py` — execute() max_tokens 기본값

## 테스트 결과

- **test_llm_models.py**: 5/6 통과 (Tier3 Gemini 403 API key 이슈 — max_tokens와 무관)
- **test_pipeline.py**: 실행 확인 (파이프라인 엔진은 model_router/에이전트 값 사용)

## 적용·배포

- 적용: ✅ 소스 반영
- 배포: aads-core Git push 완료 시 반영 (API/서비스 재시작 불필요)

## 체크사항

- [ ] Tier3 API 키 교체 후 test_llm_models.py 6/6 재확인
- [ ] 실제 파이프라인 실행 시 출력 길이/토큰 사용량 모니터링
