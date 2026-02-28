# 무료/오픈소스 특화 AI 및 도구 전수 조사 보고서

**작성일**: 2026-02-28 | **목적**: AADS 에이전트 도구 생태계 확장 전략

---

## 1. 수학/추론 특화 오픈소스 AI 모델

| 모델 | 제공사 | 파라미터 | 라이선스 | 핵심 특화 |
|---|---|---|---|---|
| **DeepSeek-R1** | DeepSeek | 671B MoE | MIT | 수학·코딩 추론 최강, RL 기반, 오픈웨이트 |
| **Qwen/QwQ-32B** | Alibaba | 32B | Apache 2.0 | 수학 추론 특화, 경량 |
| **GLM-Z1-9B** | Zhipu AI (清华) | 9B | 오픈소스 | 수학·논리 추론, 초경량 |
| **Mathstral** | Mistral AI | 7B | Apache 2.0 | 수학 추론·과학 발견 전용, 32K 컨텍스트 |
| **Phi-4-reasoning** | Microsoft | 14B | MIT | RL 기반 추론, 대형 모델급 수학 성능 |
| **OpenThinker-32B** | Bespoke Labs | 32B | 오픈소스 | 복잡 수학·코드·추론 파인튜닝 |
| **OpenThinker-7B** | Bespoke Labs | 7B | 오픈소스 | 경량 추론 |
| **Kimi K2 Thinking** | Moonshot AI (月之暗面) | - | 오픈웨이트 | 세계 최고 수준 오픈 모델 벤치마크 (Artificial Analysis) |

**AADS 활용**: 수학·통계 계산이 필요한 건강검진 수치 분석, 영양소 최적 배합 계산에 활용 가능. Mathstral(7B)은 로컬 GPU에서도 구동 가능.

---

## 2. 코딩 특화 오픈소스 모델

| 모델 | 제공사 | 파라미터 | 핵심 특화 |
|---|---|---|---|
| **MiniMax M2.5** | MiniMax | 230B (10B 활성) | 코딩·에이전틱 MIT 라이선스, Sonnet 가격의 8% |
| **MiMo-V2-Flash** | Xiaomi | 309B (15B 활성) | $0.10/1M, 150 tok/s, 코딩·추론 |
| **Kimi K2.5** | Moonshot AI | - | 비디오 생성 + 자율 에이전틱, 오픈소스 |
| **GLM-5** | Zhipu AI | 744B | 범용+코딩, 중국 최대 오픈소스 |
| **GLM-4.7** | Zhipu AI | 355B | 코딩·에이전트 |
| **Step-3.5-Flash** | Stepfun (阶跃星辰) | 196B (11B 활성) | GLM-4.7, DeepSeek V3.2 능가, 에이전틱 특화 |
| **gpt-oss-120b** | OpenAI | 120B | Apache 2.0, H100 단일 GPU, OpenAI 최초 오픈웨이트 |
| **StarCoder2** | BigCode/HuggingFace | 15B | 코드 생성 전용, 600+언어 |
| **CodeLlama** | Meta | 7B/13B/34B | Llama 기반 코드 특화 |

---

## 3. 주식/퀀트 분석 오픈소스 도구 (비-AI 포함)

### 3-1. 백테스팅/전략 프레임워크

| 도구 | GitHub Stars | 언어 | 핵심 기능 |
|---|---|---|---|
| **Backtrader** | 14K+ | Python | 가장 인기 있는 백테스팅 라이브러리, 초보~전문가 |
| **Zipline** | 17K+ | Python | Quantopian 유산, pandas 기반 이벤트 드리븐 |
| **QuantConnect/LEAN** | 10K+ | C#/Python | 멀티에셋 알고리즘 트레이딩, 27만+ 퀀트 |
| **Jesse** | 5K+ | Python | 암호화폐 알고 트레이딩 특화 |
| **Hikyuu** | 2K+ | Python/C++ | 고성능 퀀트 분석·백테스팅 |
| **AutoTrader** | - | Python | 풀 라이프사이클 자동매매 (아카이브) |

### 3-2. AI 기반 퀀트 플랫폼

| 도구 | 제공사 | 핵심 기능 |
|---|---|---|
| **Microsoft Qlib** | Microsoft | AI 기반 퀀트 투자 플랫폼, 아이디어→프로덕션 |
| **GS-Quant** | Goldman Sachs | 골드만삭스 내부 퀀트 도구 오픈소스, 리스크·파생상품·포트폴리오 |
| **FinRL** | Columbia Univ. | 강화학습 기반 금융 트레이딩 |
| **awesome-quant** | GitHub | 700+ 퀀트 리소스 큐레이션 (라이브러리·논문·데이터) |
| **awesome-quant-ai** | GitHub | AI/ML 퀀트 전략 전용 큐레이션 |

