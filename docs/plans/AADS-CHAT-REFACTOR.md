# AADS-CHAT-REFACTOR — 채팅 스트리밍 리팩토링 정밀 설계서

> **작성**: 2026-04-27 KST  
> **대상**: AADS CEO Chat (`/root/aads/aads-server` + `/root/aads/aads-dashboard`)  
> **상태**: P0/P1 패치 진행 중(`runner-fea90edb`), P2/P3 설계 완료  
> **노출 경로**: <https://aads.newtalk.kr/docs> (project_docs.py 자동 스캔, plan 카테고리)

---

## 1. 배경

CEO Chat 스트리밍에서 다음 4가지 문제가 관측되어 정밀 검토를 수행했다.

1. 502/503/504 자동 재시도 시 동일 메시지가 DB에 중복 저장되는 사례 발생
2. `thinking` 이벤트 토큰이 첫 `delta` 토큰 도착과 동시에 화면에서 사라짐(가시성 0)
3. `page.tsx`가 4,000줄 단일 컴포넌트화되어 SSE reader loop 코드가 4곳에 중복
4. `chatRecovery.ts` 모듈이 import만 되고 실제 호출 경로가 없는 dead-code 상태

본 문서는 P0~P3 우선순위로 단계별 리팩토링 설계를 제시한다.

---

## 2. 현재 아키텍처 (실측)

### 2.1 백엔드 3-layer

| 계층 | 파일 | 역할 |
|------|------|------|
| Producer/Consumer | `app/services/chat_service.py:479-720+` | `with_background_completion()` — Queue 기반. 클라이언트 끊겨도 LLM 생성 계속 + DB 저장 |
| Redis Stream | `app/services/redis_stream.py` | `publish_token()` XADD, `xread_blocking()` 재연결용 영속 버퍼 |
| Stream Worker | `app/services/stream_worker.py` | `deliver_sse()` — Queue/Redis에서 SSE 형식으로 변환 후 클라이언트 전달 |

### 2.2 프론트엔드 SSE 진입점

| 라인 | 함수 | 용도 |
|-----:|------|------|
| 1272 | `attachToExecution()` | 백그라운드 실행 재부착 |
| 1700 | `sendMessage()` | 신규 메시지 전송 |
| 2400 | `resume()` | 새로고침/재접속 시 미완료 스트림 재개 |
| (훅) | `useChatSSE.ts` | **import 됨, 실제 호출 없음** — dead-code |

### 2.3 핵심 정합성 이슈

- 4개 진입점 모두 동일한 `EventSource → for await reader` 패턴을 **개별 구현**
- `streamBuf`/`thinkingBuf`/`toolStatus` 상태 관리 중복
- 재연결 로직이 위치별로 미묘하게 다름

---

## 3. 우선순위 매트릭스

| 순위 | 작업 | 규모 | 위험 | 효과 |
|-----:|------|:---:|:---:|------|
| **P0** | 재시도 중복 저장 방지 (`idempotency_key` 영속화) | S | 0 | 데이터 무결성 |
| **P1** | `thinking` 토큰 가시성 분리 | S | 0 | UX 즉시 개선 |
| **P2** | SSE Reader Loop → `parser.ts` 공통 유틸 추출 (점진적) | M | 1 | 코드 중복 -75% |
| **P3** | `page.tsx` 컴포넌트 분할 (세션/메시지/SSE/입력) | XL | 2 | 장기 유지보수 |

---

## 4. P0 정밀 설계 — idempotency_key 영속화

### 4.1 현상

`page.tsx`의 `sendMessage()`는 매 호출마다 `crypto.randomUUID()`로 키를 생성. 502 자동 재시도 시 동일 메시지가 새 키로 백엔드에 도달 → dedup 실패 → 중복 저장.

### 4.2 패치 (3 hunk)

**Hunk 1 — 함수 시그니처 확장**

```diff
- const sendMessage = useCallback(async function sendMessage(
-   queuedContent?: string, _unused?: undefined,
-   retryCount?: number, _existingMsgId?: string
- ) {
+ const sendMessage = useCallback(async function sendMessage(
+   queuedContent?: string, _unused?: undefined,
+   retryCount?: number, _existingMsgId?: string,
+   _existingIdempotencyKey?: string
+ ) {
```

