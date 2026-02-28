# AADS Infrastructure · MCP · App-Builder Survey (2026-02-28)
> 상태: 확정 | 조사 범위: 샌드박스, PaaS, MCP 생태계, 앱 빌더, 코딩 에이전트

---

## 1. 조사 목적

AADS 인프라 구성에 필요한 샌드박스, 배포 플랫폼, MCP 서버, 경쟁 앱 빌더, 코딩 에이전트를 전수 조사하여 최적 조합을 선정한다.

---

## 2. 샌드박스 벤치마크 (7종)

| 서비스 | Cold Start | 격리 | 가격 | AADS 적합성 |
|--------|-----------|------|------|------------|
| **E2B** | ~150ms | Firecracker | $0.01667/vCPU·hr | ✅ **선정** |
| Modal | ~500ms | gVisor | 종량제 | Phase 2+ (GPU) |
| Daytona | ~150ms | Docker | $0.000014/vCPU·s | 대안 |
| Runloop | ~200ms | VM | 비공개 | 대안 |
| CodeSandbox | ~1s | 컨테이너 | Freemium | 프론트 특화 |
| StackBlitz | 즉시 | WASM | 무료/유료 | 브라우저 전용 |
| Fly Machines | ~300ms | microVM | 종량제 | 대안 |

---

## 3. PaaS 비교 (3종)

| 플랫폼 | 장점 | 단점 | 월 비용 | 적합성 |
|--------|------|------|---------|--------|
| **Fly.io** | 글로벌 엣지, Docker, SSE | 60초 idle timeout | $20–50 | ✅ **API 선정** |
| Railway | 간편, GitHub 통합 | 가격 불투명 | $20–50 | 대안 |
| Render | 무료 티어 | cold start | $0–25 | 소규모 |

Fly.io 주의: 60초 idle timeout → 15초 SSE heartbeat 필수. 2GB RAM 필요.

---

## 4. MCP 생태계 (8,600+ 서버)

8,600+ 서버, Google/OpenAI/GitHub/Atlassian 공식 지원. AADS Phase 1 필수 7개: Filesystem, Git/GitHub, PostgreSQL, Brave Search, Memory, Supabase, Fetch. supervisord 상시 4개 + 온디맨드 3개. stdio 기본, JWT 인증.

**프로덕션 베스트 프랙티스**: Bounded Context, stateless/idempotent, SSE/HTTP 원격 시, 취소 지원, Firecracker 격리.

---

## 5. AI 앱 빌더 비교 (8종)

| 서비스 | 방식 | AADS와의 차이 |
|--------|------|-------------|
| Bolt.new | 단일 프롬프트 | AADS: 다단계 체크포인트 |
| Lovable | 단일 프롬프트 | AADS: 15-20 컴포넌트 한계 없음 |
| v0 (Vercel) | UI 생성 | AADS: 전체 스택 |
| Replit Agent | 단일 에이전트 | AADS: 멀티 에이전트 |
| Copilot Workspace | 이슈 기반 | AADS: 자유 입력 |
| Cursor/Windsurf | IDE + AI | AADS: 완전 자동화 |
| Devin | 자율 코딩 | 성공률 ~15%, AADS: 점진적 자율성 |

AADS 차별점: 멀티 에이전트 조직 + 6단계 체크포인트 + SSE 스트리밍 + 점진적 자율성.

---

## 6. 코딩 에이전트 비교 (12종)

Devin (Cognition), SWE-Agent (Princeton), Aider, OpenHands, Cline, Sweep, AutoCodeRover, Claude Code (Anthropic), Copilot Workspace (GitHub), Augment Code, Factory Droids, Codex CLI (OpenAI).

---

## 7. AADS Phase 1 월간 비용

| 항목 | 최소 | 기대 | 최대 |
|------|------|------|------|
| LLM API (캐싱 후) | $20 | $55 | $120 |
| E2B Pro | $150 | $180 | $250 |
| Fly.io (2GB) | $15 | $30 | $50 |
| Supabase Pro | $25 | $30 | $40 |
| Upstash Redis | $0 | $5 | $10 |
| Vercel | $0 | $20 | $25 |
| LangSmith Free | $0 | $0 | $0 |
| **총합** | **$210** | **$320** | **$495** |

D-004 $500 예산 내 충족.

---

## 8. 결론

최적 조합: E2B(샌드박스) + Fly.io(API, 2GB) + Supabase Pro(DB, Direct:5432) + Vercel(프론트) + Upstash Redis(캐시) + LangSmith Free(관측). MCP 7개 supervisord 운영. 월 기대 $320.

---

*HANDOVER.md 업데이트 완료: {커밋해시}*
