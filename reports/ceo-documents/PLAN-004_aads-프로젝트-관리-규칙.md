# AADS 프로젝트 관리 규칙

## 작업 보고 체계
- 모든 작업은 시작/완료 시간이 기록된 보고서 생성
- 보고서 위치: `aads-core/logs/YYYY-MM-DD_HH-MM_작업명.md`
- Supabase `work_logs` 테이블에 자동 기록
- 보고서 스크립트: `scripts/work_report.sh start/end`

## Git 커밋 규칙
- 형식: `{이모지} {Sprint} {Task}: {설명}`
- Pre-commit hook: .env 및 API 키 커밋 자동 차단
- 자동 커밋+백업: `scripts/commit_and_backup.sh`

## 백업 체계
| 대상 | 방법 | 주기 |
|------|------|------|
| 코드 | Git push (aads-core) | 매 작업 완료 |
| 문서 | Git push (aads-docs) | 주요 변경 시 |
| 로그/보고서 | DO Spaces (newtalk1) | 주 1회 |
| DB | Supabase 자동 백업 | 일 1회 |
| .env | Spaces 암호화 업로드 | 변경 시 |

## 비용 관리
- 프로젝트 예산: $500 (기본)
- 일일 한도: $100
- 80% 경고, 100% 작업 중단
- Tier 3 (Gemini) 우선 → 필요시 Tier 2 → 최종 Tier 1

## 배포 체크리스트
- [ ] 모든 테스트 통과
- [ ] .env 미포함 확인
- [ ] Docker 이미지 빌드 성공
- [ ] 헬스체크 통과
- [ ] 롤백 계획 수립
- [ ] 배포 보고서 작성

## AI 모델 설정 (2026-02-26 최신)
| Tier | 모델 | API ID | 용도 |
|------|------|--------|------|
| 1 | Claude Opus 4.6 | claude-opus-4-6 | 기획·설계 |
| 2 | Claude Sonnet 4.6 | claude-sonnet-4-6 | 코드·리뷰 |
| 3 | Gemini 2.5 Flash | gemini-2.5-flash | 반복·대량 |
| 4 | Claude Haiku 4.5 | claude-haiku-4-5 | 폴백 |
