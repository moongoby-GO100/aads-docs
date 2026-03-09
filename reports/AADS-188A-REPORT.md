# AADS-188A Wrap Report: Gemini Deep Research API 통합 업그레이드

**Task ID**: AADS-188A  
**완료일**: 2026-03-09  
**담당**: Claude (서버 68, /root/aads)  
**commit**: c36c927 (aads-server)

---

## 목적
CEO가 "시장 동향 조사해", "경쟁사 분석해" 등 딥 리서치 요청 시 Gemini Deep Research Agent를 호출하여 자율 탐색 + 종합 보고서 생성.  
기존 AADS-186E-2-BRIDGE에서 구현된 `research()` 메서드를 업그레이드하여 AsyncGenerator 스트리밍, context/format 파라미터, Langfuse 연동 추가.

---

## 구현 내용

### 1. `app/models/research.py` (수정)
- `ResearchEvent` 타입 확장: `planning | searching | analyzing | complete` 추가 (기존: start/thinking/content/complete/error)
- 신규 필드: `content` (스트리밍 텍스트), `sources` (인용 목록), `phase` (단계 설명), `progress_pct` (진행률 0~100)
- `ResearchResult` 신규 필드: `sources` (기본값 빈 리스트)

### 2. `app/services/deep_research_service.py` (대폭 수정)
- **GOOGLE_GENAI_API_KEY 지원**: `os.getenv("GOOGLE_GENAI_API_KEY") or os.getenv("GEMINI_API_KEY", "")` — 두 환경변수 양쪽 지원
- **`research_stream()` 신규**: AsyncGenerator[ResearchEvent] 반환. planning → searching → analyzing → complete 단계 이벤트 yield
- **`context` 파라미터**: `research()` 및 `research_stream()` 양쪽에 추가
- **`format` 파라미터**: 'summary' | 'detailed' | 'report' 프리셋 지원 (`_format_preset()` 헬퍼)
- **`_build_prompt()` 헬퍼**: query + context + format_instructions 결합
- **`_research_impl()` 내부 라우터**: SDK/HTTP 폴백 분리
- **Langfuse span**: `research()` 및 `research_stream()` 양쪽에서 langfuse_config.py 연동 (비활성화 시 graceful 스킵)
- **월간 50건 제한 추가**: `_monthly_usage` 카운터 + `_check_monthly_limit()`

### 3. `app/services/tool_registry.py` (수정)
- `deep_research` 스키마에 `context` 파라미터 추가
- `format` 파라미터 추가 (enum: summary/detailed/report)
- input_examples 업데이트 (format 프리셋 사용)

### 4. `app/services/tool_executor.py` (수정)
- `_deep_research()`: `context`, `format` 파라미터 추출 및 `svc.research()` 전달
- Langfuse span 추가: `tool_deep_research` trace + `deep_research_tool` span (소스수/비용/elapsed/status 기록)

### 5. `app/services/intent_router.py` (수정)
- deep_research 키워드 확장:
  - 첫 번째 블록: "리서치", "경쟁사 분석", "트렌드 분석" 추가
  - 두 번째 블록: "조사해줘", "조사해서", "경쟁사", "트렌드", "보고서 작성" 추가

### 6. `tests/test_deep_research.py` (수정)
- 기존 21개 테스트 모두 유지
- 신규 25개 테스트 추가:
  - `TestResearchEventNewFields` (6개): planning/searching/analyzing/complete 이벤트 + sources 필드
  - `TestDeepResearchStreamFeatures` (11개): research_stream 메서드 존재, _build_prompt, _format_preset, API 키 없을 때 오류, 일일 한도, GOOGLE_GENAI_API_KEY, 월간 한도
  - `TestToolRegistryDeepResearchSchema` (3개): context/format 파라미터, required 검증
  - `TestIntentRouterDeepResearchKeywords` (5개): 리서치/경쟁사/트렌드/딥리서치/시장분석보고서 키워드

---

## 테스트 결과
```
46 passed, 1 warning in 3.05s
```
- 기존 21개 유지 ✅
- 신규 25개 추가 ✅
- **total: 46/46 PASS**

---

## 검증 항목

| 검증 기준 | 결과 |
|-----------|------|
| research_stream() AsyncGenerator 구현 | ✅ |
| planning/searching/analyzing/complete 단계 이벤트 | ✅ |
| GOOGLE_GENAI_API_KEY 환경변수 지원 | ✅ |
| context 파라미터 | ✅ |
| format 파라미터 (summary/detailed/report) | ✅ |
| 일일 5건 제한 | ✅ (기존 유지) |
| 월간 50건 제한 | ✅ (신규) |
| 타임아웃 60분 설정 | ✅ (_TIMEOUT_COMPLEX=3600) |
| Langfuse span 연동 | ✅ (비활성화 시 graceful 스킵) |
| 테스트 5개 이상 PASS | ✅ (46개 PASS) |

---

## 커밋
- aads-server: `c36c927` feat(AADS-188A): Gemini Deep Research API 통합 업그레이드
- aads-docs: (이 보고서 커밋)

---

## 교훈
- `async def` + `yield` 조합은 AsyncGenerator — `asyncio.iscoroutinefunction()` 대신 `inspect.isasyncgenfunction()` 사용 필요
- `google.genai.Client` + `aio.models.generate_content_stream()` 스트리밍 패턴으로 deep research 이벤트 처리
