---
doc_id: AADS-LAYOUT-020
project: AADS
type: layout
status: active
created_at: 2026-04-28
updated_at: 2026-04-28
owner_role: PromptEngineer
source_session: ce19b443
related_routes:
  - /admin/prompts
  - /chat
tags:
  - prompt-assets
  - layer3
  - layer5
  - model-routing
---

# AADS Prompt Governance Layer 3/5 통합 기획 보고서

- 작성일: 2026-04-28
- 대상: `prompt_assets`, `/chat`, `/admin/prompts`, `PromptCompiler`
- 범위: Layer 3 역할 매칭, Layer 5 모델 매칭, 회사/제공사별 모델 특성, 성능 기반 적용 기준

## 1. 핵심 결론

현재 AADS 프롬프트 운영에서 Layer 3와 Layer 5는 DB 구조상 존재하지만, 실제 채팅 실행 경로에서는 충분히 정밀하게 적용되지 않는다. Layer 3는 `PromptCompiler.compile()`이 `role`을 받을 수 있음에도 실제 채팅 경로에서 프로젝트 코드가 role처럼 전달되고 있어 `CEO`, `CTO`, `PM`, `Developer`, `QA`, `Ops` 같은 역할 자산이 안정적으로 매칭되지 않는다. Layer 5는 `target_models` exact match 중심이고 현재 등록 자산도 Claude 계열 일부에 한정되어 있어, 채팅창에서 선택한 모델과 제공사별 특성/성능을 반영하기 어렵다.

개선 방향은 세 가지다. 첫째, 세션 생성 시 프로젝트와 별도로 `role_key`를 지정하게 해서 Layer 3가 사용자/세션의 실제 역할을 기준으로 적용되게 한다. 둘째, 채팅창 모델 선택값을 `selected_model_id`, 백엔드 실행 모델을 `execution_model_id`로 분리하고 provider/family/category/capability/performance key를 만들어 Layer 5 매칭에 사용한다. 셋째, Layer 5 prompt asset은 특정 모델 ID뿐 아니라 `provider:openai`, `provider:anthropic`, `family:claude`, `capability:thinking`, `category:coding` 같은 운영 키를 지원하도록 확장한다.

## 2. 현재 코드/DB 진단

### 2.1 Layer 3 역할 매칭 문제

- `chat_sessions`에는 명시적인 `role_key` 컬럼이 없고 `settings jsonb`만 존재한다.
- `SessionCreate`/`SessionOut`에도 역할 필드가 없다.
- `PromptCompiler.compile()`은 `role` 인자를 받을 수 있고 Layer 3 자산은 `role_scope`로 필터링된다.
- 실제 채팅 경로에서는 `chat_service.py`에서 `role=_normalized_project or ""` 형태로 전달되고 있다.
- 결과적으로 프로젝트 코드 `AADS`, `KIS`, `KAKAOBOT` 등이 role로 들어가며, Layer 3 자산의 `role_scope={CEO,CTO,PM,Developer,QA,Ops}`와 매칭되지 않는다.

현재 Layer 3 자산은 다음 역할 중심이다.

- `role-ceo-command`: CEO
- `role-cto-strategist`: CTO
- `role-pm-coordinator`: PM
- `role-developer-implementer`: Developer
- `role-qa-verifier`: QA
- `role-ops-monitor`: Ops
- `role-kakaobot-handler`: KAKAOBOT

`CEO-020[프롬프트전문가]` 같은 운영 시나리오에는 `PromptEngineer` 또는 `PromptArchitect` 역할을 추가해야 한다. 특히 prompt asset 작성, 품질 검수, layer별 운영 기준 수립, DB 반영을 담당하는 세션은 일반 CEO/PM/Developer 역할과 분리하는 것이 맞다.

### 2.2 Layer 5 모델 매칭 문제

현재 Layer 5 prompt assets는 Claude 계열 일부에 치우쳐 있다.

