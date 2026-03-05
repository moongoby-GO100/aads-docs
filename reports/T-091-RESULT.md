---
project: AADS
task_id: T-091
completed_at: 2026-03-05 19:27 KST
---

# T-091 보고서
## 원격 서버 프로젝트별 작업 보고 표준화 — REMOTE_211/114 에이전트 task_result 형식 통일

**작업자**: Claude Code (claude-sonnet-4-6)
**서버**: 68 (aads.newtalk.kr) — 68 로컬 수정 완료, 211/114 SSH 불가(배포 대기)
**완료**: 2026-03-05 19:27 KST

---

## 작업 내용 요약

### Part 1: 211/114서버 aads_remote_agent.py — auto_report_task_results() 추가

**파일**: `/root/aads/scripts/aads_remote_agent.py` (→ aads-server/scripts에 동기)

**추가된 함수:**
- `_detect_task_results_from_dir(project, base_path)`: 프로젝트 디렉토리에서 24시간 이내 변경된 보고서 파일 감지 (reports/*.md, docs/reports/*.md, HANDOVER*.md, RESULT*.md 등 8개 패턴)
- `auto_report_task_results()`: 모든 PROJECTS에 대해 task_result 감지 → 68서버 POST /context/system 전송
- 중복 보고 방지: `_reported_tasks` dict로 task_id 관리
- YAML front matter 파싱: task_id, title, status 추출

**전송 형식:**
```json
{
  "category": "cross_msg_{SERVER_ID}_AADS_MGR",
  "key": "task_result_{task_id}_{timestamp}",
  "value": {
    "message_type": "task_result",
    "project": "KIS",
    "task_id": "KIS-088",
    "status": "completed",
    "title": "...",
    "summary": "...",
    "completed_at": "..."
  }
}
```

**run_collect_cycle()에 통합**: 매 COLLECT_INTERVAL(기본 5분)마다 자동 실행

**211서버 대상 프로젝트**: KIS, GO100, ShortFlow
**114서버 대상 프로젝트**: ShortFlow, NewTalk, NAS

**주의**: 211/114서버는 SSH 연결 불가(Connection timed out). 업데이트된 스크립트가 `/root/aads/scripts/aads_remote_agent.py`에 저장됨. SSH 접근 복구 시 배포 필요.

백업 명령어 (SSH 복구 후 실행):
```bash
ssh REMOTE_211 "cp /root/aads_remote_agent.py /root/aads_remote_agent.py.bak.T091"
ssh REMOTE_114 "cp /root/aads_remote_agent.py /root/aads_remote_agent.py.bak.T091"
```

---

### Part 2: 68서버 context.py — task_result 수신 시 project_tasks 자동 upsert

**파일**: `/root/aads/aads-server/app/api/context.py`

**변경 사항:**
1. `SystemMemoryRequest` 모델에 `data` 필드 추가 (원격 에이전트 curl 호환):
   - `value: Optional[Dict]` (기존)
   - `data: Optional[Dict]` (신규 — value 없을 때 사용)
   - `get_value()` 메서드: value → data 순으로 반환
2. `_upsert_task_result()` 타임스탬프 파싱 수정:
   - 기존: asyncpg에 ISO8601 문자열 직접 전달 → 422 오류
   - 수정: `_parse_ts()` 내부 헬퍼 함수 추가 — `datetime.fromisoformat()` 변환 후 전달

**기존 upsert 로직 (T-090에서 구현된 것 유지):**
- `message_type == "task_result"` 또는 카테고리 `cross_msg_REMOTE_*` + task_id 존재 시 upsert
- `ON CONFLICT (task_id, source)` DO UPDATE
- server_id 추출: `cross_msg_(REMOTE_\d+)` 정규식

---

### Part 3: project_tasks 테이블 — DB 확인

**테이블**: `project_tasks` (T-090에서 기생성 확인)

**스키마**:
```
project      VARCHAR(50) NOT NULL
task_id      VARCHAR(100) NOT NULL
status       VARCHAR(50) DEFAULT 'pending'
title        VARCHAR(200)
summary      TEXT
started_at   TIMESTAMPTZ
completed_at TIMESTAMPTZ
server_id    VARCHAR(50)
source       VARCHAR(50) DEFAULT 'manual'
raw_data     JSONB
UNIQUE(task_id, source)
```

**schema.sql에 DDL 추가** (신규 배포 시 자동 생성용)

---

## 검증 결과

### Health Check
```
GET https://aads.newtalk.kr/api/v1/health
→ {"status":"ok","graph_ready":true,...} HTTP 200 ✓
```

### Test 1: data 필드 (지시서 명세 curl)
```bash
curl -X POST https://aads.newtalk.kr/api/v1/context/system \
  -H "X-Monitor-Key: {KEY}" \
  -H "User-Agent: curl/7.64.0" \
  -d '{"category":"cross_msg_REMOTE_211_AADS_MGR","key":"test_task_result","data":{"message_type":"task_result","project":"KIS","task_id":"KIS-TEST-001","status":"completed","title":"테스트 작업","summary":"연동 테스트"}}'
```
결과:
```json
{
  "status": "ok",
  "saved": "cross_msg_REMOTE_211_AADS_MGR/test_task_result",
  "task_upsert": {"status": "ok", "project": "KIS", "task_id": "KIS-TEST-001", "source": "REMOTE_211"}
}
```
→ **PASS** ✓

### Test 2: value 필드 + timestamps
```bash
curl -X POST ... -d '{"category":"cross_msg_REMOTE_114_AADS_MGR","key":"test_sf_001","value":{"message_type":"task_result","project":"ShortFlow","task_id":"SF-TEST-001","status":"completed","title":"114 연동 테스트","summary":"114서버 ShortFlow 작업 결과","started_at":"2026-03-05T10:00:00+09:00","completed_at":"2026-03-05T10:20:00+09:00"}}'
```
결과:
```json
{
  "status": "ok",
  "task_upsert": {"status": "ok", "project": "ShortFlow", "task_id": "SF-TEST-001", "source": "REMOTE_114"}
}
```
→ **PASS** ✓

### DB 확인
```sql
SELECT project, task_id, status, title, started_at, completed_at, source
FROM project_tasks
WHERE task_id IN ('KIS-TEST-001','SF-TEST-001')
ORDER BY created_at DESC;
```
결과:
```
 project  |   task_id    |  status   |      title       |    started_at          |   completed_at         | source
-----------+--------------+-----------+------------------+------------------------+------------------------+--------------
 KIS       | KIS-TEST-001 | completed | 테스트 작업      |                        |                        | REMOTE_211
 ShortFlow | SF-TEST-001  | completed | 114 연동 테스트  | 2026-03-05 01:00:00+00 | 2026-03-05 01:20:00+00 | REMOTE_114
```
→ **PASS** ✓ (started_at/completed_at KST→UTC 자동 변환 정상)

---

## Docker 배포

```
Container aads-server Recreated → Started (healthy)
Container aads-postgres Running (healthy)
```

---

## Git 커밋

### aads-server
```
SHA: 6c042bb
feat(T-091): 원격 에이전트 task_result 자동수집 + project_tasks upsert
pushed to origin/main
```

### aads-docs
```
SHA: 47f6523
feat(T-091): HANDOVER v5.20 — 원격 에이전트 task_result 자동수집 + project_tasks upsert
pushed to origin/main
```

---

## 미완료 사항

- **211/114 SSH 배포**: Connection timed out으로 원격 서버 직접 수정 불가
  - 업데이트된 스크립트: `/root/aads/scripts/aads_remote_agent.py`
  - SSH 복구 후 배포 및 systemd 재시작 필요
  - 백업 명령: `ssh REMOTE_211 "cp /root/aads_remote_agent.py /root/aads_remote_agent.py.bak.T091"`

---

## 결론

T-091 주요 3파트 중 68서버 관련 작업 완료:
1. **원격 에이전트 업데이트** (scripts/): auto_report_task_results() — 프로젝트 디렉토리 24h 변경 파일 감지 + task_result 자동 전송
2. **context.py**: data/value 양방향 허용 + 타임스탬프 파싱 수정 → curl 검증 HTTP200+DB upsert PASS
3. **project_tasks 테이블**: schema.sql DDL 추가, DB 실존 확인 (REMOTE_211/REMOTE_114 KST 타임스탬프 정상 저장)

211/114 서버 직접 배포는 SSH 연결 불가로 지연. 스크립트 준비 완료 상태.
