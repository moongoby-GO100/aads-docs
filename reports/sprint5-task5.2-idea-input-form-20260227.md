# Task 5.2: 아이디어 입력 폼

- **일시**: 2026-02-27 KST
- **Sprint**: 5 — Task 5.2

## 변경 사항

| 파일 | 내용 |
|------|------|
| dashboard/src/app/new/page.tsx | 2단계 프로젝트 생성 폼 (템플릿 선택 → 아이디어 입력) |
| dashboard/src/app/layout.tsx | 사이드바 "🚀 새 프로젝트" 링크 추가 |

## 기능

- Step 1: 7개 템플릿 + 직접 입력 선택
- Step 2: 프로젝트명 + 아이디어 텍스트 + 실행모드(자동/수동)
- 생성 시 자동으로 run-all 호출 → 프로젝트 상세로 이동

## 검증

| 항목 | 결과 |
|------|------|
| /new 페이지 로드 | ✅ 200 (https://aads.newtalk.kr/new) |
| npm run build | ✅ Compiled successfully, Route /new 생성됨 |
| 사이드바 링크 | ✅ layout.tsx에 `/new` 링크 반영 |
| 프로젝트 생성·자동 실행 | API 연동 코드 반영 (실제 생성은 백엔드 연동 시 검증) |

## 적용·배포

- 적용: ✅ 소스 반영 (aads-core)
- 배포: ✅ 빌드 후 systemctl restart aads-dashboard 수행, /new 200 확인
- 커밋: 변경 사항은 Task 5.4 커밋에 포함되어 origin/main에 푸시됨

## 체크리스트

- [x] src/app/new/page.tsx 생성
- [x] layout.tsx에 /new 링크
- [x] npm run build 성공
- [x] https://aads.newtalk.kr/new 표시(200)
- [x] aads-core 푸시 완료 (5.4 커밋에 포함)
