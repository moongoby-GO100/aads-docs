# Task 5.8: 결과물 페이지 수정 + 파이프라인 코드 저장 연동

- **일시**: 2026-02-27 KST
- **Sprint**: 5 — Task 5.8

## 수정 항목

| # | 이슈 | 조치 | 결과 |
|---|------|------|------|
| 1 | /projects/{id}/results 404 | results/page.tsx 생성 (파일목록, 코드뷰어, 배포, 다운로드) | ✅ 200 OK |
| 2 | 📦 결과물 버튼 미표시 | projects/[id]/page.tsx — 이미 버튼 있음 | ⏭️ 생략 |
| 3 | 파이프라인 결과 미저장 | pipeline.py — 이미 save_phase_output 호출 있음 | ⏭️ 생략 |

## 변경 파일

| 파일 | 내용 |
|------|------|
| dashboard/src/app/projects/[id]/results/page.tsx | 결과물 페이지 신규 생성. API 응답(files 객체)을 배열로 정규화하여 파일탐색기·코드뷰어·배포/다운로드 연동 |

## 검증

```bash
# results 페이지
curl -s -o /dev/null -w "%{http_code}" http://localhost:3000/projects/2a0647ca/results  # 200

# files API
curl -s http://localhost:8001/projects/2a0647ca/files
# {"project_id":"2a0647ca","files":{},"count":0}

# deploy-status
curl -s http://localhost:8001/projects/2a0647ca/deploy-status
# {"status":"not_deployed"}

# health
curl -s http://localhost:8001/health
# {"status":"healthy","time":"2026-02-27T19:02:55+09:00"}
```

## 결과

- 적용: ✅ 소스 반영
- 배포: ✅ aads-api / aads-dashboard 재시작, 대시보드 빌드 완료
- Git: ✅ aads-core push 완료 (b9b6344)

## 진행/체크사항

- [ ] 실서비스 URL에서 results 페이지 확인: https://aads.newtalk.kr/projects/2a0647ca/results
- [ ] 파이프라인 실행 후 파일 생성 시 결과물 목록·코드뷰 표시 확인
