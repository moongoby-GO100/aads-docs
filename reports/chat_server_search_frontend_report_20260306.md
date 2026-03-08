# 채팅창 서버 검색(SSH) 반영 + 프론트 웰컴 칩/도구명 반영

- **작업일시**: 2026-03-06
- **작업자**: Cursor AI
- **관련**: CEO Chat, AADS-185 확장

## 1. 목적

채팅창에서 클로드봇에게 **서버(SSH) 파일 검색**을 지시할 수 있도록 백엔드·프론트 반영.
동시에 **작업현황** 확인용 웰컴 칩을 추가해 직접 지시·현황 확인 UX 강화.

## 2. 변경 파일

### 백엔드 (이전 턴에서 반영)

| 파일 | 내용 |
|------|------|
| `app/services/tool_registry.py` | `list_remote_dir` 도구 스키마 추가, `read_remote_file`에 `project`+`path` 반영 |
| `app/services/tool_executor.py` | `_list_remote_dir`, `_read_remote_file`(project 연동), 인텐트 매핑 `server_file` |
| `app/services/intent_router.py` | 인텐트 `server_file` 추가, 키워드 폴백(서버 검색, 원격 서버, SSH 등) |

### 프론트엔드 (이번 턴)

| 파일 | 내용 |
|------|------|
| `aads-dashboard/src/components/chat/ChatStream.tsx` | `_TOOL_DISPLAY_NAMES`에 `list_remote_dir` 추가, `read_remote_file` 라벨을 "원격 서버 파일 읽기"로 변경 |
| `aads-dashboard/src/components/chat/ActionChips.tsx` | 웰컴 칩: "작업현황", "서버 파일 검색" 추가; 동적 칩: 서버/원격/작업 관련 후속 액션 추가 |

## 3. 프론트 변경 상세

- **도구 표시명**: 스트리밍 시 도구 호출 블록에 `list_remote_dir` → "서버 파일 검색 (SSH)", `read_remote_file` → "원격 서버 파일 읽기" 표시.
- **웰컴 칩**: "오늘의 브리핑", **"작업현황"**, **"서버 파일 검색"**, "프로젝트 현황", "지시서 작성" (기존 경쟁사 분석·코드 리뷰 자리 조정).
- **동적 칩**: 마지막 답변이 서버/원격/SSH 관련이면 "서버 파일 검색", "원격 파일 읽기" 칩 노출; 대시보드/작업 관련이면 "작업 이력 더" 칩 노출.

## 4. 검증

- 린트: `ChatStream.tsx`, `ActionChips.tsx` 에러 없음.
- 빌드: 로컬 `npm run build` 권장 (배포 전).

## 5. 배포

- **실행**: 2026-03-09 08:28 KST, 68서버 `/root/aads/aads-server`에서 수행.
- **명령**: `docker compose -f docker-compose.prod.yml build aads-server aads-dashboard` → `up -d aads-server aads-dashboard`.
- **결과**: aads-server(8100), aads-dashboard(3100), aads-postgres(5433) 기동 완료.
- **프론트**: Docker 이미지 재빌드로 대시보드 반영됨.
- **백엔드**: aads-server 이미지 재빌드로 서버 검색·인텐트 반영됨.

## 6. 사용 방법 (채팅창)

- **작업현황**: "작업현황 알려줘" 또는 웰컴 칩 **작업현황** 클릭.
- **서버 파일 검색**: "KIS 서버에서 설정 파일 찾아줘", "SF 서버 파일 목록" 또는 웰컴 칩 **서버 파일 검색** 클릭.
- **원격 파일 읽기**: "SF 서버의 app/config.php 내용 보여줘" 등 프로젝트(KIS/GO100/SF/NTV2)+경로 지정.

## 7. 다음 작업 제안

- 배포 후 https://aads.newtalk.kr/chat 에서 웰컴 칩 "작업현황", "서버 파일 검색" 클릭으로 동작 확인.
- 필요 시 다른 프로젝트(SF, NTV2)용 웰컴 메시지 문구 추가.
