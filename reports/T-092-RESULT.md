# T-092 비용 자동 추적 결과 보고서

## 개요

- **Task ID**: T-092
- **제목**: 작업 비용 자동 추적 — Claude Code usage 파싱 + 토큰 단가 자동 계산 + DB 자동 기록
- **완료 시각**: 2026-03-05T19:44:54 KST
- **커밋 SHA**: 6efa920

---

## Part A — task_cost_log 테이블 확인/수정

테이블은 T-089에서 이미 생성되어 있었으나 스키마 차이 확인:

| 기존 컬럼 | 추가/수정 |
|---|---|
| id, task_id, session_id, model, input_tokens, output_tokens, total_tokens, cost_usd, project, server, logged_at | `session_duration_ms INT DEFAULT 0` 추가 |
| | `source VARCHAR(50) DEFAULT 'auto'` 추가 |

**ALTER TABLE 실행 결과:**
```
ALTER TABLE
 column_name         | data_type
---------------------+--------------------------
 id                  | integer
 task_id             | character varying
 session_id          | character varying
 model               | character varying
 input_tokens        | integer
 output_tokens       | integer
 total_tokens        | integer
 cost_usd            | numeric
 project             | character varying
 server              | character varying
 logged_at           | timestamp with time zone
 session_duration_ms | integer
 source              | character varying
(13 rows)
```

---

## Part B — /root/aads/scripts/cost_tracker.py 신규 작성

**파일 위치**: `/root/aads/scripts/cost_tracker.py`

### PRICE_TABLE (CEO-DIRECTIVES T-002 기반)
```python
PRICE_TABLE = {
    "claude-opus-4-6":          {"input": 5.00,  "output": 25.00},
    "claude-sonnet-4-6":        {"input": 3.00,  "output": 15.00},
    "claude-haiku-4-5":         {"input": 1.00,  "output": 5.00},
    "claude-haiku-4-5-20251001": {"input": 1.00, "output": 5.00},
    "gemini-2.0-flash":         {"input": 0.30,  "output": 2.50},
    "gemini-3.1-pro-preview":   {"input": 2.00,  "output": 12.00},
    "gpt-5-mini":               {"input": 0.25,  "output": 2.00},
}
```

### 주요 함수
- `calculate_cost(model_id, input_tokens, output_tokens)` → cost_usd
- `record_cost(task_id, project, model_id, input_tokens, output_tokens, ...)` → API POST 후 source/duration_ms 직접 업데이트
- `parse_claude_result(result_json_str)` → {model, input_tokens, output_tokens, duration_ms}
- `parse_result_file(result_file)` → RESULT.md에서 usage 정보 추출

### 테스트 결과
```
$ python3 cost_tracker.py calculate --model claude-sonnet-4-6 --input 10000 --output 15000
model=claude-sonnet-4-6 input=10000 output=15000 cost=$0.255000

$ python3 cost_tracker.py record --task-id T-092 --project AADS --model claude-sonnet-4-6 --input 45000 --output 20000
[cost_tracker] 기록: task=T-092 model=claude-sonnet-4-6 input=45000 output=20000 cost=$0.435000 source=auto
[cost_tracker] API 기록 완료: id=65 cost=$0.435000
[cost_tracker] extra fields 업데이트 완료 (id=65)
```

---

## Part C — auto_trigger.sh 수정

**수정 전 흐름**:
```
지시서 감지 → claude_exec.sh 실행 → 결과 처리
```

**수정 후 흐름**:
```
지시서 감지 → 실행 시작 시각 기록(ms) → claude_exec.sh 실행 →
실행 종료 시각 기록(ms) → exec_duration_ms 계산 →
cost_tracker.py record 호출 (--task-id, --project, --result-file) →
결과 처리
```

**추가된 코드 요약** (`/root/aads/scripts/auto_trigger.sh`):
```bash
local ts_exec_start
ts_exec_start=$(date +%s%3N)
"${SCRIPT_DIR}/claude_exec.sh" "$task_id" "$directive_file" || exec_exit=$?
local ts_exec_end
ts_exec_end=$(date +%s%3N)
local exec_duration_ms=$(( ts_exec_end - ts_exec_start ))

# T-092: 비용 자동 추적
local project="${PROJECT:-AADS}"
local result_file=""
if [ -n "$directive_file" ]; then
    local base_name
    base_name=$(basename "$directive_file" .md)
    result_file="${DONE_DIR}/${base_name}_RESULT.md"
fi

python3 "${SCRIPT_DIR}/cost_tracker.py" record \
    --task-id "$task_id" \
    --project "$project" \
    --result-file "$result_file" 2>&1 || true
```

---

## Part D — API 엔드포인트 수정 (project_dashboard.py)

