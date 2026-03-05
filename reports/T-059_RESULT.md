# T-059 결과 보고서: 대시보드 Docker 재빌드 + claudebot docker 그룹 등록

## 작업 개요
- **Task ID**: T-059
- **제목**: 대시보드 Docker 재빌드 + claudebot docker 그룹 등록
- **서버**: 68 (aads.newtalk.kr)
- **우선순위**: P1-HIGH
- **완료 시각**: 2026-03-05 10:53 KST

---

## Step 1: claudebot docker 그룹 추가

### 실행 명령
```
usermod -aG docker claudebot
id claudebot | grep docker
```

### 결과
```
[Step 1] Adding claudebot to docker group...
[Step 1] Result: uid=1002(claudebot) gid=1002(claudebot) groups=1002(claudebot),0(root),993(docker)
```

**상태**: ✅ 완료 — docker 그룹(gid=993) 포함 확인

---

## Step 2: 대시보드 최신 코드 pull + 빌드

### git pull 결과
```
From https://github.com/moongoby-GO100/aads-dashboard
 * branch            main       -> FETCH_HEAD
   2ea0348..a0125ae  main       -> origin/main
Already up to date.
HEAD: a0125ae [AADS] feat: T-049 CEO dashboard 7 pages + dark theme + SaaS admin console
```

### docker compose build 결과 (docker-compose.prod.yml 사용)
```
time="2026-03-05T10:50:46+09:00" level=warning msg="/root/aads/aads-server/docker-compose.prod.yml: `version` is obsolete"
#0 building with "default" instance using docker driver

#1 [aads-dashboard internal] load build definition from Dockerfile
#1 transferring dockerfile: 522B done
#1 DONE 0.0s

#2 [aads-dashboard internal] load metadata for docker.io/library/node:20-alpine
#2 DONE 1.4s

#3 [aads-dashboard internal] load .dockerignore
#3 transferring context: 2B done
#3 DONE 0.0s

#4 [aads-dashboard builder 1/6] FROM docker.io/library/node:20-alpine@sha256:09e2b3d9726018aecf269bd35325f46bf75046a643a66d28360ec71132750ec8
#4 DONE 0.0s

#5 [aads-dashboard internal] load build context
#5 transferring context: 3.44MB 1.4s done
#5 DONE 1.5s

#6 [aads-dashboard builder 2/6] WORKDIR /app
#6 CACHED

#7 [aads-dashboard builder 3/6] COPY package*.json ./
#7 CACHED

#8 [aads-dashboard builder 4/6] RUN npm ci
#8 CACHED

#9 [aads-dashboard builder 5/6] COPY . .
#9 DONE 17.8s

#10 [aads-dashboard builder 6/6] RUN npm run build
#10 1.187
#10 1.187 > aads-dashboard@0.1.0 build
#10 1.187 > next build
#10 1.187
#10 3.576 ▲ Next.js 16.1.6 (Turbopack)
#10 3.576 - Environments: .env.local
#10 3.577
#10 3.581 ⚠ The "middleware" file convention is deprecated. Please use "proxy" instead.
#10 3.651   Creating an optimized production build ...
#10 19.37 ✓ Compiled successfully in 14.4s
#10 19.39   Running TypeScript ...
```

### docker compose up -d 결과
```
docker ps 출력:
CONTAINER ID   IMAGE                        COMMAND                  CREATED         STATUS                            PORTS                                                      NAMES
44d896ccd0b3   aads-server-aads-dashboard   "docker-entrypoint.s…"   Up (seconds)    0.0.0.0:3100->3100/tcp            aads-dashboard
```

### 컨테이너 기동 로그
```
▲ Next.js 16.1.6
- Local:         http://44d896ccd0b3:3100
- Network:       http://44d896ccd0b3:3100

✓ Starting...
✓ Ready in 144ms
```

**상태**: ✅ 완료 — aads-dashboard 컨테이너 신규 이미지(a0125ae)로 재기동

---

## Step 3: 검증 (6개 URL HTTP 응답 코드)

```
curl -sL -o /dev/null -w "%{http_code}" https://aads.newtalk.kr/
curl -sL -o /dev/null -w "%{http_code}" https://aads.newtalk.kr/project-status
curl -sL -o /dev/null -w "%{http_code}" https://aads.newtalk.kr/conversations
curl -sL -o /dev/null -w "%{http_code}" https://aads.newtalk.kr/managers
curl -sL -o /dev/null -w "%{http_code}" https://aads.newtalk.kr/decisions
curl -sL -o /dev/null -w "%{http_code}" https://aads.newtalk.kr/settings
```

| URL | HTTP 코드 | 결과 |
|-----|-----------|------|
| https://aads.newtalk.kr/ | 200 | ✅ |
| https://aads.newtalk.kr/project-status | 200 | ✅ |
| https://aads.newtalk.kr/conversations | 200 | ✅ |
| https://aads.newtalk.kr/managers | 200 | ✅ |
| https://aads.newtalk.kr/decisions | 200 | ✅ |
| https://aads.newtalk.kr/settings | 200 | ✅ |

**상태**: ✅ 전체 6개 URL 200 반환

---

## 참고: Health Check 이슈

docker-compose.prod.yml의 aads-dashboard healthcheck 설정에 버그 존재:
- `test: ["CMD", "curl", "-f", "http://localhost:3000"]`
- 문제 1: node:20-alpine 이미지에 curl 미설치
- 문제 2: 포트 3000 사용 (실제 서비스 포트는 3100)
- 영향: 컨테이너 Health 상태가 "starting"으로 유지됨 (실제 서비스는 정상 작동)
- 권고: healthcheck를 wget 또는 node 기반으로 수정 필요 (별도 태스크)

---

## 완료 기준 체크

| 기준 | 결과 |
|------|------|
| id claudebot 출력에 docker 그룹 포함 | ✅ groups=1002(claudebot),0(root),993(docker) |
| docker ps에서 aads-dashboard 최신 이미지로 실행 중 | ✅ aads-server-aads-dashboard (a0125ae 기준 빌드) |
| 6개 URL 모두 200 반환 | ✅ 전체 200 |
| 보고서 push 완료 | ✅ (본 파일) |

---

## 전체 결론

**T-059 완료** — claudebot docker 그룹 등록, 대시보드 최신 코드(a0125ae) Docker 재빌드 및 배포, 6개 URL 모두 HTTP 200 정상 응답 확인.
