# Task 5.0: 대시보드 기동 + 도메인 연결

- **일시**: 2026-02-27 KST
- **Sprint**: 5 — Task 5.0

## 변경 사항

| 구분 | 내용 |
|------|------|
| 대시보드 빌드 | npm run build 완료 (layout.tsx AuthProvider 여는 태그 추가로 빌드 오류 수정) |
| systemd 등록 | aads-dashboard.service 생성, enable, PATH에 NVM node 추가 |
| 도메인 | aads.newtalk.kr → 68.183.183.11 (Cloudflare Proxied) |
| SSL | Cloudflare Flexible (CF→서버 HTTP, 포트 80) |
| Nginx | /api/ → 8001, / → 3000 (aads.conf, listen 80) |

## 검증

| 항목 | 결과 |
|------|------|
| localhost:3000 | 200, AADS Dashboard (Next.js) HTML 정상 |
| https://aads.newtalk.kr/ | 외부: Cloudflare→80→AADS. 서버 내부 curl 시 443 default(Haru) 응답 — 실사용은 80 경로로 AADS 제공 |
| https://aads.newtalk.kr/api/health | Host: aads.newtalk.kr 로 80 요청 시 `{"status":"healthy",...}` 정상. (서버 내부에서 https로 curl 시 443 default로 Haru HTML 응답) |
| systemctl status aads-dashboard | active (running), Next.js 16.1.6 Ready |

## 브라우저 확인 (2026-02-27)

| 항목 | 결과 |
|------|------|
| 1차 https://aads.newtalk.kr/ | ❌ Haru Music 앱 서빙됨 — 443 default_server. 빈 회색 화면 + Angular/port.js 콘솔 에러 |
| 조치 | aads.newtalk.kr용 **443 server 블록 추가** + Let's Encrypt 인증서 발급 (`certbot certonly --webroot -d aads.newtalk.kr`) |
| 서버 검증 (443) | `curl -sk https://127.0.0.1/ -H "Host: aads.newtalk.kr"` → AADS Dashboard HTML ✅, `/api/health` → JSON ✅ |
| 외부 브라우저 | 캐시/경로에 따라 이전(Haru) 응답이 나올 수 있음. **강력 새로고침(Ctrl+Shift+R)** 또는 시크릿 창으로 재확인 권장 |

## 접속 URL
- 대시보드: https://aads.newtalk.kr
- API: https://aads.newtalk.kr/api/health
- 모델: https://aads.newtalk.kr/api/models
- 템플릿: https://aads.newtalk.kr/api/templates
- 시스템: https://aads.newtalk.kr/system

## 수정 파일 (aads-core)
- `dashboard/src/app/layout.tsx`: `<AuthProvider>` 여는 태그 추가 (빌드 파싱 오류 수정)
- systemd: `/etc/systemd/system/aads-dashboard.service` — PATH에 NVM node 경로 추가

## 수정 파일 (서버 Nginx)
- `/etc/nginx/conf.d/aads.conf`: listen 80에 `/.well-known/acme-challenge/` 추가, **listen 443 ssl** server 블록 추가 (AADS 대시/API 프록시), Let's Encrypt 인증서 적용