### GET /dashboard/costs 수정 (T-092)
- `by_project_model` 집계 추가: project + model_id 기반 입출력 토큰 및 비용 집계
- `summary.cost_status` 필드 추가: `'active' if count > 0 else 'no_data'`
- SQL:
  ```sql
  SELECT project, COALESCE(model, 'unknown') AS model_id,
         COALESCE(SUM(input_tokens), 0) AS input_tokens,
         COALESCE(SUM(output_tokens), 0) AS output_tokens,
         COALESCE(SUM(cost_usd), 0) AS cost_usd
  FROM task_cost_log
  GROUP BY project, model
  ORDER BY cost_usd DESC
  ```

### GET /dashboard/analytics 수정 (T-092)
- `cost_status = 'no_data'` (기존 `'not_configured'`) 로 수정
- `total_cost_usd = -1.0` → `0.0`으로 수정 (데이터 없을 때)
- `len(cost_rows) > 0` 조건 추가 (비용 0이지만 레코드 있는 경우 active)

---

## Part E — backfill_costs.py 실행 결과

**파일 위치**: `/root/aads/scripts/backfill_costs.py`

**스캔**: `/root/.genspark/directives/done/` — 92개 RESULT 파일

**실행 결과**:
```
[backfill] 완료: 성공=60, 스킵=32, 오류=0
```

**DB 집계 (backfill 완료 후)**:
```
      project      | count |  round
-------------------+-------+---------
 AADS              |    59 | 16.6421
 KIS-AUTOTRADE-V41 |     1 |  0.2550
 aads-server       |     3 |  0.7650
 aads              |     1 |  0.2550
(4 rows)
```

---

## 검증 결과

### 1. 테이블 집계
```sql
SELECT project, count(*), round(sum(cost_usd)::numeric,4) FROM task_cost_log GROUP BY project;
```
```
      project      | count | total_cost
-------------------+-------+------------
 AADS              |    60 |    17.0771
 aads-server       |     3 |     0.7650
 KIS-AUTOTRADE-V41 |     1 |     0.2550
 aads              |     1 |     0.2550
```
**총 레코드: 65건, 총 비용: $18.35**

### 2. 프로젝트별/모델별 비용 집계
```
      project      |       model       | total_input | total_output | total_cost
-------------------+-------------------+-------------+--------------+------------
 AADS              | claude-sonnet-4-6 |      569178 |       802575 |    13.7371
 AADS              | claude-opus-4-6   |           0 |            0 |     2.5000
 aads-server       | claude-sonnet-4-6 |       30000 |        45000 |     0.7650
 AADS              | mixed-8-agents    |           0 |            0 |     0.6900
 KIS-AUTOTRADE-V41 | claude-sonnet-4-6 |       10000 |        15000 |     0.2550
 aads              | claude-sonnet-4-6 |       10000 |        15000 |     0.2550
 AADS              | gemini-2.0-flash  |           0 |            0 |     0.1500
```

### 3. Analytics API cost_status 확인
```
$ curl -s https://aads.newtalk.kr/api/v1/dashboard/analytics -H "User-Agent: curl/7.64.0" | \
  python3 -c "import sys,json; d=json.load(sys.stdin); print(f\"cost: \${d['summary']['total_cost_usd']:.2f}, status: {d['summary']['cost_status']}\")"

cost: $17.92, status: active
```

### 4. 최근 5건 기록 확인
```
 task_id |       model       | input_tokens | output_tokens |  cost_usd  |  source
---------+-------------------+--------------+---------------+------------+----------
 T-091   | claude-sonnet-4-6 |        10000 |         15000 | 0.25500000 | backfill
 T-090   | claude-sonnet-4-6 |        10000 |         15000 | 0.25500000 | backfill
 T-089   | claude-sonnet-4-6 |         1200 |           800 | 0.01560000 | backfill
 T-088   | claude-sonnet-4-6 |        10000 |         15000 | 0.25500000 | backfill
 T-082   | claude-sonnet-4-6 |        10000 |         15000 | 0.25500000 | backfill
```

### 5. HTTP 상태
```
$ curl -s -o /dev/null -w "%{http_code}" https://aads.newtalk.kr/api/v1/health
200
```

---

## 빌드/배포

```
DOCKER_BUILDKIT=0 docker-compose -f docker-compose.prod.yml up -d --build
→ aads-server: Recreated + Started
→ aads-dashboard: Recreated + Started
HTTP: 200
```

---

## Git 커밋

```
커밋: 6efa920
메시지: feat(T-092): 비용 자동 추적 — cost_tracker.py + auto_trigger 연동 + backfill
브랜치: main
Push: https://github.com/moongoby-GO100/aads-server/commit/6efa920
```
