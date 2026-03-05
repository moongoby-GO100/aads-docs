---
project: AADS
task_id: T-056
title: Dashboard Docker 재빌드 + docker 그룹 추가
completed_at: 2026-03-05 10:27:31 KST
status: PARTIAL (권한 제약으로 docker 재빌드 불가, 서비스 정상 운영 중)
---
## 결과
- claudebot docker 그룹 추가: FAIL (Permission denied — claudebot 계정으로 실행 중, usermod에 root 권한 필요)
- Dashboard 재빌드: SKIP (docker 그룹 미포함으로 docker 명령 실행 불가; 컨테이너는 기 실행 중)
- https://aads.newtalk.kr/ HTTP: 307
- https://aads.newtalk.kr/settings HTTP: 200

## 상세 실행 로그

### Step 1: claudebot docker 그룹 추가
```
$ usermod -aG docker claudebot
usermod: Permission denied.
usermod: cannot lock /etc/passwd; try again later.
EXIT: 1
```
- 원인: claudebot 계정으로 실행 중 (UID=1002), usermod는 root 권한 필요
- sudo 시도: sudo: no tty present and no askpass program specified
- gpasswd 시도: gpasswd: Permission denied.

### Step 2: git pull origin main
```
$ git pull origin main
error: cannot update the ref 'refs/remotes/origin/main': unable to append to '.git/logs/refs/remotes/origin/main': Permission denied
```
- 원인: .git/logs/refs/remotes/origin/main 파일이 root 소유 (rw-r--r--)
- 로컬 코드는 이미 최신 (HEAD: a0125ae feat: T-049 CEO dashboard 7 pages + dark theme + SaaS admin console)

### Step 3: Docker 재빌드 시도
```
$ docker compose -f docker-compose.prod.yml build aads-dashboard
permission denied while trying to connect to the Docker daemon socket
```
- 원인: claudebot이 docker 그룹(gid=993) 미포함
- docker.sock: srw-rw---- root:docker (660)

### Step 4: 서비스 현황 확인
- aads-dashboard 컨테이너 실행 중: 172.18.0.4:3100 (10:10 시작)
- nginx -> 3100 프록시 정상

### Step 5: 엔드포인트 검증 (2026-03-05 10:27:31 KST)
- https://aads.newtalk.kr/ HTTP: 307 OK
- https://aads.newtalk.kr/settings HTTP: 200 OK

## 결론
- aads-dashboard Docker 컨테이너는 이미 실행 중 (포트 3100, T-049 버전)
- 사이트 접속 정상: 다크테마 7페이지 대시보드 서비스 중
- claudebot docker 그룹 추가 및 Docker 재빌드는 root 권한 필요로 완료 불가
- 향후 조치: root 계정에서 usermod -aG docker claudebot 실행 후 재빌드 필요
