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

## 오류 사전 (R-ERRBOOK)

원인을 밝혔으면 사전에 넣는다. 넣지 않으면 다음 사람이 같은 추적을 처음부터
반복한다. 2026-09-14, 같은 실패를 세 세션이 "러너 계정 문제" 로 보고했다.
실제 원인은 호스트/컨테이너 경로 불일치였다.

사전은 DB 한 벌(`ohvis_wiki_error_book`)이고 모든 서버가 같은 것을 본다.
contabo116 은 컨테이너 경유, 원격 서버는 PGHOST 터널로 붙는다 — 도구는
`scripts/error_book.py` 하나다. **서버별 사본을 만들지 마라.**

    # 이 오류가 알려진 것인가 (조사 시작할 때 먼저)
    error_book.py match <오류파일|->

    # 원인을 밝혔을 때 — 자동 기록된 후보를 채운다
    error_book.py list --candidates
    error_book.py promote --key auto.xxxx \
        --cause "..." --prevention "..." \
        --fix-commit <sha> --fix-file <경로> --fix-note "무엇을 고쳤나"

    # 후보가 없는 새 항목
    error_book.py register --key <영역>.<증상> --symptom "..." \
        --cause "..." --prevention "..." --signature "<정규식>" \
        --fix-commit <sha> --fix-note "..."

세 가지를 지킨다.

1. **추측을 넣지 않는다.** 확인한 원인만 넣는다. 틀린 사전은 없느니만 못하다 —
   다음 사람이 잘못된 원인을 믿고 엉뚱한 데를 판다. 모르면 candidate 로 둔다.
2. **prevention 과 fix 를 구분한다.** prevention 은 "앞으로 이렇게 해라",
   fix 는 "이번에 무엇을 고쳤나(커밋)". fix 가 없으면 재발했을 때 고친 것이
   되돌아간 건지 다른 경로가 같은 버그를 밟은 건지 알 수 없다.
3. **코드가 막는 것과 사람이 지켜야 하는 것을 구분해 적는다.** 규칙만 적어둔
   항목은 신뢰도가 낮다 — 나도 적어놓고 어겼다.

러너·프론트 진단기·채팅 오류 보고는 실패 시 자동으로 조회하고, 모르는 오류는
`status=candidate` 로 남긴다. 조사해서 알아낸 것은 자동으로 들어오지 않으므로
사람이 promote 해야 한다.

---

## 이 사본에 대하여

원본은 `/root/aads/AGENTS.md` 이고 **git 밖에 있다**. 전역 릴리스 계약이 버전
관리되지 않으면 누가 언제 무엇을 바꿨는지 남지 않고, 서버를 다시 세울 때 같이
사라진다. 이 사본은 그 이력을 남기기 위한 것이다.

원본을 고치면 이 사본도 같이 갱신한다.
