# TECH-002: AADS 정적 스냅샷 시스템 연구

## 목적
AADS API 서버가 다운되거나 응답이 느릴 때 CEO가 핵심 정보에 즉시 접근할 수 있도록
정적 JSON 스냅샷을 주기적으로 생성·저장하는 시스템.

## 아키텍처
```
cron (5분 주기)
└── generate_manager_snapshot.py
    ├── GET /api/v1/context/public-summary  → public/manager/summary.json
    ├── GET /api/v1/watchdog/errors         → public/manager/errors.json
    ├── GET /api/v1/documents               → public/manager/documents.json  ← T-102
    └── GET /api/v1/channels                → public/manager/channels.json   ← T-103
```

## 파일 구조 (public/manager/)
```
public/manager/
├── summary.json      — 전체 시스템 상태 요약
├── errors.json       — 최근 에러 목록
├── documents.json    — CEO 문서 목록 (T-102)
└── channels.json     — 매니저 대화창 목록 (T-103)
```

## CEO 접근 URL
- https://aads.newtalk.kr/manager/documents.json

## 크론 설정
```
*/5 * * * * /usr/bin/python3 /root/aads/scripts/generate_manager_snapshot.py >> /root/aads/logs/snapshot.log 2>&1
```

## 생성 일시
2026-03-06 KST (현재 세션)
