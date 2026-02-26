# Task 4.5: API 레이트 리밋 + 보안 강화

- **일시**: 2026-02-26
- **스프린트**: Cursor 4 — Sprint 4.5

## 변경 사항

| 항목 | 내용 |
|------|------|
| slowapi | 설치 및 `requirements.txt` 추가 |
| `core/rate_limiter.py` | 신규 — 기본 60/min, 파이프라인 5/min 상수 |
| `api/main.py` | limiter 통합, CORS 유지, 보안 헤더 미들웨어, 파이프라인 엔드포인트 제한 |

## 레이트 리밋

- **기본**: 60회/분 (전역, `default_limits`)
- **파이프라인 실행**: 5회/분 — `POST /projects/{id}/run`, `POST /projects/{id}/run-all`에 `@limiter.limit(PIPELINE_LIMIT)` 적용

## 보안

- **CORS**: 기존 설정 유지 (`allow_origins=["*"]` 등)
- **보안 헤더**: `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `X-XSS-Protection: 1; mode=block` 미들웨어로 추가
- **RateLimitExceeded**: 429 응답 처리 (`_rate_limit_exceeded_handler`)

## 검증

- `python -c "from api.main import app"` 통과
- `uvicorn api.main:app` 기동 정상
- `curl -D - http://127.0.0.1:8765/health` 응답에 보안 헤더 포함 확인

## 적용/배포

- 적용: ✅ 소스 반영
- 배포: aads-core 서비스 재시작 시 반영
