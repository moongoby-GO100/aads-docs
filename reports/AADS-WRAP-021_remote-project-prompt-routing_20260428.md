---
doc_id: AADS-WRAP-021
project: AADS
type: wrap
status: active
created_at: 2026-04-28
updated_at: 2026-04-28
owner_role: PromptEngineer
source_session: ce19b443
related_files:
  - /root/aads/aads-server/migrations/062_remote_project_access_contract.sql
  - /root/aads/aads-server/app/services/prompt_compiler.py
  - /root/aads/aads-server/app/services/chat_service.py
  - /root/aads/aads-server/app/services/tool_executor.py
  - /root/aads/aads-server/app/api/ceo_chat_tools.py
  - /root/aads/aads-server/app/services/tool_registry.py
  - /root/aads/aads-server/scripts/claude_relay_server.py
  - /root/aads/aads-dashboard/src/app/chat/page.tsx
related_routes:
  - /chat
  - /admin/prompts
tags:
  - prompt-assets
  - remote-project
  - go100
  - tool-routing
  - codex
---

# AADS 원격 프로젝트 프롬프트·도구 라우팅 적용 완료 보고서

- 작성일: 2026-04-28
- 대상 프로젝트: AADS, GO100, KIS, SF, NTV2
- 목적: GO100 등 외부 프로젝트 채팅 세션에서 실제 서버 코드와 프로젝트 DB를 확인하지 못하고 AADS 기본 컨텍스트로 작업하는 문제 방지

## 1. 핵심 결론

GO100 세션의 서버·코드·DB 접근 문제는 매핑 부재가 아니라 세션 프롬프트, 역할/모델 라우팅, Codex/SDK 실행 경로, 도구 파라미터 보정이 한 흐름으로 묶이지 않은 구조 문제였다. 이번 조치로 기존 세션도 새 대화턴마다 최신 `prompt_assets`를 다시 조립하고, 원격 프로젝트 도구 호출에서 `project`가 누락되면 세션/워크스페이스 기준으로 자동 보정한다.

## 2. DB 적용

`migrations/062_remote_project_access_contract.sql`을 적용했다.

- `project-remote-access-contract`: KIS/GO100/SF/NTV2 세션에서 코드, DB, 서버 상태, 오류, 개발, 수정, 배포 요청 시 추정 답변 금지와 원격 도구 우선 사용을 지시한다.
- `project-go100-context`: GO100은 211 서버 `/root/kis-autotrade-v4`를 사용하며, DB는 `query_project_database(project='GO100')`로 확인하도록 지시한다.
- `intent-remote-code-db-preflight`: 코드/DB/원격 실행/Runner 인텐트에서 실제 파일, 쿼리, 로그 근거를 먼저 확보하도록 지시한다.

각 자산은 400자 이상 실행 지침으로 보강했고, `pipeline_runner` 인텐트까지 매칭되도록 확장했다.

## 3. 세션·워크스페이스 반영

GO100/KIS/SF/NTV2 워크스페이스 `settings`에 다음 값을 저장했다.

- `project_key`
- `server_profile`
- `workdir`
- `db_profile`

기존 세션도 워크스페이스를 통해 다음 턴부터 이 값을 참조한다. GO100은 `project_key=GO100`, `workdir=/root/kis-autotrade-v4`, `db_profile=GO100`으로 반영했다.

## 4. 코드 적용

### 4.1 PromptCompiler

Layer 3 역할과 Layer 5 모델 매칭을 강화했다. 세션의 `role_key`, 선택 모델, 실제 실행 모델, provider/family/category/capability 토큰이 `prompt_assets.target_models` 매칭에 사용된다.

### 4.2 채팅 실행 경로

일반 채팅, Autonomous/Runner, Agent SDK 직접 실행 경로 모두 실행 직전에 `PromptCompiler`를 거치도록 정리했다. 특히 “직접 해”, “직접 수정” 같은 Agent SDK 경로도 최신 시스템 프롬프트를 SDK `system_prompt`로 전달하게 보완했다.

### 4.3 도구 라우팅

`read_remote_file` 호출에서 `file_path`와 `path`를 모두 받을 수 있게 했다. 또한 `read_remote_file`, `query_project_database`, `run_remote_command`, `pipeline_runner_submit` 등 프로젝트형 도구에서 `project`가 빠지면 세션/워크스페이스 기준으로 자동 보정한다.

### 4.4 Codex Relay

Codex relay가 항상 AADS로 실행되던 문제를 보완했다. 채팅 세션의 프로젝트를 relay 요청에 전달하고, relay는 active project hint와 프로젝트별 cwd/fallback을 사용한다. GO100/KIS/SF/NTV2는 로컬 AADS 코드베이스가 아니라 프로젝트 전용 SSH/MCP 도구를 사용하도록 힌트를 주입한다.

### 4.5 UI

채팅 화면에 세션 역할 선택값을 저장/수정하는 UI를 추가했다. 새 세션 생성 시 `role_key`가 함께 저장되고, 기존 세션도 역할 변경 후 다음 턴부터 Layer 3 매칭에 반영된다.

## 5. 검증

- `py_compile` 통과: `chat_service.py`, `agent_sdk_service.py`, `tool_executor.py`, `ceo_chat_tools.py`, `tool_registry.py`, `seed_prompt_assets.py`
- DB 적용 완료: `project-remote-access-contract`, `project-go100-context`, `intent-remote-code-db-preflight`
- `GO100 + pipeline_runner` PromptCompiler dry-run에서 GO100 컨텍스트와 원격 코드·DB preflight 자산 매칭 확인
- GO100 세션에서 `project` 없는 `read_remote_file` 입력이 `project=GO100`으로 자동 보정되는 것 확인
- `aads-server` 재시작 후 `/api/v1/health` 정상

## 6. 운영 기준

앞으로 원격 프로젝트 채팅 세션에서 코드/DB/서버 작업을 요청받으면 다음 순서가 기본이다.

1. 현재 세션의 `project_key`를 active project로 해석한다.
2. 코드 확인은 `list_remote_dir` 또는 `read_remote_file`로 시작한다.
3. 프로젝트 DB 확인은 `query_database`가 아니라 `query_project_database`를 사용한다.
4. 수정/배포는 근거 파일, 쿼리, 로그를 확보한 뒤 `patch_remote_file` 또는 `pipeline_runner_submit`을 선택한다.
5. 확인하지 못한 내용은 완료 또는 문제 없음으로 단정하지 않는다.

## 7. 남은 과제

- Prompt asset 변경 dry-run 결과를 `/admin/prompts`에서 세션별로 더 쉽게 확인하는 UI 보강
- 프로젝트별 문서 저장 룰을 Documents API와 관리자 화면까지 연결
- 원격 프로젝트별 실제 DB connection health를 정기 점검 리포트로 저장
