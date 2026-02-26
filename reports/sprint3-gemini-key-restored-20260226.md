# Gemini API 키 교체 완료

- **일시**: 2026-02-26
- **이전 키**: AIzaSyBCr8... (차단됨, 삭제 예정)
- **신규 키**: AIzaSyCX6... (newtalk 프로젝트)

## 검증 결과

| 항목 | 결과 | 비고 |
|------|------|------|
| Tier 3 (Gemini 2.5 Flash) | ✅ 통과 | `AADS_TIER3_OK` (1.4s) |
| 전체 모델 테스트 | **6/6 통과** | Redis, Supabase, Tier1~4 모두 OK |
| Monitoring 단계 (POST /projects/{id}/run) | ❌ 403 | "API key was reported as leaked" — 8001 서비스가 신규 키 미반영 가능성, **서비스 재시작 필요** |

## 후속 조치

- [ ] 8001 포트 서비스(aads-api 등) 재시작 후 `/projects/{id}/run` 재호출 검증
- [ ] 이전 키(GOOGLE_AI_API_KEY 구값) Google Cloud Console에서 삭제 권장
