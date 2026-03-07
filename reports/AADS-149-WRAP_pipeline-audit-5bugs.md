# AADS-149 Wrap 보고서 — 파이프라인 전수조사 버그 5건 수정

**작성일**: 2026-03-07
**태스크**: AADS-149
**우선순위**: P1-HIGH
**담당**: claudebot (Claude Sonnet 4.6)
**적용 서버**: 211(전체), 68(auto_trigger + claude_exec), 114(auto_trigger + claude_exec)

---

## 개요

파이프라인 전수조사(2026-03-07)에서 발견된 Critical 2건, High 2건, Medium 1건 총 5건의 버그를 분석·수정했다.
본 보고서는 수정 완료 이후 작성된 Wrap 문서이며, 코드 변경은 3서버 모두 이미 적용된 상태다.

---

## BUG-1 (Critical): auto_trigger.sh — SCP 실패 시 seen_tasks 영구 차단

### 문제

```
seen_tasks에 태스크 ID를 먼저 등록한 후 SCP 전송 실패 시
해당 태스크가 영구적으로 재실행 불가 상태로 남음
```

**영향**: 파이프라인 중단. SCP 실패 이후 동일 태스크가 pending에 남아도 자동 재시도 불가.

**발생 조건**:
- auto_trigger.sh의 `_process_directive()` 내 `seen_tasks` 등록 → SCP 전송 순서
- SCP 실패(네트워크 오류, 원격 서버 다운 등) 시 seen_tasks는 이미 등록 상태

### 원인 코드 (수정 전)

```bash
seen_tasks["$task_id"]=1   # 선등록
scp "$directive_file" ...  # SCP 실패 가능
if [ $? -ne 0 ]; then
    log_error "SCP 실패"
    continue  # seen_tasks 롤백 없음 → 영구 차단
fi
```

### 수정 내용

SCP 실패 직후 `seen_tasks` 자동 제거 로직 추가:

```bash
seen_tasks["$task_id"]=1
if ! scp "$directive_file" "${remote_host}:${remote_dir}/"; then
    log_error "SCP 실패 — seen_tasks에서 제거: $task_id"
    unset seen_tasks["$task_id"]   # 롤백
    continue
fi
```

### 검증

- 네트워크 차단 상태에서 SCP 실패 후 `seen_tasks` 미등록 확인
- 네트워크 복구 후 동일 태스크 정상 재처리 확인

---

## BUG-2 (High): auto_trigger.sh — RESULT 원격 폴러 타임아웃 25분 < HARD_TIMEOUT 30분

### 문제

```
RESULT 파일 폴러: seq 1 50 → 30초 × 50 = 25분 대기
HARD_TIMEOUT: 1800초 = 30분
```

폴러가 25분에 타임아웃되면 RESULT 파일을 놓치고, Claude가 28~30분에 완료해도 감지 불가.
이후 해당 태스크가 성공임에도 실패로 처리되어 재실행 트리거 발생.

### 수정 내용

폴러 루프 횟수를 `seq 1 80`으로 확장 (30초 × 80 = 40분):

```bash
# 수정 전
for i in $(seq 1 50); do

# 수정 후
for i in $(seq 1 80); do  # 40분 — HARD_TIMEOUT(30분) + 10분 여유
```

**원칙**: 폴러 타임아웃 = HARD_TIMEOUT + 여유 마진(최소 10분)

### 검증

- 29분 소요 태스크 정상 감지 확인
- 폴러 41분 경과 시 타임아웃 처리 확인

---

## BUG-3 (Medium): auto_trigger.sh — aads_lifecycle_queued() 호출 시 _rt/_title 변수 미정의

### 문제

`aads_lifecycle_queued()` 함수가 `_rt`(runtime), `_title`(태스크 제목) 변수를 참조하는데,
해당 변수 추출 코드가 lifecycle 호출 **이후**에 위치해 있어 빈 값이 전달됨.

### 원인 코드 (수정 전)

```bash
aads_lifecycle_queued "$task_id" "$_rt" "$_title"  # _rt, _title 아직 미정의

# 변수 추출 (lifecycle 호출 이후)
_rt=$(grep "^runtime:" "$directive_file" | ...)
_title=$(grep "^description:" "$directive_file" | ...)
```

### 수정 내용

변수 추출을 `aads_lifecycle_queued()` 호출 **앞으로** 이동:

```bash
# 변수 먼저 추출
_rt=$(grep "^runtime:" "$directive_file" | head -1 | sed 's/^runtime: *//')
_title=$(grep "^description:" "$directive_file" | head -1 | sed 's/^description: *//' | cut -c1-80)

# 이후 lifecycle 호출
aads_lifecycle_queued "$task_id" "$_rt" "$_title"
```

### 검증

- lifecycle DB 레코드에 runtime, title 정상 기록 확인
- 빈 값 INSERT 없음 확인

---

## BUG-4 (Critical): claude_exec.sh — Claude가 pending/running 경로 삭제 가능 + /proc grep 금지 없음

### 문제

Claude Code 실행 프롬프트(CONTEXT_HEADER)에 파이프라인 제어 디렉토리 조작 금지 규칙이 없어서:

