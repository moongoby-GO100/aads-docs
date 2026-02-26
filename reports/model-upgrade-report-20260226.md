# 최신 모델 전면 교체 보고 (2026-02-26)

## 요약
- .env 및 `core/model_router.py` MODEL_CONFIG를 Opus 4.6 / Sonnet 4.6 / Haiku 4.5 / Gemini 2.5 Flash 기준으로 전면 교체.
- aads-core, aads-docs 커밋 및 push 완료.

## 변경 파일
- **aads-core**: `.env`, `core/model_router.py`, `core/llm_client.py`, `tests/test_llm_models.py`
- **aads-docs**: `architecture/system-architecture.md`, `phase-reports/sprint-1-progress.md`, `phase-reports/sprint-2-progress.md`, `reports/cost-analysis.md`

## 테스트 결과 (python tests/test_llm_models.py)
| 항목 | 결과 | 비고 |
|------|------|------|
| Opus 4.6 (claude-opus-4-6-20260205) | ❌ | API 404 – 모델 ID 미지원 또는 계정/플랜 제한 |
| Sonnet 4.6 (claude-sonnet-4-6-20260217) | ❌ | API 404 – 동일 |
| Gemini 2.5 Flash | ✅ | AADS_TIER3_OK |
| Haiku 4.5 | ✅ | AADS_TIER4_OK |
| Redis 6380 | ✅ | read=OK |
| Supabase | ✅ | projects 1건 |

## Git push 결과
- **aads-core**: `15635b4` → `origin main` ✅
- **aads-docs**: `110b3a0` → `origin main` ✅

## 후속 조치
- Opus 4.6 / Sonnet 4.6가 404인 경우: Anthropic 대시에서 사용 가능한 최신 모델 ID 확인 후 .env 및 MODEL_CONFIG에 반영.
- 또는 기존 동작하던 Opus 4.5 / Sonnet 4 ID로 일시 롤백 가능.
