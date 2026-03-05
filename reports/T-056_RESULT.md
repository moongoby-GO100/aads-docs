---
project: AADS
task_id: T-056
title: Dashboard Docker 재빌드 + docker 그룹 추가
completed_at: 2026-03-05 10:21 KST
status: PARTIAL_OK
---
## 결과
- claudebot docker 그룹 추가: FAIL (Permission denied — claudebot은 root 권한 없음, usermod 실패)
- Dashboard 재빌드: OK (10:07~10:08 KST root 프로세스가 T-049 rebuild_dashboard.sh 통해 선행 완료)
  - 빌드 이미지: aads-server-aads-dashboard (sha256:cfdfecfbeee8...)
  - 배포 시각: 2026-03-05 10:10:21 KST
  - 실행 중 PID: 5238 (Docker cgroup: dc6c14b59bf3...)
- https://aads.newtalk.kr/ HTTP: 307
- https://aads.newtalk.kr/settings HTTP: 200

## 상세 내용
- claudebot 사용자는 docker 그룹에 미포함 (docker:x:993: 비어있음)
- usermod -aG docker claudebot → Permission denied (claudebot uid=1002, /etc/passwd 쓰기 불가)
- gpasswd -a claudebot docker → Permission denied (gshadow 접근 불가)
- 직접 docker 명령 실행 → /var/run/docker.sock 접근 거부 (srw-rw---- root docker)
- git pull origin main → 실패 (로컬 브랜치가 origin/main보다 3커밋 앞섬 + .git/logs 권한 없음)
- 로컬 코드: a0125ae [AADS] feat: T-049 CEO dashboard 7 pages + dark theme + SaaS admin console
- Docker build T-056 시도 → permission denied (buildx 및 docker.sock 접근 불가)

## 선행 완료 확인
- T-049 rebuild_dashboard.sh 스크립트가 root로 실행되어 오늘(10:07~10:21) 빌드+배포 완료
- 현재 aads-dashboard 컨테이너: 정상 동작 중 (307→200 확인)

## 권고사항
- claudebot을 docker 그룹에 추가하려면 root로 수동 실행 필요:
  `usermod -aG docker claudebot`
- 이후 claudebot 세션 재시작하면 docker 명령 사용 가능