### 3-3. 기술적 분석 라이브러리

| 라이브러리 | 핵심 기능 |
|---|---|---|
| **TA-Lib** | 200+ 기술 지표 (RSI, MACD, 볼린저밴드 등) |
| **pandas-ta** | pandas 통합 130+ 지표 |
| **QuantPy** | 포트폴리오 이론, 파생상품, 리스크 분석 |
| **PyPortfolioOpt** | 포트폴리오 최적화 (MVO, Black-Litterman, HRP) |
| **empyrical** | 금융 성과 측정 지표 (샤프비율, 최대손실 등) |

### 3-4. 시장 데이터 소스 (무료)

| 소스 | 핵심 기능 |
|---|---|---|
| **yfinance** | Yahoo Finance 주가 데이터 Python API |
| **OpenBB Terminal** | 무료 금융 터미널 (Bloomberg 대체), 오픈소스 |
| **Alpha Vantage** | 무료 API (주식·FX·암호화폐) |
| **FRED** | 미국 연방 경제 데이터 |
| **KRX 데이터** | 한국거래소 공공데이터 |

---

## 4. 의료/건강 특화 오픈소스 AI

| 모델/도구 | 제공사 | 핵심 특화 |
|---|---|---|
| **BioMistral** | Mistral AI | 생의학 특화 파인튜닝, 오픈소스 |
| **BioGPT** | Microsoft | 생의학 텍스트 마이닝·생성 |
| **BiomedGPT** | Harvard MGH | 오픈소스 범용 의료 비전-언어 모델 |
| **Med-PaLM 2** | Google | USMLE 86.5% (전문가 수준), MedLM API |
| **MedGemma** | Google | 의료 영상 + 텍스트 멀티모달 |
| **OpenScribe** | 오픈소스 | MIT 라이선스, AI 의료 스크라이브 (무료) |
| **PubMed.ai** | 오픈소스 | 의학 논문 AI 검색 |
| **MedLLMs Practical Guide** | GitHub | 의료 LLM 가이드 큐레이션 |

**AADS/HealthMate 활용**: BioMistral로 약물 상호작용 DB 분석, BioGPT로 논문 기반 영양소 연구 자동 요약.

---

## 5. 문서 처리/OCR 오픈소스

| 도구 | GitHub Stars | 핵심 기능 |
|---|---|---|
| **Docling** | IBM | PDF·이미지→구조화 데이터, 테이블·수식·읽기 순서 감지 |
| **Marker** | Datalab | PDF→Markdown/JSON/HTML, Surya 기반, 빠름 |
| **Surya OCR** | Datalab | 90+언어, 텍스트 감지·레이아웃 분석·인식 |
| **PaddleOCR** | Baidu | 80+언어, 경량, 프로덕션급 |
| **Tesseract** | Google | 전통적 OCR 엔진, 100+언어 |
| **MinerU** | OpenDataLab | PDF 파싱 특화 |
| **Mistral OCR 3** | Mistral AI | AI OCR, 마크다운+HTML 테이블 재구성 (API) |

---

## 6. RAG/검색 프레임워크 오픈소스

| 프레임워크 | GitHub Stars | 핵심 특화 |
|---|---|---|
| **LangChain** | 100K+ | 가장 대중적, 체이닝·에이전트·도구 통합 |
| **LlamaIndex** | 40K+ | 데이터 연결 전문, 프라이빗 데이터→LLM |
| **Haystack** | 18K+ | deepset, 프로덕션급 RAG 파이프라인 |
| **DSPy** | 20K+ | 프로그래밍적 프롬프트 최적화 |
| **LangGraph** | 10K+ | 상태 기반 순환 그래프, 에이전트 오케스트레이션 |
| **LightRAG** | 신규 | 경량 RAG |
| **FlashRAG** | 신규 | 빠른 RAG 벤치마킹 |
| **Ragas** | - | RAG 평가 메트릭 프레임워크 |

---

## 7. 벡터 데이터베이스 오픈소스

| DB | GitHub Stars | 언어 | 핵심 특화 |
|---|---|---|---|
| **Milvus** | 35K+ | Go/C++ | 최대 규모, 10억+ 벡터, 프로덕션 검증 |
| **Qdrant** | 9K+ | Rust | 고성능, 필터링 강점, AADS Phase 2 도입 후보 |
| **Weaviate** | 8K+ | Go | 클라우드 네이티브, GraphQL, 다양한 모듈 |
| **ChromaDB** | 15K+ | Python | 가장 간단한 시작, 내장형 |
| **pgvector** | - | PostgreSQL 확장 | 기존 PostgreSQL에 벡터 검색 추가 |
| **FAISS** | Meta | C++/Python | 유사도 검색 라이브러리 (DB 아님), 가장 빠름 |
| **Redis Vector Search** | Redis | Redis 모듈 | 기존 Redis에 벡터 검색 추가 |

