# Sprint 3 Task 3.3 + 3.4 — 게이트 UI + 비용 차트

**일시**: 2026-02-26 20:35 KST

## Task 3.3: 게이트 승인/거부

- `dashboard/src/components/GateApproval.tsx` 추가
- `waiting_approval` 상태에서만 표시
- 승인 → 다음 단계, 거부 → 재작업 (API 연동: `/projects/{id}/approve`, `/projects/{id}/reject`)
- 거부 시 `reason` body 전달 (기본값: `거부됨`)

## Task 3.4: 비용 모니터링

- `/costs` 페이지: `dashboard/src/app/costs/page.tsx`
- 예산 카드: 누적 비용, 예산 소진율(80% 경고), 잔여 예산
- 일별 비용 BarChart (Recharts)
- Tier별 비용 PieChart (Tier1~4)
- 최근 20건 비용 로그 테이블
- 예산: $500 총, $100 일일, 80% 경고
- `dashboard/src/hooks/useRealtimeCosts.ts`: GET `/costs` 30초 폴링
- 백엔드 `api/main.py`: GET `/costs` 스텁 추가 (현재 `{"costs": []}` — 추후 LLM 호출 로그 연동)

## 변경 파일 요약

| 위치 | 파일 |
|------|------|
| aads-core | dashboard/src/components/GateApproval.tsx (신규) |
| aads-core | dashboard/src/hooks/useRealtimeCosts.ts (신규) |
| aads-core | dashboard/src/app/costs/page.tsx (신규) |
| aads-core | dashboard/src/lib/api.ts (getCosts, reject body 추가) |
| aads-core | api/main.py (GET /costs 스텁) |

## 검증

- `cd dashboard && npm run build` 통과
- 린트 에러 없음

## 후속

- [ ] 백엔드에서 LLM 호출 시 비용 로그 저장 후 GET `/costs` 실데이터 연동
- [ ] 프로젝트 상세 페이지에 `<GateApproval>` 노출 (필요 시)
