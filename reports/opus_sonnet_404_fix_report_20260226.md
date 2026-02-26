# Opus 4.6 / Sonnet 4.6 404 해결 보고서

**작성일**: 2026-02-26
**작업**: Anthropic snapshot ID → alias 전환 (404 해결)

---

## 1. 요약

- **원인**: API 호출 시 snapshot ID(`claude-opus-4-6` 등) 사용 시 404 발생.
- **조치**: 모델 ID를 alias(`claude-opus-4-6`, `claude-sonnet-4-6`, `claude-haiku-4-5`)로 통일.

---

## 2. 변경 사항

| 구분 | 파일 | 내용 |
|------|------|------|
| 환경 | `.env` | `DEFAULT_TIER1/2/4_MODEL` 값을 snapshot → alias로 변경 (로컬만, git 미포함) |
| 코드 | `core/model_router.py` | `MODEL_CONFIG` fallback 및 `AVAILABLE_MODELS`를 alias로 통일 |
| 테스트 | `tests/test_llm_models.py` | 기본값을 alias로 통일 |

---

## 3. 검증 결과

| 항목 | 결과 |
|------|------|
| SDK 버전 | 0.84.0 (업그레이드 전·후 동일, 목표 이상) |
| 직접 테스트 | Opus 4.6 ✅, Sonnet 4.6 ✅, Haiku 4.5 ✅ |
| 전체 테스트 | `tests/test_llm_models.py` 6/6 통과 |

---

## 4. 적용·배포

- **적용**: ✅ 소스 반영 완료
- **배포**: ✅ `git push origin main` 완료 (f7cba9b)
- **주의**: `.env`는 버전 관리 제외. `.env.backup.*`도 .gitignore 추가함.

## 5. Task 2.0.2 (2026-02-26) 추가 조치

| 항목 | 내용 |
|------|------|
| `tests/test_sdk.py` | `claude-sonnet-4-6-20250514` → `claude-sonnet-4-6` alias로 수정 |
| `.gitignore` | `.env.backup.*` 추가 (백업 파일 미커밋) |
| 검증 | `test_llm_models.py` 6/6 통과, `test_pipeline.py` exit 0 (max_tokens 400 이슈는 별건) |

---

## 6. llm_client.py 확인

- `anthropic.Anthropic()` 사용, `client.messages.create(model=model_id, ...)` 호출.
- `model_id`는 `model_router`에서 오며, alias 적용으로 정상 동작 확인.