---

## 8. 웹 크롤링/스크래핑 오픈소스

| 도구 | GitHub Stars | 핵심 기능 |
|---|---|---|
| **Crawl4AI** | 60K+ | GitHub 트렌딩 1위, LLM 친화적, 로컬 모델 통합 |
| **Firecrawl** | 30K+ | AI용 웹 크롤링 API, 셀프호스팅 가능 |
| **Scrapy** | 55K+ | 가장 성숙한 Python 크롤링 프레임워크 |
| **ScrapeGraphAI** | - | LLM 기반 구조화 스크래핑 |
| **Playwright** | Microsoft | 브라우저 자동화, JS 렌더링 |
| **Puppeteer** | Google | Chrome 자동화 |
| **Selenium** | - | 전통적 브라우저 자동화 |

---

## 9. 워크플로우/자동화 오픈소스

| 도구 | GitHub Stars | 핵심 기능 |
|---|---|---|
| **n8n** | 50K+ | 노코드 워크플로우 자동화, 셀프호스팅, AI 에이전트 |
| **Temporal** | 12K+ | 내구성 있는 워크플로우 실행, 장기 실행 태스크 |
| **Prefect** | 18K+ | 데이터 파이프라인 오케스트레이션 |
| **Apache Airflow** | 38K+ | 가장 성숙한 워크플로우 스케줄러 |
| **Windmill** | 10K+ | 스크립트→UI→워크플로우, 셀프호스팅 |

---

## 10. LLM 로컬 실행/서빙 오픈소스

| 도구 | 핵심 기능 |
|---|---|---|
| **Ollama** | "Docker for LLMs", 가장 쉬운 로컬 LLM 실행, 한 줄 설치 |
| **vLLM** | 고처리량 추론 엔진, 프로덕션급, PagedAttention |
| **TGI (Text Generation Inference)** | HuggingFace, 프로덕션 서빙, 양자화 지원 |
| **llama.cpp** | C++ 기반, CPU/GPU 혼합, GGUF 포맷 |
| **LocalAI** | OpenAI 호환 API, 멀티모달, 셀프호스팅 |
| **LM Studio** | GUI 기반 로컬 LLM, 초보자 친화적 |
| **LiteLLM** | 100+ LLM API를 단일 인터페이스로 통합 (OpenAI 포맷) |
| **OpenRouter** | 멀티 모델 라우터, 단일 API로 100+ 모델 접근 |

---

## 11. LLM 파인튜닝 오픈소스

| 도구 | 핵심 기능 |
|---|---|---|
| **Unsloth** | 30배 빠른 파인튜닝, 90% 메모리 절약, LoRA/QLoRA, TTS·BERT 지원 |
| **Axolotl** | 다양한 파인튜닝 방법 통합 (LoRA, QLoRA, Full) |
| **Torchtune** | PyTorch 공식, 투명한 파인튜닝 |
| **LLaMA-Factory** | 100+ 모델 파인튜닝 GUI, 웹 UI |
| **PEFT** | HuggingFace, LoRA/Prefix Tuning/P-Tuning |
| **TRL** | HuggingFace, RLHF/DPO/PPO 트레이닝 |

---

## 12. 컴퓨터 비전 오픈소스

| 모델/도구 | 핵심 기능 |
|---|---|---|
| **YOLO26** | 최신 객체 감지, NMS-Free, 에지 최적화, 실시간 |
| **SAM 3 (Segment Anything)** | Meta, 제로샷 이미지 분할 |
| **YOLOE** | 제로샷 객체 감지 (고정 카테고리 불필요) |
| **OpenCV** | 컴퓨터 비전 기초 라이브러리 |
| **Detectron2** | Meta, 객체 감지/분할 프레임워크 |
| **Roboflow** | 학습→배포 파이프라인 (무료 티어) |

---

## 13. 데이터 라벨링/어노테이션 오픈소스

| 도구 | 핵심 기능 |
|---|---|---|
| **Label Studio** | 텍스트·이미지·오디오·비디오·시계열, AI 보조 라벨링 |
| **CVAT** | 이미지·비디오·3D, AI 통합, 10배 빠른 어노테이션 |
| **LabelMe** | 이미지 폴리곤 어노테이션 |
| **LabelImg** | 바운딩 박스 어노테이션 |
| **Doccano** | 텍스트 어노테이션 (NER, 감성분석, 번역) |

---

## 14. 기타 특화 오픈소스 도구

### 14-1. 과학/연구

| 도구 | 핵심 기능 |
|---|---|---|
| **Elicit** | AI 논문 검색·요약·분석 |
| **Semantic Scholar API** | AI2, 학술 논문 검색 API (무료) |
| **SciSpace** | 논문 이해·요약 AI |
| **Paperguide** | 연구 논문 AI 어시스턴트 |

