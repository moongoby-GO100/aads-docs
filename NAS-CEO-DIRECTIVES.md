# NAS CEO DIRECTIVES
> 최종 업데이트: 2026-03-07 (v1.0) — AADS-143 신규 배포
> 관리자: CEO (moongoby)
> 용도: NAS AI 세션 필수 읽기. 공통 규칙(CEO-DIRECTIVES.md D-016~D-021, R-001~R-016) 참조.

---

## NAS 전용 규칙

### NAS-D-001 파일 안전 최우선
- NAS 작업 시 기존 파일 덮어쓰기 전 반드시 백업 확인
- 대용량 파일 삭제는 CEO 승인 필수 (100MB 초과)
- 원본 미디어 파일 수정 금지 — 사본에서만 작업

### NAS-D-002 WORKDIR 준수
- 모든 작업은 /root 내부에서만 (단, NAS 마운트 경로 제외)
- NAS 마운트 경로: CEO가 별도 지정한 경우만 사용
- /tmp 임시 파일 작업 후 반드시 정리

### NAS-D-003 유지보수 원칙
- NAS는 유지보수 단계 — 신규 기능보다 안정성 우선
- 서비스 중단 없는 패치만 허용 (무중단 배포)
- 변경 사항은 반드시 changelog 작성

### NAS-D-004 데이터 보호
- 개인정보 포함 파일 처리 시 로그 기록 의무
- 외부 API 전송 데이터 최소화 원칙
- 백업 주기: 작업 전 스냅샷 필수

---

## 공통 규칙 참조 (CEO-DIRECTIVES.md v3.1)

### 필수 준수 항목
- **D-016**: FLOW 프레임워크 (Find→Layout→Operate→Wrap up) — 모든 작업 의무
- **D-017**: 소스코드 모듈화 원칙 (agents/graphs/models/services 4개 디렉토리)
- **D-018**: 4계층 자기치유 원칙 — L1 하트비트 기반 세션 관리
- **D-019**: 서버 상호 감시 의무화 — 3서버 2분 주기 크로스 모니터링
- **D-020**: 복구 이력 DB 의무화 — recovery_logs 테이블 기록
- **D-021**: 하트비트 기반 세션 관리 — claude_exec 하트비트, 10초 감시
- **R-001**: 작업 완료 후 HANDOVER.md 업데이트 의무
- **R-014**: WRAP 게이트 — auto_trigger.sh WRAP 보고서 확인 후 다음 태스크 투입
- **R-016**: 서킷브레이커 준수 — 3회 연속 실패 시 5분 쿨다운
- **R-008**: GitHub 브라우저 경로로 결과 보고

### 공통 파이프라인
- 8단계 파이프라인: CEO지시→Bridge감지→사전검증→우선순위전송→Claude실행→결과보고→DB기록→교차검증
- 하드 타임아웃: 7200초 (2시간)
- git-push 감시: commit SHA RESULT_FILE 기록 + auto_trigger HTTP 200 확인

전체 공통 규칙: /root/aads/aads-docs/CEO-DIRECTIVES.md

---

## HANDOVER 참조
- NAS HANDOVER: /root/aads/aads-docs/NAS-HANDOVER.md
- 워크플로우: /root/aads/shared/rules/WORKFLOW-PIPELINE.md
- 규칙 매트릭스: /root/aads/shared/rules/RULE-MATRIX.md
