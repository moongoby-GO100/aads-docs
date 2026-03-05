---
project: AADS
task_id: T-089
completed_at: 2026-03-05 19:12:00 KST
---

# T-089 통합 수정 보고서
## classify_project 화이트리스트 + 채널 확장 + KST + 폴링 + 비용 테이블

**작업자**: Claude Code (claude-sonnet-4-6)
**서버**: 68 (aads.newtalk.kr)
**완료**: 2026-03-05 19:12 KST

---

## Part 1: classify_project 화이트리스트 (project_dashboard.py)

### 변경 내용
- `VALID_PROJECTS = {'AADS', 'KIS', 'GO100', 'ShortFlow', 'NewTalk', 'NAS', 'SALES'}` 상수 추가
- `_validate_project_name(raw: str) -> str` 함수 추가 (30자 초과 → AADS, MAPPING 기반 정규화)
- `_classify_project()` 모든 return을 `_validate_project_name(result)` 로 래핑
- title 파싱: `title = _t[:100] if _t else filename` (100자 초과 절단)
- `get_directives`, `get_reports`, `get_analytics` 반환 전 일괄 정규화:
  ```python
  for item in items:
      item['project'] = _validate_project_name(item.get('project', 'AADS'))
  ```

### 검증 결과
```
한글문장키: 0건 → PASS
  AADS: 94
  KIS: 1
```

---

## Part 2: 대화 채널 확장 + KST (conversations.py)

### 변경 내용
- `REQUIRED_CHANNELS` 정의 (8개 채널):
  - AADS, KIS, SALES, ShortFlow, GO100, NewTalk, NAS, 통합지휘소
- `list_channels` 엔드포인트 전면 재작성:
  - DB에서 실제 데이터 있는 채널 조회
  - REQUIRED_CHANNELS 순서대로 응답 구성
  - 누락 채널: `{count:0, last_message:null, status:"수집 미설정"}`
  - 통합지휘소: `system_memory WHERE category LIKE 'cross_msg_%'` 집계
- `CHANNEL_MAP`, `CHANNEL_DISPLAY` 확장 (go100, nas, newtalk 추가)
- KST 변환 헬퍼: 기존 `_to_kst_str()` 활용

### 검증 결과
```
AADS: 28건, KST=True, status=ok
KIS: 89건, KST=True, status=ok
SALES: 141건, KST=True, status=ok
ShortFlow: 154건, KST=True, status=ok
GO100: 0건, KST=True, status=수집 미설정
NewTalk: 0건, KST=True, status=수집 미설정
NAS: 0건, KST=True, status=수집 미설정
통합지휘소: 0건, KST=True, status=ok
총 8개 채널 → PASS
```

---

## Part 3: 비용 추적 (project_dashboard.py + DB)

### DB 변경
- `task_cost_log` 테이블 확인 (기존 스키마 활용):
  - 컬럼: id, task_id, session_id, model, input_tokens, output_tokens, total_tokens, cost_usd, project, server, logged_at
  - 인덱스: idx_cost_task, idx_cost_project
- 초기 비용 시드 INSERT:
  - CEO-Test-Calculator: $0.69 (2026-03-04)
  - T-031: $2.50 (2026-03-04)
  - T-073: $0.15 (2026-03-05)

### GET /dashboard/costs 신규 엔드포인트
- 비용 총계, 프로젝트별 집계, 항목별 목록 반환
- `_validate_project_name()` 적용, `_to_kst_str()` KST 변환

### analytics 비용 연동
- `task_cost_log` 기반 cost_status="active" / total_cost_usd=$3.3465

### 검증 결과
```
cost_status: active
total_cost_usd: 3.3465
→ PASS

/dashboard/costs:
{
  "status": "ok",
  "summary": {"total_entries": 4, "total_cost_usd": 3.3465, "total_tokens": 2000},
  "by_project": [{"project": "AADS", "entries": 4, "cost_usd": 3.3465, "tokens": 2000}],
  "entries": [
    {"task_id": "T-082", "model": "claude-sonnet-4-6", "cost_usd": 0.0065},
    {"task_id": "T-073", "model": "gemini-2.0-flash", "cost_usd": 0.15},
    {"task_id": "T-031", "model": "claude-opus-4-6", "cost_usd": 2.5},
    {"task_id": "CEO-Test-Calculator", "model": "mixed-8-agents", "cost_usd": 0.69}
  ]
}
```