- `model-claude-haiku`: `claude-haiku-4-5`
- `model-claude-sonnet`: `claude-sonnet-4-6`, `claude-sonnet-4-7`
- `model-claude-opus`: `claude-opus-4-6`, `claude-opus-4-7`

채팅 UI는 `current_model`을 세션에 저장하고 메시지 전송 시 `model_override`를 백엔드로 전달한다. 백엔드는 명시 선택된 모델이면 `get_model_for_override(model_override)`로 실행 모델을 잠그고, 이후 `PromptCompiler`에는 `model=model_override or intent_result.model` 형태로 넘긴다. 따라서 명시 선택 모델은 UI 선택값 기준으로 Layer 5가 매칭될 수 있지만, auto/mixture 또는 alias가 개입되면 실제 실행 모델과 프롬프트 매칭 모델이 달라질 수 있다.

문제는 `target_models`가 단일 문자열 exact match 구조로 사용된다는 점이다. 예를 들어 UI 선택값이 `claude-opus-4-7`인데 registry의 canonical 모델이 `claude-opus`이면, 어떤 ID를 넘기느냐에 따라 Layer 5 적용 여부가 달라진다. OpenAI, Gemini, Qwen, Groq, Kimi, DeepSeek, MiniMax, Codex까지 운영하려면 exact model ID만으로는 부족하다.

### 2.3 모델 registry 현황

로컬 `llm_models` registry 기준으로 active/executable 모델은 여러 provider에 분포한다. 확인 시점 기준 provider 축은 `openai`, `qwen`, `gemini`, `groq`, `kimi`, `anthropic`, `codex`, `deepseek`, `minimax` 등이다. `llm_models`에는 `provider`, `model_id`, `display_name`, `family`, `category`, `supports_tools`, `supports_thinking`, `supports_vision`, `supports_coding`, `input_cost`, `output_cost`, `capabilities`, `pricing` 같은 Layer 5에 필요한 운영 정보가 이미 존재한다.

따라서 Layer 5는 정적 prompt asset만이 아니라 registry metadata와 실행 telemetry를 함께 읽는 모델 운영 계층으로 재정의해야 한다.

## 3. 목표 아키텍처

### 3.1 세션 생성 시 필수 선택 축

세션 생성 화면은 최소 3개 축을 명시해야 한다.

1. 프로젝트: `AADS`, `KIS`, `KAKAOBOT` 등 workspace/project 범위
2. 역할: `CEO`, `CTO`, `PM`, `Developer`, `QA`, `Ops`, `PromptEngineer` 등 Layer 3 적용 기준
3. 모델: `mixture/auto` 또는 특정 모델 선택값. Layer 5 적용 기준

프로젝트는 업무/회사/서비스 범위를 결정하고, 역할은 답변 방식과 책임 범위를 결정하며, 모델은 추론 방식과 비용/성능 정책을 결정한다. 이 세 축을 섞어서 쓰면 prompt asset 적용이 불안정해진다.

### 3.2 PromptCompiler 입력 확장

현재 `PromptCompiler.compile(project, role, model, intent)` 형태를 다음처럼 확장한다.

- `project_key`: 세션의 프로젝트/워크스페이스
- `role_key`: 세션 생성 시 지정된 역할
- `selected_model_id`: 채팅창에서 사용자가 선택한 값
- `execution_model_id`: 백엔드에서 실제 실행하기로 결정한 모델
- `model_provider`: registry의 provider
- `model_family`: registry의 family
- `model_category`: registry의 category
- `model_capabilities`: tools/thinking/vision/coding 등 capability set
- `performance_tier`: 최근 telemetry 기반 fast/balanced/deep/unstable/costly 등 운영 등급
- `intent_key`: intent classifier 결과

Layer 5는 단일 `model` 문자열 대신 위 정보를 조합한 `model_match_keys`로 asset을 매칭한다.

예시 match key:

- `claude-opus-4-7`
- `claude-opus`
- `provider:anthropic`
- `family:claude`
- `category:reasoning`
- `capability:tools`
- `capability:thinking`
- `performance:deep`
- `cost:high`

