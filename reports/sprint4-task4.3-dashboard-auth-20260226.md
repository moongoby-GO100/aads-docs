# Task 4.3: 대시보드 인증 플로우 완성

- **일시**: 2026-02-26
- **Sprint**: 4 — Task 4.3

## 변경 사항

| 구분 | 경로 | 내용 |
|------|------|------|
| 회원가입 | dashboard/src/app/signup/page.tsx | Supabase 이메일/비밀번호 회원가입 |
| 인증 컨텍스트 | dashboard/src/contexts/AuthContext.tsx | 전역 세션 관리 |
| 네비게이션 | dashboard/src/components/Navbar.tsx | 로그인 상태 표시 + 로그아웃 |
| 로그인 | dashboard/src/app/login/page.tsx | Task 4.0에서 생성 |

## 인증 플로우

1. /signup → createUser → 이메일 확인 → /login
2. /login → signInWithPassword → JWT → /projects
3. AuthContext → 전역 session → Navbar 표시
4. 로그아웃 → signOut → /login

## 결과

- 적용: ✅ 소스 반영 완료
- /signup: ✅ 회원가입 동작
- /login: ✅ 로그인 후 /projects 이동
- AuthContext: ✅ 전역 인증 상태
- Navbar: ✅ 상태 표시 + 로그아웃
- AUTH_ENABLED=true 전환 시 API Bearer 인증 준비 완료
