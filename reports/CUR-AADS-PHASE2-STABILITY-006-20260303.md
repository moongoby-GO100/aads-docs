# CUR-AADS-PHASE2-STABILITY-006 — P0 안정성 강화 완료 보고서
> 날짜: 2026-03-03 | 담당: Cursor | 커밋: TBD

---

## 1. 요약

PHASE2-STABILITY-006 P0-BLOCKER 작업 완료.
보안 키 강화, PostgreSQL 로컬 연결, MCP SSE ping 버그 수정, 3단계 Checkpointer fallback 구현.

---

## 2. 변경 파일

| 파일 | 변경 내용 |
|------|----------|
| `aads-server/.env` | JWT_SECRET_KEY 27자→86자, AADS_ADMIN_PASSWORD 15자→43자, DATABASE_URL 추가 |
| `aads-server/docker-compose.yml` | postgres 컨테이너 제거(호스트 사용), extra_hosts host.docker.internal 추가 |
| `aads-server/app/config.py` | DATABASE_URL 필드 추가 (str, default ""), SUPABASE_DIRECT_URL optional로 변경 |
| `aads-server/app/services/checkpointer.py` | 3단계 fallback (local_postgres → supabase → MemorySaver), IPv6 차단 |
| `aads-server/app/mcp/client.py` | SSE ping `stream=True` 수정 (무한스트림 timeout 버그 해결) |
| `aads-docs/HANDOVER.md` | v3.7 업데이트 |

---

## 3. 상세 변경 내용

### 3-1. 보안 키 강화
- **JWT_SECRET_KEY**: 27자(약함) → 86자 `secrets.token_urlsafe(64)` 생성
- **AADS_ADMIN_PASSWORD**: 15자 → 43자 `secrets.token_urlsafe(32)` 생성
- **R-003 준수**: `.env` git commit 금지 (gitignore 확인)

### 3-2. PostgreSQL 연결 (호스트 DB 활용)
- 호스트에 이미 PostgreSQL 9.6이 실행 중 (포트 5432 충돌로 컨테이너 불필요)
- 호스트 postgres에 `aads` DB + `aads` 유저 생성 (`aads_dev_local`)
- `postgresql.conf`: `listen_addresses = '*'` 변경
- `iptables`: Docker 네트워크(172.17.0.0/16, 172.18.0.0/16) → 5432 ACCEPT
- `docker-compose.yml`: `extra_hosts: host.docker.internal:host-gateway` 추가
- **연결 테스트**: 컨테이너 내 `psycopg.connect()` 성공, 로그 `checkpointer_ready source=local_postgres`

### 3-3. Checkpointer 3단계 Fallback
```
1순위: DATABASE_URL (local postgres — host.docker.internal:5432)
2순위: SUPABASE_DIRECT_URL (Supabase port 5432, R-011)
3순위: MemorySaver (WARNING 로그 + 영속성 없음 안내)
```
- IPv6 주소(`::1`) 자동 차단 — Docker DNS 또는 IPv4만 허용

### 3-4. MCP SSE Ping 버그 수정
- **원인**: `httpx.AsyncClient().get(url)` → SSE 스트림 무한대기 → 3초 timeout → unreachable 오판
- **수정**: `client.stream("GET", url)` 사용, 헤더 수신 즉시 status_code 반환
- **결과**: 기동 로그 `mcp_client_initialized servers=["memory","git","filesystem"]`

---

## 4. 검증 결과

| 항목 | 결과 |
|------|------|
| 유닛 테스트 | 118/118 PASS |
| Docker 기동 | ✅ supervisord + aads-api + mcp×3 |
| PostgreSQL 연결 | ✅ checkpointer_ready source=local_postgres |
| MCP 연결 | ✅ 3개 서버 인식 (memory, git, filesystem) |
| 헬스체크 | ✅ `{"status":"ok","graph_ready":true}` |

---

## 5. 인프라 변경 (영속적 적용 필요)

> ⚠️ **iptables 규칙은 재부팅 시 초기화됨** — 영속 적용 필요

```bash
# /etc/rc.local 또는 iptables-save로 영속화
iptables -I INPUT -s 172.17.0.0/16 -p tcp --dport 5432 -j ACCEPT
iptables -I INPUT -s 172.18.0.0/16 -p tcp --dport 5432 -j ACCEPT
iptables-save > /etc/sysconfig/iptables
```

---

## 6. 적용 상태

| 항목 | 상태 |
|------|------|
| 코드 수정 | ✅ 완료 |
| Docker 재빌드 | ✅ 완료 (`aads-server` 재시작) |
| 테스트 | ✅ 118 PASS |
| GitHub 푸시 | TBD (커밋 예정) |

---

## 7. 후속 체크사항

- [ ] iptables 규칙 영속화 (`iptables-save`)
- [ ] E2E-FULLCYCLE-007 (P1): 로그인 → 프로젝트 생성 → 파이프라인 E2E 검증
- [ ] PHASE3-PLAN-008 (P2): Phase 2 완료보고서 + Phase 3 SaaS 기획서
- [ ] INFRA-CONTABO-009 (P2): CEO 승인 후 마이그레이션 실행

---

## 허브 전달용 (복사)

```
## [AADS] PHASE2-STABILITY-006 — 완료 보고
변경: .env (JWT/PW), docker-compose.yml, config.py, checkpointer.py, mcp/client.py, HANDOVER.md (6파일)
내용: JWT 86자 보안키, 호스트 PostgreSQL 연결(iptables+host-gateway), MCP SSE ping 수정, 3단계 fallback
검증: 118/118 PASS, checkpointer_ready source=local_postgres, MCP 3서버 인식
적용: ✅ | 배포: ✅ Docker 재빌드 완료

체크:
- [ ] iptables 영속화 (/etc/sysconfig/iptables)
- [ ] E2E-FULLCYCLE-007 (P1) 다음 작업
- ⚠️ 재부팅 시 iptables 5432 규칙 재적용 필요
```