SQL은 `target_models && $model_match_keys::text[] OR '*' = ANY(target_models)` 형태로 바꾸면 기존 `target_models text[]` 컬럼을 그대로 활용할 수 있다.

## 4. Layer 3 역할 운영 설계

### 4.1 권장 역할 목록

초기 역할은 다음 8개로 시작한다.

- `CEO`: 최종 의사결정, 우선순위, 승인/보류 판단
- `CTO`: 기술 구조, 아키텍처, 리스크, 운영 품질
- `PM`: 요구사항 정리, 일정/범위, 인수 기준
- `Developer`: 구현, 코드 수정, 테스트 실행, 배포 준비
- `QA`: 재현, 검증, 회귀 테스트, 오류 보고
- `Ops`: 장애 대응, 로그/모니터링, 서버/배포 운영
- `PromptEngineer`: prompt_assets 작성, Layer별 기준, 프롬프트 품질 검수, DB 반영
- `Analyst`: 데이터/로그/성과 분석, 보고서 작성

`PromptEngineer`는 현재 요청과 직접 연결된다. 이 역할은 prompt asset을 400-800자 수준의 실행 가능한 운영 지침으로 작성하고, Layer 1-5 간 중복/충돌을 제거하며, 적용 후 dry-run으로 매칭 결과를 확인하는 책임을 가진다.

### 4.2 프로젝트별 기본 역할 매핑

역할은 세션 생성 시 직접 선택하되, 프로젝트별 추천값을 제공한다.

- `AADS`: CEO, CTO, PM, Developer, QA, Ops, PromptEngineer
- `KAKAOBOT`: PM, Developer, QA, Ops, PromptEngineer
- `KIS`: Analyst, Developer, QA, Ops
- 신규 프로젝트: PM, Developer, QA를 기본 노출하고 관리자에서 확장

중요한 점은 `project_key`와 `role_key`를 분리 저장하는 것이다. `KAKAOBOT`은 프로젝트이면서 현재 Layer 3 role asset에도 들어가 있으나, 장기적으로는 프로젝트 전용 지침은 Layer 2, 세션 역할은 Layer 3에 배치해야 한다.

### 4.3 DB/API 변경

권장 migration:

```sql
ALTER TABLE chat_sessions
ADD COLUMN IF NOT EXISTS role_key varchar(40);

CREATE INDEX IF NOT EXISTS idx_chat_sessions_project_role
ON chat_sessions(project, role_key);
```

API 변경:

- `SessionCreate.role_key` 추가
- `SessionOut.role_key` 추가
- 세션 update API에서 `role_key` 변경 허용
- 기존 세션은 `settings->>'role_key'`, 세션명 prefix, 프로젝트 기본값 순서로 backfill

UI 변경:

- 새 세션 모달에 역할 select 추가
- 기존 세션 header 또는 settings menu에서 역할 변경 허용
- 역할 변경 시 다음 turn부터 PromptCompiler에 반영
- 세션 목록에는 작은 role badge만 표시하고 채팅 본문을 방해하지 않음

## 5. Layer 5 모델 운영 설계

### 5.1 원칙

Layer 5는 "선택한 모델에 맞춰 프롬프트 문체만 바꾸는 계층"이 아니라 "모델의 회사/제공사 특성, 기능, 비용, 최근 성능 상태에 맞춰 실행 방식을 조정하는 계층"이어야 한다.

Layer 5 자산은 다음 기준을 모두 반영한다.

- 채팅창에서 사용자가 선택한 모델
- auto/mixture에서 백엔드가 실제로 선택한 실행 모델
- provider/family/category/capability
- 최근 latency, error rate, tool success rate, cost per success
- 프로젝트와 역할의 위험도

### 5.2 target_models 토큰 체계

기존 `target_models text[]`를 유지하되 값의 의미를 확장한다.

