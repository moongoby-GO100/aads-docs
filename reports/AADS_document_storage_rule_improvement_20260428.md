# AADS 문서 저장 룰 개선안 보고서

- 작성일: 2026-04-28
- 대상: `aads-docs`, `aads-server/docs`, `aads-dashboard/docs`, Documents API, 보고서/기획서/운영문서 저장 흐름
- 결론: 현재 문서 저장소와 일부 명명 규칙은 있으나, 문서 유형별 단일 저장 기준은 없다.

## 1. 현재 진단

현재 AADS에는 문서 저장 관련 규칙이 부분적으로만 존재한다.

- `aads-dashboard/README.md`는 `aads-docs`를 문서/HANDOVER 저장소로 설명한다.
- `aads-docs/README.md`는 `/reports`, `/architecture`, `/phase-reports`, `/specs` 같은 큰 디렉터리 구조를 정의한다.
- `aads-docs/shared/rules/flow-rules.md`는 FLOW 산출물 이름을 `{PROJECT}-FIND-{SEQ}`, `{PROJECT}-LAYOUT-{SEQ}`, `{PROJECT}-WRAP-{SEQ}`처럼 일부 정의한다.
- `aads-server/app/api/documents.py`는 CEO 문서를 `/root/aads/aads-docs/reports/ceo-documents/{DOC_ID}_{slug}.md`와 `system_memory(category=ceo_document)`에 이중 저장한다.
- 실제 repo에는 `aads-server/docs/reports`, `aads-dashboard/docs`, `aads-dashboard/reports`, `aads-server/reports`, `aads-core/reports`, `aads-docs/reports`가 공존한다.

따라서 현재는 “문서 저장 기능”은 있지만 “문서 저장 룰”은 통합되어 있지 않다. 이 상태에서는 같은 성격의 보고서가 서버 repo와 공식 docs repo에 나뉘어 쌓이고, 관리자 화면이나 검색 기능에서 누락될 가능성이 있다.

## 2. 문제점

첫째, source of truth가 불명확하다. 공식 문서 저장소는 `aads-docs`로 보이지만, 개발 중 생성되는 보고서는 각 코드 repo의 `docs/reports`에도 저장되고 있다.

둘째, 문서 유형과 저장 위치가 1:1로 매핑되어 있지 않다. 기획, 기술 설계, 장애 보고, 작업 결과, handover, prompt governance 문서가 모두 report 또는 docs라는 넓은 이름 아래 섞인다.

셋째, 파일명 규칙이 일관되지 않다. 일부는 `T-100-RESULT.md`, 일부는 `AADS-188A-REPORT.md`, 일부는 날짜 suffix, 일부는 날짜 prefix를 사용한다.

넷째, DB/파일 이중 저장 기준이 CEO 문서 API에만 있고, 일반 보고서나 자동 생성 보고서에는 동일한 index/update 규칙이 없다.

다섯째, 문서 생명주기가 없다. draft, active, superseded, archived 상태가 없어서 최신 문서와 과거 문서를 구분하기 어렵다.

## 3. 개선 원칙

문서 저장은 다음 원칙으로 통일한다.

1. 공식 보존 문서는 `aads-docs`를 source of truth로 둔다.
2. repo 내부 `docs/`는 해당 repo 코드와 강하게 결합된 기술 스펙, 실행 절차, API 문서만 둔다.
3. 사용자가 “보고서로 저장”을 요청한 산출물은 기본적으로 `aads-docs/reports`에 저장한다.
4. 특정 코드 변경과 함께 필요한 설계/운영 문서는 해당 repo `docs/`에 둘 수 있으나, 공식 보고 가치가 있으면 `aads-docs/reports`에 링크 또는 사본을 둔다.
5. 문서 metadata를 표준화해 관리자 화면, 검색, 세션 요약, memory 등록이 같은 기준으로 동작하게 한다.

## 4. 권장 저장 위치

| 문서 유형 | 저장 위치 | 예시 |
| --- | --- | --- |
| CEO/경영 보고서 | `aads-docs/reports/ceo-documents` | `STATUS-007_*.md` |
| 일반 기획/분석 보고서 | `aads-docs/reports` | `AADS-LAYOUT-020_prompt-governance.md` |
| 작업 완료/검증 보고서 | `aads-docs/reports` | `AADS-WRAP-201_admin-prompts-fix.md` |
| 프로젝트별 handover | `aads-docs/{PROJECT}-HANDOVER.md` 또는 `aads-docs/handover/` | `AADS-HANDOVER.md` |
| 공통 운영 규칙 | `aads-docs/shared/rules` | `DOCUMENT-STORAGE-RULES.md` |
| 공통 교훈 | `aads-docs/shared/lessons` | `L-009_prompt-layer-routing.md` |
| repo 전용 기술 문서 | `{repo}/docs` | `aads-server/docs/chat/CHAT-BACKEND-SPEC.md` |
| repo 전용 임시 조사 | `{repo}/docs/reports` 또는 `/tmp` | 배포 전 내부 분석 |
| API로 등록되는 문서 | `aads-docs/reports/ceo-documents` + `system_memory` | Documents API |

## 5. 파일명 규칙

기본 규칙은 다음과 같이 통일한다.

```text
{PROJECT}-{FLOW}-{SEQ}_{slug}_{YYYYMMDD}.md
```

