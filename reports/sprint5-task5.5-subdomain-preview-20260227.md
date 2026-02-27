# Task 5.5: 서브도메인 자동 배포 (프리뷰)

- **일시**: 2026-02-27 KST
- **Sprint**: 5 — Task 5.5

## 변경 사항

| 대상 | 내용 |
|------|------|
| 서버 `/etc/nginx/conf.d/aads.conf` | HTTP/HTTPS 양쪽에 `include .../aads-previews/*.conf` 추가 (기존 SSL·API·Dashboard 유지) |
| 서버 `/etc/nginx/conf.d/aads-previews/` | 프로젝트별 프록시 설정 디렉터리 생성, `00-placeholder.conf`로 include 정상 동작 보장 |
| aads-core `core/deployer.py` | `create_nginx_preview`, `remove_nginx_preview` 추가; `deploy_project`에서 배포 성공 시 Nginx 설정 생성·reload; `stop_project`에서 프리뷰 설정 제거 |

## 프리뷰 URL 형식

- `https://aads.newtalk.kr/preview/{project_id 앞 8자리}/`
- `/preview/{short_id}` 접근 시 `/preview/{short_id}/` 로 301 리다이렉트

## 검증

| 항목 | 결과 |
|------|------|
| nginx -t | syntax is ok, test is successful |
| systemctl reload nginx | 성공 |
| aads-core push | main 푸시 완료 (41fb69c, a86afd7) |

## 적용·배포

- **적용**: ✅ aads.conf 수정, aads-previews 디렉터리 생성, deployer.py 수정 반영
- **배포**: ✅ Nginx reload 완료; aads-core Git push 완료

## 진행/체크사항

- [ ] 실제 프로젝트 배포 후 `https://aads.newtalk.kr/preview/{short_id}/` 접속 확인
- [ ] deployer 실행 시 Nginx 설정 쓰기·reload 권한 확인 (root 또는 sudo)
- ⚠️ `create_nginx_preview`/`remove_nginx_preview`는 `/etc/nginx` 쓰기 및 `systemctl reload nginx` 필요

## 다음 작업 제안

- API/대시보드에서 배포 완료 시 `external_url` 노출 및 프리뷰 링크 클릭 테스트
