# AADS 채팅창 터미널 동등권한 및 Privileged 실행구조 개선안 보고서 (2026-04-23 KST)

## 1. 보고 목적

본 보고서는 아래 2개 요구를 동시에 만족시키기 위한 개선안을 정리한다.

- CEO 요구 1: 채팅창은 현재 이 터미널에서 가능한 작업을 동일하게 수행할 수 있어야 한다.
- CEO 요구 2: 파일 수정 누락, commit/push 누락, 배포 반영 누락이 없어야 한다.

결론부터 말하면, 현재 구조에 권한만 더 주는 방식은 적합하지 않다.  
정답은 `채팅창 = 실제 터미널 세션`을 소유하게 만들고, 파일수정/반영/배포를 그 위에 재정렬하는 것이다.

---

## 2. 최종 결론

### 권고 방향

가장 적절한 방향은 `Full Terminal Runner + Structured Tool 공존 구조`다.

- 평소에는 기존 구조화 도구를 사용한다.
- 실제 운영/개발/배포 작업이 필요할 때는 채팅창이 `실제 터미널 세션`을 연다.
- 이 터미널 세션은 현재 운영 터미널과 같은 `host / user / cwd / env / TTY / network / filesystem` 문맥을 사용한다.
- 파일수정 후 즉시 commit/push는 제거하고, `ledger + finalize + deploy preflight` 구조로 재배치한다.

### 핵심 판단

현재 문제의 본질은 권한 부족 자체가 아니다.

- 권한이 필요한 작업과 아닌 작업이 섞여 있다.
- 같은 privileged 작업이 여러 실행 경로에 흩어져 있다.
- 파일수정과 commit/push/build/restart가 과도하게 결합돼 있다.
- 채팅창이 실제 터미널이 아니라, 여러 우회 도구의 집합처럼 동작한다.

따라서 목표는 `more tools`가 아니라 `real shell parity`여야 한다.

### 2.1 2026-04-23 구현 현황 업데이트 — Terminal P0

보고서 작성 이후 P0 백엔드 기반을 실제 코드로 추가했다.

- 신규 모델: `aads-server/app/models/terminal.py`
- 신규 서비스: `aads-server/app/services/terminal_runner.py`
- 신규 API: `aads-server/app/api/terminal.py`
- 라우터 등록: `aads-server/app/main.py`

현재 추가된 API는 아래와 같다.

- `GET /api/v1/terminal/sessions`
- `POST /api/v1/terminal/sessions`
- `GET /api/v1/terminal/sessions/{session_id}`
- `POST /api/v1/terminal/sessions/{session_id}/input`
- `POST /api/v1/terminal/sessions/{session_id}/execute`
- `POST /api/v1/terminal/sessions/{session_id}/resize`
- `POST /api/v1/terminal/sessions/{session_id}/close`
- `GET /api/v1/terminal/sessions/{session_id}/stream`

구현 범위는 다음이다.

- 사용자별 in-memory terminal session 생성/조회/종료
- 최근 출력 ring buffer와 `seq` 기반 replay
- `Last-Event-ID` 기반 SSE 재연결
- 기본 민감정보 마스킹 (`token`, `password`, `Bearer`, `sk-*`, private key 패턴)
- PTY 우선 시도, 실패 시 `pipe fallback`

실검증 결과도 남긴다.

- 서버 런타임 Python 3.11에서 `py_compile` 통과
- 로컬 smoke test에서 세션 생성, `pwd`, `echo TOKEN=...`, 종료까지 정상 동작 확인
- 현재 작업 환경은 `pty.openpty()`가 `out of pty devices`로 실패했다
- 원인은 현재 컨테이너/샌드박스의 `devpts` 상태 문제로 보이며, 이 환경에서는 자동으로 `pipe fallback`이 사용된다
- 따라서 일반 쉘 명령은 동작하지만 `vim`, `top`, `less`, job control 같은 완전한 interactive TTY는 아직 이 환경에서 검증되지 않았다

즉, “터미널 세션 API 자체”는 구현됐고, “완전한 TTY parity”는 런타임의 `devpts` 준비 상태에 추가로 의존한다.

### 2.2 2026-04-23 구현 현황 업데이트 — Direct Edit Finalize

