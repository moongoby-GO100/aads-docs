# AADS Global Release Rules

These rules apply to every repository and operator under `/root/aads`.

## Mandatory Blue/Green release contract

1. Preserve unrelated worktree changes. Inspect the exact target files before editing, commit the intended files, and push before declaring completion.
2. Build one immutable image per repository release SHA. Start the candidate and standby slots with `--no-build`; both slots must report the same Docker image digest.
3. Never hold `/tmp/aads-nginx-upstream.lock` during a build, dependency install, drain wait, standby sync, or QA. Acquire it only after candidate health passes, keep it through the routing/state-marker update and immediate routed health check, then release it.
4. A chat execution is mutable only by the holder of its DB `owner_instance` + `owner_epoch` lease. Heartbeat the lease during model/tool waits. An inactive slot must not start recovery or automatic reactions; persist them for the active slot.
5. Do not consume retry budget for scanning, claiming, waiting for a relay slot, or resolving a placeholder conflict. Increment it only immediately before an actual model call.
6. Do not restart or rebuild the previous active slot until its streams have drained. Synchronize it from the already-built release image, never by rebuilding.
7. Roll out sequentially: candidate start → direct health → cutover → routed/external health → same-image standby → QA. Roll back routing immediately if a cutover check fails.
8. “Cutover complete” and “release certified” are different states. Do not report the release as complete until required tests, external checks, and at least five minutes of P0/P1 monitoring pass without new errors.
9. Never deploy the full compose stack for an application-only change, never directly restart the active API supervisor, and never bypass commit hooks with `--no-verify`.
10. Build only from a clean, committed release worktree. If unrelated tracked changes exist, stop before the build or create an isolated clean worktree at the release SHA; never silently include uncommitted files in a release image.
11. Keep Docker build contexts bounded. Exclude local virtual environments, repository history, caches, generated media, and other runtime-irrelevant artifacts; investigate any unexpected context growth before certifying the release.

The deploy scripts must fail closed when the immutable-image, short-lock, health, or same-digest gates cannot be verified.

---

## 이 사본에 대하여

원본은 `/root/aads/AGENTS.md` 이고 **git 밖에 있다**(2026-09-14 확인). 전역 릴리스
계약이 버전 관리되지 않으면 누가 언제 무엇을 바꿨는지 남지 않고, 서버를 다시
세울 때 같이 사라진다. 이 사본은 그 이력을 남기기 위한 것이다.

원본을 고치면 이 사본도 같이 갱신한다. 두 벌을 유지하는 비용을 아는 채로
선택한 것이며, 대안(원본을 저장소 안으로 옮기고 `/root/aads/AGENTS.md` 를
심볼릭 링크로 두는 것)은 CEO 판단이 필요하다.
