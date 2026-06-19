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
