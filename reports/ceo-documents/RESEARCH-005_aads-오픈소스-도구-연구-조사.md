# AADS Open-Source Tools Survey (2026-02-28)
> 상태: 확정 | 조사 범위: 멀티 에이전트 관련 오픈소스 도구 전수 조사

---

## 1. 조사 목적

AADS 구축에 활용 가능한 오픈소스 도구와 모델을 전수 조사하여, 상용 SaaS 대체 가능 여부와 Phase 1 적용 가능성을 평가한다.

---

## 2. 수학/추론 오픈소스 모델 (7종)

| 모델 | 파라미터 | 특화 | AADS 활용 |
|------|---------|------|----------|
| DeepSeek-R1 | 671B MoE | 추론, 수학 | Phase 2 셀프호스팅 후보 |
| Qwen 3.5 (72B) | 72B | 범용+추론 | 한국어 지원, 대안 후보 |
| Llama 4 Maverick | 400B MoE | 범용 | 대안 후보 |
| Llama 4 Scout | 109B | 범용, 효율 | 경량 대안 |
| Phi-4 | 14B | 수학, 코딩 | 로컬 테스트용 |
| Mistral Large 2 | 123B | 범용 | 대안 |
| Command A | 111B | 비즈니스 | RAG 특화 |

---

## 3. 코딩 오픈소스 모델 (6종)

| 모델 | 특화 | AADS 활용 |
|------|------|----------|
| DeepSeek-Coder-V3 | 코드 생성 | Phase 2 Developer 대안 |
| Qwen 3.5 Coder | 코드 생성 | 경량 대안 |
| StarCoder2 | 코드 생성 | 코드 자동완성 |
| CodeLlama (34B) | 코드 생성 | 레거시 |
| Granite Code | IBM, 코드 | 엔터프라이즈 |
| OpenCoder | 코드 생성 | 연구용 |

---

## 4. RAG 프레임워크 (8종)

| 도구 | 특징 | 라이선스 | Phase 1 |
|------|------|---------|---------|
| LangChain | 최대 생태계 | MIT | ✅ 사용 |
| LlamaIndex | 데이터 연결 | MIT | Phase 2 |
| Haystack | 파이프라인 | Apache 2.0 | 대안 |
| RAGFlow | 비주얼 RAG | Apache 2.0 | Phase 2 |
| Cognita | 모듈형 | Apache 2.0 | 대안 |
| Verba | 대화형 | BSD | 대안 |
| Canopy | 관리형 | Apache 2.0 | 대안 |
| FlashRAG | 벤치마크 | MIT | 연구용 |

---

## 5. 벡터 DB (6종)

| 도구 | 특징 | Phase 1 |
|------|------|---------|
| pgvector | PostgreSQL 확장 | ✅ Supabase 내장 |
| Chroma | 경량 | 개발용 |
| Weaviate | 그래프+벡터 | Phase 2 |
| Qdrant | 고성능 | 대안 |
| Milvus | 대규모 | 대안 |
| FAISS | 라이브러리 | 로컬 테스트 |

---

## 6. 기타 도구

**크롤링**: Scrapy, Playwright, Puppeteer, Crawl4AI, FireCrawl, Apify. **워크플로우**: Prefect, Airflow, Temporal, n8n, Windmill. **라벨링**: Label Studio, Argilla, Prodigy, Doccano. **에이전트 프레임워크**: CrewAI, AutoGen, Swarm, MetaGPT, ChatDev. **퀀트**: QuantLib, Zipline, Backtrader, VectorBT, FreqTrade, FinRL 등 20+종.

---

## 7. 결론

Phase 1은 LangChain/LangGraph + pgvector(Supabase) + LangSmith Free. 오픈소스 LLM은 Phase 2 셀프호스팅 비용 절감 전략으로 검토.

---

*HANDOVER.md 업데이트 완료: 7dfd454*
