# KIS AutoTrade V4 & GO100 — 차트/대시보드 웹 경로 맵
최종 업데이트: 2026-03-07 | 작성: Claude Opus 4.6 (CEO 직접 지시)

---

## 1. 서비스 현황

| 서비스 | 포트 | 프로세스 | 상태 |
|--------|------|---------|------|
| KIS Frontend (Next.js 14) | 3000 | next-server | 운영중 |
| KIS V4.1 API (FastAPI) | 8003 | uvicorn | 운영중 |
| KIS V4.0 Backtest API | 8002 | uvicorn | 운영중 |

---

## 2. KIS AutoTrade V4 — 차트 페이지

### 2.1 레거시 정적 HTML 대시보드 (Port 3000)

| 페이지 | URL | 차트 라이브러리 | 주요 기능 |
|--------|-----|----------------|----------|
| **DESK2 실시간 모니터링** | `http://localhost:3000/desk2-live.html` | Plotly.js 2.27.0 | 실시간 시그널, 포지션 P&L, 일별 손익 라인차트, 후보종목 랭킹 |
| **DESK2 백테스트 분석** | `http://localhost:3000/desk2-backtest.html` | Plotly.js 2.27.0 | 세션별 성과, 거래 테이블, 일중 캔들스틱, 승률/PF/수익률 카드 |
| **AI 모델 대시보드** | `http://localhost:3000/ai-model.html` | — (통계 카드) | V3 모델 AUC, 모델 상태, 학습 결과, 피처 중요도 |
| **어드민 패널** | `http://localhost:3000/admin.html` | — | 시스템 관리 |
| **트레이드 뷰어** | `http://localhost:3000/trades.html` | — | 거래 이력 |

### 2.2 V4 시스템 대시보드 (Port 8003)

| 페이지 | URL | 주요 기능 |
|--------|-----|----------|
| **V4 대시보드** | `http://localhost:8003/static/v4-dashboard/index.html` | 5 DESK 현황, 실시간 시스템 메트릭(디스크/메모리), 총자산/일손익/레짐, 포지션 테이블, 최근 20건 거래 |

**V4 대시보드 API 엔드포인트** (`/api/v4/dashboard/`):

| Method | Path | 용도 |
|--------|------|------|
| GET | `/api/v4/dashboard/overview` | 총자산, 일 손익, 시장 레짐 |
| GET | `/api/v4/dashboard/desks` | DESK별 자금 현황 |
| GET | `/api/v4/dashboard/positions` | 보유 포지션 |
| GET | `/api/v4/dashboard/trades` | 거래 이력 |
| GET | `/api/v4/dashboard/commanders` | 커맨더 센터 활동 |
| GET | `/api/v4/dashboard/risk` | 리스크 메트릭 |
| GET | `/api/v4/dashboard/signals` | 오늘 시그널 |

### 2.3 Next.js 보호 경로 (Port 3000, 인증 필요)

| 페이지 | URL | 차트 라이브러리 | 주요 기능 |
|--------|-----|----------------|----------|
| **백테스트 차트** | `/(protected)/admin/backtest/charts` | Recharts 3.7.0 | 백테스트 시각화 |
| **백테스트 일별 분석** | `/(protected)/admin/backtest/daily/{sessionId}/{date}` | Recharts | 일별 거래 분석 |
| **거래 상세** | `/(protected)/admin/backtest/trades/{tradeId}` | Recharts | 개별 거래 차트 |
| **퍼포먼스 대시보드** | `/(protected)/admin/performance` | Recharts | 성과 분석 |
| **포트폴리오** | `/(protected)/portfolio` | Recharts | 포트폴리오 현황 |
| **종목 상세 차트** | `/(protected)/stock/{code}` | Recharts | 종목별 캔들/지표 |

### 2.4 백테스트 API (Port 8003)

| Method | Path | 용도 |
|--------|------|------|
| GET | `/api/v1/backtest/sessions` | 세션 목록 |
| GET | `/api/v1/backtest/sessions/{id}` | 세션 상세 |
| GET | `/api/v1/backtest/sessions/{id}/trades` | 세션 거래 |
| GET | `/api/v1/backtest/sessions/{id}/daily-pnl` | 일별 P&L 차트 |
| GET | `/api/v1/backtest/sessions/{id}/performance` | 성과 메트릭 |
| GET | `/api/v1/backtest/sessions/{id}/goal-tracking` | 목표 진행 |
| GET | `/api/v1/backtest/trend` | 성과 추세 |
| GET | `/api/v1/backtest/readiness` | 실전 준비도 |

