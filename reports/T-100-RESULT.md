# T-100 결과보고: 브릿지 완료보고 재감지 방지

- **Task ID**: T-100
- **Title**: 브릿지 완료보고 메시지 재감지 방지 — 무한루프 차단
- **완료일**: 2026-03-06 11:00 KST
- **담당**: Claude Code (claude-sonnet-4-6)

---

## 수정 내역

### Part A — bridge.py (scripts/bridge.py)

**수정 1: SKIP_PATTERNS 10개 추가 + 메시지 스킵 로직**
- `process_message()` 진입 시 SKIP_PATTERNS 체크
- "작업 완료", "push 완료", "에러 종료", "BRIDGE_RESULT", "현재 작업 현황", "다음 지시서 작성 전 필수 확인", "맥락유지 필수문서", "HANDOVER.md 반드시 갱신", "지시서 작성규칙", "auto_trigger 자동 실행" 포함 시 즉시 스킵

**수정 2: 자기발송 마커 [BRIDGE-SENT]**
- `BRIDGE_SENT_MARKER = "[BRIDGE-SENT]"` 상수 정의
- 메시지에 마커 포함 시 즉시 스킵

**수정 3: 중복 처리 방지 (processed_ids)**
- `_processed_ids` set + `_processed_ids_order` list (MD5 해시 기반)
- 동일 메시지 재처리 시 "duplicate" reason으로 스킵
- 최대 1000개 유지 (초과 시 오래된 것 제거)

**수정 4: DIRECTIVE_START 감지 강화**
- "지시서 작성규칙" / "작성규칙" 포함 메시지 → 스킵 (규칙 설명)
- Task ID가 실제 숫자(T-NNN)가 아닌 경우 → 스킵 (템플릿)
- done/ 폴더에 이미 있는 Task ID → 스킵 (완료된 작업)

### Part B — auto_trigger.sh

- `_process_directive()` 함수 초반에 RESULT 파일 감지 로직 추가
- 파일명에 "RESULT" 포함 시 즉시 return 0 (스킵)

---

## 테스트 결과

| 테스트 | 입력 | 결과 |
|--------|------|------|
| SKIP_PATTERNS | "작업 완료: T-099 처리됨" | skipped: skip_pattern: 작업 완료 ✅ |
| 자기발송 마커 | "push 완료 T-099 [BRIDGE-SENT]" | skipped: bridge-sent marker ✅ |
| 실제 DIRECTIVE | ">>>DIRECTIVE_START Task ID: T-200..." | directive_blocks_saved: [{status: ok}] ✅ |
| 규칙설명 DIRECTIVE | "지시서 작성규칙 ... >>>DIRECTIVE_START..." | skipped: skip_pattern: 지시서 작성규칙 ✅ |
| 중복 메시지 | 동일 메시지 2회 | 1차 처리 → 2차 duplicate ✅ |

---

## HTTP 상태

- API 연결: 200 (conversation_saved, directive_blocks_saved 정상)
- git push: 완료

## HANDOVER 업데이트

HANDOVER v5.24에 T-100 내역 기록 완료