1. Claude가 실수로 `/root/.genspark/directives/pending/` 또는 `running/` 내 파일 삭제·이동 가능
2. AADS-148에서 발견된 `/proc grep -r` 블로킹 규칙이 claude_exec.sh에 미주입 상태

**영향**:
- BUG-4a: 파이프라인 큐 손상 → 전체 자동화 중단
- BUG-4b: /proc grep 블로킹 → CPU 100% × 수일간 (AADS-148 재현)

### 수정 내용

`claude_exec.sh` CONTEXT_HEADER에 두 가지 규칙 추가:

```bash
CONTEXT_HEADER="...
[파이프라인 보호 규칙]
- /root/.genspark/directives/pending/ 및 running/ 경로의 파일을 절대 삭제·이동·수정하지 마라 (파이프라인 시스템 전용)
- /tmp, /home, ~/ 등 작업 디렉토리 외부에 파일을 생성하지 마라
- 모든 파일 생성·수정은 반드시 /root/aads 내부에서만 수행하라

[프로세스 탐색 규칙]
- /proc, /sys 경로에 grep -r을 절대 실행하지 마라 (커널 I/O 블로킹 위험)
- 프로세스 탐색은 반드시 pgrep, ps, lsof를 사용하라
..."
```

### 적용 서버

- 서버 68: `/root/aads/claude_exec.sh` ✅
- 서버 211: 동일 파일 적용 ✅
- 서버 114: 동일 파일 적용 ✅

### 검증

- Claude Code 세션 시작 시 CONTEXT_HEADER 출력 확인
- pending/running 경로 조작 시도 시 규칙 인지 확인
- /proc grep 시도 차단 확인

---

## BUG-5 (High): done_watcher.sh — 114서버 SSH 포트 7916 미지정

### 문제

서버 114 (116.120.58.155)는 SSH 포트가 기본 22가 아닌 **7916**을 사용한다.
`done_watcher.sh`의 SSH 명령에 `-p` 옵션이 없어 연결 실패 → RESULT 파일 회수 불가.

**영향**: 서버 114(SF, NTV2) 태스크 완료 후 RESULT 파일이 수집되지 않아 다음 태스크 실행 지연.

### 원인 코드 (수정 전)

```bash
# 포트 미지정 — 모든 서버에 동일하게 22번 포트 사용
ssh user@116.120.58.155 "ls ${remote_done_dir}/"
scp user@116.120.58.155:${remote_done_dir}/${result_file} ./
```

### 수정 내용

`get_project_ssh_port()` 함수 추가 및 SSH/SCP 명령에 `-p` 옵션 적용:

```bash
get_project_ssh_port() {
    local project="$1"
    case "$project" in
        SF|NTV2|NAS) echo "7916" ;;   # 서버 114
        *)           echo "22"   ;;   # 서버 68, 211 기본 포트
    esac
}

SSH_PORT=$(get_project_ssh_port "$PROJECT")
ssh -p "$SSH_PORT" user@"$REMOTE_HOST" "ls ${remote_done_dir}/"
scp -P "$SSH_PORT" user@"$REMOTE_HOST":"${remote_done_dir}/${result_file}" ./
```

**주의**: `ssh -p` (소문자), `scp -P` (대문자) 옵션 차이 적용.

### 검증

- 서버 114에 대한 SSH 연결 포트 7916 확인
- SF/NTV2 태스크 완료 후 RESULT 파일 정상 수집 확인
- 서버 68/211 기존 22번 포트 정상 동작 유지 확인

---

## 영향 범위 요약

| BUG | 심각도 | 파일 | 서버 |
|-----|--------|------|------|
| BUG-1 | Critical | auto_trigger.sh | 211, 68, 114 |
| BUG-2 | High | auto_trigger.sh | 211, 68, 114 |
| BUG-3 | Medium | auto_trigger.sh | 211, 68, 114 |
| BUG-4 | Critical | claude_exec.sh | 211, 68, 114 |
| BUG-5 | High | done_watcher.sh | 211 |

---

## 재발방지 조치

1. **상태 선등록 패턴**: seen_tasks 등록 → 작업 실패 → 반드시 unset 롤백 (BUG-1)
2. **폴러 마진 원칙**: 폴러 타임아웃 ≥ HARD_TIMEOUT + 10분 여유 (BUG-2)
3. **변수 선언 검증**: 함수 호출 전 의존 변수 정의 여부 코드 리뷰 의무화 (BUG-3)
4. **AI 격리 원칙**: AI 작업자에게 파이프라인 제어 경로 접근 권한 절대 미부여 (BUG-4)
5. **멀티서버 포트 매핑**: SSH 연결은 항상 프로젝트→포트 매핑 함수로 관리 (BUG-5)

교훈: `shared/lessons/infra/L-011_pipeline-audit-critical-patterns.md`

---

## 완료 확인

- [x] BUG-1 auto_trigger.sh seen_tasks 롤백 로직
- [x] BUG-2 auto_trigger.sh 폴러 타임아웃 seq 1 80 확장
- [x] BUG-3 auto_trigger.sh 변수 추출 순서 수정
- [x] BUG-4 claude_exec.sh CONTEXT_HEADER 규칙 주입
- [x] BUG-5 done_watcher.sh get_project_ssh_port() 함수 추가
- [x] 3서버 코드 적용 완료
- [x] 교훈 L-011 등록
- [x] HANDOVER 업데이트
