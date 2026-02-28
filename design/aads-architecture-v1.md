# AADS Phase 1 Architecture Design Document (v1.0-draft)
> 작성: 2026-02-28 | 상태: draft — CEO 검토·승인 후 확정
> 기준: HANDOVER.md, CEO-DIRECTIVES.md, 리서치 Phase 결과

---

## 1. 개요

- **AADS (Autonomous AI Development System)**: 멀티 AI 에이전트가 조직처럼 협업하여 소프트웨어를 자율 개발하는 시스템
- **AADS 자체가 프로덕트** — Phase 1에서는 코어(에이전트 체인 + MCP + 샌드박스)의 첫 동작을 목표로 함
- 본 설계서는 Phase 1 범위를 정의하며, Phase 2(대시보드)·Phase 3(서비스 배포)는 별도 문서로 확장

---

## 2. 목표와 범위

### 2.1 Phase 1 목표
- LangGraph 기반 Supervisor + 전문 에이전트 체인이 한 사이클 동작
- MCP 서버 7개 연동으로 파일/ Git/ DB/ 검색/ 메모리/ Supabase/ Fetch 사용
- E2B 샌드박스에서 에이전트가 생성한 코드 실행·검증
- 체크포인팅으로 실패 시 마지막 성공 단계에서 재개
- 고위험 작업(배포, DB 변경 등)에 Human-in-the-Loop 승인 게이트

### 2.2 범위 제외
- AADS 대시보드 UI(Phase 2)
- 외부 사용자 멀티테넌시(Phase 3)
- HealthMate 등 구체 프로젝트 개발(Phase 5)

---

## 3. 아키텍처 원칙

- **2단계 계층**: Supervisor 1개 + 전문 에이전트(Architect, PM, Developer, QA, DevOps, Researcher) — 3단계 이상 금지
- **상태 기반**: 방향 그래프로 흐름 명시, 예측·테스트 가능
- **비용 제어**: 모델 라우팅(nano/flash vs Opus/Sonnet), 토큰 예산, 캐싱, 조기 종료
- **오픈소스 우선**: 상용 대체 가능 시 오픈소스 활용
- **MVP 우선**: 완벽보다 동작하는 것 먼저

---

## 4. 에이전트 역할 구성

| 에이전트 | 역할 | 추천 모델 | 대안 |
|----------|------|-----------|------|
| Supervisor | 오케스트레이션, 태스크 분배·합성 | Claude Opus 4.6 | Gemini 3.1 Pro |
| Architect | 시스템 설계, 기술 의사결정 | Claude Opus 4.6 | GPT-5.2 Pro |
| PM | 태스크 분해, 우선순위, 일정 | Claude Sonnet 4.6 | GPT-5.2 Chat |
| Developer | 코드 생성·수정·리팩터 | Claude Sonnet 4.6 | GPT-5.3 Codex |
| QA | 테스트 생성·실행·버그 리포트 | Claude Sonnet 4.6 | GPT-5 mini |
| DevOps | CI/CD, 배포, 모니터링 | GPT-5 mini | Claude Haiku 4.5 |
| Researcher | 데이터 수집·분석·웹 검색 | Gemini 2.5 Flash | GPT-5 nano |

---

## 5. LangGraph Supervisor 패턴

- **프레임워크**: LangGraph (상태 기반 방향 그래프)
- **노드**: Supervisor 노드 1개, 전문 에이전트 노드 각 1개
- **엣지**: Supervisor → 에이전트(라우팅), 에이전트 → Supervisor(결과 반환)
- **종료 조건**: Supervisor가 "완료" 판단 시 그래프 종료
- **재시도**: 에이전트 실패 시 Supervisor가 재할당 또는 HITL 요청

---

## 6. 상태 관리 및 체크포인팅

- **저장소**: PostgreSQL (LangGraph 체크포인트 어댑터)
- **캐싱**: Redis (세션·중간 결과)
- **복구**: 실패 시 마지막 성공 체크포인트 ID로 재개
- **보존 기간**: Phase 1에서는 7일 (설정 가능)

---

## 7. Human-in-the-Loop (HITL)

- **대상**: 배포 실행, DB 스키마/데이터 변경, 외부 API 키 사용, 프로덕션 리소스 변경
- **방식**: Supervisor가 "승인 대기" 상태로 전환 → CEO/운영자 승인 후 재개
- **로깅**: 승인 요청·승인/거부·타임스탬프 전부 기록

