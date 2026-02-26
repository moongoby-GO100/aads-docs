# Node.js 20 업그레이드 + 대시보드 빌드

**일시**: 2026-02-26 21:35 KST

## 업그레이드 요약

| 항목 | 이전 | 이후 |
|------|------|------|
| Node.js | 18.19.1 | v20.20.0 (nvm) |
| npm | 10.2.4 | 10.8.2 |

## 설치 방법

- **yum(NodeSource)**: CentOS 7 glibc 2.17 제약으로 Node 20 패키지 설치 실패 (glibc ≥ 2.28 필요).
- **nvm + unofficial-builds**: `NVM_NODEJS_ORG_MIRROR=https://unofficial-builds.nodejs.org/download/release`, `nvm_get_arch() => x64-glibc-217` 설정 후 `nvm install 20`으로 **glibc 2.17 호환** Node 20.20.0 설치 완료.
- 프로젝트 사용 시 셸에서 `nvm use 20` 또는 `nvm alias default 20` 적용 필요.

## 대시보드 빌드

- **npm install**: `--legacy-peer-deps` 사용 (일부 postinstall 이슈 회피).
- **next.config.ts**: `output: "standalone"` 추가 (Docker 빌드용).
- **타입 수정**: `dashboard/src/app/page.tsx` setProjects 인자 `(p as Record<string, unknown>[])`, `dashboard/src/hooks/useRealtimePipeline.ts` map 콜백 인자 `PipelineStage`로 수정.
- **npm run build**: 종료코드 0, 성공.
- **standalone 출력**: `dashboard/.next/standalone` 생성 확인 (server.js, node_modules 포함).

## next.config

- `output: "standalone"` 설정 완료 (Docker/프로덕션 배포용).

## 적용/배포

- 적용: aads-core 소스 반영 (next.config, page.tsx, useRealtimePipeline.ts, .env.local 빌드용).
- 배포: 미배포 (로컬 빌드 검증만 수행). CI/Docker에서 Node 20 사용 시 동일 절차 또는 공식 Node 20 이미지 사용 권장.

## 체크사항

- [ ] Dockerfile 등에서 Node 20 사용 시 `nvm` 또는 공식 `node:20` 이미지 사용.
- [ ] `.env.local` 실제 키는 버전 관리 제외 유지 (빌드 시에만 사용).