### 2.5 차트 데이터 API (Port 8003)

| Method | Path | 용도 |
|--------|------|------|
| GET | `/api/v4/chart/stocks` | 종목 유니버스 |
| GET | `/api/v4/chart/daily/{code}` | 일봉 OHLCV |
| GET | `/api/v4/chart/minute/{code}` | 분봉 OHLCV |
| GET | `/api/v4/chart/indicators/{code}` | 기술 지표 |
| GET | `/api/v4/chart/investor/{code}` | 투자자 심리 |
| GET | `/api/v4/chart/trades/{code}` | 거래 오버레이 |
| GET | `/api/v1/backtest/chart/candles` | 백테스트 캔들 |
| GET | `/api/v1/backtest/chart/indicators` | 백테스트 지표 |
| GET | `/api/v1/backtest/chart/trade-overlay` | 진입/청산 마커 |

### 2.6 AI 모델 API (Port 8003)

| Method | Path | 용도 |
|--------|------|------|
| GET | `/api/v4/ai-model/status` | V3 모델 상태 |
| GET | `/api/v4/ai-model/predictions` | 최근 예측 |
| GET | `/api/v4/ai-model/performance` | 예측 정확도 |

---

## 3. GO100 — 차트 페이지

### 3.1 메인 대시보드 (Port 3000, 인증 필요)

| 페이지 | URL | 차트 라이브러리 | 주요 기능 |
|--------|-----|----------------|----------|
| **GO100 메인 대시보드** | `/(protected)/go100/dashboard` | Recharts + Lightweight Charts | 포트폴리오 현황, 성과 차트, 포지션, 전략 카드, 레짐 타임라인 |
| **트레이딩 대시보드** | `/(protected)/go100/trading/dashboard` | Next.js | 실시간 트레이딩 현황 |
| **라이브 트레이딩** | `/(protected)/go100/live-trading` | Next.js | 실전 매매 세션 |
| **페이퍼 트레이딩** | `/(protected)/go100/paper-trading` | Next.js | 모의 매매 세션 |
| **커맨더** | `/(protected)/go100/commander` | Next.js | 에이전트 성과, 조직도, 토론 이력 |

### 3.2 매니저 대시보드 (Public)

| 페이지 | URL | 주요 기능 |
|--------|-----|----------|
| **GO100 매니저 대시보드** | `http://localhost:3000/manager/go100-dashboard.html` | 멀티에이전트 조직도, 시스템 연결 상태(KIS API/Kiwoom/Redis/DB), 에이전트 성과, 자동 폴링 |

### 3.3 GO100 대시보드 API (Port 8003)

| Method | Path | 용도 |
|--------|------|------|
| GET | `/api/go100/dashboard/overview` | 포트폴리오 요약 |
| GET | `/api/go100/dashboard/summary` | 종합 현황 + 전략 카드 + 시그널 |
| GET | `/api/go100/dashboard/performance?period=` | 성과 시계열 (1m/3m/6m/1y/all) |
| GET | `/api/go100/dashboard/positions` | 보유 종목 + 비중 |
| GET | `/api/go100/dashboard/strategies` | 전략별 성과 |
| GET | `/api/go100/dashboard/signals?days=7` | 시장 시그널 (VIX/환율/지수) |
| GET | `/api/go100/dashboard/integrity?hours=24` | 데이터 건전성 |
| GET | `/api/go100/dashboard/regime-history?days=90` | 레짐 타임라인 |
| GET | `/api/go100/dashboard/goal-progress` | 목표 대비 진행 |
| GET | `/api/go100/dashboard/activity-log?limit=50` | 활동 피드 |

### 3.4 GO100 차트 컴포넌트 (12개)

경로: `/root/kis-autotrade-v4/frontend/src/go100/components/charts/`

