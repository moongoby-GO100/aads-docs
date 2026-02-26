# Gemini 최종 수정

- 일시: 2026-02-26
- 변경: 모델명 gemini-2.0-flash → gemini-2.5-flash (deprecated 대응, 코드베이스는 이미 2.5-flash 통일 상태)
- 변경: nohup → systemd 전환 (EnvironmentFile로 .env 로드)
- 변경: workers 2 → 1 (메모리 분리 이슈 방지)
- 검증: systemctl active, /health OK, /models tier3=gemini-2.5-flash
- Monitoring 결과: POST /projects/17262de7/run → "프로젝트 없음" (해당 ID 미등록, API·서비스 정상 동작)