---

## 8. MCP 서버 스택

### Phase 1 필수 (7개)
- **Filesystem**: 로컬/원격 파일 읽기·쓰기
- **Git/GitHub**: 커밋, 푸시, PR 생성, 이슈
- **PostgreSQL**: 쿼리 실행, 스키마 조회 (체크포인트 DB와 분리 가능)
- **Brave Search**: 웹 검색
- **Memory**: 에이전트 간·세션 간 기억
- **Supabase**: DB·Auth·Storage (프로젝트별)
- **Fetch**: HTTP 요청 (API 호출 등)

### Phase 2 확장 (3개)
- Puppeteer, Sentry, Slack

---

## 9. 샌드박스

- **도구**: E2B (에이전트 코드 실행 격리)
- **용도**: Developer/QA가 생성한 코드 실행, 테스트 실행
- **런타임**: Python, Node 등 E2B 지원 런타임
- **제한**: 네트워크·파일 시스템 정책으로 프로덕션 격리
- **비용**: E2B Pro 약 $150/월 (초기 $100 무료 크레딧)

---

## 10. 배포 인프라

| 구성요소 | 도구 | 비고 |
|----------|------|------|
| AADS API | Fly.io | FastAPI 또는 Supabase Edge |
| AADS 프론트(Phase 2) | Vercel Pro | Next.js |
| DB + Auth | Supabase Pro | 체크포인트·앱 DB |
| MCP 서버 | Fly.io 동일 또는 별도 VM | 오픈소스 호스팅 |
| GPU 필요 시 | Modal | 종량제 |

---

## 11. 데이터 흐름 (Phase 1)

1. **입력**: CEO/시스템이 "태스크" 생성 (예: "API 엔드포인트 X 추가")
2. **Supervisor**: 태스크 수신 → PM/Architect/Developer 등에게 하위 작업 분배
3. **에이전트**: MCP로 파일·Git·검색·DB 사용, 필요 시 E2B에서 코드 실행
4. **체크포인트**: 각 단계 완료 시 PostgreSQL에 상태 저장
5. **HITL**: 고위험 작업 시 승인 대기 → 승인 후 재개
6. **출력**: 최종 아티팩트(Git 커밋, 배포 로그 등) + 보고

---

## 12. 보안 및 규정

- **시크릿**: .env/API 키 커밋 금지, 런타임에만 주입
- **프로덕션 DB**: 직접 편집 금지, 마이그레이션 스크립트만
- **네트워크**: 샌드박스는 허용 목록 외 접근 차단
- **감사**: 모든 에이전트 결정·MCP 호출·HITL 이벤트 로깅 (Phase 1에서는 단순 파일/DB 로그)

---

## 13. 관측성

- **트레이싱**: LangSmith 또는 오픈소스 대안 (에이전트 호출·토큰·지연)
- **로그**: 구조화 로그(JSON)로 저장, 에러 시 알림 연동(Phase 2)

---

## 14. 비용 목표 (Phase 1)

- **LLM**: 월 약 $85 (혼합 사용 가정)
- **인프라**: Fly.io + Vercel + Supabase + E2B ≈ $320–370/월 (프로토타입 시 일부 $0 가능)
- **총합**: $500/월 이하 유지

---

## 15. Phase 로드맵 요약

| Phase | 내용 | 상태 |
|-------|------|------|
| Phase 0 | 리서치 + 아키텍처 설계 + 인계서 | 진행중 |
| Phase 1 | 코어 — LangGraph + 에이전트 + MCP + 샌드박스 | 대기 |
| Phase 2 | 대시보드 | 대기 |
| Phase 3 | 서비스 배포 | 대기 |
| Phase 4 | 자율 개선 | 대기 |
| Phase 5 | HealthMate 첫 프로젝트 | 대기 |

---

## 16. 다음 단계

- CEO 설계서 검토·승인
- 확정판 반영 후 `design/aads-architecture-v1.md` 최종화
- Phase 1.1 구현 지시서 발행 (에이전트 체인·MCP·E2B 연동 순서 정의)

---

*본 draft는 HANDOVER.md·CEO-DIRECTIVES.md 및 리서치 결과를 반영한 초안이며, CEO 승인 후 수정·보완됩니다.*
