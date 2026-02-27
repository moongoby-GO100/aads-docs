# Task 5.6: 결과물 미리보기 + 배포 버튼

- **일시**: 2026-02-27 KST
- **Sprint**: 5 — Task 5.6

## 변경 사항

| 파일 | 내용 |
|------|------|
| dashboard/src/app/projects/[id]/results/page.tsx | 결과물 페이지 (파일 목록 + 뷰어 + 배포 + 다운로드) |
| dashboard/src/app/projects/[id]/page.tsx | 📦 결과물 버튼 추가 |

## 기능

- 파일 목록 표시 (좌측 트리)
- 파일 내용 보기 (우측 코드 뷰어)
- 🚀 배포하기 버튼 (Docker 빌드 + 실행)
- 📥 ZIP 다운로드
- 🌐 프리뷰 보기 (배포 후)

## 검증

| 항목 | 결과 |
|------|------|
| results/page.tsx 생성 | ✅ 생성됨 (src/app/projects/[id]/results/page.tsx) |
| 프로젝트 상세 버튼 추가 | ✅ 거부 버튼 옆에 📦 결과물 링크 추가 |

## 의존성

- Task 5.3 (code_store), 5.4 (deployer) API 연동 필요
- 백엔드: `GET /projects/:id/files`, `GET /projects/:id/deploy-status`, `POST /projects/:id/deploy`, `GET /projects/:id/download`

## 배포

- dashboard 빌드: `cd /root/aads/aads-core/dashboard && npm run build`
- 서비스 재시작: `systemctl restart aads-dashboard` (전체 Sprint 5 작업 완료 후 권장)