### 14-2. 법률

| 도구 | 핵심 기능 |
|---|---|---|
| **Harvey AI** | 법률 AI (유료, 대형 로펌 사용) |
| **CoCounsel** | Thomson Reuters, 법률 리서치 AI |
| **Lexis+ AI** | LexisNexis, 법률 DB + 생성 AI |

### 14-3. 데이터 분석/시각화 (무료)

| 도구 | 핵심 기능 |
|---|---|---|
| **Apache Superset** | 오픈소스 BI 대시보드 (Tableau 대체) |
| **Metabase** | 오픈소스 데이터 시각화·질의 |
| **Streamlit** | Python 데이터 앱 즉시 생성 |
| **Gradio** | ML 모델 데모 UI 즉시 생성 |
| **Panel/Holoviz** | Python 대시보드 프레임워크 |

### 14-4. AI 보안/가드레일

| 도구 | 핵심 기능 |
|---|---|---|
| **Guardrails AI** | LLM 출력 검증·구조화 |
| **NeMo Guardrails** | NVIDIA, 대화 가드레일 |
| **LLM Guard** | 입출력 보안 스캔 |

### 14-5. AI 모니터링/관측

| 도구 | 핵심 기능 |
|---|---|---|
| **LangSmith** | LangChain 공식, LLM 트레이싱·평가 |
| **LangFuse** | 오픈소스 LLM 관측 플랫폼 |
| **Phoenix (Arize)** | LLM 관측·평가, 오픈소스 |
| **Helicone** | LLM 프록시, 비용·사용량 추적 |
| **OpenLLMetry** | OpenTelemetry 기반 LLM 모니터링 |

---

## 15. AADS 에이전트 도구 매핑 (권고안)

| AADS 에이전트 | 필요 도구 | 추천 오픈소스 |
|---|---|---|
| **수학/통계 전문가** | 수학 추론 모델 | Mathstral(7B), DeepSeek-R1 |
| **퀀트 분석가** | 백테스팅+데이터 | Backtrader + yfinance + TA-Lib |
| **의료 도메인 전문가** | 의학 지식 | BioMistral + PubMed.ai |
| **문서 파싱 전문가** | OCR+구조화 | Docling + Surya + Marker |
| **데이터 파이프라인** | ETL+오케스트레이션 | Prefect + LangChain |
| **시장조사 애널리스트** | 웹 크롤링 | Crawl4AI + Tavily |
| **프로젝트 메모리** | 벡터 DB | Qdrant or ChromaDB |
| **로컬 모델 서버** | 추론 엔진 | Ollama (개발) / vLLM (프로덕션) |
| **모델 파인튜닝** | 학습 도구 | Unsloth + LoRA |
| **QA 엔지니어** | 테스트+모니터링 | LangFuse + Ragas |
| **비용 감시자** | API 추적 | LiteLLM + Helicone |
| **UI/시각화** | 대시보드 | Streamlit or Gradio |
| **보안/가드레일** | 출력 검증 | Guardrails AI + NeMo |

---

## 16. 비용 절감 전략

**전략 1: 모델 라우팅** — 간단한 작업은 GPT-5 nano($0.05/1M), 중간은 Gemini Flash-Lite($0.10/1M), 복잡한 것만 Opus 사용 → 평균 비용 80% 절감.

**전략 2: 로컬 모델 병용** — Mathstral(7B), Phi-4(14B) 등을 Ollama로 로컬 실행 → 반복적 분류·계산 작업의 API 비용 제거.

**전략 3: 오픈소스 도구 스택** — Crawl4AI + Docling + Qdrant + LangGraph 조합 → 유료 SaaS 대비 인프라 비용만 발생.

**전략 4: 캐싱** — Gemini 컨텍스트 캐싱(75% 할인), Claude 프롬프트 캐싱(90% 할인) 적극 활용.

---

## 보고서 관리 현황

| # | 보고서명 | 버전 | 파일명 |
|---|---|---|---|
| 1 | AADS 심층 진단 및 진화 기획 | v1.0 | `aads-evolution-strategy-20260228.md` |
| 2 | Phase 0 에이전트 레지스트리 설계 | v1.0 | `aads-phase0-agent-registry-20260228.md` |
| 3 | Phase 0 AI 회사 전체 조직 + 동적 확장 | v3.0 | `aads-phase0-full-company-agents-20260228.md` |
| 4 | AI 모델 전수 조사 보고서 | v1.0 | `aads-ai-models-survey-20260228.md` |
| 5 | 무료/오픈소스 특화 도구 전수 조사 | v1.0 | `aads-opensource-tools-survey-20260228.md` |

---

*이 보고서는 AADS 에이전트 도구 생태계 확장 및 비용 절감 전략 수립에 활용됩니다.*
