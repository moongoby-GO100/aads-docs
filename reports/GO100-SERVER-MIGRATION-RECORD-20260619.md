> ## ⚠️ 과거 자료 (historical) — 2026-06-19 이전 기록이다
>
> - 이 문서는 **2026-06-19 당시의 서버 이전 기록**이다. 6월 사건 자체는 역사 기록으로 보존하며 본문을 치환하지 않았다.
> - 이 기록의 결론(`GO100 → contabo14 / 5.104.86.14`)은 2026-10-01 05:43 KST 현재 원장과 **일치한다**.
>   다만 현재 사실의 근거는 이 문서가 아니라 운영 원장 API `/api/v1/ops/servers`(`/ops/status` 와 동일한
>   `list_ledger_servers()`)와 `aads-server/app/services/server_registry.py` 의 `LEDGER_SERVER_IDS`, DB `server_registry` 다.
> - **현재 담당 원장은 4대**다: contabo116(AADS) · contabo14(GO100·KIS) · cafe24_114(SF·NTV2·NAS) ·
>   jinah244 / 진아실장 서버(ACCT). 건강 감시 대상은 앞의 3대뿐이며 jinah244 의 `unknown` 은 장애가 아니다.
>   "운영 3대" 로 읽지 마라.
> - **후속 정본 링크**: AADS 프로젝트 문서 `/projects/AADS/documents/aads-current-authority-context`
>   — "AADS 현재 정본·운영 근거 제공 PRD v1.0.0" (revision 1, hash `c6e3105c…`).
>   2026-10-01 05:43 KST 실측 기준 **draft** 다(`approved_revision_id` 비어 있음). 구현 승인을 정본 승인·배포 완료와 혼동하지 마라.
> - 아래 "AADS Update Scope" 의 파일 목록은 **2026-06-19 시점 스냅샷**이다. 현재 코드 위치·내용의 근거로 쓰지 말고
>   해당 파일을 직접 확인하라. legacy `211.188.51.113` 은 현재 폐기 별칭(`legacy_ids`)이며 운영 주소가 아니다.
> - 파일 변경시각(2026-06-19)과 운영 실측시각은 다른 값이다.

# GO100 Server Migration Record - 2026-06-19

## Summary

GO100 백억이 운영 서버 정보가 AADS 기준으로 최신화됐다.

| 항목 | 값 |
|---|---|
| 프로젝트 | GO100 |
| 신규 서버 | contabo14 |
| IPv4 | 5.104.86.14 |
| IPv6 | 2400:d320:2338:1565::1 |
| 위치 | Contabo Tokyo |
| OS | Ubuntu 24.04 |
| WORKDIR | /root/kis-autotrade-v4 |
| 도메인 | go100.newtalk.kr |
| 이전 상태 | 2026-06-19 KST 기준 신서버 운영 전환 확인 |
| legacy 서버 | 211.188.51.113 |
| legacy 처리 | GO100 기준 폐지 예정. 신규 GO100 작업은 contabo14 우선 |

## AADS Update Scope

- `aads-server/app/core/project_config.py`: GO100 서버를 `5.104.86.14`로 변경
- `aads-server/app/services/server_registry.py`: GO100을 `contabo14` 서버로 분리
- `aads-server/app/api/admin.py`: 관리자 서버 그룹에 `contabo14` 추가
- `aads-server/app/api/project_docs.py`: GO100 문서 조회 SSH 대상을 신규 서버로 변경
- `aads-server/app/api/ceo_chat_tools_db.py`: GO100 DB 대상을 KIS 별칭에서 분리
- `aads-server/scripts/claude_relay_server.py`, `app/services/model_selector.py`: GO100 런타임 힌트 변경
- `aads-docs/HANDOVER.md`, `aads-docs/HANDOVER-RULES.md`: 운영 문서 갱신

## Operational Rule

GO100 코드, 배포, SSH, 문서 조회, DB 조회 기본 대상은 `contabo14 / 5.104.86.14`다. KIS와 GO100을 같은 서버로 단정하지 않는다.
