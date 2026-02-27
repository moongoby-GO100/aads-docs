# Task 5.9: 회원가입/로그인 활성화

- **일시**: 2026-02-27 KST
- **Sprint**: 5 — Task 5.9

## 변경 사항

| 항목 | 내용 |
|------|------|
| AUTH_ENABLED | false → true (aads-core/.env) |
| 미들웨어 | middleware.ts 추가 (미인증 → /login 리디렉트) |
| 사이드바 | 🔑 로그인 / 📝 회원가입 링크 추가 (layout.tsx) |
| 대시보드 환경변수 | NEXT_PUBLIC_SUPABASE_URL, NEXT_PUBLIC_SUPABASE_ANON_KEY (.env.production) |

## 사용자 플로우

1. https://aads.newtalk.kr 접속 → /login 리디렉트
2. 회원가입: /signup → 이메일/비밀번호 입력 → Supabase 계정 생성
3. 로그인: /login → 인증 → /projects로 이동
4. 로그아웃: Navbar → 로그아웃 → /login

## 검증

| 항목 | 결과 |
|------|------|
| /login | HTTP 200 |
| /signup | HTTP 200 |
| 미인증 /projects | 307 → /login?redirect=%2Fprojects |
| API health | healthy |
| AUTH_ENABLED | true |

## 결과

- 적용: ✅ aads-core .env, dashboard .env.production, layout.tsx, middleware.ts
- 배포: ✅ systemctl restart aads-api, aads-dashboard 반영
- Git: ✅ aads-core push 완료 (main)
- 로그인 페이지: 사이드바에 로그인/회원가입 링크 노출, AADS Login 폼 정상 표시

## 스크린샷

- 로그인 페이지: aads-login-page-20260227.png (동일 세션에서 캡처)

## 참고

- .env, .env.production은 로컬만 수정(비공개). 배포 서버에서 동일 값 설정 필요.
- Supabase 쿠키: middleware에서 `sb-*-auth-token` 패턴으로 확인.
