# T-060 결과 보고서: JS 에러 5건 수정 + 진행률 표시 + 사용자 프로젝트 생성 화면 구현

- 완료일: 2026-03-05
- 담당: Claude Code (claude-sonnet-4-6)
- 커밋: 3e17ec8 (aads-dashboard)

## 수정 내용

### 1. Dashboard (/) — y.slice 에러 수정
- `r.alerts ?? r ?? []` → `Array.isArray(r.alerts) ? r.alerts : []`
- `r.decisions ?? r ?? []` → `Array.isArray(r.data) ? r.data : Array.isArray(r.decisions) ? r.decisions : []`
- `p.progress ?? 0` → `p.progress_percent ?? 0` (2곳)

### 2. CEO Decisions (/decisions) — e.map 에러 수정
- `r.decisions ?? r.results ?? r ?? []` → `Array.isArray(r.data) ? r.data : Array.isArray(r.decisions) ? r.decisions : []`
- `r.alerts ?? r ?? []` → `Array.isArray(r.alerts) ? r.alerts : []`

### 3. Project Status (/project-status) — 진행률 0% 수정
- `p.progress ?? 0` → `p.progress_percent ?? 0` (2곳)

### 4. Pipeline (/projects) — 전체 재구현
- 프로젝트 생성 폼 (textarea + 생성 버튼)
- api.getProjects() 목록 조회
- 10초 자동 갱신 (setInterval)
- 각 카드: project_id, checkpoint_stage, progress_percent, current_agent, total_cost_usd
- 자동 실행 버튼 (api.autoRunProject)
- 승인/반려 버튼 (api.resumeProject, checkpoint_stage=interrupted일 때)
- 비용 상세 토글 (api.getProjectCosts)
- 빈 상태 메시지

## 검증 결과
- npm build: 0 에러 ✅
- Docker 빌드: 성공 ✅
- HTTP 200: / /project-status /conversations /managers /decisions /settings /projects 모두 200 ✅
