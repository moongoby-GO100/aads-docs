# Gemini API 키 교체 + Tier3 복구

**일시**: 2026-02-26 20:25 KST

## 상태

| 항목 | 결과 |
|------|------|
| GOOGLE_AI_API_KEY 존재 | ✅ 존재 (AIzaSyBCr8...) |
| Gemini 2.5 Flash 테스트 | ❌ 403 Your API key was reported as leaked |
| 전체 모델 테스트 | ⚠️ 5/6 통과 (Tier3만 실패) |

## 조치

- **키 유효**: ❌ **키 교체 필요** — 현재 키가 유출로 차단됨. 새 키 발급 후 `.env`에 반영 필요.
- Tier1 Opus / Tier2 Sonnet / Tier4 Haiku + Redis / Supabase: ✅ 정상.

## 미해결 시 안내

- 키 발급: https://aistudio.google.com/apikey
- `.env` 수정: `GOOGLE_AI_API_KEY=새키` (경로: `/root/aads/aads-core/.env`)
- 테스트: `cd /root/aads/aads-core && source .venv/bin/activate && python tests/test_llm_models.py`

## 후속

- [ ] 대표님께서 새 Gemini 키 발급 후 `.env` 교체
- [ ] 교체 후 동일 스크립트 또는 `python tests/test_llm_models.py` 재실행하여 6/6 통과 확인