터미널 P0 이후, direct edit 반영 경로도 실제 코드로 일부 전환했다.

- 신규 서비스: `aads-server/app/services/workspace_change_tracker.py`
- 신규 운영 API:
  - `GET /api/v1/ops/workspace-changes`
  - `POST /api/v1/ops/workspace-changes/finalize`
- startup 테이블 보장:
  - `chat_workspace_change_ledger`

반영된 구조는 다음이다.

- `write_remote_file`, `patch_remote_file`, `run_remote_command`가 파일 변경을 감지하면
  - 더 이상 즉시 `git commit/push`를 하지 않는다
  - 세션별 ledger에 `dirty` 상태로 기록한다
  - AADS server는 hot-reload만 유지한다
  - CHANGELOG와 AI diff review는 유지한다
- 배포/재기동 성격의 명령(`deploy.sh`, 일부 `docker compose up`, GO100/KIS systemctl restart`) 실행 전에는
  - 현재 채팅 세션의 pending 변경을 `finalize`한다
  - finalize에서만 `git add -> git commit --only -> git push`를 수행한다
  - 실패 시 원 명령 실행 전에 차단한다

실코드 기준 변경 파일:

- `aads-server/app/services/tool_executor.py`
- `aads-server/app/services/workspace_change_tracker.py`
- `aads-server/app/api/ops.py`
- `aads-server/app/main.py`

중요한 설계 정정도 있다.

- 초기 기획에서는 `수정 직후 auto add`를 권고했지만, 실제 코드/운영 구조상 shared repo에서 여러 세션이 같은 git index를 공유한다
- 따라서 `auto add`는 세션 간 staged 상태가 섞일 위험이 크다
- 그래서 실제 구현은 더 안전한 방식으로 조정했다
  - `수정 시 ledger only`
  - `finalize 시점에만 git add/commit/push`

이 정정은 기획 변경이 아니라, 실코드 제약을 반영한 보정이다.

### 2.3 2026-04-23 구현 현황 업데이트 — Chat UI 연결

백엔드만 추가된 상태를 넘어서, 채팅창에서 바로 쓸 수 있는 UI도 실제로 연결했다.

- 신규 프론트 컴포넌트:
  - `aads-dashboard/src/components/chat/ChatOpsDock.tsx`
- 채팅 페이지 연결:
  - `aads-dashboard/src/app/chat/page.tsx`

현재 UI에서 가능한 범위는 다음과 같다.

- `작업 상태` 패널
  - 현재 활성 채팅 세션 기준 workspace ledger 조회
  - `dirty / committed / pushed / deployed` 상태 카운트 표시
  - 파일 경로, repo, source tool, commit SHA, 배포 시각 표시
  - 수동 `Finalize` 버튼으로 `POST /api/v1/ops/workspace-changes/finalize` 호출
- `Terminal` 패널
  - terminal session 목록 조회/선택/생성/종료
  - SSE stream을 통한 출력 실시간 수신
  - 명령 실행, `Ctrl+C` 입력, 출력 clear, 최근 명령 재사용
  - `PTY`/`pipe` backend 상태 표시

이 단계에서 백엔드도 같이 보강했다.

- `workspace_change_tracker.py`
  - `deployed` 상태를 실제 ledger status로 기록 가능하게 확장
  - `deployed_at` 컬럼 보장
- `tool_executor.py`
  - deploy/restart 성격 명령이 preflight finalize 후 성공하면
  - 현재 채팅 세션의 pushed 변경을 `deployed`로 승격

즉 현재는 “반영 후보 추적”까지만이 아니라, “push 이후 배포 반영 여부”도 채팅창 상태판에서 확인 가능한 구조가 됐다.

추가 smoke test로 `dirty -> pushed -> deployed` 전이와 `deployed_at` 기록까지 로컬 DB에서 직접 검증했다.

배포 및 라이브 반영도 완료했다.

- `aads-server`: commit `68a5468`
- `aads-dashboard`: commit `52af866`
- 서버 배포: `bash /root/aads/aads-server/deploy.sh code`
- 대시보드 배포: `bash /root/aads/aads-dashboard/deploy.sh`
- 라이브 검증:
  - `GET /api/v1/health` → `status=ok`
  - OpenAPI에 terminal/workspace route 노출 확인
  - `GET /api/v1/ops/workspace-changes` 응답 확인
  - 대시보드 blue-green 배포 완료, 활성 슬롯 `blue`

---

## 3. 현재 구조 진단

## 3.1 도구 노출 범위

현재 `ceo_chat_tools.py`의 `TOOL_DEFINITIONS` 기준으로 채팅창에는 53개 도구가 노출된다.

대표 분류는 다음과 같다.

- 파일/명령: `read_remote_file`, `write_remote_file`, `patch_remote_file`, `run_remote_command`
- Git: `git_remote_add`, `git_remote_commit`, `git_remote_push`
- 운영: `search_logs`, `health_check`, `inspect_service`, `get_all_service_status`
- 실행: `pipeline_runner_submit`, `pipeline_runner_approve`, `execute_sandbox`
- 자동화: `schedule_task`
- 원격 제어: `pc_execute`
- 브라우저/검색/문서/메모리 등 일반 도구

## 3.2 실제 실행 경로

표면상 도구는 많지만, 실제 실행 경로는 대체로 아래 4개로 수렴한다.

1. `MCP Bridge -> ToolExecutor -> ceo_chat_tools`
2. `MCP Bridge -> ToolExecutor -> internal API`
3. `MCP Bridge -> ToolExecutor -> Pipeline Runner`
4. `MCP Bridge -> ToolExecutor -> Docker / SSH / PC Agent`

즉 채팅창은 이미 단순 대화 UI가 아니라 운영 자동화 허브처럼 동작하고 있다.

## 3.3 핵심 privileged surface

가장 중요한 privileged surface는 `run_remote_command`다.

- 허용 목록에 `systemctl`, `docker`, `nginx`, `git`, `journalctl`, `kill` 등이 포함된다.
- AADS는 `ssh root@host.docker.internal`로 실행한다.
- 원격 프로젝트는 `ssh root@<server>`로 실행한다.

근거:

- `aads-server/app/api/ceo_chat_tools.py:1242`
- `aads-server/app/api/ceo_chat_tools.py:2029`

이 구조는 사실상 `채팅창 -> root SSH bridge`다.

## 3.4 숨은 결합: 파일수정 = 운영반영

현재 `write_remote_file`, `patch_remote_file`는 단순 편집 도구가 아니다.

수정 후 자동으로 다음이 붙는다.

- hot reload
- git add
- git commit
- git push
- changelog 기록
- AI code review
- dashboard build trigger
- 일부 프로젝트 restart

근거:

- `aads-server/app/services/tool_executor.py:391`
- `aads-server/app/services/tool_executor.py:792`

즉 이름은 “파일 수정”이지만 실질적으로는 `수정 + 반영 + 운영 연쇄 작업`이다.

## 3.5 Pipeline Runner도 privileged path

`pipeline_runner_submit`, `pipeline_runner_approve`는 내부 API처럼 보이지만 실제로는 worktree, deploy lock, 승인 후 배포까지 이어지는 privileged 실행 경로다.

근거:

- `aads-server/app/services/tool_executor.py:2548`
- `aads-server/scripts/pipeline-runner.sh:1017`

## 3.6 Docker 의존 surface

다음은 Docker daemon 권한 의존성이 강하다.

- `search_logs`: `docker logs` 또는 `journalctl`
- `execute_sandbox`: `docker.from_env()` 후 컨테이너 실행
- `git_remote_commit`: `docker exec aads-server ... importlib ...`
- `ops health`: `docker.sock` 또는 socket proxy 조회

근거:

- `aads-server/app/api/ceo_chat_tools.py:1426`
- `aads-server/app/services/sandbox.py:27`
- `aads-server/app/api/ops.py:558`

## 3.7 별도 권한 경계

`pc_execute`는 로컬 root 권한과는 다르지만 별도 권한 경계다.

- WebSocket으로 연결된 PC Agent에 쉘/파일/프로세스 작업을 명령한다.
- 즉 서버 권한상승이 아니라 `원격 PC 제어 권한`이다.

근거:

- `aads-server/app/services/pc_agent_manager.py:61`

## 3.8 실코드 재검증 정정 사항

초기 보고서 작성 당시에는 개념적 보완 요구를 포함해 기술했지만, 실제 코드 재검증 결과 다음 항목은 이미 기반이 존재한다.

### 이미 존재하는 기반

1. 채팅 SSE 재연결 및 세션 복구

- 프론트는 `Last-Event-ID` 기반 재연결 상태를 유지한다.
- 서버는 `/chat/sessions/{id}/stream-resume`를 제공한다.
- Redis Stream 기반 토큰 재전송이 구현돼 있다.
- 서버 재시작 후 `resume_interrupted_streams()`와 `_resume_single_stream()`로 중단 응답 복구가 구현돼 있다.

근거:

- `aads-dashboard/src/app/chat/page.tsx:924`
- `aads-dashboard/src/app/chat/page.tsx:2293`
- `aads-server/app/routers/chat.py:251`
- `aads-server/app/routers/chat.py:327`
- `aads-server/app/services/chat_service.py:297`
- `aads-server/app/services/chat_service.py:777`
- `aads-server/app/services/redis_stream.py:50`
- `aads-server/app/services/stream_worker.py:23`

2. 장기 작업용 SSE/로그 기반

- `task_logger`는 DB 저장 + SSE 브로드캐스트를 제공한다.
- `task_monitor`는 활성 작업 조회, 로그 조회, 작업별 SSE 스트림을 제공한다.
- 즉 terminal runner를 새로 만들더라도 장기 실행 작업의 상태 전송층은 처음부터 새로 짤 필요가 없다.

근거:

- `aads-server/app/services/task_logger.py:57`
- `aads-server/app/api/task_monitor.py:13`
- `aads-server/app/api/task_monitor.py:144`

3. 락 인프라

- `deploy_lock.py`는 이미 work/file/deploy 3단계 잠금을 제공한다.
- `git_lock.py`는 cross-process flock 기반 git 락을 제공한다.
- ops API를 통해 외부에서도 잠금 획득/해제가 가능하다.

근거:

- `aads-server/app/services/deploy_lock.py:1`
- `aads-server/app/core/git_lock.py:1`
- `aads-server/app/api/ops.py:19`

4. 메시지 중복 전송 방지

- 채팅 전송에는 `idempotency_key`가 구현돼 있다.
- 이는 향후 terminal command 실행에도 유사하게 재활용 가능한 패턴이다.

근거:

- `aads-dashboard/src/app/chat/page.tsx:2190`
- `aads-server/app/services/chat_service.py:2013`

### 아직 없는 것

1. PTY 기반 터미널 세션 매니저

- 코드 전체 기준으로 `pty`, `forkpty`, `openpty`, `pexpect`, `terminal_session` 계층이 없다.
- 현재 subprocess 사용은 모두 비-PTY 파이프 방식이다.

근거:

- `aads-server/scripts/claude_relay_server.py:1011`
- `aads-server/app/api/ceo_chat_tools.py:2076`
- `aads-server/app/services/pipeline_runner_service.py:1050`
- 코드베이스 검색 결과 `terminal_session`, `pty_session` 계열 부재

2. 명령 세션 재접속/복구

- 채팅 메시지 스트림 복구는 존재하지만, 쉘 명령 세션의 `cwd/env/stdin/tty`를 이어붙이는 기능은 없다.
- 즉 “채팅 응답 복구”와 “터미널 세션 복구”는 전혀 다른 문제이며, 후자는 새로 구현해야 한다.

3. 일반 출력 마스킹 계층

- 민감정보 마스킹은 `context public summary` 쪽에 한정되어 있다.
- `task_logger`는 content를 그대로 SSE와 DB에 저장한다.
- 따라서 터미널 출력/작업 로그에 대한 공통 redaction 계층은 아직 없다.

근거:

- `aads-server/app/api/context.py:313`
- `aads-server/app/services/task_logger.py:71`

4. interactive shell UX

- 현재 채팅 UI는 SSE token stream과 placeholder 복구는 강하지만, PTY 입력/프롬프트/alternate screen을 처리하는 UI는 없다.
- 즉 `vim`, `top`, `less`, `mysql`, `psql`, shell prompt 입력 대기 등은 아직 설계/구현되지 않았다.

### 정정

이 항목 중 `PTY 기반 터미널 세션 매니저`는 본 보고서 작성 후 P0 백엔드가 추가됐다.  
다만 현재 런타임에서는 `devpts` 제약 때문에 `pipe fallback`으로 검증됐고, 완전한 PTY 경로는 배포 컨테이너에서 추가 확인이 필요하다.

### 정정

이 항목 중 `PTY 기반 터미널 세션 매니저`는 본 보고서 작성 후 P0 백엔드가 추가됐다.  
다만 현재 런타임에서는 `devpts` 제약 때문에 `pipe fallback`으로 검증됐고, 완전한 PTY 경로는 배포 컨테이너에서 추가 확인이 필요하다.

---

## 4. 현재 구조의 문제점

## 4.1 채팅창이 실제 터미널이 아니다

현재 채팅창은 다음이 부족하다.

- 지속 세션
- 현재 쉘 상태 유지
- TTY 기반 상호작용
- stdin 입력
- 작업 디렉터리 지속
- 환경변수 지속
- 장기 실행 프로세스 추적
- `fg/bg`에 준하는 세션 제어

즉 도구는 많지만 터미널 동등성은 없다.

## 4.2 권한 실패가 랜덤하게 보인다

사용자 체감상은 “채팅창이 권한 때문에 자꾸 실패한다”인데, 실제 원인은 매번 다르다.

- docker 권한
- ssh bridge 실패
- systemd 명령 실패
- git hook 실패
- push 실패
- build 실패
- runner down
- lock 충돌

하지만 UI에서는 거의 동일한 tool error처럼 보인다.

## 4.3 수정과 배포가 너무 붙어 있다

현재 구조에서는 작은 수정 1회에도 아래가 터질 수 있다.

- git commit
- git push
- dashboard build
- remote restart

그 결과:

- 권한 실패 빈도 상승
- 중간 상태가 너무 빨리 원격 반영
- 커밋 단위가 지나치게 잘게 쪼개짐
- 여러 파일 작업 중 partial state가 올라갈 수 있음

## 4.4 privileged 정책이 단일화돼 있지 않다

동일한 성격의 작업이 서로 다른 경로로 실행된다.

- 직접 `run_remote_command`
- 파일수정 후 hook
- Pipeline Runner
- Scheduler

이 상태에서는 정책, 감사로그, 실패처리, 락, 롤백이 모두 분산된다.

---

## 5. 요구사항 재정의

CEO 요구를 기술적으로 다시 정의하면 아래와 같다.

### 목표 상태

채팅창은 다음 조건을 만족해야 한다.

- 이 터미널과 같은 호스트 문맥에서 동작
- 임의 쉘 명령 실행 가능
- TTY 상호작용 가능
- 장기 실행 프로세스 유지 가능
- 파일 읽기/쓰기/명령/배포 모두 같은 실행 경로에서 처리 가능
- commit/push 누락 없이 배포 반영 보장

### 비목표

아래는 우선 목표가 아니다.

- 모든 작업을 구조화 도구로만 제한
- 각 도구에 안전 제한을 잔뜩 붙여 터미널 자유도를 떨어뜨리는 것
- 파일 수정 때마다 자동으로 즉시 원격 push하는 것

---

## 6. 선택지 비교

## 6.1 옵션 A: 현재 도구들에 권한만 더 부여

설명:

- `run_remote_command` 화이트리스트 확대
- Docker/systemctl/git 권한 더 개방
- 기존 파일수정 hook 유지

장점:

- 구현이 가장 빠름
- 기존 구조 변경이 적음

단점:

- 터미널 동등성은 여전히 없음
- privileged 정책 분산 유지
- 파일수정과 push/build/restart 결합 문제 지속
- 실패 원인 불명확 문제 해결 안 됨

판단:

- 비권고

## 6.2 옵션 B: privileged 작업은 전부 Runner로만 제한

설명:

- 채팅창은 구조화 도구만 사용
- 운영 작업은 전부 Pipeline Runner나 전용 API로만 실행

장점:

- 통제가 쉬움
- 운영 사고 경로를 줄일 수 있음

단점:

- CEO 요구인 “현재 터미널과 동등한 작업”을 만족시키지 못함
- 즉흥적인 운영/조사/실험/수동 복구 작업에 불편

판단:

- 안전하지만 CEO 요구와 불일치

## 6.3 옵션 C: Full Terminal Runner + Structured Tool 공존

설명:

- 채팅창에 실제 터미널 세션 기능 추가
- 구조화 도구는 유지
- 운영 변경성 작업은 같은 terminal runner 또는 그 wrapper로 통일

장점:

- CEO 요구 충족
- 실제 터미널과 동일한 작업 가능
- 실행 경로 단일화 가능
- 권한 경계와 감사로그 정리 가능

단점:

- 구현 난이도 높음
- 세션/TTY/스트리밍 관리 필요

판단:

- 최종 권고안

---

## 7. 권고 아키텍처

## 7.1 2모드 구조

채팅창은 아래 2개 모드로 동작하는 것이 가장 좋다.

### Standard Mode

- 기존 구조화 도구 위주
- 검색, 조회, 문서 읽기, 브라우저 검사, 일반 수정
- 빠르고 안정적

### Terminal Full Mode

- 실제 PTY 세션 생성
- 임의 쉘 명령 실행
- stdin/stdout 스트리밍
- cwd/env 지속
- long-running process 제어

핵심은 모드를 나누되, 실행 엔진은 하나의 terminal runner에 수렴시키는 것이다.

## 7.2 Terminal Runner 필수 기능

신규 runner는 최소 아래 기능이 필요하다.

- `terminal_open(session_id, cwd, env, tty=true)`
- `terminal_exec(session_id, command)`
- `terminal_stdin(session_id, data)`
- `terminal_poll(session_id)`
- `terminal_kill(session_id)`
- `terminal_close(session_id)`
- `terminal_list()`

기능 세부:

- PTY 지원
- stdin 입력
- stdout/stderr 스트리밍
- 종료 코드 반환
- 현재 작업 디렉터리 유지
- 환경변수 유지
- 세션별 프로세스 트리 관리
- 장기 실행 프로세스 추적

## 7.3 Host parity 원칙

CEO 요구를 만족하려면 runner는 아래 원칙을 따라야 한다.

- 현재 운영 터미널과 같은 host에서 실행
- 같은 user 컨텍스트 사용
- 같은 filesystem view 사용
- 같은 network reachability 사용
- 같은 docker/systemctl/git/nginx 권한 사용

즉 “도구”가 아니라 “실제 쉘 세션”이어야 한다.

## 7.4 Structured Tool의 위치 조정

구조화 도구는 유지하되 역할을 줄인다.

- 읽기/검색/문서/메모리/브라우저/검색: 유지
- 운영 변경 명령: terminal runner 또는 그 wrapper로 단일화
- legacy `run_remote_command`: 점진적 축소

즉 structured tool은 편의 레이어이고, 궁극 실행층은 terminal runner가 된다.

---

## 8. 파일수정/반영/배포 재설계

## 8.1 현 구조 문제

현재 `_post_file_modify_hook()`이 너무 많은 책임을 가진다.

- hot reload
- git add
- git commit
- git push
- changelog
- AI review
- dashboard build
- restart

이 구조는 “누락 방지”에는 유리했지만, 다음 부작용이 크다.

- 수정 1회마다 privileged 작업 발생
- 권한 실패 빈도 증가
- 중간 수정 상태가 너무 빨리 원격 반영
- 커밋 단위 파편화

## 8.2 목표 구조

아래 4단계로 분리하는 것이 가장 적절하다.

### 단계 1: Modify

- 파일 수정
- `git add <file>` 자동 수행
- `change ledger` 기록
- 필요 시 hot reload

### 단계 2: Finalize

- 세션 단위 또는 작업 단위로 commit
- push 수행
- 실패 시 dirty 상태 유지

### 단계 3: Deploy Preflight

배포 시작 전 반드시 검사:

- uncommitted change 존재 여부
- unpushed commit 존재 여부
- lock 충돌 여부
- health 상태

필요 시:

- auto commit
- auto push

실패 시:

- 배포 차단

### 단계 4: Deploy

- preflight 통과 후에만 수행
- 결과와 SHA 기록

## 8.3 왜 이 구조가 좋은가

- 파일 수정 누락 방지 유지
- commit/push 누락도 배포 직전에 최종 차단
- 중간 수정 상태의 과도한 push 감소
- 커밋 단위가 작업 단위로 정리
- privileged 작업 횟수 감소

---

## 9. 우선순위별 개선안

## P0-1. Full Terminal Runner 도입

### 목표

채팅창이 실제 터미널과 동등한 실행 경로를 갖도록 한다.

### 산출물

- PTY 세션 매니저
- session open/exec/stdin/poll/kill/close API
- 채팅창 스트리밍 연결

### 적용 대상 파일

- 신규 `aads-server/app/services/terminal_runner.py`
- 신규 `aads-server/app/api/terminal.py`
- `aads-server/app/services/tool_executor.py`
- `aads-server/app/api/ceo_chat_tools.py`
- `aads-dashboard/src/app/chat/page.tsx`

### 우선순위 이유

이게 없으면 채팅창은 도구 모음일 뿐, 터미널이 아니다.

## P0-2. 운영 변경 작업의 실행층 단일화

### 목표

아래 작업을 전부 같은 실행층으로 모은다.

- docker
- systemctl
- nginx
- git push
- deploy
- journalctl
- ssh

### 조치

- `run_remote_command`는 점진적으로 read-only/diagnostic 중심으로 축소
- mutating 명령은 terminal runner wrapper 또는 privileged executor로 단일화
- Scheduler의 `remote_command`도 동일 실행층 사용

### 우선순위 이유

현재는 privileged 정책이 분산되어 있어 운영 일관성이 없다.

## P0-3. 파일수정과 commit/push/deploy 분리

### 목표

파일수정 직후 즉시 push를 제거하고, 누락 방지는 유지한다.

### 조치

- `_post_file_modify_hook()`에서 즉시 `commit/push` 제거
- `auto add + ledger + hot reload + changelog`까지만 수행
- `finalize service` 추가
- `deploy preflight`에서 auto finalize/auto push 강제

### 적용 대상 파일

- `aads-server/app/services/tool_executor.py`
- 신규 `aads-server/app/services/workspace_finalize_service.py`
- `aads-server/scripts/pipeline-runner.sh`

### 우선순위 이유

반영 누락과 권한 실패를 동시에 줄이는 핵심 축이다.

## P1-1. 채팅창 권한 모드/배지 도입

### 목표

사용자가 지금 어떤 권한 경로를 쓰는지 명확히 보이게 한다.

### UI 항목

- `Standard`
- `Terminal Full`
- 도구별 배지:
  - `Green`: 일반 조회/분석
  - `Yellow`: 상태 변경 가능
  - `Red`: 운영/배포/시스템 영향

### 추가 표시

- 실행 경로: `tool`, `runner`, `remote pc`
- 실패 이유: `docker`, `ssh`, `git`, `lock`, `timeout`, `runner`

## P1-2. Change Ledger + Audit Log

### 목표

“누가 무엇을 바꿨고 어디까지 반영됐는가”를 구조화한다.

### 권고 필드

- `session_id`
- `project`
- `repo`
- `file_path`
- `status`: `dirty`, `committed`, `pushed`, `deployed`
- `last_modified_at`
- `commit_sha`
- `deploy_id`

### 효과

- 반영 누락 추적 가능
- 배포 전 미반영 상태 자동 감지 가능
- 운영 감사 로그 확보

## P1-3. Failure Taxonomy 정리

### 목표

실패 원인을 사람이 바로 판별할 수 있게 한다.

예시:

- `PERMISSION_DOCKER`
- `PERMISSION_SYSTEMD`
- `PERMISSION_GIT_PUSH`
- `RUNNER_DOWN`
- `LOCK_CONFLICT`
- `PRECHECK_FAILED`
- `REMOTE_SSH_FAILED`

현재처럼 generic tool error로 섞여 보이면 운영성이 계속 떨어진다.

## P2-1. Legacy Tool 축소

- `run_remote_command`는 read-only 중심 재정의
- `write_remote_file`/`patch_remote_file`는 terminal parity 구조와 역할 중복 시 축소
- hook 기반 build/restart 자동화는 preflight 기반으로 재정렬

## P2-2. Sandbox / PC Agent / Scheduler 정합화

- `execute_sandbox`의 docker 의존 상태 UI 노출
- `pc_execute`는 별도 권한 영역으로 표시
- `schedule_task(remote_command)`는 terminal runner wrapper 경유

---

## 10. 구현 우선순위 상세 제안

## 10.1 1차 구현 범위

반드시 먼저 할 것:

1. Terminal Runner 신규 도입
2. 채팅 UI에 Terminal Full Mode 추가
3. `_post_file_modify_hook()`에서 즉시 `commit/push` 제거
4. change ledger 추가
5. deploy preflight finalize 강제

이 5개가 끝나면 체감 문제가 가장 크게 줄어든다.

## 10.2 2차 구현 범위

다음으로 할 것:

1. run_remote_command 재정의
2. failure taxonomy 정리
3. scheduler remote_command 단일화
4. structured audit log 대시보드 반영

## 10.3 3차 구현 범위

마지막으로 할 것:

1. legacy tool 축소
2. terminal parity 중심 UX 정리
3. PC Agent / Sandbox / Runner 통합 상태판

---

## 11. 권고 데이터 모델

예시 테이블: `chat_workspace_changes`

- `id`
- `session_id`
- `project`
- `repo`
- `file_path`
- `change_summary`
- `change_status`
- `commit_sha`
- `deploy_id`
- `created_at`
- `updated_at`

예시 테이블: `terminal_sessions`

- `session_id`
- `owner_session_id`
- `cwd`
- `env_snapshot`
- `started_at`
- `last_activity_at`
- `status`
- `pid`

예시 테이블: `terminal_audit_log`

- `id`
- `terminal_session_id`
- `command`
- `cwd`
- `exit_code`
- `started_at`
- `finished_at`
- `stdout_size`
- `stderr_size`

---

## 12. 배포/운영 관점 체크포인트

## 12.1 배포 전

- dirty repo 여부
- unpushed commit 여부
- lock 여부
- target health
- active dashboard slot

## 12.2 배포 중

- 실행 커맨드
- commit SHA
- container switch 결과
- service restart 결과

## 12.3 배포 후

- health check
- chat stream 정상 여부
- docs/Claude health 상태
- dashboard upstream 상태

---

## 13. 하면 안 되는 것

아래 방식은 비권고다.

- 현재 `run_remote_command` 화이트리스트만 더 넓히기
- 파일수정 후 즉시 commit/push를 계속 유지하기
- 배포를 파일수정 hook에서 바로 연동하기
- privileged 작업을 runner, scheduler, direct command에 계속 분산 유지하기
- 채팅창을 실제 터미널처럼 보이게만 하고 세션/PTY를 제공하지 않기

---

## 14. 최종 권고안

### 최종 아키텍처 문장

`채팅창은 실제 터미널 세션을 소유해야 한다. 구조화 도구는 편의 계층으로 유지하되, 운영 변경 작업은 terminal runner로 단일화한다. 파일수정은 auto add + ledger까지만 수행하고, commit/push는 finalize 및 deploy preflight에서 강제한다.`

### 최우선 실행 순서

1. `Full Terminal Runner` 구현
2. `채팅 UI Terminal Full Mode` 추가
3. `파일수정 후 즉시 commit/push 제거`
4. `Change Ledger + Finalize Service` 추가
5. `Deploy Preflight 강제`

### 기대 효과

- 채팅창이 실제 터미널과 동등해짐
- 권한 문제 체감 급감
- commit/push 누락 방지 유지
- 배포 반영 누락 차단
- 운영 추적성과 원인 분석성 향상

---

## 15. 참고 코드 근거

- `aads-server/app/api/ceo_chat_tools.py:1242`
- `aads-server/app/api/ceo_chat_tools.py:2029`
- `aads-server/app/api/ceo_chat_tools.py:2177`
- `aads-server/app/api/ceo_chat_tools.py:1426`
- `aads-server/app/services/tool_executor.py:391`
- `aads-server/app/services/tool_executor.py:792`
- `aads-server/app/services/tool_executor.py:2548`
- `aads-server/app/services/tool_executor.py:1379`
- `aads-server/scripts/pipeline-runner.sh:1017`
- `aads-server/app/services/sandbox.py:27`
- `aads-server/app/api/ops.py:558`
- `aads-server/app/services/pc_agent_manager.py:61`
