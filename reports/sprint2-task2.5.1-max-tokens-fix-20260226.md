# Sprint 2 Task 2.5.1 — max_tokens 이슈 수정

**일시**: 2026-02-26 18:55 KST  
**이슈**: 파이프라인 Planning 단계에서 API 400 에러 — `max_tokens: 200000 > 128000` (Claude Opus 4.6 출력 한도 초과)  
**원인**: `model_router.py`의 `max_tokens`가 컨텍스트 창 크기(200000 등)로 설정되어 있는데, 에이전트가 이를 API **출력 토큰 한도**로 그대로 전달함.  
**수정**: API 호출용 출력 한도를 tier별 적정값으로 변경. 컨텍스트 창은 `context_window` 필드로 분리.

## 변경 파일

- `core/model_router.py` (1파일)

## 변경 내용

| Tier | 용도 | max_tokens (출력 한도) | context_window (참고) |
|------|------|------------------------|------------------------|
| tier1 | Planner/Designer (Opus) | 4096 | 200000 |
| tier2 | Developer/QA/DevOps (Sonnet) | 4096 | 1000000 |
| tier3 | Ops (Gemini) | 2048 | 1048576 |
| tier4 | Cost (Haiku) | 1024 | 200000 |

- `MODEL_CONFIG`에 `context_window` 필드 추가(기존 max_tokens 값 이관).
- `max_tokens`를 API 출력 한도(4096/2048/1024)로 재정의.
- 에이전트(`agents/*/agent.py`)는 기존대로 `model["max_tokens"]` 사용 → 수정 없음.

## 테스트

```text
python tests/test_pipeline.py
```

- **수정 전**: Planning 단계에서 `Error code: 400 - max_tokens: 200000 > 128000` 발생.
- **수정 후**: Ideation → Planning 완료, Design 단계로 진행. exit 0.  
  - 예: 모델 Claude Opus 4.6, output 4096 토큰, 비용 $0.1043.

## 적용·배포

- 적용: ✅ `core/model_router.py` 반영.
- 배포: 로컬/CI만 해당, 별도 서버 배포 없음.

## 진행/체크사항

- [ ] 필요 시 Tier별 출력 한도(4096/2048/1024) 재조정.
- ⚠️ `context_window`를 참조하는 다른 코드가 있으면 해당 경로에서 사용처 확인.
