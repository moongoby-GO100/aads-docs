# Task 4.9: 베타 테스트 + 피드백

- **일시**: 2026-02-27
- **Sprint**: 4 — Task 4.9

## 테스트 항목

| # | 테스트 | 대상 |
|---|--------|------|
| 1 | Health check | GET /health |
| 2 | Models list | GET /models |
| 3 | Templates list | GET /templates |
| 4 | System metrics | GET /metrics/system |
| 5 | LLM metrics | GET /metrics/llm |
| 6 | Security headers | curl -D 헤더 확인 |
| 7 | Project create | POST /projects |
| 8 | Project get | GET /projects/{id} |
| 9 | Project list | GET /projects |
| 10 | Pipeline run | POST /projects/{id}/run |
| 11 | Project delete | DELETE /projects/{id} |
| 12 | Template create | POST /projects/from-template/todo-app |
| 13 | pytest suite | python3 -m pytest tests/ -v |

## 실행 결과

```
=== AADS Beta Test Fri Feb 27 11:10:49 KST 2026 ===
❌ FAIL: Health — got: 
❌ FAIL: Models — got: 
❌ FAIL: Templates — got: 
❌ FAIL: Metrics/system — got: 
❌ FAIL: Metrics/llm — got: 
❌ FAIL: Security headers — got: 
```

**참고**: 실행 시점에 AADS API(8001)가 응답하지 않음(`systemctl is-active aads-api` → activating). API 기동 후 재실행 시 CRUD·파이프라인·pytest 결과가 추가됨.

## 결과 파일

- 스크립트: aads-core/tests/beta_test.sh
- 결과: /tmp/beta-test-result-YYYYMMDD.txt

## 피드백 및 개선점

- 테스트 스크립트: ✅ 등록 및 push 완료
- API 미기동으로 본 실행은 Health~Security까지 6건 FAIL
- **재검증**: `systemctl start aads-api` 후 `bash /root/aads/aads-core/tests/beta_test.sh` 실행 권장
- 파이프라인 실행 시 LLM 비용 발생 ($0.5~1.0 예상)
