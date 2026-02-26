# Sprint 3 Task 3.5 — 모델 뷰 + 로그 페이지

**일시**: 2026-02-26 20:35 KST

## 생성 페이지

| 경로 | 기능 |
|------|------|
| /models | 4개 모델 카드 (Tier/가격/역할/상태) + 시스템 상태 |
| /logs | 작업 로그 실시간 테이블 (Supabase work_logs) |

## /models 기능

- Tier 1~4 모델 카드 (정상/차단 상태 표시)
- 시스템 상태: API서버, SDK, Anthropic Tier, 잔액

## /logs 기능

- work_logs 테이블 실시간 구독
- 최근 50건 표시, INSERT 이벤트 자동 추가