- exact model: `claude-opus-4-7`, `gpt-5.5`, `gemini-2.5-pro`
- provider: `provider:anthropic`, `provider:openai`, `provider:gemini`, `provider:qwen`
- family: `family:claude`, `family:gpt`, `family:codex`, `family:gemini`, `family:qwen`
- category: `category:coding`, `category:reasoning`, `category:fast`, `category:multimodal`
- capability: `capability:tools`, `capability:thinking`, `capability:vision`, `capability:coding`
- performance: `performance:fast`, `performance:balanced`, `performance:deep`, `performance:unstable`
- cost: `cost:low`, `cost:medium`, `cost:high`

이 방식은 DB schema를 크게 바꾸지 않고도 Layer 5를 provider/company 특성까지 확장할 수 있다.

### 5.3 회사/제공사별 적용 기준

| Provider | 권장 적용 | Layer 5 지침 방향 |
| --- | --- | --- |
| Anthropic/Claude | 장문 맥락, 도구 사용, 코드 리뷰, 안정적 지시 준수, 고난도 판단 | Sonnet은 기본 균형 모델, Opus는 고비용 고난도 작업에 한정, Haiku는 단순 응답/분류에 사용한다. 답변은 근거와 리스크를 분리하고 불확실한 실행은 확인 절차를 둔다. |
| OpenAI/GPT | 범용 추론, 구조화 출력, 코딩, 멀티모달, 복잡한 의사결정 | GPT-5.x 계열은 복잡한 설계/분석/코딩에 사용하고 mini/nano 계열은 비용 민감 작업에 사용한다. JSON/스키마/검증 루프가 필요한 작업에서 강하게 활용한다. |
| Codex | 코드베이스 수정, 테스트, patch 중심 에이전트 작업 | 파일 경계, 테스트 명령, 변경 요약을 명확히 요구한다. repo 작업에서는 role이 Developer/QA/Ops일 때 우선 후보로 둔다. |
| Google/Gemini | 대용량 컨텍스트, 빠른 요약, 멀티모달, 비용 효율 | 긴 문서/로그/이미지 분석에 강점을 둔다. preview 모델은 최종 결정 전 검증 모델 또는 테스트 루프를 붙인다. |
| Alibaba/Qwen | 비용 효율, 한국어/아시아권 언어, 코딩/추론 대량 처리 | 반복 분석, 초안, 다국어 변환, 비용 민감 세션에 적합하다. 고위험 사실 판단은 검증 모델을 추가한다. |
| Groq | 초저지연 응답, 간단 분류, 상태 확인, 초안 생성 | 빠른 turn이 중요한 UI 응답에 사용한다. 최종 코드 변경/장기 추론/고위험 결론에는 단독 사용하지 않는다. |
| Kimi/Moonshot | 긴 컨텍스트, 코드/수학/재무 성격 분석 | 대용량 자료를 길게 읽고 정리하는 작업에 배치한다. 산출물은 QA 또는 고신뢰 모델로 재검증한다. |
| DeepSeek | 저비용 추론/코딩 대안 | 비용 제한이 강한 reasoning/coding 후보로 사용한다. 프로덕션 변경은 테스트와 리뷰를 필수로 둔다. |
| MiniMax | agent/tool 실험, 대체 reasoning 후보 | 충분한 telemetry가 쌓이기 전까지 기본값으로 두지 않고 실험 또는 fallback 후보로 운영한다. |

위 기준은 외부 벤치마크를 고정적으로 신뢰하는 방식이 아니라, 로컬 registry metadata와 실제 AADS telemetry로 계속 보정해야 한다.

### 5.4 Layer 5 prompt_assets 권장 세트

초기에는 exact model asset을 많이 만들기보다 provider/family/capability asset을 우선 구축한다. 각 asset content는 400-800자 수준으로 작성해 실제 실행 지침이 되게 한다.

