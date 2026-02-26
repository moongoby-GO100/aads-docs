# Task 4.1: 멀티테넌트 프로젝트 분리

- **일시**: 2026-02-26
- **변경**: `project_store.py`에 user_id 기반 Redis Set 분리, pipeline·API 연동

## 변경 사항

| 파일 | 내용 |
|------|------|
| `aads-core/core/project_store.py` | `USER_PREFIX`(aads:user_projects:), create 시 user_id 저장 및 Set 등록, list_projects(user_id=), delete 시 사용자 Set 제거, get_user_project_count() |
| `aads-core/core/pipeline.py` | create_project(user_id=), list_projects(user_id=), store_get_user_project_count import |
| `aads-core/api/main.py` | POST/GET /projects, from-template에 optional_auth 연동, user_id 전달 |

## 기능

- **생성**: `create_project(..., user_id="anonymous")` — Redis 키 `aads:user_projects:{user_id}` Set에 project_id 추가
- **목록**: `list_projects(user_id=None)` — user_id 지정 시 해당 사용자 프로젝트만 반환
- **삭제**: `delete_project(pid)` — 사용자 Set에서도 제거
- **개수**: `get_user_project_count(user_id)` — 사용자별 프로젝트 수

## 결과

- 테스트: `create_project` / `list_projects(user_id=)` / `get_user_project_count` 검증 통과
- 기존 파이프라인 상태(PipelineState) 호환 유지(create_initial_state 사용)
- 적용: ✅ 소스 반영
- 배포: aads-api 재시작(activating)

## 비고

- AUTH_ENABLED=false 시 optional_auth는 `{"id": "anonymous"}` 반환 → 동일 사용자로 집계됨.
