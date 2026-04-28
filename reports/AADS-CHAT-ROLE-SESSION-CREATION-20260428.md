# AADS Chat Role Session Creation Deployment Report

Date: 2026-04-28 KST

## Scope

- Added a role-aware "new session" flow to the AADS chat sidebar.
- Restored left session row actions: open in new window, rename, delete, and more menu.
- Added project-level default role storage through `workspace.settings`.
- Added server fallback so sessions without `role_key` use the workspace `default_role_key`.
- Normalized dashboard blue-green nginx upstream handling so only one dashboard slot is active.

## Code Changes

- Dashboard commit: `247b443` (`Add role-aware chat session creation`)
  - `src/app/chat/page.tsx`
  - `src/app/chat/ChatSidebar.tsx`
  - `src/app/chat/types.ts`
  - `src/lib/api.ts`
  - `deploy.sh`
- Server commit: `447c13e` (`Apply workspace role fallback for chat sessions`)
  - `app/services/chat_service.py`
  - `nginx-aads-upstream.conf`

## Deployment

- Dashboard blue-green deploy completed at 2026-04-28 15:46 KST.
  - Active slot: `green`
  - Container: `aads-dashboard-green`
  - Nginx upstream: `3101` active, `3100` backup
  - Frontend QA step completed with `UNKNOWN` verdict from QA API, deploy script reported pass.
- Server deploy completed at 2026-04-28 15:48 KST.
  - Health check passed after restart.
  - DB schema check passed.
  - Chat table access check passed.
  - LLM service check passed.

## Verification

- `docker exec aads-server python -m py_compile /app/app/services/chat_service.py`: passed.
- `git diff --check`: passed for dashboard and server before commit.
- `https://aads.newtalk.kr/chat`: returned expected `307` login redirect.
- `http://127.0.0.1:8100/api/v1/health`: returned `status=ok`.
- Recent filtered logs:
  - `aads-server`: no matching `Traceback`, `ERROR`, SQL syntax, or role fallback errors.
  - `aads-dashboard-green`: no matching `Error`, `TypeError`, `ReferenceError`, or `failed` lines.

## Notes

- `npx tsc --noEmit --incremental false` still fails on pre-existing admin API typing gaps unrelated to this change.
- Local sandbox `next build` stalled under Turbopack, but Docker blue-green build completed successfully in the deploy path.
- `public/manager/env_unknown.json` remains an unrelated local dashboard modification and was intentionally excluded from the dashboard commit.
