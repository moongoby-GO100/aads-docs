---
project: AADS
task_id: T-049
title: CEO 대시보드 7페이지 + 다크테마
completed_at: 2026-03-05T10:07 KST
status: partial
commit_sha: a0125aefd331b13b0317647ff17b92bf5e6bfa4c
commit_url: https://github.com/moongoby-GO100/aads-dashboard/commit/a0125aefd331b13b0317647ff17b92bf5e6bfa4c
---
## 결과
- globals.css 다크테마: OK
- Sidebar 7메뉴: OK
- api.ts 확장 (9함수): OK
- npm run build: OK (에러 0)
- 페이지별 HTTP: / 307, /project-status 307, /conversations 307, /managers 307, /decisions 307, /settings 307, /login 200
  (307은 인증 미들웨어 → /login 리다이렉트, -L 플래그로 최종 200 확인)
- Docker 재빌드: FAIL — claudebot 사용자가 docker 그룹에 미포함, Permission denied (/var/run/docker.sock). 컨테이너는 v0.2.0으로 계속 서빙 중. 루트 권한으로 수동 `docker compose build aads-dashboard && docker compose up -d aads-dashboard` 필요.
- Git push: OK (main 브랜치에 푸시 완료)

## 파일 변경
- src/app/globals.css (수정) — 다크테마 CSS 변수 완성
- src/components/Sidebar.tsx (수정) — 7개 메뉴: Dashboard, Project Status, Conversations, Managers, CEO Decisions, Pipeline, Settings
- src/lib/api.ts (수정) — 9개 신규 함수 추가: getProjectDashboard, getProjectDetail, getTimeline, getAlerts, getConversationStats, getConversations, getMemorySearch, getCeoDecisions, getManagerInbox
- src/app/page.tsx (리디자인) — 4 통계 카드 + 6 프로젝트 그리드 + 최근 대화 5건 + 알림/CEO결정
- src/app/project-status/page.tsx (신규) — 프로젝트 카드 리스트, 진행률 바, 클릭 시 상세 이동
- src/app/project-status/[id]/page.tsx (신규) — 프로젝트 헤더, 대화 타임라인, 매니저 inbox 요약
- src/app/conversations/page.tsx (수정) — 프로젝트별 탭, 키워드 검색, 대화 카드
- src/app/managers/page.tsx (수정) — 프로젝트 매니저 + 코어 에이전트 카드/테이블
- src/app/decisions/page.tsx (신규) — CEO 결정 타임라인 + 알림 목록 (기간 필터)
- src/app/settings/page.tsx (신규) — 시스템 상태, 버전 정보, 빠른 링크

## 주요 이슈
1. Docker 재빌드 불가 — claudebot 계정이 docker 그룹 미소속. `/var/run/docker.sock` Permission denied.
   해결 방법: `usermod -aG docker claudebot` 후 재로그인 또는 root로 수동 빌드.
2. Git push 로컬 ref 에러 — `.git/logs/refs/remotes/origin/main` 에 Permission denied 발생하나 원격 푸시는 성공.

## 빌드 출력 (요약)
```
▲ Next.js 16.1.6 (Turbopack)
✓ Compiled successfully in 14.1s
✓ Generating static pages using 7 workers (12/12) in 702.9ms

Route (app)
├ ○ /
├ ○ /conversations
├ ○ /decisions
├ ○ /login
├ ○ /managers
├ ○ /project-status
├ ƒ /project-status/[id]
├ ○ /projects
├ ○ /settings
```

## 스크린샷
Visual QA 미수행 (Docker 재빌드 실패로 인해 새 코드가 컨테이너에 배포되지 않음).
