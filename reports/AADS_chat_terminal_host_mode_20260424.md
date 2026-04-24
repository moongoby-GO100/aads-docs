# AADS 채팅창 터미널 Host 모드 반영 보고서 (2026-04-24 KST)

## 1. 목적

기존 채팅창 터미널은 `aads-server` 컨테이너 내부 셸만 열 수 있었고, 호스트에 설치된 `codex`를 사용할 수 없었다.  
이번 작업의 목적은 채팅창 터미널이 `container`뿐 아니라 `host` 셸도 열 수 있게 하고, 브라우저 UI에서 실제로 `codex` 실행이 가능한 상태까지 검증하는 것이다.

## 2. 반영 내용

### 2-1. 백엔드

다음 파일에 `session_mode` 개념을 추가했다.

- `aads-server/app/models/terminal.py`
- `aads-server/app/api/terminal.py`
- `aads-server/app/services/terminal_runner.py`

핵심 변경:

- `TerminalSessionCreate`에 `session_mode: "container" | "host"` 추가
- `TerminalSessionOut`에 `session_mode` 노출
- 세션 생성 API가 `session_mode`를 terminal runner로 전달
- `host` 모드일 때 아래 명령으로 호스트 셸을 연다
  - `ssh -tt root@host.docker.internal 'cd /root/aads && exec /bin/bash -il'`
- 상태 이벤트와 session snapshot에도 `session_mode`를 포함

즉, 브라우저 터미널은 이제

- `container`: 기존 컨테이너 셸
- `host`: 호스트 root bash 셸

두 모드를 선택할 수 있다.

### 2-2. 프론트엔드

다음 파일을 반영했다.

- `aads-dashboard/src/components/chat/TerminalPane.tsx`
- `aads-dashboard/src/app/chat/terminal/page.tsx`

핵심 변경:

- `Host / Container` 토글 추가
- 터미널 상태 라인에 `session_mode / backend_mode / status` 표시
- 세션 생성 시 `session_mode`를 API로 전달
- 세션 목록에도 `host/container` 표시
- 새 창 `/chat/terminal`을 열 때 `mode` query를 같이 넘기고, 새 창도 같은 모드로 시작
- UI 기본 생성 모드는 `host`

## 3. 검증 결과

### 3-1. API 스모크

localhost API로 실제 host session을 생성해 아래를 확인했다.

- `create_session_mode = host`
- `backend_mode = pty`
- `pwd -> /root/aads`
- `whoami -> root`
- `command -v codex -> /root/.nvm/versions/node/v20.20.0/bin/codex`
- `codex --version -> codex-cli 0.123.0`

검증 세션:

- `4c0173ae-b4ae-4c87-a7bf-007dd5f208eb`

### 3-2. 브라우저 E2E

컨테이너 내부 Playwright로 실제 로그인 후 `/chat` UI 기준 E2E를 수행했다.

검증 항목:

- 로그인 성공
- `Terminal` 패널 열림
- `Host` 토글 클릭 성공
- 새 세션 생성 성공
- 생성된 세션이 `session_mode=host`
- `backend_mode=pty`
- UI에서 `codex --version` 입력 성공
- session API `recent_commands=["codex --version"]` 확인
- stream 출력에 `codex-cli 0.123.0` 포함 확인
- 종료 후 running session 0 확인

검증 세션:

- `862aca93-bcb2-4f3f-b3e2-611f56e2177e`

스크린샷:

- `/tmp/aads_terminal_host_mode_e2e_host/terminal_panel.png`
- `/tmp/aads_terminal_host_mode_e2e_host/after_codex.png`

## 4. 현재 상태

2026-04-24 08:42 KST 기준:

- `aads-server`: healthy
- `aads-dashboard`: healthy
- `GET /api/v1/health` -> `status=ok`
- terminal host mode: 라이브 반영 완료
- 브라우저 UI에서 host terminal + codex 실행: 확인 완료

## 5. 제약 및 메모

- 이 구조는 “기존 호스트 터미널 프로세스 attach”가 아니다.
- 정확히는 채팅창에서 `새 호스트 셸 세션`을 여는 방식이다.
- `host` 모드는 사실상 웹 UI에서 호스트 root shell을 여는 것이므로 권한 범위가 매우 크다.
- 현재는 호스트 셸을 `/root/aads`에서 `/bin/bash -il`로 고정해 열고 있다.
- 세션 종료 시 SSH 쪽 return code가 `255`로 보일 수 있으나, 현재 E2E 기준 명령 실행과 종료 정리는 정상이다.

## 6. 결론

채팅창 터미널은 이제 실제로 `host` 셸을 열 수 있고, 호스트에 설치된 `codex`도 브라우저에서 바로 실행된다.  
즉 “컨테이너 셸 보조 위젯” 수준에서 벗어나, 운영용 호스트 터미널로 사용할 수 있는 상태까지 올라왔다.