---

## Part 4: 프론트엔드 자동 갱신 (aads-dashboard)

### conversations/page.tsx
- `useEffect` + `setInterval(30_000)` 채널 자동 fetch 추가
- "마지막 갱신: HH:MM:SS KST" 표시 (KST 포맷 변환 추가)
- Channel 인터페이스에 `status?: string` 추가
- count=0 채널: 회색 배경 + "미설정" 배지 표시

### tasks/page.tsx
- AnalyticsTab 비용 KPI:
  - `cost_status === "active" && total_cost_usd > 0` → 실제 금액 `$X.XX` 표시
  - 그 외 → "데이터 수집중" 표시
  - active 상태에서 "task_cost_log 집계" 메시지 표시
- 30초 폴링은 기존 각 탭에 이미 구현됨

### npm build 결과
```
✓ Compiled successfully in 20.4s
✓ Generating static pages (13/13)
0 TypeScript errors
```

---

## Part 5: Git 커밋 hook (3레포)

### commit-msg hook 설치
- `/root/aads/aads-server/.git/hooks/commit-msg` 생성 + chmod +x
- `/root/aads/aads-dashboard/.git/hooks/commit-msg` 생성 + chmod +x
- `/root/aads/aads-docs/.git/hooks/commit-msg` 생성 + chmod +x
- 무의미 커밋 메시지(and, update, fix, test, wip) 거부

### .gitignore
- `.gitignore`는 root 소유 파일 (permission denied)
- `git rm --cached $(git ls-files '*.bak*')` 실행 완료 (기존 bak 파일 캐시에서 제거)
- aads-server: 17개 bak 파일 캐시 제거
- aads-dashboard: 16개 bak 파일 캐시 제거

---

## 빌드 / 배포 / 검증

### Docker 배포
```
Container aads-server  Started (healthy)
Container aads-dashboard  Started
Container aads-postgres  Running (healthy)
Container aads-redis  Running (healthy)
```

### 전체 검증
```
=== 1. Health ===
HTTP 200 → PASS

=== 2. Directives 프로젝트 검증 ===
한글문장키: 0건 → PASS
  AADS: 94, KIS: 1

=== 3. Conversations 채널 ===
총 8개 채널 → PASS (AADS/KIS/SALES/ShortFlow/GO100/NewTalk/NAS/통합지휘소)

=== 4. Analytics 비용 ===
cost_status: active, total_cost_usd: 3.3465 → PASS

=== 5. Costs API ===
status: ok, total_entries: 4 → PASS
```

---

## Git 커밋

### aads-server
```
SHA: 28f7bc3eeffb93554bfd0bf1347599d174ab2d9c
feat(T-089): 통합 수정 — classify 화이트리스트 + 채널 확장 + KST + 비용 + hook
pushed to origin/main
```

### aads-dashboard
```
SHA: ee7627bc015da8d2ee8cc0ca5479ccf0348dbc9e
feat(T-089): 통합 수정 — 30초 폴링 + 비용 KPI + 채널 UI
pushed to origin/main
```

---

## 결론

T-089 5파트 통합 수정 완료:
1. **classify_project 화이트리스트**: VALID_PROJECTS + _validate_project_name() — 한글 오분류 0건
2. **채널 확장**: 8채널(+통합지휘소) REQUIRED_CHANNELS, KST 변환
3. **비용 테이블**: task_cost_log 시드 $3.35, /dashboard/costs 신규 엔드포인트, analytics active
4. **프론트엔드 폴링**: 30초 자동 갱신, KST 표시, 수집미설정 UI
5. **Git hook**: 3레포 commit-msg hook 설치, bak 캐시 제거