**Hunk 2 — 키 재사용**

```diff
  // Stage 3: idempotency key — 502 재시도 시 동일 메시지 중복 저장 방지
- const _idempotencyKey = crypto.randomUUID();
+ // P0-FIX (2026-04-27): 재시도 호출 시 첫 호출의 키를 그대로 재사용
+ const _idempotencyKey = _existingIdempotencyKey || crypto.randomUUID();
```

**Hunk 3 — 재시도 호출에 키 전달**

```diff
  await new Promise((r) => setTimeout(r, delay));
  setToolStatus(null);
- return sendMessage(content, undefined, attempt, userMsg.id);
+ return sendMessage(content, undefined, attempt, userMsg.id, _idempotencyKey);
```

### 4.3 검증

```bash
cd /root/aads/aads-dashboard
npx tsc --noEmit 2>&1 | head -30   # 0 error 기대
```

수동 시나리오: 채팅창에서 백엔드 일시 중단(502) → 자동 재시도 → DB `chat_messages` 테이블에서 동일 `idempotency_key`로 1건만 적재되는지 확인.

---

## 5. P1 정밀 설계 — thinking 가시성 분리

### 5.1 현상

`thinking` 이벤트 처리부:

```ts
} else if (ev.type === "thinking" && ev.content) {
  setToolStatus("💭 사고 중...");
  if (!isStale() && !full) setStreamBuf(ev.content || "분석 중...");
}
```

문제 2가지:
- `streamBuf`에 직접 표시 → `delta` 토큰 도착 시 즉시 덮어씀
- `!full` 가드로 인해 delta 1바이트만 들어와도 thinking 누적 중단

### 5.2 패치 (5 hunk)

1. **state 분리**: `const [thinkingBuf, setThinkingBuf] = useState<string>("");`
2. **이벤트 핸들러 변경**: `setThinkingBuf(prev => prev + (ev.content || ""))`
3. **시작 시 초기화**: `setStreamBuf(""); setThinkingBuf("");`
4. **종료 시 초기화**: `done` 이벤트 핸들러에 `setThinkingBuf("")` 추가
5. **UI 렌더 추가** — streaming placeholder 직전에 보라색 구분 영역:

```tsx
{thinkingBuf && (
  <div className="text-xs text-gray-500 italic px-3 py-1.5 mb-1
                  border-l-2 border-purple-300
                  bg-purple-50/30 dark:bg-purple-900/10 rounded">
    <span className="font-medium text-purple-600 dark:text-purple-400">
      💭 사고 과정
    </span>
    <div className="mt-0.5 whitespace-pre-wrap text-gray-600 dark:text-gray-400">
      {thinkingBuf}
    </div>
  </div>
)}
```

### 5.3 검증

수동: 긴 도구 호출(`run_agent_team` 등) 시 보라색 영역에 사고 과정이 누적되며, delta 도착 후에도 사라지지 않고 답변 위에 유지되는지 확인.

---

## 6. P2 정밀 설계 — SSE Reader Loop 공통 유틸 추출 (점진적)

### 6.1 목표

4곳의 SSE reader 중복(`attach`/`send`/`resume` + `useChatSSE` 훅 4 패턴)을 1개 유틸로 통합한다. **점진적**으로 진행하여 중간에도 항상 동작 가능 상태를 유지한다.

### 6.2 새 파일 구조

```
aads-dashboard/src/lib/chat/
  ├─ parser.ts        ← Phase 1: SSE 라인 파싱 + 이벤트 디스패치 (순수 함수)
  ├─ reader.ts        ← Phase 2: ReadableStream → AsyncIterator 래퍼
  ├─ recovery.ts      ← Phase 3: 502 재시도 + Redis 복구 정책
  └─ index.ts
```

### 6.3 Phase 1 — parser.ts (1일, 위험 0)

