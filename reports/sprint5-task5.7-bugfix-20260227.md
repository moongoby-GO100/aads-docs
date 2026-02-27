# Task 5.7: 대시보드 버그 수정 3건

- **일시**: 2026-02-27 KST
- **Sprint**: 5 — Task 5.7

## 수정 항목

| # | 이슈 | 원인 | 조치 | 결과 |
|---|------|------|------|------|
| 1 | 비용 페이지 $0.00 | 파이프라인→Redis 비용 미기록 | cost_tracker.py 생성, /costs API 연동, pipeline에서 record_cost 호출, 기존 $0.82 보정 | ✅ total_cost 0.8207 반영 |
| 2 | 로그 작업명 비어있음 | 로그 저장 시 task 필드 없음 | logger.py 생성, log_task(project_name, phase → task), /logs API 추가, 로그 페이지 추가 | ✅ 파이프라인 실행 시 task 표시 |
| 3 | Gemini ❌ 키 차단 | .env에 GOOGLE_API_KEY/GEMINI_API_KEY 없음 | 키 테스트 스크립트 실행 — 키 없음 확인 | ⚠️ .env에 키 추가 후 재검증 필요 |

## 변경 파일 (aads-core)

| 파일 | 내용 |
|------|------|
| core/cost_tracker.py | 비용 추적 (record_cost, get_cost_summary), 대시보드용 costs 배열 포함 |
| core/logger.py | 작업 로그 (log_task, get_logs) |
| api/main.py | get_cost_summary/get_logs import, /costs·/logs 엔드포인트 |
| core/pipeline.py | Redis record_cost + log_task 호출 (phase 완료 시) |
| dashboard/src/lib/api.ts | getLogs(limit) 추가 |
| dashboard/src/app/logs/page.tsx | 로그 페이지 (작업명·단계·상태·상세) |

## 검증 결과

### /costs API
```json
{
  "total_cost": 0.8207,
  "budget": 500.0,
  "usage_percent": 0.16,
  "remaining": 499.1793,
  "daily": { "2026-02-27": 0.8207 },
  "by_tier": { "tier_1": 0.3902, "tier_2": 0.4295, "tier_3": 0.001 },
  "costs": [ ... ]
}
```

### /logs API
- `GET /logs?limit=50` → 배열 반환. 파이프라인 실행 시 task(작업명) 포함하여 기록됨.

### Gemini API
- GOOGLE_API_KEY / GEMINI_API_KEY 미설정으로 테스트 스킵. 키 추가 후 `aads:model:gemini-2.5-flash:status` = active 설정 가능.

### Git Push
- aads-core: `main -> main` 푸시 완료 (b871f4e).

## 후속 조치

- [ ] .env에 `GOOGLE_API_KEY=` 또는 `GEMINI_API_KEY=` 추가 후 Gemini 호출 테스트 및 Redis 상태 업데이트.
- [ ] 브라우저에서 비용 페이지·로그 페이지 실제 표시 확인 (https://aads.newtalk.kr).
