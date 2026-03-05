# T-058 결과 보고서: project_dashboard.py 상세API 및 conversation_count 버그 수정

## 개요
- Task ID: T-058
- 서버: 68 (aads.newtalk.kr)
- 우선순위: P1-HIGH
- 의존성: T-057 완료
- 완료 시각 (KST): 2026-03-05T10:51:17+09:00

---

## 버그 원인 분석

### Bug 1: 상세 API progress_percent=0
`get_project_detail` 함수에 `go100_user_memory` 테이블의 `project_status` 레코드를 조회하여 `progress_percent` 등을 override하는 로직이 누락되어 있었음. 목록 API(`get_dashboard`)에는 `status_map` override 로직이 있었으나 상세 API에는 없었음.

### Bug 2: conversation_count mapping 오류
`get_dashboard`의 `conv_rows` 처리 시 `system_memory`의 `conversation:sf`, `conversation:sales`, `conversation:kis` 카테고리에서 추출한 proj_key ("sf", "sales", "kis")에 `CONV_PROJECT_MAP`을 적용하지 않아 `conv_stats`에 "sf", "sales", "kis" 키로 저장됨. 이후 프로젝트 목록 조립 시 `conv_stats.get("shortflow", {})` 등 실제 project_id로 조회하면 매칭되지 않아 conversation_count=0 반환.

DB 확인 결과 (`system_memory`의 `conversation:%` 카테고리):
- `conversation:sf`: 69건
- `conversation:sales`: 58건
- `conversation:kis`: 34건
- `conversation:aads`: 22건

참고: `aads_conversations` 테이블은 해당 서버 DB에 존재하지 않음 (테이블 없음). 대화 데이터는 `system_memory`의 `conversation:*` 카테고리로 관리됨.

---

## 수정 내용

### 파일: `/root/aads/aads-server/app/api/project_dashboard.py`
백업: `project_dashboard.py.bak.T058`

#### 수정 1: `get_project_detail` — project_status override 추가

**추가된 DB 쿼리** (async with conn 블록 내):
```python
# go100_user_memory project_status 최신 레코드 조회
status_row = await conn.fetchrow(
    """
    SELECT content FROM go100_user_memory
    WHERE user_id = 2 AND memory_type = 'project_status'
      AND content->>'project_id' = $1
    ORDER BY created_at DESC LIMIT 1
    """, project_id
)
```

**추가된 변수 초기화**:
```python
total_tasks = 0
completed_tasks = 0
handover_url = ""
key_issues: List[str] = []
```

**추가된 override 로직** (sys_rows 루프 이후):
```python
# project_status override from go100_user_memory
if status_row:
    s = status_row["content"] if isinstance(status_row["content"], dict) else json.loads(status_row["content"])
    progress_percent = s.get("progress_percent", progress_percent)
    total_tasks = s.get("total_tasks", total_tasks)
    completed_tasks = s.get("completed_tasks", completed_tasks)
    handover_url = s.get("handover_url", handover_url)
    key_issues = s.get("key_issues", key_issues)
    if s.get("status"):
        status = s["status"]
```

**return 딕셔너리에 필드 추가**:
- `total_tasks`, `completed_tasks`, `handover_url`, `key_issues`

#### 수정 2: `get_dashboard` — conversation_count mapping 수정

**변경 전**:
```python
for r in conv_rows:
    # category: conversation:go100 → project_id: go100
    proj_key = r["category"].replace("conversation:", "")
    cnt = r["cnt"]
    total_conversations += cnt
    conv_stats[proj_key] = {
        "count": cnt,
        "last_updated": str(r["last_updated"]),
    }
```

**변경 후**:
```python
for r in conv_rows:
    # category: conversation:sf → proj_key: sf → mapped: shortflow (CONV_PROJECT_MAP)
    proj_key = r["category"].replace("conversation:", "")
    proj_key = CONV_PROJECT_MAP.get(proj_key, proj_key)
    cnt = r["cnt"]
    total_conversations += cnt
    if proj_key in conv_stats:
        conv_stats[proj_key]["count"] += cnt
    else:
        conv_stats[proj_key] = {
            "count": cnt,
            "last_updated": str(r["last_updated"]),
        }
```

---

## 배포

컨테이너 ID: `e48383aa0587ef010416a1d7d61074b3470ab0025986d2cda7b66edece7b201e`

```
INJECT_OK
aads-api: stopped
aads-api: started
RESTART_OK
```

---

## 검증 결과

### 목록 API: GET /projects/dashboard
```
go100: progress=97, conv=0
kis_v41: progress=85, conv=34
shortflow: progress=80, conv=69
nas: progress=65, conv=0
newtalk_v2: progress=60, conv=58
aads: progress=75, conv=22
```

- shortflow conversation_count=69 (≥50 ✓)
- kis_v41 conversation_count=34 (≥30 ✓)
- newtalk_v2 conversation_count=58 ✓
- aads conversation_count=22 ✓

### 상세 API: GET /projects/dashboard/go100
```
progress=97
```
- progress_percent=97 ✓

---

## Git

- 저장소: aads-server
- 커밋: `7824f3d`
- 메시지: `[AADS] fix: T-058 detail API progress override + conversation_count mapping`
- Push: origin/main 완료

---

## 완료 기준 충족 여부

| 기준 | 결과 | 충족 |
|------|------|------|
| GET /projects/dashboard/go100 → progress_percent=97 | progress=97 | ✓ |
| GET /projects/dashboard → shortflow conversation_count≥50 | conv=69 | ✓ |
| GET /projects/dashboard → kis_v41 conversation_count≥30 | conv=34 | ✓ |
| Git push 완료 (aads-server) | 커밋 7824f3d push됨 | ✓ |
