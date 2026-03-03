# CONTABO-MIGRATION-001 — Contabo VPS 마이그레이션 체크리스트

> task_id: CUR-AADS-INFRA-CONTABO-009
> 목표일: 2026-04-01
> 작성일: 2026-03-03
> 작성자: Claude Agent (Cursor)
> 상태: DRAFT — CEO 승인 대기 중

---

## 0. 현재 인프라 현황 (DigitalOcean, 68.183.183.11)

| 항목 | 현재값 |
|------|--------|
| OS | CentOS Linux 7 (Core) |
| Kernel | 3.10.0-1062.9.1.el7.x86_64 |
| RAM | 15GB (사용 8.3GB / 여유 1.7GB) |
| 루트 디스크 | 160GB (사용 148GB / **93% 사용 — 위험 수위**) |
| 마운트 볼륨 | /mnt/volume_sgp1_01 50GB (사용 29GB / 60%) |
| aads-core | 1.2GB |
| aads-dashboard | 503MB |
| aads-docs | 2.9MB |
| aads-server | 1.9MB |

> ⚠️ 루트 디스크 93% 사용 중 — 마이그레이션 추가 동기

---

## 1. Contabo VPS 스펙 선정

| 항목 | 값 |
|------|----|
| 플랜 | VPS 30 |
| vCPU | 8 코어 |
| RAM | 30GB |
| SSD | 800GB NVMe |
| 월 비용 | $12.99/월 |
| OS | Ubuntu 22.04 LTS |
| 리전 | Singapore (현 서비스 지역과 동일) |

> ⚠️ **CEO 승인 필요**: Contabo VPS 구매 승인 ($12.99/월)

---

## 2. 마이그레이션 단계별 체크리스트

### Phase 1 — Contabo 서버 초기화

- [ ] 2-1. Contabo VPS 주문 및 결제 (CEO 승인 후 진행)
- [ ] 2-2. Contabo 관리 패널에서 Ubuntu 22.04 LTS 설치
- [ ] 2-3. root 비밀번호 변경 및 SSH 키 등록
- [ ] 2-4. ufw 방화벽 설정 (22, 80, 443, 8000 개방)
- [ ] 2-5. swap 설정 (4GB 권장)

### Phase 2 — 소프트웨어 설치

- [ ] 2-6. apt update && apt upgrade -y
- [ ] 2-7. Docker CE 설치 (공식 스크립트)
- [ ] 2-8. Docker Compose v2 설치
- [ ] 2-9. Nginx 설치
- [ ] 2-10. Certbot (Let's Encrypt) 설치
- [ ] 2-11. Git 설치
- [ ] 2-12. Python 3.11+ 확인

### Phase 3 — 코드 배포

- [ ] 3-1. GitHub clone: `git clone https://github.com/moongoby-GO100/aads-server`
- [ ] 3-2. GitHub clone: `git clone https://github.com/moongoby-GO100/aads-dashboard`
- [ ] 3-3. GitHub clone: `git clone https://github.com/moongoby-GO100/aads-docs`
- [ ] 3-4. .env 파일 수동 복사 (R-003 준수 — 키는 git 커밋 금지)
  ```bash
  # 구 서버에서 실행:
  scp /root/aads/aads-server/.env root@<NEW_IP>:/root/aads/aads-server/.env
  ```

### Phase 4 — 데이터베이스 마이그레이션

- [ ] 4-1. 구 서버에서 PostgreSQL dump
  ```bash
  docker exec <pg_container> pg_dump -U aads aads_db > /tmp/aads_db_$(date +%Y%m%d).sql
  ```
- [ ] 4-2. dump 파일 전송
  ```bash
  scp /tmp/aads_db_*.sql root@<NEW_IP>:/tmp/
  ```
- [ ] 4-3. 새 서버에서 PostgreSQL 복원
  ```bash
  docker exec -i <pg_container> psql -U aads aads_db < /tmp/aads_db_*.sql
  ```

### Phase 5 — Nginx + SSL 설정

- [ ] 5-1. `/etc/nginx/sites-available/aads` 설정 파일 작성
- [ ] 5-2. Certbot으로 SSL 인증서 발급
  ```bash
  certbot --nginx -d aads.newtalk.kr
  ```
- [ ] 5-3. Nginx 재시작 및 HTTPS 확인

### Phase 6 — Docker 서비스 기동

- [ ] 6-1. `docker compose up -d --build`
- [ ] 6-2. health check 확인
  ```bash
  curl -s http://localhost:8000/health | python3 -m json.tool
  ```
- [ ] 6-3. 로그 확인: `docker compose logs -f`

### Phase 7 — DNS 전환

- [ ] 7-1. DNS 레코드 변경: `aads.newtalk.kr` → Contabo 신규 IP
- [ ] 7-2. DNS 전파 확인 (5~30분 소요)
  ```bash
  nslookup aads.newtalk.kr 8.8.8.8
  ```
- [ ] 7-3. 전체 E2E 테스트 수행

### Phase 8 — 정리

- [ ] 8-1. 마이그레이션 완료 확인 후 **7일 모니터링**
- [ ] 8-2. 7일 후 DigitalOcean Droplet 삭제 (비용 절감)
- [ ] 8-3. 마이그레이션 완료 보고서 작성

---

## 3. 위험 요소 및 대응

| 위험 | 대응 |
|------|------|
| DNS 전파 지연 (최대 30분) | 점검 시간대(새벽) 선정 |
| PostgreSQL 데이터 손실 | 마이그레이션 전 전체 dump 백업 |
| .env 파일 유실 | 마이그레이션 전 안전한 위치에 임시 보관 |
| 서비스 다운타임 | DNS TTL 300초로 낮춰두기 (24시간 전) |

---

## 4. 예상 다운타임

| 단계 | 소요 시간 |
|------|-----------|
| DNS 전파 | 5~30분 |
| Docker 빌드 | 5~10분 |
| DB 복원 | 1~5분 (DB 크기에 따라) |
| **합계** | **최대 45분** |

---

## 5. 롤백 계획

DNS 전환 후 문제 발생 시:
```bash
# DNS를 DigitalOcean IP로 즉시 복원
# aads.newtalk.kr → 68.183.183.11
```
DigitalOcean Droplet은 **7일간 유지** 후 삭제.

---

## 6. 관련 제약사항

- **R-003**: .env 등 비밀 키 파일 git 커밋 금지 — scp 수동 전송 필수
- **R-007**: 서비스 중단 예상 시 CEO 사전 승인 필요

---

## 7. 참고 스크립트

→ `/root/aads/aads-server/scripts/migrate-contabo.sh` 참조

---

*HANDOVER: v4.0 | 생성: CUR-AADS-INFRA-CONTABO-009*