- `model-provider-anthropic`: `target_models={provider:anthropic,family:claude}`
- `model-provider-openai`: `target_models={provider:openai,family:gpt}`
- `model-provider-codex`: `target_models={provider:codex,family:codex,category:coding}`
- `model-provider-gemini`: `target_models={provider:gemini,family:gemini}`
- `model-provider-qwen`: `target_models={provider:qwen,family:qwen}`
- `model-provider-groq`: `target_models={provider:groq,performance:fast}`
- `model-provider-kimi`: `target_models={provider:kimi,family:kimi}`
- `model-provider-deepseek`: `target_models={provider:deepseek,family:deepseek}`
- `model-provider-minimax`: `target_models={provider:minimax}`
- `model-capability-tools`: `target_models={capability:tools}`
- `model-capability-thinking`: `target_models={capability:thinking,category:reasoning}`
- `model-capability-vision`: `target_models={capability:vision}`
- `model-cost-high`: `target_models={cost:high}`
- `model-performance-unstable`: `target_models={performance:unstable}`

exact model asset은 정말 필요한 경우에만 만든다. 예를 들어 특정 Claude Opus 모델만 비용 제한을 강제하거나, 특정 preview 모델만 검증 루프를 강제해야 할 때 사용한다.

## 6. 성능 기반 모델 적용 설계

### 6.1 telemetry source

성능 판단은 다음 데이터에서 산출한다.

- `chat_turn_executions`: turn별 실행 모델, 상태, 오류, latency
- `bg_llm_usage_log`: 토큰, 비용, provider별 사용량
- `oauth_usage_log`: Claude/Codex OAuth 계열 사용량과 제한 상태
- `llm_models`: provider, family, capability, cost metadata
- `chat_model_preferences`: pinned/favorite/hidden/display_order 운영 선호도

### 6.2 model_performance_profile

권장 view 또는 table:

```sql
CREATE MATERIALIZED VIEW model_performance_profile AS
SELECT
  provider,
  actual_model AS model_id,
  count(*) AS turns,
  percentile_cont(0.5) WITHIN GROUP (ORDER BY latency_ms) AS p50_latency_ms,
  percentile_cont(0.95) WITHIN GROUP (ORDER BY latency_ms) AS p95_latency_ms,
  avg(CASE WHEN status = 'error' THEN 1 ELSE 0 END) AS error_rate,
  avg(total_cost) AS avg_cost,
  max(created_at) AS last_seen_at
FROM chat_turn_executions
WHERE created_at >= now() - interval '7 days'
GROUP BY provider, actual_model;
```

실제 컬럼명은 현재 테이블 정의에 맞춰 조정한다. 이 profile에서 `performance:fast`, `performance:balanced`, `performance:deep`, `performance:unstable`, `cost:high` 같은 key를 산출해 Layer 5 매칭에 넣는다.

### 6.3 선택 우선순위

명시 모델 선택 시:

1. 사용자가 선택한 모델을 우선한다.
2. 해당 모델이 inactive/unexecutable이면 UI에서 즉시 경고하고 `mixture` 또는 기본 모델로 fallback한다.
3. PromptCompiler에는 selected/execution/provider/family/capability/performance key를 모두 전달한다.
4. 고비용 또는 불안정 모델이면 Layer 5가 검증/요약/확인 절차를 추가한다.

auto/mixture 선택 시:

1. intent, role, project risk를 먼저 평가한다.
2. 모델 후보를 capability와 cost로 필터링한다.
3. 최근 telemetry가 불안정한 모델은 제외하거나 fallback으로 낮춘다.
4. 최종 실행 모델을 결정한 뒤 그 모델 기준으로 Layer 5를 적용한다.

## 7. 구현 계획

### Phase 1. DB/계약 정리

- `chat_sessions.role_key` 추가
- session create/update/out schema에 `role_key` 추가
- 기존 세션 backfill
- `PromptCompiler`에 `model_match_keys` 입력 추가
- Layer 5 SQL을 exact match에서 overlap match로 변경

### Phase 2. UI 반영

- 새 세션 생성 모달에 역할 select 추가
- 모델 selector는 현재처럼 유지하되 선택 모델의 provider/family/capability badge를 작게 표시
- admin prompts 화면에서 Layer 3/5 scope를 명확히 편집할 수 있게 개선
- 모델이 inactive이면 세션 진입 시 경고와 fallback 표시