- `PROJECT`: `AADS`, `KIS`, `GO100`, `NT`, `SF`, `KAKAOBOT`
- `FLOW`: `FIND`, `LAYOUT`, `OPERATE`, `WRAP`, `STATUS`, `TECH`, `RESEARCH`, `DIRECTIVE`
- `SEQ`: 프로젝트별 3자리 증가 번호 또는 기존 task id
- `slug`: 영문 kebab-case 권장, 필요한 경우 한글 허용
- `YYYYMMDD`: 작성일

예시:

- `AADS-LAYOUT-020_prompt-layer3-layer5-routing_20260428.md`
- `AADS-WRAP-021_admin-prompts-error-fix_20260428.md`
- `KIS-TECH-014_order-routing-architecture_20260428.md`

CEO Documents API의 기존 `PLAN-001`, `TECH-001`, `STATUS-001` 체계는 유지하되, project prefix가 필요한 문서는 `PLAN-AADS-001`, `TECH-KIS-001`처럼 확장한다.

## 6. Markdown metadata 표준

모든 공식 문서 상단에는 다음 metadata를 넣는다.

```yaml
---
doc_id: AADS-LAYOUT-020
project: AADS
type: layout
status: active
created_at: 2026-04-28
updated_at: 2026-04-28
owner_role: PromptEngineer
source_session: ce19b443
related_files:
  - /root/aads/aads-server/app/services/chat_service.py
related_routes:
  - /admin/prompts
  - /chat
tags:
  - prompt-assets
  - layer3
  - layer5
---
```

`status`는 `draft`, `active`, `superseded`, `archived` 중 하나로 둔다. 최신 운영 기준은 반드시 `active` 하나만 유지한다.

## 7. Documents API 개선안

현재 Documents API는 `plan`, `tech`, `research`, `status`, `directive` 타입만 지원한다. 다음 확장이 필요하다.

- type에 `layout`, `wrap`, `handover`, `rule`, `lesson`, `spec` 추가
- `project`, `status`, `owner_role`, `related_files`, `related_routes`, `supersedes`, `superseded_by` 필드 추가
- `_index.json`뿐 아니라 DB에도 동일 metadata 저장
- 파일 저장 위치를 type에 따라 분기하되 공식 문서는 기본 `aads-docs` 아래 저장
- repo-local 문서는 `canonical_doc_path`로 공식 문서와 연결

권장 DB 저장 category:

- `ceo_document`: CEO 문서
- `project_document`: 프로젝트 공식 문서
- `rule_document`: 운영 규칙
- `technical_spec`: 기술 스펙
- `wrap_report`: 완료/검증 보고서

## 8. 채팅/어드민 화면 반영안

채팅창에서 “보고서로 저장” 요청이 들어오면 저장 전 다음 값을 자동 결정한다.

- `project`: 현재 세션 project
- `role`: 현재 세션 role
- `type`: intent 기반 `layout`, `wrap`, `status`, `tech`, `research`
- `canonical_repo`: 기본 `aads-docs`
- `source_session`: 현재 session id
- `related_files`: 이번 대화에서 언급/수정한 파일

관리자 화면에는 다음 기능을 추가한다.

- 문서 저장 위치 미리보기
- 같은 slug 또는 같은 doc_id 중복 경고
- supersede 처리 버튼
- `aads-docs`와 repo-local docs 간 canonical 링크 표시
- prompt asset 보고서처럼 DB 반영이 필요한 문서는 관련 migration/asset 링크 표시

## 9. Prompt Layer 기획 보고서에 대한 적용 판단

방금 작성한 `20260428_PROMPT_LAYER3_LAYER5_MODEL_ROUTING_PLAN.md`는 코드 repo 내부 분석과 직접 연결되어 있어 `aads-server/docs/reports`에 저장한 것은 기술적으로 틀리지는 않다. 다만 공식 운영 기준으로 남길 문서이므로 최종본은 `aads-docs/reports`에도 저장하거나, 앞으로는 처음부터 `aads-docs/reports/AADS-LAYOUT-020_prompt-layer3-layer5-routing_20260428.md` 형태로 저장하는 것이 맞다.

즉, 현재 저장은 “개발 repo-local 보고서”이고, 앞으로 승인/운영 기준 문서는 “공식 docs 보고서”로 승격해야 한다.

## 10. 우선 적용 순서

1. `aads-docs/shared/rules/DOCUMENT-STORAGE-RULES.md` 신설
2. Documents API request/metadata schema 확장
3. 채팅 저장 기능에서 저장 위치 자동 결정
4. 기존 `aads-server/docs/reports`, `aads-dashboard/docs`, `aads-dashboard/reports` 문서 inventory 작성
5. 공식 보존 대상 문서를 `aads-docs`로 이동 또는 링크
6. admin 문서 화면에서 canonical path, status, supersede 표시

## 11. 최종 권고

문서 저장 룰은 현재 “있다/없다”로 보면 없다. 더 정확히는 저장소 구조와 일부 FLOW 명명 규칙은 있지만, 실제 운영자가 따라야 하는 통합 저장 정책은 부재하다.

앞으로는 `aads-docs`를 공식 source of truth로 정하고, repo 내부 `docs/`는 코드와 함께 버전 관리되어야 하는 기술 문서로 제한해야 한다. 채팅창과 어드민에서 저장되는 보고서는 metadata를 포함해 `aads-docs`에 저장하고, 필요 시 DB `system_memory`에는 검색/회상용 사본을 넣는 방식이 가장 안정적이다.
