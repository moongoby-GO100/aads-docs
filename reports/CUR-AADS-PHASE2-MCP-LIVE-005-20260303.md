# CUR-AADS-PHASE2-MCP-LIVE-005 완료 보고서
**날짜**: 2026-03-03  
**작업자**: Cursor (Claude)  
**커밋**: moongoby-GO100/aads-server@046111f  
**HTTP**: 200  

---

## 1. 작업 요약

Phase 2 MCP 서버 실제 구동 + 에이전트 프롬프트 튜닝 + 테스트 커버리지 확대

---

## 2. 변경 파일 (21개)

### MCP 서버 신규 (4파일)
| 파일 | 내용 |
|------|------|
| `mcp_servers/__init__.py` | 패키지 초기화 |
| `mcp_servers/filesystem_server.py` | FastMCP SSE 서버 (포트 8765) — read/write/list/create/delete/file_info 6개 도구 |
| `mcp_servers/git_server.py` | FastMCP SSE 서버 (포트 8766) — log/status/diff/show/branches/file_history 6개 도구, 읽기 전용 |
| `mcp_servers/memory_server.py` | FastMCP SSE 서버 (포트 8767) — store/retrieve/delete/list/search/namespaces/clear 7개 도구 |

### 인프라 변경 (2파일)
| 파일 | 내용 |
|------|------|
| `supervisord.conf` | aads-api + mcp-filesystem + mcp-git + mcp-memory 4개 프로세스 관리 |
| `Dockerfile` | supervisord + git 설치, EXPOSE 8765/8766/8767 추가, CMD → supervisord -c /app/supervisord.conf |

### MCP 클라이언트 개선 (1파일)
| 파일 | 내용 |
|------|------|
| `app/mcp/client.py` | initialize()에 asyncio.gather 병렬 HTTP ping 추가 — 실제 도달 가능 서버만 _available_servers에 등록 |

### 에이전트 프롬프트 (8파일)
모든 에이전트 시스템 프롬프트 전면 개선:
- **PM**: 역할/규칙/응답형식 구체화, `priority`·`assumptions` 필드 추가
- **Supervisor**: 유효성 검증, research 자동 감지, 비용 경고 로직 추가
- **Architect**: YAGNI 원칙, 테스트 전략, entry_point 출력 추가
- **Developer**: 다언어 지원, 재작업 프로토콜, 품질 기준 명시
- **QA**: 커버리지 우선, 경계값 테스트, 명확한 실패 메시지 원칙
- **Judge**: 점수 체계 구체화, criteria_met/criteria_failed 필드 추가
- **DevOps**: 롤백 전략, runtime/install_cmd/run_cmd 필드 추가
- **Researcher**: 옵션 비교 테이블, 구현 코드 스니펫, [불확실] 태그 규칙

### 신규 테스트 (6파일)
| 파일 | 커버 대상 | 테스트 수 |
|------|----------|----------|
| `tests/unit/test_routing.py` | graph/routing.py | 23개 |
| `tests/unit/test_supervisor_agent.py` | agents/supervisor.py | 8개 |
| `tests/unit/test_model_router.py` | services/model_router.py | 14개 |
| `tests/unit/test_sandbox.py` | services/sandbox.py | 4개 |
| `tests/unit/test_api_health.py` | api/health.py | 3개 |
| `tests/unit/test_mcp_servers.py` | mcp_servers/ (3개) | 21개 |

---

## 3. 검증 결과

| 항목 | 결과 |
|------|------|
| 단위 테스트 | **118/118 PASS** |
| 테스트 커버리지 | **62%** (목표 55+ 달성, 기존 43% 대비 +19%p) |
| routing.py 커버리지 | **100%** (기존 56%) |
| supervisor.py 커버리지 | **97%** (기존 20%) |
| model_router.py 커버리지 | **90%** (기존 48%) |
| sandbox.py 커버리지 | **92%** (기존 24%) |
| health.py 커버리지 | **100%** (기존 0%) |
| mcp_servers/ 커버리지 | **신규 — Filesystem 91%, Memory 96%, Git 60%** |

---

## 4. 적용 및 배포 상태

- **적용**: ✅ aads-server 코드 반영 완료
- **커밋**: 046111f (aads-server main)
- **push**: ✅ GitHub 반영 완료
- **Docker rebuild**: ❌ 미적용 (supervisord 포함 이미지 재빌드 필요)
  - 재빌드 명령: `cd /root/aads/aads-server && docker compose up -d --build`
  - 주의: 재빌드 중 약 3~5분 서비스 중단 발생 가능
- **HANDOVER**: ✅ v3.6 업데이트 완료

---

## 5. 후속 체크사항

- [ ] `docker compose up -d --build` — supervisord 포함 이미지 재빌드
- [ ] 재빌드 후 포트 8765/8766/8767 응답 확인 (`curl http://localhost:8765/sse`)
- [ ] E2B 실제 API 키 설정 후 실전 코드 생성 테스트
- [ ] AADS_ADMIN_PASSWORD 환경변수 실제 값 설정
- ⚠️ Docker rebuild 시 컨테이너 재시작 — 실사용 중이면 유지보수 시간대 권장

---

## 6. 다음 작업 제안

1. **Docker rebuild 배포** — supervisord + MCP 서버 실서비스 반영
2. **E2B 실전 연동** — API 키 설정 후 실제 코드 생성/실행 E2E 테스트
3. **Phase 3 기획** — SaaS 멀티유저, 결제 연동 범위 확정

HANDOVER.md 업데이트 완료: 046111f
