# 2026년 2월 최신 AI 모델 전수 조사 보고서

> **작성일**: 2026-02-28
> **작성자**: AADS AI Research Agent
> **목적**: AADS 에이전트 레지스트리 모델 배정 전략 수립
> **버전**: v1.0
> **GitHub**: `reports/aads-ai-models-survey-20260228.md`

---

## 목차

1. [텍스트/추론 대형 모델 (Frontier LLM)](#1-텍스트추론-대형-모델-frontier-llm)
2. [한국 특화 AI 모델](#2-한국-특화-ai-모델)
3. [코딩 특화 도구/모델](#3-코딩-특화-도구모델)
4. [이미지 생성 모델](#4-이미지-생성-모델)
5. [비디오 생성 모델](#5-비디오-생성-모델)
6. [음성/TTS/STT 모델](#6-음성ttsstt-모델)
7. [음악 생성 모델](#7-음악-생성-모델)
8. [임베딩/검색 특화 모델](#8-임베딩검색-특화-모델)
9. [검색 증강 모델](#9-검색-증강search-augmented-모델)
10. [멀티에이전트/오케스트레이션 프레임워크](#10-멀티에이전트오케스트레이션-프레임워크)
11. [API 가격 종합 비교](#11-api-가격-종합-비교-2026-02-기준-1m-tokens)
12. [AADS 에이전트별 모델 배정 전략](#12-aads-에이전트별-모델-배정-전략-권고안)
13. [월간 비용 시뮬레이션](#13-월간-비용-시뮬레이션-aads-운영-기준)
14. [핵심 트렌드 요약](#14-핵심-트렌드-요약-2026-02-기준)

---

## 1. 텍스트/추론 대형 모델 (Frontier LLM)

### 1-1. Anthropic — Claude 4 패밀리

| 모델 | 출시일 | 파라미터 | 컨텍스트 | API 가격 (Input/Output, /1M tokens) | 핵심 특화 |
|---|---|---|---|---|---|
| **Claude Opus 4.6** | 2026-02-05 | 비공개 | 200K (1M 베타) | $5 / $25 | 최고 정밀 코딩(SWE-Bench 80.8%), 인간 선호도 Elo 1,606, 장기 에이전틱 태스크 |
| **Claude Sonnet 4.6** | 2026-02-17 | 비공개 | 1M (베타) | $3 / $15 | GDPval-AA Elo 1,633(전 모델 1위), 코딩·에이전트·콘텐츠 최적 밸런스, Claude.ai 기본 모델 |
| Claude Opus 4.5 | 2025-11-24 | 비공개 | 200K | $5 / $25 | Opus 4.6 이전 플래그십 |
| Claude Sonnet 4.5 | 2025 | 비공개 | 200K | $3 / $15 | 이전 기본 모델 |
| Claude Opus 4.1 | 2025 | 비공개 | 200K | $5 / $25 | 코딩 강화 버전 |
| Claude Opus 4 | 2025 | 비공개 | 200K | $5 / $25 | Claude 4 패밀리 초기 릴리스 |
| Claude Sonnet 4 | 2025 | 비공개 | 200K | $3 / $15 | - |
| **Claude Haiku 4.5** | 2025-10-15 | 비공개 | 200K | $1 / $5 | 최저가 경량 모델, 빠른 응답, 높은 볼륨 처리 |

**구독 가격**: Claude Pro ~$20/월, Claude Max ~$100/월

**AADS 활용 포인트**:
- Opus 4.6 → DUR 약물상호작용, 의료 추천 등 고정밀 판단
- Sonnet 4.6 → 코드 생성, 에이전트 오케스트레이션, 일반 업무
- Haiku 4.5 → 분류·태깅·요약 등 대량 호출 작업

### 1-2. OpenAI — GPT-5 패밀리

| 모델 | 출시일 | 컨텍스트 | API 가격 (Input/Output) | 핵심 특화 |
|---|---|---|---|---|
| **GPT-5.3 Codex** | 2026-02-05 | 400K | ~$1.25 / ~$10 (추정) | Terminal-Bench 77.3%(1위), SWE-Bench Pro 56.8%, 전문 코딩 에이전트 |
| GPT-5.2 Chat | 2026-02 | 128K | $1.75 / (미공개) | 범용 대화, 기업 업무 |
| GPT-5.2 Pro | 2026 | - | $10.50 / $84 | 최고 성능 추론, 고가 |
| **GPT-5** | 2025-08-07 | 128K | $1.25 / $10 | GPT-4o·o3·o4-mini 대체, ChatGPT 기본 모델 |
| **GPT-5 mini** | 2025-08 | 400K | $0.25 / $2 | 경량, 비용 효율 |
| **GPT-5 nano** | 2025-08 | 128K | $0.05 / $0.40 | 초경량, 레이턴시 최적화, 에지 디바이스 |
| GPT-5 Pro | 2025 | - | $7.50 / $60 | 고급 추론 |
| **gpt-oss-120b** | 2025 | - | 오픈소스 (Apache 2.0) | H100 단일 GPU 구동 가능, OpenAI 최초 오픈웨이트 |
| **GPT Image 1 / 1.5** | 2025~2026 | - | 이미지별 과금 | 이미지 생성, 텍스트 정확도 높음 |

**구독 가격**: ChatGPT Plus ~$20/월, ChatGPT Pro ~$200/월

**AADS 활용 포인트**:
- GPT-5 nano → 분류·라우팅 등 초저가 태스크
- GPT-5.3 Codex → 터미널·DevOps 코딩 특화
- gpt-oss-120b → 셀프호스팅 가능

### 1-3. Google DeepMind — Gemini 패밀리

| 모델 | 출시일 | 컨텍스트 | API 가격 (Input/Output) | 핵심 특화 |
|---|---|---|---|---|
| **Gemini 3.1 Pro** (Preview) | 2026-02-19 | 1M | $2 / $12 | ARC-AGI-2 77.1%(1위), GPQA 94.3%, 13/16 벤치마크 선두, 가성비 최고 프론티어 |
| **Gemini 3 Pro** | 2025-11 | 1M | $2 / $12 | Gemini 2.5 Pro 대비 50%+ 향상 |
| **Gemini 3 Flash** | 2025-12-17 | 1M | $0.50 / $3 | 속도 최적화, 무료 티어 제공 |
| Gemini 3 Flash (Preview) | 2026 | 1M | 무료~$0.50 / $3 | 최신 Flash, 사고 비용 절감 |
| Gemini 2.5 Pro | 2025-03 | 1M | $1.25 / $5 | 이전 세대 프로 (GA 안정) |
| **Gemini 2.5 Flash** | 2025 | 1M | $0.30 / $2.50 | 생산 안정 모델 |
| **Gemini 2.5 Flash-Lite** | 2025 | 1M | $0.10 / $0.40 | 초저가, 대량 처리 |

**특수 모델**: Gemini 2.5 Flash Image (이미지 편집·생성), Gemini 2.5 Pro TTS (텍스트→음성)

**구독 가격**: Gemini Advanced ~$20/월, Ultra ~$125/3개월

**AADS 활용 포인트**:
- Gemini 3.1 Pro → 복잡한 추론, 과학적 판단
- Gemini 2.5 Flash-Lite → 초저가 대량 호출
- 무료 티어 → PoC, 프로토타이핑

### 1-4. xAI — Grok 패밀리

| 모델 | 출시일 | 파라미터 | 컨텍스트 | 구독 가격 | 핵심 특화 |
|---|---|---|---|---|---|
| **Grok 4.20** | 2026-02-17 | 500B (소형) | 256K~2M | SuperGrok $30/월, Heavy $300/월 | **4개 AI 에이전트 병렬 아키텍처** (Grok+Harper+Benjamin+Lucas), 실시간 X 데이터, Alpha Arena 유일 수익 모델 |
| **Grok 4.1** | 2026-01 | - | 2M | - | 65% 환각 감소, 30-40% 속도 향상, 감정 인텔리전스 |
| Grok 4 | 2025-07-09 | 1.7T (추정) | 256K~2M | - | 네이티브 멀티모달, 첫 번째 원리 추론 |

**API 상태**: Grok 4.20 API "Coming soon" (2026 Q2 예상). Azure AI Foundry에서 Grok 4 이용 가능.

**AADS 활용 포인트**: 멀티 에이전트 병렬 아키텍처 → AADS 동적 팀 구성 설계에 참고할 아키텍처 혁신

### 1-5. DeepSeek (중국)

| 모델 | 출시일 | 파라미터 | API 가격 | 핵심 특화 |
|---|---|---|---|---|
| **DeepSeek-V3.2** | 2025-12-01 | MoE (671B 추정) | 매우 저가 | 범용, 사고+도구 통합, GPT-4.5급 성능 |
| **DeepSeek-R1** | 2025-01 | - | 저가 | 추론 특화, 수학·코딩 |
| DeepSeek-VL2 | 2025 | - | - | 비전-언어 멀티모달 |

> **주의**: 중국 법률 하 데이터 접근 우려. 한국 기업 환경에서는 셀프호스팅 권장.
> 새 모델 테스트 중 (2026-02-13 Reddit 확인).

### 1-6. Alibaba — Qwen 패밀리

| 모델 | 출시일 | 파라미터 | 컨텍스트 | API 가격 | 핵심 특화 |
|---|---|---|---|---|---|
| **Qwen 3.5** (Plus, Medium 등) | 2026-02-15 | 다양 | 1M | - | 네이티브 멀티모달 에이전트, 201개 언어, 빌트인 도구 |
| **Qwen 3.5-Medium** | 2026-02-26 | - | - | 오픈소스 | Sonnet 4.5급 성능, 로컬 구동 가능 |
| Qwen3.5-122B-A10B | 2026 | 122B(10B 활성) | 262K | $0.40 / $1.20 | 초저가 프론티어급, Apache 2.0 |
| Qwen3-Max-Thinking | 2026-01-25 | - | - | - | 적응적 도구 사용, 사고 모드 |

> **참고**: 알리바바 클라우드 API는 싱가포르 경유 → GDPR/개인정보 이슈. 셀프호스팅으로 데이터 주권 확보 가능.

### 1-7. Meta — Llama 4 패밀리

| 모델 | 출시일 | 파라미터 | 컨텍스트 | 라이선스 | 핵심 특화 |
|---|---|---|---|---|---|
| **Llama 4 Maverick** | 2025-04 | 400B (17B 활성, 128 Experts) | 1M | 오픈웨이트 | 네이티브 멀티모달(텍스트+이미지), GPT-4o·Gemini 2.0 Flash 능가 |
| **Llama 4 Scout** | 2025-04 | - | **10M** | 오픈웨이트 | 10M 토큰 컨텍스트 윈도, 대규모 코드베이스 분석 |
| Llama 4 Behemoth | 미출시 | - | - | - | Maverick·Scout 증류 원본 |

> **참고**: Meta "Avocado" 후속 모델은 클로즈드 소스 전환 검토 중 (2025-12 보도).

### 1-8. Mistral AI (프랑스)

| 모델 | 출시일 | 파라미터 | 컨텍스트 | 핵심 특화 |
|---|---|---|---|---|
| **Mistral Large 3** | 2025-12-02 | 675B (41B 활성 MoE) | 256K | 비전 포함, 다국어, 오픈웨이트 |
| **Mistral Medium 3** | 2025-05-07 | - | 128K | GPT-5급 대비 8배 저가, 기업 배포 최적화 |
| **Mistral Small 3.1** | 2025 | 24B | 128K | 에지 배포 가능 |
| Magistral Medium 1.2 | 2025-09 | - | - | 추론 특화 |
| Ministral 3 | 2025-12-02 | 3B/8B/14B | - | 초소형 모델 (에지용) |
| **Mistral OCR 3** | 2025 | - | - | 문서 OCR 특화, 마크다운+HTML 테이블 재구성 |
| **Voxtral** | 2026 | - | - | 실시간 음성 번역 |

### 1-9. Microsoft — Phi 패밀리 (SLM)

| 모델 | 파라미터 | 핵심 특화 |
|---|---|---|
| **Phi-4** | 14B | 복잡 추론, 수학, 오픈웨이트 |
| **Phi-4-reasoning** | 14B | RL 기반 추론, 대형 모델 수준 |
| **Phi-4-multimodal-instruct** | 14B | 텍스트+이미지+음성 |
| **Phi-4-mini-instruct** | 3.8B | 추론·수학·코딩·함수호출, 엣지 최적화 |

### 1-10. 기타 주요 텍스트 모델

| 모델 | 제공사 | 파라미터 | 핵심 특화 |
|---|---|---|---|
| **Xiaomi MiMo-V2-Flash** | Xiaomi | 309B (15B 활성 MoE) | 오픈소스, $0.10/1M input, 150 tok/s 추론 속도 |
| **MiniMax M2.5** | MiniMax | 230B (10B 활성 MoE) | 코딩·에이전틱 특화, Sonnet 가격의 8% |
| **Amazon Nova 2 Pro** | AWS | 비공개 | 텍스트+이미지+비디오+음성 멀티모달, Bedrock 전용 |
| Amazon Nova 2 Lite | AWS | 비공개 | 비용 효율적 추론 |
| Amazon Nova 2 Sonic | AWS | 비공개 | 음성 특화 |
| Amazon Nova 2 Omni | AWS | 비공개 | 옴니모달 (프리뷰) |
| **Cohere Command R+** | Cohere | - | RAG 최적화, 기업 검색·요약 |
| **Perplexity Sonar Pro** | Perplexity | - | 실시간 웹 검색+LLM 통합 API |

---

## 2. 한국 특화 AI 모델

| 모델 | 제공사 | 핵심 특화 | 상태 |
|---|---|---|---|
| **HyperCLOVA X Think** | NAVER Cloud | 추론 AI, 한국어 최적화, 한국 주권 AI | 2025-06 출시 |
| **HyperCLOVA X SEED** | NAVER Cloud | 오픈소스 한국어 모델 3종, 상업 이용 | 2025-04 출시 |
| **Samsung Gauss 2** | Samsung | 언어·코드·이미지 3종, 갤럭시·가전 내장 | 2024-11 업데이트 |

**AADS 활용 포인트**: 한국 공공데이터 API 연동 시 한국어 처리 검증에 활용 가능. HyperCLOVA X의 한국어 이해도가 글로벌 모델 대비 유리할 수 있으나 API 접근성 한계.

---

## 3. 코딩 특화 도구/모델

| 도구 | 기반 모델 | 가격 | 핵심 특화 |
|---|---|---|---|
| **Claude Code** | Claude Sonnet/Opus 4.6 | API 종량제 | CLI 에이전틱 코딩, 파일 읽기/쓰기, 터미널 실행, Git 연동 |
| **OpenAI Codex** (CLI/App) | GPT-5.3 Codex | ChatGPT Pro 포함 | 샌드박스 코드 실행, PR 생성, 터미널 자동화 |
| **Cursor** | Claude/GPT-5/Gemini 선택 | $20~40/월 | VS Code 기반 IDE, 멀티모델 지원 |
| **GitHub Copilot** | Claude Sonnet 4.6 기본 | $10~39/월 | IDE 통합 자동완성, PR 리뷰 |
| **Devin** | 자체 | $500/월 | 자율 AI 소프트웨어 엔지니어, PR 완성, ~13% 성공률 |
| **Replit Agent** | 자체+멀티모델 | 무료~$25/월 | 브라우저 기반 풀스택 앱 생성 |
| **Lovable** | Gemini 3 Flash 등 | 무료~ | 풀스택 앱 생성, GitHub 연동, ARR $200M+, 밸류 $6.6B |
| **Bolt.new** | 멀티모델 | 무료~ | 브라우저 기반 프로토타이핑, 프론트엔드 속도 |

---

## 4. 이미지 생성 모델

| 모델 | 제공사 | 핵심 특화 |
|---|---|---|
| **FLUX.2 [dev/pro]** | Black Forest Labs | 32B 파라미터, 4MP 포토리얼리즘 1위, 오픈소스 |
| FLUX.2 [klein] 4B | Black Forest Labs | 실시간 생성(1초 미만), 오픈소스, 소비자 GPU 구동 |
| **Midjourney V7** | Midjourney | 예술적·시네마틱 이미지 최강 |
| **DALL-E 4** | OpenAI | 리얼리즘, 상업용 |
| GPT Image 1.5 | OpenAI | 텍스트 in 이미지 정확도 최고 |
| **Adobe Firefly 3** | Adobe | 상업 라이선스 안전, 편집 워크플로우 통합 |
| **Ideogram 3.0** | Ideogram | 타이포그래피 정확도 |
| **Recraft V3** | Recraft | 로고·벡터 HuggingFace 1위 |
| **Stable Diffusion 3.5** | Stability AI | 오픈소스, 커스터마이징 |
| **Gemini 2.5 Flash Image** | Google | 이미지 편집·생성, 캐릭터 일관성 |

---

## 5. 비디오 생성 모델

| 모델 | 제공사 | 핵심 특화 |
|---|---|---|
| **Sora 2** | OpenAI | 가장 긴 일관된 비디오, 동기화된 대화·음향 생성 |
| **Veo 3.1** | Google DeepMind | 4K 출력, 세로 비디오, 네이티브 오디오, YouTube 통합 |
| Veo 3 | Google | 텍스트→비디오+오디오, 4/6/8초 |
| **Runway Gen-4.5** | Runway | 세계 1위 비디오 모델 평가, 편집·이펙트 통합 |
| **Kling AI 2.6** | Kuaishou | 동기화 음향·대화, 중국 플랫폼 |
| **Hailuo AI** | MiniMax | 비디오+음악 생성 |
| Pika | Pika Labs | 단편 비디오 특화 |

---

## 6. 음성/TTS/STT 모델

| 모델 | 제공사 | 유형 | 핵심 특화 |
|---|---|---|---|
| **ElevenLabs v3** | ElevenLabs | TTS | 70+언어, 감정 태그, 보이스 클로닝, 에이전트 플랫폼, 음악 생성 |
| **OpenAI GPT-4o mini TTS** | OpenAI | TTS | 초자연적 음성, Realtime API 통합 |
| **Cartesia Sonic-3** | Cartesia | TTS | 실시간 스트리밍 최적화 |
| **OpenAI Whisper v3** | OpenAI | STT | 99개 언어, 오픈소스 |
| **Deepgram** | Deepgram | STT | 실시간 STT, 개발자 API |
| **Gemini 2.5 Pro TTS** | Google | TTS | Gemini 통합 TTS |
| Chatterbox | Resemble AI | TTS | 오픈소스, 프로덕션급 |
| MiniMax Speech-02 | MiniMax | TTS | 다국어 TTS |
| **Mistral Voxtral** | Mistral AI | 번역 | 실시간 AI 음성 번역 |

---

## 7. 음악 생성 모델

| 모델 | 제공사 | 핵심 특화 |
|---|---|---|
| **Suno v4** | Suno | 200만+ 유료 구독자, 1억+ 사용자, 가장 대중적 |
| **Udio** | Udio | 가사 리얼리즘·보컬 다양성 최고 |
| **ElevenLabs Music** | ElevenLabs | TTS 통합 음악 생성 |

---

## 8. 임베딩/검색 특화 모델

| 모델 | 제공사 | 핵심 특화 |
|---|---|---|
| **Voyage 4** | Voyage AI (Anthropic 인수) | MoE 공유 임베딩 공간, 2026-01 출시 |
| **Cohere Embed v4** | Cohere | RAG 검색 최적화, 다국어 |
| **OpenAI text-embedding-3-large** | OpenAI | 범용 시맨틱 검색, $0.13/1M tokens |
| OpenAI text-embedding-3-small | OpenAI | $0.02/1M tokens, 경량 |
| **Qwen3-Embedding** | Alibaba | 오픈소스 임베딩 |
| **BGE-M3** | BAAI | 오픈소스 다국어 임베딩 |
| Jina v3 | Jina AI | 오픈소스, 다국어 |

---

## 9. 검색 증강(Search-Augmented) 모델

| 모델 | 제공사 | 핵심 특화 |
|---|---|---|---|
| **Perplexity Sonar / Sonar Pro** | Perplexity AI | 실시간 웹 검색+LLM 통합 API, 2-3배 더 많은 출처 인용, Llama 3.3 70B 기반 |
| **Perplexity "Computer"** | Perplexity AI | 다른 AI 에이전트에게 작업 위임하는 메타 에이전트 (2026-02 발표) |

---

## 10. 멀티에이전트/오케스트레이션 프레임워크

| 프레임워크 | 제공사 | 핵심 특화 | AADS 관련성 |
|---|---|---|---|
| **LangGraph** | LangChain | 상태 기반 순환 그래프, 가장 유연한 에이전트 오케스트레이션 | **Phase 2 도입 예정** |
| **CrewAI** | CrewAI | 역할 기반 다중 에이전트, 쉬운 설정 | 참고 |
| **Microsoft AutoGen** | Microsoft | 대화형 멀티에이전트 | 참고 |
| **MetaGPT** | DeepWisdom | SOC 기반 소프트웨어 회사 시뮬레이션 | **AADS 경쟁 벤치마크** |
| **Google ADK** | Google | 프로덕션 레디 에이전트 프레임워크 | 참고 |
| **Semantic Kernel** | Microsoft | .NET/Python 에이전트 통합 | 참고 |
| **OpenAI Agents SDK** | OpenAI | 핸드오프·가드레일·트레이싱 | 참고 |

---

## 11. API 가격 종합 비교 (2026-02 기준, /1M tokens)

| 모델 | Input | Output | 가성비 등급 |
|---|---|---|---|
| GPT-5 nano | $0.05 | $0.40 | **S** (초저가) |
| Gemini 2.5 Flash-Lite | $0.10 | $0.40 | **S** |
| MiMo-V2-Flash (Xiaomi) | $0.10 | $0.30 | **S** |
| GPT-5 mini | $0.25 | $2.00 | **A** |
| Gemini 3 Flash | $0.50 | $3.00 | **A** |
| Qwen 3.5 (122B) | $0.40 | $1.20 | **S** (셀프호스팅 시) |
| Claude Haiku 4.5 | $1.00 | $5.00 | **B** |
| GPT-5 / GPT-5.2 Chat | $1.25~$1.75 | $10.00 | **B** |
| Gemini 3.1 Pro | $2.00 | $12.00 | **A+** (성능 대비) |
| Claude Sonnet 4.6 | $3.00 | $15.00 | **A** (품질 대비) |
| Claude Opus 4.6 | $5.00 | $25.00 | **B** (최고 품질) |
| GPT-5.2 Pro | $10.50 | $84.00 | **C** (프리미엄) |
| DeepSeek-V3.2 | 매우 저가 | 매우 저가 | **S** (데이터 주권 주의) |

### 월 비용 시뮬레이션 (5M tokens / 25M tokens, 70:30 input:output 비율)

| 모델 | 5M tokens/월 | 25M tokens/월 |
|---|---|---|
| Qwen 3.5 | $3.20 | $16.00 |
| GPT-5.3 Codex (추정) | $19.38 | $96.88 |
| Gemini 3.1 Pro | $25.00 | $125.00 |
| Claude Sonnet 4.6 | $33.00 | $165.00 |
| Claude Opus 4.6 | $55.00 | $275.00 |

---

## 12. AADS 에이전트별 모델 배정 전략 (권고안)

| 에이전트 역할 | 주 모델 | 보조/대체 모델 | 선정 근거 |
|---|---|---|---|
| **CEO (Supervisor AI)** | Claude Opus 4.6 | Gemini 3.1 Pro | 최고 판단력, 팀 편성 정확도 |
| **PM / PO** | Claude Sonnet 4.6 | GPT-5.2 Chat | 기획·PRD·이슈 생성 범용성 |
| **System Architect** | Claude Opus 4.6 | Gemini 3.1 Pro | 복잡한 아키텍처 설계 정밀도 |
| **Full-stack Developer** | Claude Sonnet 4.6 | GPT-5.3 Codex | 코드 생성 밸런스 |
| **Terminal/DevOps Engineer** | GPT-5.3 Codex | Claude Sonnet 4.6 | Terminal-Bench 1위 |
| **QA Engineer** | Claude Sonnet 4.6 | - | 테스트 케이스 생성 |
| **UI/UX Designer** | Gemini 3.1 Pro | Claude Sonnet 4.6 | 멀티모달 비전·디자인 이해 |
| **시장조사 애널리스트** | Perplexity Sonar Pro | Gemini 3.1 Pro + Tavily | 실시간 검색 |
| **데이터 파이프라인 전문가** | Gemini 2.5 Flash | DeepSeek-V3.2 | 대량 처리 저가 |
| **의료/건강 도메인 전문가** | Claude Opus 4.6 | - | 정확성 최우선 |
| **분류/태깅/라우터** | GPT-5 nano | Gemini 2.5 Flash-Lite | 초저가 대량 |
| **문서 관리자** | Claude Haiku 4.5 | - | 요약·정리 효율 |
| **비용 감시자 (CFO)** | GPT-5 mini | - | 중간 성능·저가 |
| **콘텐츠/마케팅** | Claude Sonnet 4.6 | Qwen 3.5 | 다국어·창의성 |
| **번역** | Mistral Voxtral | Qwen 3.5 | 실시간 다국어 번역 특화 |
| **OCR/문서 파싱** | Mistral OCR 3 | Gemini 3 Flash | 문서 구조 보존 |
| **이미지 생성** | FLUX.2 [dev] | GPT Image 1.5 | 품질+오픈소스 |
| **음성 합성 (TTS)** | ElevenLabs v3 | OpenAI TTS | 자연스러움 최고 |
| **음성 인식 (STT)** | OpenAI Whisper v3 | Deepgram | 다국어·오픈소스 |
| **임베딩/검색** | Voyage 4 | Cohere Embed v4 | RAG 정확도 |

---

## 13. 월간 비용 시뮬레이션 (AADS 운영 기준)

**시나리오: 월 2,500만 토큰 사용 (약 10,000 페이지 분량)**

| 구성 | 월 추정 비용 |
|---|---|
| 전부 Claude Opus 4.6 | ~$275 |
| 전부 Claude Sonnet 4.6 | ~$165 |
| 전부 Gemini 3.1 Pro | ~$125 |
| **권장 혼합** (Opus 10%, Sonnet 50%, Haiku 20%, GPT-5 nano 20%) | **~$85** |
| 전부 Qwen 3.5 (API) | ~$16 |

---

## 14. 핵심 트렌드 요약 (2026-02 기준)

### 트렌드 1: 모델의 용도별 분화
단일 "최고" 모델은 없다. 작업별 최적 모델을 선택하는 라우팅 전략이 핵심.
→ **AADS의 동적 에이전트 레지스트리 + 모델 배정 시스템의 존재 이유**.

### 트렌드 2: 가격 98% 하락
2023년 GPT-4 수준 성능이 $60/1M → $0.75/1M 이하로 하락.
→ AADS의 대량 에이전트 호출이 경제적으로 가능.

### 트렌드 3: 에이전틱 AI 주류화
- Grok 4.20의 4-에이전트 병렬 실행
- Perplexity Computer의 메타 에이전트
- Qwen 3.5의 네이티브 에이전트
→ 모든 제공사가 에이전트 우선 설계로 전환.

### 트렌드 4: 오픈소스가 프론티어에 근접
- Qwen 3.5-Medium → Sonnet 4.5급
- MiMo-V2-Flash, Llama 4 Maverick, Mistral Large 3 → 상용 모델과 경쟁
→ 셀프호스팅 옵션으로 비용·데이터 주권 동시 확보 가능.

### 트렌드 5: 한국어 특화 모델은 제한적
NAVER HyperCLOVA X가 유일한 한국어 최적화 모델이나 API 접근성 한계.
글로벌 모델(Claude, GPT-5, Gemini)의 한국어 성능이 실무에 충분한 수준.

---

## 부록: 벤치마크 비교표 (2026-02 주요 모델)

| Benchmark | Gemini 3.1 Pro | Claude Opus 4.6 | Claude Sonnet 4.6 | GPT-5.3 Codex | Grok 4.20 | Qwen 3.5 |
|---|---|---|---|---|---|---|
| ARC-AGI-2 (Novel reasoning) | **77.1%** | 68.8% | 60.4% | 52.9% | ~16%† | 12% |
| GPQA Diamond (PhD science) | **94.3%** | 91.3% | 89.9% | 92.4% | ~88%† | 88.4% |
| SWE-Bench Verified (GitHub) | 80.6% | **80.8%** | 79.6% | — | ~72-75%† | 76.4% |
| Terminal-Bench 2.0 (DevOps) | 68.5% | 65.4% | 59.1% | **77.3%** | — | — |
| Humanity's Last Exam | 51.4% | **53.1%** | 49.0% | — | ~41%† | 28.7% |
| GDPval-AA Elo (Office work) | 1,317 | 1,606 | **1,633** | ~1,580* | — | — |
| Context Window | **1M** | 200K (1M β) | **1M (β)** | 400K | 256K-2M | **1M** |

*추정값 | †Grok 4.20 공식 벤치마크 미공개 (2026-03 예상)

---

## 변경 이력

| 버전 | 날짜 | 변경 내용 |
|---|---|---|
| v1.0 | 2026-02-28 | 초판 작성 — 전수 조사 완료 |

---

*이 보고서는 AADS Phase 0-A 에이전트 레지스트리 구축 시 모델 배정 기준으로 활용됩니다.*
