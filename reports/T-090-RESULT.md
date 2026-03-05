# T-090 멀티프로젝트 데이터 파이프라인 결과 보고서

## project_tasks 테이블 레코드 수 (프로젝트별)

```
  project  | count
-----------+-------
 AADS      |    80
 GO100     |    61
 ShortFlow |    52
 NewTalk   |    16
 KIS       |     9
 합계        |   218
```

- 로컬 지시서 (done/) → 114건 (source='local')
- go100_user_memory cross_msg → 101건 (source='REMOTE_211'/'REMOTE_114')
- system_memory cross_msg → 2건

## API 응답 예시

### ?project=KIS
```
KIS: 9건
```

### ?project=GO100
```
GO100: 61건
```

### GET /api/v1/dashboard/directives?project=KIS 상세
```json
{
  "status": "ok",
  "total": 9,
  "running": 0,
  "completed": 9,
  "error": 0,
  "by_project": {"KIS": 9}
}
```

## 프론트엔드

- URL: https://aads.newtalk.kr/tasks
- 프로젝트 탭 클릭 → `?project=KIS` 등 API 호출
- 각 프로젝트 탭에 실제 건수 뱃지 표시 (by_project 기반)
- 데이터 없을 시: "이 프로젝트의 작업 기록이 없습니다. 원격 서버에서 수집 대기중입니다." 안내

## Git SHA

- server: e0d75f5b9c47cf6983736e043fdb99040177dba0
- dashboard: c2b503e9b080af8d476fafad98fb629e69e90fca

## 빌드 에러

- 빌드 에러 수: 0
- HTTP /health: 200