### Phase 3. Layer 3/5 prompt_assets 보강

- Layer 3 역할별 content를 400-800자로 확장
- `PromptEngineer` 역할 asset 추가
- Layer 5 provider/family/capability asset 추가
- 기존 Claude exact asset은 provider/family asset과 중복되지 않게 정리
- dry-run으로 project/role/model/intent 조합별 적용 asset 확인

### Phase 4. 성능 profile 적용

- 최근 7일 telemetry 기반 `model_performance_profile` 산출
- model selector와 model router에서 error rate/latency/cost를 참고
- Layer 5에 `performance:*`, `cost:*` key 주입
- 성능 하락 모델은 admin에서 자동 review_required 상태로 표시

## 8. 검증 시나리오

1. AADS 프로젝트에서 `PromptEngineer` 역할, `claude-opus-4-7` 선택 시 Layer 1 + Layer 2(AADS) + Layer 3(PromptEngineer) + Layer 5(Anthropic/Claude/High-cost)가 적용되어야 한다.
2. AADS 프로젝트에서 `Developer` 역할, Codex 모델 선택 시 Layer 3 Developer와 Layer 5 Codex/Coding asset이 적용되어야 한다.
3. KAKAOBOT 프로젝트에서 `QA` 역할, Gemini vision 모델 선택 시 프로젝트 지침, QA 지침, Gemini/Vision 지침이 함께 적용되어야 한다.
4. `mixture` 선택 시 실제 실행 모델이 결정된 뒤 해당 provider/family 기준 Layer 5가 적용되어야 한다.
5. inactive 모델이 세션에 저장되어 있으면 UI가 fallback을 표시하고 백엔드가 실행 불가 모델을 사용하지 않아야 한다.
6. `/admin/prompts`에서 Layer 5 asset의 `target_models`에 `provider:*`, `family:*`, `capability:*` 토큰을 저장/조회/수정할 수 있어야 한다.

## 9. 리스크 및 보완점

- 현재 `context_builder.build_messages_context()`와 `chat_service` 양쪽에서 PromptCompiler 호출 가능성이 있어, Layer 3/5 적용 전 중복 compile 경로를 정리해야 한다.
- role과 project를 혼용한 기존 자산(`role-kakaobot-handler`)은 장기적으로 Layer 2 프로젝트 지침으로 이동하는 것이 좋다.
- provider/company 특성은 고정 문구로 끝내면 빠르게 낡는다. registry와 telemetry를 기준으로 주기적으로 갱신해야 한다.
- exact model ID와 alias/canonical ID가 다를 수 있으므로 `selected_model_id`와 `execution_model_id`를 모두 match key에 넣어야 한다.
- 고비용 모델에 대한 Layer 5 지침은 품질 향상뿐 아니라 비용 제한, 요약 우선, 검증 후 실행 같은 운영 규칙을 포함해야 한다.

## 10. 최종 권고

Layer 3와 Layer 5는 별도 기능이 아니라 같은 세션 라우팅 계약으로 묶어야 한다. 세션은 `project_key`, `role_key`, `selected_model_id`를 보유하고, 백엔드는 실제 실행 시 `execution_model_id`, provider, family, capability, performance key를 계산한다. PromptCompiler는 이 모든 key를 기준으로 prompt_assets를 조합한다.

다음 실행 순서는 `role_key` 컬럼과 API/UI 반영을 먼저 끝낸 뒤, Layer 5의 provider/family/capability token matching을 적용하는 것이다. 그 다음 Layer 3/5 prompt_assets를 400-800자 실행 지침으로 보강하고, admin prompts 화면에서 실제 적용 결과를 dry-run으로 확인하게 만들면 운영자가 채팅창에서 선택한 역할과 모델에 따라 프롬프트가 어떻게 달라지는지 명확히 검증할 수 있다.