```ts
// aads-dashboard/src/lib/chat/parser.ts
export type SSEEvent =
  | { type: "delta"; content: string }
  | { type: "thinking"; content: string }
  | { type: "tool_use"; name: string; input?: any }
  | { type: "tool_result"; name: string; result?: any }
  | { type: "done"; finalMsg?: any }
  | { type: "error"; message: string }
  | { type: "heartbeat" };

export function parseSSELine(line: string): SSEEvent | null {
  if (!line.startsWith("data:")) return null;
  const data = line.slice(5).trim();
  if (!data || data === "[DONE]") return { type: "done" };
  try { return JSON.parse(data) as SSEEvent; }
  catch { return null; }
}

export interface SSEHandlers {
  onDelta?: (content: string) => void;
  onThinking?: (content: string) => void;
  onToolUse?: (name: string, input?: any) => void;
  onToolResult?: (name: string, result?: any) => void;
  onDone?: (finalMsg?: any) => void;
  onError?: (message: string) => void;
  onHeartbeat?: () => void;
}

export function dispatchSSEEvent(ev: SSEEvent, h: SSEHandlers): void {
  switch (ev.type) {
    case "delta": h.onDelta?.(ev.content); break;
    case "thinking": h.onThinking?.(ev.content); break;
    case "tool_use": h.onToolUse?.(ev.name, ev.input); break;
    case "tool_result": h.onToolResult?.(ev.name, ev.result); break;
    case "done": h.onDone?.(ev.finalMsg); break;
    case "error": h.onError?.(ev.message); break;
    case "heartbeat": h.onHeartbeat?.(); break;
  }
}
```

**적용 순서**: `page.tsx` 4개 위치의 `if/else if` 체인을 `dispatchSSEEvent(parseSSELine(line)!, {...})`로 교체. 단위 테스트 우선 작성(`parser.test.ts`).

### 6.4 Phase 2 — reader.ts (1.5일, 위험 1)

```ts
export async function* readSSEStream(
  response: Response,
  signal?: AbortSignal,
): AsyncGenerator<SSEEvent> {
  const reader = response.body!.getReader();
  const decoder = new TextDecoder();
  let buffer = "";
  while (true) {
    if (signal?.aborted) { reader.cancel(); return; }
    const { done, value } = await reader.read();
    if (done) break;
    buffer += decoder.decode(value, { stream: true });
    const lines = buffer.split("\n");
    buffer = lines.pop() ?? "";
    for (const line of lines) {
      const ev = parseSSELine(line);
      if (ev) yield ev;
    }
  }
}
```

**적용 순서**: `attachToExecution` → `resume` → `sendMessage` 순서로 한 곳씩 마이그레이션. 각 단계마다 빌드 + 수동 검증.

### 6.5 Phase 3 — recovery.ts (1일, 위험 1)

기존 `chatRecovery.ts`의 dead-code를 정리하여 `recovery.ts`로 통합. 502/503/504 backoff 정책과 Redis Stream `xread` 복구 로직을 일원화.

### 6.6 마이그레이션 일정

| Phase | 작업 | 소요 | 위험 |
|------:|------|:---:|:---:|
| 1 | `parser.ts` + 단위 테스트, page.tsx 1곳 적용 | 1일 | 0 |
| 2 | `reader.ts`, page.tsx 4곳 순차 적용 | 1.5일 | 1 |
| 3 | `recovery.ts`, `chatRecovery.ts` 제거 | 1일 | 1 |
| **합계** | — | **3.5일** | — |

---

## 7. P3 정밀 설계 — page.tsx 컴포넌트 분할

### 7.1 현재 문제

`page.tsx` ≈ 4,000줄 단일 클라이언트 컴포넌트. 세션/메시지/SSE/입력/툴바/모달이 모두 한 파일에 혼재.

### 7.2 목표 구조

