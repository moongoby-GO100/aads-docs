# AADS 능력 확장 — 자체 구축 확정 계획표

**문서 ID**: CAPABILITY-MAP-003
**작성일**: 2026-03-02
**상태**: CEO 승인 완료

---

## 목적

최종 결정: 9개 자체구축 + 3개 외부API 확정 및 구축 일정.

## 자체 구축 (9개)

| # | 영역 | 도구 | 방식 | 구축 일수 | Phase |
|---|------|------|------|----------|-------|
| 1 | 데스크톱 | ScreenEnv | Docker 자체호스팅 | 1~2일 | 3 |
| 2 | 브라우저 | browser-use | pip install | 1~2일 | 2 |
| 3 | 모바일 | Droidrun | pip + ADB | 3일 | 3~4 |
| 4 | 외부 서비스 | Activepieces | Docker 자체호스팅 | 0.5일 | 2 |
| 5 | DB/파일 | PostgreSQL MCP + Filesystem MCP | 이미 완료 | 0일 | 1 (완료) |
| 6 | 배포 | Fly.io + Terraform MCP | 무료 티어 | 완료 | 2 |
| 7 | 보안 | Trivy | apt install | 0.5일 | 2 |
| 8 | 모니터링 | GlitchTip | Docker (512MB RAM) | 1일 | 3 |
| 9 | 시각 분석 | Claude Vision | 내장 기능 | 0일 | 2 |

## 외부 API (3개)

| # | 영역 | 서비스 | 월 비용 | 이유 |
|---|------|--------|--------|------|
| 1 | STT | Groq Whisper API | $0~$1 | GPU 불필요, 10x 빠름 |
| 2 | TTS | Google Cloud TTS | $0 | 100만 글자/월 무료 |
| 3 | 이미지 | DALL-E 3 API | $2~$4 | GPU $2,000 투자 대비 무의미 |

## 총 구축 일수: ~18일 (Phase 2~4에 분산)

## 월 추가 비용: $2~$10 (LLM API 별도)
