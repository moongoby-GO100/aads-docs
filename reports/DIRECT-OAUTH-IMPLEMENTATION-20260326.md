# Direct OAuth 구현 보고서 (2026-03-26)

## 개요
LiteLLM 프록시 경유 → Anthropic 직접 OAuth 전환 완료.
CEO 채팅창 무영향 원칙 준수, 기능 플래그 + 롤백 스크립트 구비.

## 변경 사항

### 1. Relay Server Direct OAuth (`claude_relay_server.py`)
- **기능 플래그**: `AADS_CLAUDE_DIRECT_OAUTH=1` (systemd drop-in)
- **HOME 격리**: `/tmp/.claude-relay/` 사용 → `settings.json` 충돌 방지
- **토큰 동적 읽기**: `.env.oauth`에서 요청 시점에 CURRENT_OAUTH 읽기
- **oauth_slot 파라미터**: 폴백 체인에서 계정 지정 가능
- **429 감지**: rate_limit/overloaded 패턴 감지 → 다음 요청 토큰 전환
- **resume 실패 처리**: CLI exit code != 0 + resume=True → 세션 매핑 자동 정리
- **`/oauth/switch` API**: POST로 토큰 슬롯 전환 → `.env.oauth` 업데이트

### 2. auth_provider.py
- `ANTHROPIC_AUTH_TOKEN_2` fallback 읽기 추가 (기존 누락 수정)
- 토큰 라벨 자동 판별 (prefix 기반: 3VVR=Gmail, 5ZED=Naver)
- `set_token_order()` → relay `/oauth/switch` API 호출로 `.env.oauth` 동기화
- 프론트 스위치 버튼 → 백엔드 → relay 연동 완료

### 3. model_selector.py — 계정 교차 폴백
```
Opus 요청:  Opus(계정1) → Opus(계정2) → Sonnet(계정1) → Sonnet(계정2) → Gemini
Sonnet 요청: Sonnet(계정1) → Sonnet(계정2) → Gemini
Haiku 요청:  Haiku(계정1) → Haiku(계정2) → Gemini
```
- 같은 모델 다른 계정 우선 시도 (품질 저하 최소화)
- $200 × 2계정 = $400 rate limit 풀 활용
- 각 단계: Tier1(Relay) → Tier2(Agent SDK) 순서

### 4. --no-verify 차단 3계층
| 계층 | 파일 | 역할 |
|------|------|------|
| 차단 | `~/.claude/hooks/pre_tool_use_git_guard.sh` | Claude 세션에서 `--no-verify`, `-n` 실행 전 차단 |
| 강제 | `~/.claude/hooks/post_tool_use_commit_guide.sh` | git commit 실패 시 원인 분석 가이드 자동 주입 |
| 감사 | `.git/hooks/pre-commit` + `commit-msg` | hook 통과 서명 생성 → commit-msg에서 서명 검증 → 우회 시 커밋 차단 |

### 5. 롤백
```bash
bash /root/aads/aads-server/scripts/oauth_switch_rollback.sh
```
- 플래그 OFF + 원본 relay 복원 + systemd drop-in 제거 + 재시작

## 테스트 결과

### Direct OAuth 스트리밍
| 계정 | Opus | Sonnet | Haiku |
|------|:---:|:---:|:---:|
| Gmail | PASS | PASS | PASS |
| Naver | PASS | PASS | PASS |

### 계정 스위치
| 테스트 | 결과 |
|--------|------|
| 프론트 버튼 → relay 전파 | PASS |
| Gmail↔Naver 전환 후 채팅 | PASS |
| `.env.oauth` 동기화 | PASS |

### 폴백
| 테스트 | 결과 |
|--------|------|
| Relay 중단 → Agent SDK | PASS |
| Relay + SDK 실패 → Gemini | PASS |
| Gemini 폴백 (Tier3) | PASS |

### --no-verify 차단
| 테스트 | 결과 |
|--------|------|
| PreToolUse hook 차단 | PASS |
| 정상 git 명령 동작 | PASS |

## 파일 목록
- `scripts/claude_relay_server.py` — Direct OAuth relay
- `scripts/oauth_switch_rollback.sh` — 롤백 스크립트
- `app/core/auth_provider.py` — 토큰 관리 + relay 연동
- `app/services/model_selector.py` — 계정 교차 폴백
- `~/.claude/hooks/pre_tool_use_git_guard.sh` — --no-verify 차단
- `~/.claude/hooks/post_tool_use_commit_guide.sh` — 커밋 실패 가이드
- `.git/hooks/pre-commit` — 서명 생성 추가
- `.git/hooks/commit-msg` — 서명 검증 추가
- `/etc/systemd/system/claude-relay.service.d/oauth-direct.conf` — 플래그

## 발견 및 수정한 기존 버그
1. `auth_provider.py`: `ANTHROPIC_AUTH_TOKEN_2` fallback 누락 → Gmail 토큰 미인식
2. 토큰 라벨 하드코딩 → prefix 기반 자동 판별로 개선
3. Docker 컨테이너에서 `ANTHROPIC_API_KEY_FALLBACK` 환경변수 미전달 (docker-compose 변수 치환 실패)