```
aads-dashboard/src/app/chat/
  ├─ page.tsx                ← 80줄 미만, layout + provider 만
  ├─ context/
  │   ├─ ChatContext.tsx     ← 전역 상태 (session, messages, streaming)
  │   └─ useChat.ts          ← 핵심 훅
  ├─ components/
  │   ├─ SessionSidebar.tsx  ← 좌측 세션 목록
  │   ├─ MessageList.tsx     ← 메시지 영역
  │   ├─ MessageBubble.tsx   ← 개별 메시지 (thinking/tool/delta 렌더)
  │   ├─ ChatInput.tsx       ← 입력창 + 첨부
  │   ├─ ToolStatusBar.tsx   ← 도구 진행 표시
  │   └─ StreamingPlaceholder.tsx
  └─ hooks/
      ├─ useSendMessage.ts
      ├─ useResume.ts
      └─ useAttachExecution.ts
```

### 7.3 마이그레이션 순서 (위험 최소)

1. **Step 1 (1일)**: `ChatContext` 생성, 기존 state를 그대로 옮김. `page.tsx`는 그대로 유지하면서 context를 `useState` 대신 사용.
2. **Step 2 (2일)**: 입력/사이드바부터 분리(`ChatInput`, `SessionSidebar`). 가장 독립적인 영역.
3. **Step 3 (2일)**: `MessageBubble` 분리 — thinking/tool/markdown 렌더 로직을 모듈화.
4. **Step 4 (3일)**: SSE 훅 3종(`useSendMessage`/`useResume`/`useAttachExecution`) 분리. P2 완료 의존.
5. **Step 5 (1일)**: `MessageList` 분리, `page.tsx`를 80줄 미만 layout으로 축소.
6. **Step 6 (1일)**: 회귀 테스트, dead-code 제거.

총 소요: **약 10일(2주)**.

### 7.4 의존 관계

P3는 P2(parser/reader 추출) 완료 후 진행 권장. 그 전에 시작하면 SSE 훅 분리가 중복 작업이 된다.

---

## 8. 통합 로드맵

```
Week 0 (현재)
  ├─ P0 패치 (runner-fea90edb 진행 중)
  └─ P1 패치 (동일 러너에 포함)

Week 1
  └─ P2 Phase 1 (parser.ts) — 1일

Week 2
  ├─ P2 Phase 2 (reader.ts) — 1.5일
  └─ P2 Phase 3 (recovery.ts) — 1일

Week 3~4
  └─ P3 Step 1~6 — 10일
```

---

## 9. 검증 계획

| 단계 | 검증 방법 | 기준 |
|------|-----------|------|
| 빌드 | `npx tsc --noEmit` | 0 error |
| 단위 테스트 | `parser.test.ts`, `reader.test.ts` | 100% 통과 |
| 회귀 (수동) | 채팅 1턴/긴 도구 호출/재시도/새로고침 후 재개 | 4 시나리오 모두 정상 |
| 회귀 (자동) | E2E (Playwright, 추후) | TBD |
| 성능 | DevTools → Performance | 첫 토큰 지연 ≤ 500ms 유지 |

---

## 10. 리스크 & 롤백

| 리스크 | 완화책 |
|--------|--------|
| P2 적용 중 SSE 파싱 차이로 일부 이벤트 누락 | 단위 테스트로 100% 커버 후 점진 적용 |
| P3 분할 중 context provider 누락 → 빈 화면 | Step별로 빌드/배포, 1단계 단위 PR |
| 사용자 미완료 스트림 손실 | Redis Stream `xread`로 복구 보장 |

**롤백**: 모든 변경은 단일 PR/Runner 작업 단위. `git revert <hash>` 1회로 즉시 복구 가능.

---

## 11. 다음 액션

- [x] 본 설계서 저장 (`/app/docs/plans/AADS-CHAT-REFACTOR.md`)
- [ ] `runner-fea90edb` 완료 후 P0/P1 검증 보고
- [ ] CEO 승인 시 P2 Phase 1(`parser.ts`) Runner 제출
- [ ] P2 완료 후 P3 일정 확정

---

> **저자**: AADS CTO AI (Claude Opus)  
> **참고**: `app/services/chat_service.py`, `app/services/redis_stream.py`, `app/services/stream_worker.py`, `aads-dashboard/src/app/chat/page.tsx`, `aads-dashboard/src/hooks/useChatSSE.ts`
