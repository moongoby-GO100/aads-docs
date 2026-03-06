# TECH-001: Adaptive UI 컴포넌트 구조 + 라우터 설계

## 컴포넌트 구조

### AdaptiveRouter
```
AdaptiveRouter
├── KeywordParser       — 입력 텍스트에서 키워드 추출
├── UIMapper            — 키워드 → UI 컴포넌트 매핑
├── DataFetcher         — 매핑된 API 엔드포인트 호출
└── RenderEngine        — 컴포넌트 렌더링 (React)
```

### 컴포넌트 목록
- `CostChart`       : 비용 시계열 차트
- `StatusBoard`     : 시스템 상태 그리드
- `ErrorList`       : 에러 로그 테이블
- `DocList`         : 문서 목록 카드
- `ChannelList`     : 대화창 목록
- `ProjectBoard`    : 프로젝트 칸반
- `MemoryView`      : 메모리 조회 패널
- `ApprovalQueue`   : 승인 대기 큐
- `DeployStatus`    : 배포 상태 타임라인
- `AgentList`       : 매니저 에이전트 목록

## 라우터 로직
```python
def route(keyword: str) -> (component, api_url):
    for pattern, comp, url in ROUTING_TABLE:
        if pattern in keyword:
            return comp, url
    return DefaultView, "/api/v1/context/public-summary"
```

## API 엔드포인트 맵 (현재 구현됨)
- GET /api/v1/health
- GET /api/v1/context/system/{category}
- GET /api/v1/watchdog/errors
- POST /api/v1/memory/log
- GET /api/v1/documents  ← T-102 신규
- GET /api/v1/channels   ← T-103 신규

## 생성 일시
2026-03-05 KST (소급 저장: 2026-03-06 KST)
