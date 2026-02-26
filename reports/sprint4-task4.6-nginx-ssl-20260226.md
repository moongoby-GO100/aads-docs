# Task 4.6: Nginx 리버스 프록시 + SSL 준비

- 일시: 2026-02-26
- 변경: /etc/nginx/conf.d/aads.conf (80 → 8001 API, 80 → 3000 Dashboard)
- SSL: certbot 설치 완료, 도메인 연결 시 `certbot --nginx -d 도메인` 실행
- 결과: Nginx 설정 적용 완료, certbot 사용 가능
