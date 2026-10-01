> ## ⚠️ 과거 자료 (historical) — 2026-06-19 당시 기록이다
>
> - 이 문서는 원문 작성 시점(2026-06-19)의 서버 이전 기록이다. 6월 사건 자체는 역사 기록으로 보존하며 본문을 치환하지 않았다.
> - 이 기록의 결론(`GO100 → contabo14 / 5.104.86.14`)은 현재 원장과 **일치한다.** 다만 현재 사실의 근거는 이 문서가 아니라 아래 단일 source 다.
>
> ### 현재 근거(authority) — 단일 source 와 API 경로
>
> | 구분 | 값 | 비고 |
> |---|---|---|
> | 원장 단일 source | `aads-server/app/services/server_registry.py` → `list_ledger_servers()` | `LEDGER_SERVER_IDS`=4 / `CANONICAL_SERVER_IDS`=감시 3 (코드 89~90행, 2026-10-01 15:11 KST 확인) |
> | API 권위 경로 | `GET /api/v1/ops/status` | 2026-10-01 14:25:12 KST 확인 |
> | 화면 | `https://aads.newtalk.kr/ops/servers` | 대시보드 경로다. API route 가 아니다 |
> | DB `server_registry` 테이블 | 대조 증거 | 값이 현재와 맞더라도 단일 source 가 아니다 |
>
> **정정 (2026-10-01)**: 이 머리표시의 이전 판은 API 경로를 `/api/v1/ops/servers` 로 적었다. 그 route 는 코드에 없다 —
> `app/api/ops.py` 와 OpenAPI 에 선언된 것은 `/ops/status` 뿐이고, 이전 판이 근거로 삼은 401 응답은 라우트 존재 증거가
> 아니라 전역 인증 미들웨어 응답이었다. `/api/v1/ops/status` 로 정정했다.
>
> ### 현재 담당 원장 4대 (2026-10-01 15:11 KST, `list_ledger_servers()` 기준)
>
> | 서버 | 주소 | 프로젝트 | 건강 감시 |
> |---|---|---|---|
> | contabo116 | 5.104.86.116 | AADS | 포함 |
> | contabo14 | 5.104.86.14 | GO100, KIS | 포함 |
> | cafe24_114 | 114.207.244.86 (SSH 7916) | SF, NTV2, NAS | 포함 |
> | jinah244 (진아실장 서버 / 회계비서) | 5.104.85.244 | ACCT | **제외** — `http_health_urls` 없음. `unknown` 을 장애로 읽지 마라 |
>
> 감시 대상은 앞의 3대뿐이다. "운영 3대" 로 읽지 마라 — 원장 4대와 감시 3대는 별개 집계다.
>
> ### 후속 정본 링크
>
> - AADS 프로젝트 문서 `/projects/AADS/documents/aads-current-authority-context`
>   — "AADS 현재 정본·운영 근거 제공 PRD v1.0.0" (latest revision `1aed18f5-6541-47e5-8d4b-c0d427863769`,
>   hash `c6e3105cf68727446358210e57f4b70b033e2da425a2008990260b24ab1c1b32`).
> - 2026-10-01 15:11 KST DB 실측 기준 이 PRD 는 **draft** 다 (`approved_revision_id` = NULL).
>   구현 승인을 정본 승인·배포 완료와 혼동하지 마라. 최신 draft 를 승인 정본으로 승격하지 않는다.
> - AADS current guide: `aads-server/docs/operations/CURRENT_AUTHORITY_AND_SERVER_LEDGER.md`
>   (root 소관, `df17e6173ebf6ab4fbb67aeb821e9b66c18656b0` origin/main 반영).
> - 같은 6월 토폴로지 기록의 정본: GO100 저장소 `docs/AADS-3SERVER-OPERATING-TOPOLOGY-20260623.md`
>   (contabo14 `/root/kis-autotrade-v4`, origin/main `bf770026`).
>
> ### 시각 표기 주의 (2026-10-01 정정)
>
> - 파일 mtime 은 **운영 관측시각도 작성시각도 아니다.** 본문의 모든 시각은 2026-06-19 당시 기준이다.
> - 이 머리표시의 이전 판은 UTC 값을 KST 로 잘못 표기했다 — `05:43 KST` 로 적힌 것은 실제 `05:43 UTC` = **14:43 KST** 다.
>   contabo116 의 셸 `date` 는 CEST(+0200)이고 DB·도구 타임스탬프는 UTC 이므로 KST 는 UTC+9 로 환산해야 한다.
> - 아래 "AADS Update Scope" 의 파일 목록은 2026-06-19 시점 스냅샷이다. 현재 코드 위치의 근거로 쓰지 말고 해당 파일을
>   직접 확인하라. legacy `211.188.51.113` 은 현재 폐기 별칭(`legacy_ids`)이며 운영 주소가 아니다.

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
