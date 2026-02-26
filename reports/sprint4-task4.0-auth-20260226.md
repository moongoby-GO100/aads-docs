# Task 4.0: Supabase Auth 인증

- **일시**: 2026-02-26
- **변경**: core/auth.py, core/auth_middleware.py, dashboard/login 페이지, .env.example
- **설정**: AUTH_ENABLED=false (개발 모드), true 전환 시 JWT 필수
- **결과**: 적용 완료. API에 optional_auth 의존성 주입 시 선택적 인증 가능.

## 변경 파일

| 경로 | 내용 |
|------|------|
| aads-core/core/auth.py | Supabase /auth/v1/user로 JWT 검증 |
| aads-core/core/auth_middleware.py | optional_auth: AUTH_ENABLED=false면 anonymous 반환 |
| aads-core/dashboard/src/app/login/page.tsx | 이메일/비밀번호 로그인 → /projects 리다이렉트 |
| aads-core/.env.example | AUTH_ENABLED=false 추가 |

## 사용 방법

- **API**: `Depends(optional_auth)` 사용 시 AUTH_ENABLED=true면 Authorization Bearer 필수.
- **대시보드**: `/login` 접속 후 Supabase 로그인 → 성공 시 `/projects`로 이동.
