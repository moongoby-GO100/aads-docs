# Task 5.3: AI 생성 코드 파일 저장

- **일시**: 2026-02-27 KST
- **Sprint**: 5 — Task 5.3

## 변경 사항

| 파일 | 내용 |
|------|------|
| core/code_store.py | 파일 저장/조회/ZIP/삭제 모듈 |
| api/main.py | /projects/{id}/files, /files/{path}, /download 엔드포인트 |
| core/pipeline.py | save_phase_output 연동 (단계 완료 시 자동 저장) |

## 저장 구조

```
/mnt/volume_sgp1_01/projects/{project_id}/{phase}/
  meta.json
  output.json   (또는 output.txt)
  code/         (code 키가 있을 때 개별 파일)
```

## API

- `GET /projects/{id}/files` — 파일 목록
- `GET /projects/{id}/files/{path}` — 파일 내용
- `GET /projects/{id}/download` — ZIP 다운로드

## 검증

| 항목 | 결과 |
|------|------|
| code_store.py 문법 | OK |
| import 테스트 | OK |
| API /projects/{id}/files | 8001 응답 정상 |
| health | healthy |

## 적용·배포

- 적용: ✅ aads-core 소스 반영
- 배포: ✅ aads-api 서비스에 반영됨 (이미 배포된 상태에서 검증 완료)

## 비고

- aads-core 레포는 Task 5.4 커밋에 code_store 포함되어 있어 별도 커밋 없음.
- 파이프라인 `run_phase` 완료 시 `save_phase_output(project_id, current, result)` 호출로 단계별 출력·코드 자동 저장.
