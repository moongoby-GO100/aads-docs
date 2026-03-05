# AADS Handover v5.15
- 작성일: 2026-03-05 KST
- 작성: Claude Sonnet 4.6
- Task: T-074

## 변경 사항 (T-074: 분석탭 g.reduce 에러 수정 + classify_project 정확도 개선 + 시간 KST 변환)

### Part A - 분석탭 g.reduce 에러 수정

**원인**: `error_distribution` API 응답이 dict (`{memory_type: count}`)로 반환되어
프론트엔드의 `.reduce()` 호출 시 TypeError 발생.

**Backend 수정** (`project_dashboard.py`):
- `error_distribution` 반환 형식을 dict → array로 변경:
  ```python
  error_distribution = [
      {"error_type": r["memory_type"], "count": int(r["cnt"])}
      for r in cross_type_rows
  ]
  ```
- `avg_task_duration_min: 0.0` 을 summary에 추가 (프론트 `.toFixed(1)` 크래시 방지)

**Frontend 수정** (`page.tsx`):
- `Array.isArray` 가드 추가:
  ```typescript
  const dailyTrend = Array.isArray(data.daily_trend) ? data.daily_trend : [];
  const byProject = Array.isArray(data.by_project) ? data.by_project : [];
  const byServer = Array.isArray(data.by_server) ? data.by_server : [];
  const errorDist = Array.isArray(data.error_distribution) ? data.error_distribution : [];
  ```

### Part B - classify_project 정확도 개선

**파일**: `project_dashboard.py` → `_classify_project()` 함수

**수정 내용**: AADS 키워드를 1순위로 체크 후 다른 프로젝트 확인. 'nas', 'KIS' 단독 키워드 제거.
```python
def _classify_project(content: str) -> str:
    content_lower = content.lower()
    # 1순위: AADS (dashboard, handover, ceo chat 등 포함)
    aads_keywords = ['aads', 'dashboard', 'ceo chat', 'ceo 채팅', '대시보드', 'handover',
                     'tasks 페이지', 'task-history', 'project_dashboard',
                     'cost', '비용', '분석', 'remote', '원격', 'bridge', '브릿지',
                     'memory', 'context api', '계층 메모리', '모델 분기', '실행 엔진']
    if any(kw in content_lower for kw in aads_keywords):
        return 'AADS'
    # 2순위: 프로젝트별 정확 매칭
    if any(kw in content_lower for kw in ['kis-autotrade', 'kis_autotrade', '주식', 'autotrade', '백억이']):
        return 'KIS'
    if any(kw in content_lower for kw in ['shortflow', '쇼츠', 'shorts', '템빨', 'youtube short']):
        return 'ShortFlow'
    if any(kw in content_lower for kw in ['newtalk', '뉴톡', 'newtalk_v2']):
        return 'NewTalk'
    if any(kw in content_lower for kw in ['nasync', 'nas동기화']):
        return 'NAS'
    if any(kw in content_lower for kw in ['go100', 'go_100']):
        return 'GO100'
    return 'AADS'
```

**결과**: "CEO 채팅 v2" → AADS, "HANDOVER" → AADS 정확 분류

### Part C - 시간 KST 변환

**Backend** (`project_dashboard.py`):
- `KST = timezone(timedelta(hours=9))` 정의 (이미 있음)
- `_to_kst_str(dt)` 헬퍼 함수: datetime/문자열 → `+09:00` KST ISO 형식 변환
- `started_at`/`finished_at` (task history): `_to_kst_str()` 적용
- `last_report` (by_server): `_to_kst_str(lr_aware)` 적용
- 지시서 파일 시각: `datetime.fromtimestamp(mtime, tz=KST).strftime("%Y-%m-%dT%H:%M:%S+09:00")`

**Frontend** (`page.tsx`):
- `toKST(dateStr)` 헬퍼 함수 추가:
  ```typescript
  function toKST(dtStr: string | null | undefined, len = 16): string {
    if (!dtStr) return "-";
    const d = new Date(dtStr);
    const kst = d.toLocaleString("ko-KR", { timeZone: "Asia/Seoul", hour12: false });
    // 포맷: "2026-03-05 16:39"
  }
  ```
- DirectivesTab 시작/완료 컬럼: `toKST(d.started_at)`, `toKST(d.completed_at)`
- RemoteTab 시작/완료 컬럼: `toKST(t.started_at, 19)`, `toKST(t.finished_at, 19)`

### Part D - 빌드 및 배포

- `npm run build`: 성공 (0 에러, TypeScript 통과)
- Docker 이미지 빌드:
  - `aads-server-aads-server:latest` (SHA: 7bdae4ba8d37) ✅
  - `aads-server-aads-dashboard:latest` (SHA: 6268d681b1bf) ✅
- `docker compose up -d`: 성공
- `curl https://aads.newtalk.kr/tasks` → 200 ✅
- `curl https://aads.newtalk.kr/api/v1/health` → 200 ✅

### Part E - Git

- aads-server: `f9a7929` fix(T-074): classify_project accuracy + KST timezone
- aads-dashboard: `55e59ae` fix(T-074): analytics tab g.reduce fix + KST display
- aads-docs: handover-v5.15.md 작성

## 현재 시스템 상태

| 컴포넌트 | 상태 |
|----------|------|
| aads-server | healthy (port 8100) |
| aads-dashboard | healthy (port 3100) |
| aads-postgres | healthy (port 5433) |
| aads-redis | healthy |
| 분석탭 g.reduce 에러 | 수정됨 |
| classify_project | T-073→AADS, T-071→AADS 정확 분류 |
| 시간 KST 표시 | backend _to_kst_str + frontend toKST |

## 주요 파일 위치

| 파일 | 설명 |
|------|------|
| `/root/aads/aads-server/app/api/project_dashboard.py` | 백엔드 API (T-066~T-074 누적) |
| `/root/aads/aads-dashboard/src/app/tasks/page.tsx` | Tasks 페이지 (4탭 구조) |
| `/root/aads/aads-server/docker-compose.prod.yml` | 프로덕션 배포 설정 |
