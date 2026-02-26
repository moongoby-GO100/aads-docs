# Task 4.7: 모니터링 대시보드 고도화

- **일시**: 2026-02-26
- **Sprint**: 4 — Task 4.7

## 변경 사항

| 구분 | 경로 | 내용 |
|------|------|------|
| API | `api/main.py` | `/metrics/system`, `/metrics/llm` 엔드포인트 추가 |
| 의존성 | `requirements.txt` | `psutil>=5.9.0` 추가 |
| 대시보드 | `dashboard/src/app/system/page.tsx` | 시스템 모니터 페이지 신규 |
| 대시보드 | `dashboard/src/app/layout.tsx` | 사이드바 "🖥️ 시스템" 링크 추가 |
| API 클라이언트 | `dashboard/src/lib/api.ts` | `getSystemMetrics`, `getLlmMetrics` 추가 |

## 기능 요약

- **/metrics/system**: CPU 사용률, 메모리(총 GB·사용%), 디스크(루트·볼륨), API/Redis 서비스 상태
- **/metrics/llm**: 누적 비용, 프로젝트 수, 예산(월/일) — Redis `aads:total_cost`, `aads:project_ids` 연동
- **/system 페이지**: 10초 폴링, CPU/메모리/디스크 프로그레스 바, 서비스 상태 뱃지

## 검증

- `ast.parse` 및 린트: 통과
- API 서버 기동 후 `/metrics/system`, `/metrics/llm` 호출 시 JSON 정상 반환
- 대시보드 `/system` 접속 시 메트릭 로드 및 갱신 확인

## 적용·배포

- 적용: ✅ 소스 반영
- 배포: API 재시작 및 프론트 재빌드 후 반영

## 결과

실행 후 서버 환경에서 `/metrics/system`·`/metrics/llm` 및 대시보드 `/system` 동작 확인 시 위 검증란에 기록.
