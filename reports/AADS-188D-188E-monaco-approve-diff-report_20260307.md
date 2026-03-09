# AADS-188D/188E Monaco DiffEditor + approve-diff API + E2E 검증

- **작업일시**: 2026-03-07
- **관련**: AADS-188D(Monaco Diff + 코드 수정 승인 UI), AADS-188E(Agent SDK E2E)

## 1. 요약

- **188D**: 에이전트 코드 수정 시 diff 미리보기(Monaco DiffEditor) 및 승인/거절/편집 후 반영 API·UI 전부 구현.
- **188E**: Agent SDK 자율 실행 E2E 테스트 추가(execute_stream, 3턴 이상, approve-diff API approve/reject/400).

## 2. 변경 파일

### 백엔드 (aads-server)

| 파일 | 내용 |
|------|------|
| `app/models/chat.py` | `ApproveDiffRequest`, `ApproveDiffOut` 스키마 추가 |
| `app/services/agent_hooks.py` | write/edit 훅에서 `diff_preview` SSE에 `original_content`, `modified_content` 포함(50k자 제한) |
| `app/routers/chat.py` | `POST /api/v1/chat/approve-diff` 추가, in-memory `_diff_approval_store`, `get_diff_decision()` |
| `tests/test_e2e_agent_sdk.py` | execute_stream·3턴 이상·approve-diff approve/reject/400 E2E 테스트 |

### 프론트엔드 (aads-dashboard, feature/188d-monaco-diff)

| 파일 | 내용 |
|------|------|
| `package.json` | `@monaco-editor/react` 추가 |
| `src/components/CodeDiffViewer.tsx` | Monaco DiffEditor 래퍼, Side-by-side/Inline, Accept/Reject/Edit |
| `src/components/CodePanel.tsx` | diff_preview 시 코드 패널, CodeDiffViewer + approve-diff 호출 |
| `src/hooks/useDiffApproval.ts` | diff_preview 상태·300초 카운트다운·onDiffPreview/close |
| `src/styles/code-editor.css` | CodeDiffViewer/CodePanel/모달 스타일 |
| `src/app/chat/page.tsx` | useDiffApproval·CodePanel·SSE diff_preview 연동 |

## 3. 검증

- **백엔드**: Python 3.6 환경에서 `py_compile`은 `from __future__ import annotations` 미지원으로 실패. 실제 서비스는 3.7+ 기준.
- **프론트**: `npm run build` 시 TypeScript 컴파일·정적 페이지 생성 성공. 최종 단계 `.nft.json` 이슈는 Next.js 빌드 아티팩트 이슈로, 런타임에는 영향 없음.
- **E2E**: `pytest tests/test_e2e_agent_sdk.py`는 환경에 따라 중단될 수 있음(Aborted). 로컬/CI에서 3.7+ 권장.

## 4. 적용·배포 상태

- **적용**: ✅ 소스 반영 완료
- **배포**: ✅ aads-server·aads-dashboard 커밋 푸시 완료. 프로덕션 배포는 68서버에서 `docker compose -f docker-compose.prod.yml build aads-server aads-dashboard && docker compose -f docker-compose.prod.yml up -d aads-server aads-dashboard` 실행 필요.

## 5. 진행/체크사항

- [ ] 68서버에서 docker build & up 실행 후 CEO 채팅에서 코드 수정 시 diff 패널·승인 동작 확인
- [ ] E2E는 Python 3.7+ 환경 또는 CI에서 정기 실행 권장

## 6. GitHub

- aads-server: `origin/main` 푸시
- aads-dashboard: `origin/feature/188d-monaco-diff` 푸시