| 컴포넌트 | 용도 | 라이브러리 |
|---------|------|-----------|
| `EquityCurveChart.tsx` | 자산 곡선 | Recharts |
| `DrawdownChart.tsx` | 최대 낙폭 (MDD) | Recharts |
| `CumulativeReturnChart.tsx` | 누적 수익률 | Recharts |
| `DailyPLChart.tsx` | 일별 손익 바차트 | Recharts |
| `MonthlyReturnsChart.tsx` | 월별 수익률 히트맵 | Recharts |
| `TradeDistributionChart.tsx` | 거래 분포 | Recharts |
| `WinRateChart.tsx` | 승률 도넛 차트 | Recharts |
| `AssetTrendChart.tsx` | 자산 추세 라인 | Recharts |
| `AISignalHistoryChart.tsx` | AI 시그널 이력 | Recharts |
| `PositionWeightChart.tsx` | 포지션 비중 파이 | Recharts |
| `SectorExposureChart.tsx` | 섹터 노출 바차트 | Recharts |
| `chartConfig.ts` | 공통 차트 설정 | — |

### 3.5 대시보드 UI 컴포넌트

경로: `/root/kis-autotrade-v4/frontend/src/go100/components/dashboard/`

| 컴포넌트 | 용도 |
|---------|------|
| `PerformanceChart.tsx` | 성과/P&L 시각화 |
| `ActivityFeed.tsx` | 활동 로그 피드 |
| `PositionTable.tsx` | 보유 포지션 테이블 |
| `GoalProgressBar.tsx` | 목표 달성 프로그레스 |
| `RegimeTimeline.tsx` | 시장 레짐 타임라인 |
| `StrategyCards.tsx` | 전략 성과 카드 |
| `OverviewCard.tsx` | 요약 메트릭 카드 |

---

## 4. 차트 라이브러리 현황

| 라이브러리 | 버전 | 사용처 | 특징 |
|-----------|------|--------|------|
| **Recharts** | 3.7.0 | GO100, 어드민, 포트폴리오 | React 차트 (라인/바/에어리어/파이) |
| **Lightweight Charts** | 5.1.0 | GO100 금융 차트 | 캔들스틱, OHLCV, 기술 지표 |
| **Plotly.js** | 2.27.0 | DESK2 레거시 대시보드 | 인터랙티브 3D, 복합 차트 |

---

## 5. 소스 파일 경로 요약

### Backend (API 라우터)
| 파일 | 용도 |
|------|------|
| `backend/app/routers/v4_dashboard.py` | V4 시스템 대시보드 API |
| `backend/app/routers/v4_chart.py` | V4 차트 데이터 API |
| `backend/app/routers/bt_dashboard.py` | 백테스트 대시보드 API |
| `backend/app/routers/bt_chart.py` | 백테스트 차트 API |
| `backend/app/routers/go100/dashboard_router.py` | GO100 대시보드 API |
| `backend/app/routers/ai_model_dashboard_router.py` | AI 모델 API |

### Frontend (페이지)
| 파일 | 용도 |
|------|------|
| `frontend/static/desk2-live.html` | DESK2 실시간 |
| `frontend/static/desk2-backtest.html` | DESK2 백테스트 |
| `frontend/static/ai-model.html` | AI 모델 |
| `backend/static/v4-dashboard/index.html` | V4 대시보드 |
| `frontend/src/app/(protected)/go100/dashboard/page.tsx` | GO100 메인 |
| `frontend/src/app/(protected)/go100/commander/page.tsx` | GO100 커맨더 |
| `frontend/src/app/(protected)/admin/backtest/charts/page.tsx` | 백테스트 차트 |
| `frontend/src/app/(protected)/portfolio/page.tsx` | 포트폴리오 |
| `frontend/public/manager/go100-dashboard.html` | GO100 매니저 |

---

## 6. 통계 요약

| 항목 | KIS V4 | GO100 | 합계 |
|------|--------|-------|------|
| 웹 페이지 | 11개 | 6개 | **17개** |
| API 엔드포인트 | 22개 | 11개 | **33개** |
| 차트 컴포넌트 | — | 12개 | **12개** |
| 대시보드 컴포넌트 | — | 7개 | **7개** |
| 차트 라이브러리 | 3종 | 2종 | **3종** (중복 제외) |
