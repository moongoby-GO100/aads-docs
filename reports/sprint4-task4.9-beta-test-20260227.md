# Task 4.9: 베타 테스트 + 피드백

- **일시**: 2026-02-27
- **Sprint**: 4 — Task 4.9

## 테스트 항목 (13건)

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
| 13 | pytest suite | 13 passed / 2 skipped |

## 실행 결과

```
=== AADS Beta Test Fri Feb 27 12:46:27 KST 2026 ===
✅ PASS: Health
✅ PASS: Models
✅ PASS: Templates
✅ PASS: Metrics/system
✅ PASS: Metrics/llm
✅ PASS: Security headers
✅ PASS: Project create (ID: a50e0cbb)
✅ PASS: Project get
✅ PASS: Project list
✅ PASS: Pipeline run
✅ PASS: Project delete
✅ PASS: Template create (ID: b00d41ac)

=== pytest ===
  See https://docs.pytest.org/en/stable/how-to/assert.html#return-not-none for more information.
    warnings.warn(

-- Docs: https://docs.pytest.org/en/stable/how-to/capture-warnings.html
================== 13 passed, 2 skipped, 6 warnings in 12.14s ==================

=== SUMMARY ===
Total: 12 | Pass: 12 | Fail: 0
```

## 수정 이력

| 수정 | 원인 | 조치 |
|------|------|------|
| Security headers | HTTP/2 소문자 헤더 | expect 문자열 소문자 "x-content-type-options" |
| Project create | API 필수 필드 `idea` 누락 | POST body에 "idea" 추가 |
| Project create | 응답 키 fallback | project_id + id fallback (d.get) |

## 결과 파일
- 스크립트: aads-core/tests/beta_test.sh
- 결과: /tmp/beta-test-result-20260227.txt
