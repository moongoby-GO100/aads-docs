# Task 5.4: Docker 자동 빌드 + 배포

- **일시**: 2026-02-27 KST
- **Sprint**: 5 — Task 5.4

## 변경 사항
| 파일 | 내용 |
|------|------|
| core/deployer.py | Dockerfile 생성, 빌드, 컨테이너 실행, 상태 관리 |
| api/main.py | /build, /deploy, /deploy-status, /stop 엔드포인트 |

## 기능
- 프로젝트 코드 분석 → 적절한 Dockerfile 자동 생성
- docker build → docker run (포트 9000~9999 동적 할당)
- 컨테이너 메모리 256MB, CPU 0.5 제한
- Redis에 배포 상태 저장

## 검증
| 항목 | 결과 |
|------|------|
| deployer.py 문법 | OK |
| API health | `{"status":"healthy","time":"2026-02-27T17:50:11+09:00"}` |

## 적용/배포
- 적용: ✅ aads-core 반영
- 배포: ✅ systemctl restart aads-api 후 health 정상
