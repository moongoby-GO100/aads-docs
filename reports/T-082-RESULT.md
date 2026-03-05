---
project: AADS
task_id: T-082
status: success
completed_at: 2026-03-05T18:54:00+09:00
---

# T-082: classify_project 전면 재작성 결과 보고서

## 작업 개요
- **Task ID**: T-082
- **제목**: classify_project 전면 재작성 — 파일명 접두사 + AADS 인프라 1순위 + 한글 오분류 제거
- **파일**: `/root/aads/aads-server/app/api/project_dashboard.py`
- **커밋**: `2179ff9` — fix(T-082): classify_project 전면 재작성 — 3단계 분류 + 한글 오분류 제거

## 변경 내용

### 1. `_classify_project()` 전면 재작성 (3단계 분류)
기존 로직(AADS 인프라 키워드 최우선)을 다음 순서로 변경:

| 단계 | 조건 | 결과 |
|------|------|------|
| 1단계 | 파일명 접두사: KIS_, GO100_, SF_, NT_, SALES_, NAS_ | 해당 프로젝트 |
| 2단계 | AADS 인프라 키워드 매칭 (dashboard, bridge, handover 등 30+ 키워드) | AADS |
| 3단계 | 프로젝트 고유 키워드 매칭 (KIS, GO100, ShortFlow, NewTalk, NAS, SALES) | 해당 프로젝트 |
| 기본값 | — | AADS |

SALES_ 파일명 접두사 지원 추가 (기존 없음).

### 2. `validate_project_name()` 함수 추가
허용 프로젝트명: `AADS, KIS, GO100, ShortFlow, NewTalk, NAS, SALES, aads-server, aads-dashboard`

비허용값(한글 문장, 50자 초과 등)은 자동으로 `AADS`로 대체.

### 3. 한글 오분류 버그 수정
- **title 파싱**: 50자 초과 시 파일명 사용 (description으로 취급)
- **project 파싱**: `validate_project_name()` 래핑으로 비정상 프로젝트명 차단
  - YAML 프런트매터의 `project:` 필드 적용
  - 일반 텍스트의 `프로젝트:` 필드 적용 (50자 이하 제한 유지)
  - `_parse_directive_file()` 및 `_parse_report_file()` 양쪽 적용

### 4. analytics API by_project 통합
- `by_project_dir`: `validate_project_name()` 적용
- `by_project_conv`: `validate_project_name(CONV_PROJECT_MAP.get(...))` 적용
- directives/reports 엔드포인트 `by_project` 집계에도 `validate_project_name()` 적용

## 검증 결과

### health check
```
HTTP 200 OK
```

### directives AADS 프로젝트 카운트
```bash
curl -s https://aads.newtalk.kr/api/v1/dashboard/directives | python3 -m json.tool | grep -c '"project": "AADS"'
# 결과: 172
```

### analytics by_project (한글 문장 0건 확인)
```bash
curl -s https://aads.newtalk.kr/api/v1/dashboard/analytics | python3 -c "..."
# 결과:
# AADS 97
# KIS 2
# (한글 문장 프로젝트명 0건 확인)
```

## Git 정보
- **커밋 SHA**: `2179ff9`
- **커밋 URL**: https://github.com/moongoby-GO100/aads-server/commit/2179ff9
- **브랜치**: main
- **변경**: 1 file changed, 186 insertions(+), 55 deletions(-)
