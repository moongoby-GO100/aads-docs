# AADS 능력 확장 — 무료/자체개발 옵션 비교

**문서 ID**: CAPABILITY-MAP-002
**작성일**: 2026-03-02
**상태**: CEO 승인 완료

---

## 목적

CAPABILITY-MAP-001에서 식별한 12개 영역별로 무료 오픈소스 및 자체개발 가능한 옵션을 비교한다.

## 영역별 최선 무료 옵션

| # | 영역 | 최선 무료 옵션 | 라이선스 | 비용 | 비고 |
|---|------|--------------|---------|------|------|
| 1 | 데스크톱 | ScreenEnv (자체호스팅) | MIT | $0 | Docker 기반 |
| 2 | 브라우저 | browser-use v0.12.0 | MIT | $0 | pip install |
| 3 | 모바일 | Droidrun | MIT | $0 | 91.4% 정확도 |
| 4 | 외부 서비스 | Activepieces | MIT | $0 | 280+ 앱, Docker |
| 5 | STT | Groq Whisper API | — | $0 (무료 티어) | 100분/월 무료 |
| 6 | TTS | Google Cloud TTS | — | $0 (무료 티어) | 100만 글자/월 무료 |
| 7 | 시각 분석 | Claude Vision (내장) | — | $0 | 기존 LLM 활용 |
| 8 | 이미지 생성 | DALL-E 3 API | — | $0.04/장 | 최저 품질 대비 가격 |
| 9 | DB/파일 | PostgreSQL MCP + Filesystem MCP | MIT | $0 | Phase 1 완료 |
| 10 | 배포 | Fly.io 무료 티어 + Terraform MCP | — | $0~5 | 이미 구성 |
| 11 | 보안 | Trivy | Apache-2.0 | $0 | 32K GitHub Stars |
| 12 | 모니터링 | GlitchTip 자체호스팅 | MIT | $0 | Sentry SDK 호환, RAM 512MB |

## 결론

12개 영역 중 9개가 완전 무료 자체 구축 가능. 3개(STT, TTS, 이미지)는 무료 티어 API가 자체 GPU 호스팅보다 경제적.
