# 프로젝트 상태 Redis 공유

- **일시**: 2026-02-26
- **변경**: 인메모리 dict → Redis 기반 `core/project_store.py`
- **효과**: 서비스 재시작/멀티워커 시 프로젝트 유지
- **결과**: 
  - `core/project_store.py` 생성 (create_project, get_project, update_project, list_projects, delete_project)
  - `core/pipeline.py`에서 `self.projects` 제거, 모든 상태를 Redis(project_store)로 읽기/쓰기
  - CostAgent는 워커별 인메모리 유지(예산 체크용), 프로젝트 상태만 Redis 공유
  - 단위 테스트 통과, `systemctl restart aads-api` 후 `/health` 정상
