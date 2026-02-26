# Gemini API 키 점검 + Tier3 403 해결

**일시**: 2026-02-26 19:50 KST

## 진단

| 항목 | 결과 |
|------|------|
| GOOGLE_AI_API_KEY 존재 | ✅ .env에 설정됨 |
| Gemini 2.5 Flash 직접 호출 | ❌ 403 Your API key was reported as leaked. Please use another API key. |
| llm_client.py 연동 | ✅ GOOGLE_AI_API_KEY 사용 중 |
| test_llm_models.py Tier3 | ❌ 동일 403 (키 유출 신고로 차단) |

## 원인

- **403 원인**: 현재 `.env`의 `GOOGLE_AI_API_KEY`가 Google 측에서 **유출된 키로 신고되어 차단**된 상태입니다.
- Tier3(Gemini 2.5 Flash)만 실패하며, Tier1/2/4(Claude)는 정상 동작합니다.

## 조치 (필수)

1. **새 API 키 발급**
   - https://aistudio.google.com/apikey 접속 후 **새 API 키 생성**
   - 기존 키는 폐기 권장(이미 유출 처리됨)

2. **.env 수정**
   - `GOOGLE_AI_API_KEY=새로_발급한_키` 로 교체
   - `.env`는 Git에 커밋하지 않음(이미 .gitignore 대상)

3. **검증**
   - `cd /root/aads/aads-core && source .venv/bin/activate && python tests/test_llm_models.py`
   - Tier3 Gemini 2.5 Flash가 ✅ 로 나오면 해결

## 코드 변경 (이번 작업)

- `core/llm_client.py`: Gemini 호출 시 `GOOGLE_API_KEY`, `GEMINI_API_KEY` 폴백 지원 추가  
  → 문서/스크립트에서 `GOOGLE_API_KEY` 사용 시에도 동작하도록 통일

## 참고

- Gemini API 키 발급: https://aistudio.google.com/apikey
- 무료 한도: 15 RPM, 100만 토큰/일 (제품 문서 기준)
- 구 패키지 `google.generativeai`는 deprecated 안내가 뜨지만 현재 동작함. 추후 `google.genai`로 이전 검토 가능.
