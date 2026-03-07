# NTV2 (NewTalk V2) CEO DIRECTIVES
> 최종 업데이트: 2026-03-07 (v1.0) — AADS-143 신규 배포
> 관리자: CEO (moongoby)
> 용도: NTV2 AI 세션 필수 읽기. 공통 규칙(CEO-DIRECTIVES.md D-016~D-021, R-001~R-016) 참조.

---

## NTV2 전용 규칙

### NTV2-D-001 Phase 1 우선 완료
- NTV2는 현재 **Phase 1 환경 구축** 단계
- Phase 1 완료 전 Phase 2(기능 구현) 태스크 착수 금지
- Phase 1 체크리스트: 서버 환경 설정 + 기본 파이프라인 연동 + HANDOVER 구축

### NTV2-D-002 서버 라우팅 확인
- 기본 서버: server-114 (116.120.58.155, 포트 7916)
- rfree-009 (114.207.244.86) 사용 여부 CEO 확인 후 반영
- 라우팅 변경 시 auto_trigger.sh REMOTE_HOST_MAP["NTV2"] 업데이트 필수
- 변경 전 CEO 승인 필요

### NTV2-D-003 콘텐츠 안전 기준
- 생성 콘텐츠는 뉴스 기반 사실 정보만
- 허위 정보 생성 절대 금지
- 저작권 있는 원본 그대로 복사 금지 (요약·분석만 허용)

### NTV2-D-004 WORKDIR 준수
- 모든 작업은 /srv/newtalk-v2 내부에서만
- /tmp, /home, ~/ 등 다른 경로 파일 생성 절대 금지
- 영상 출력 파일: /srv/newtalk-v2/output/ 폴더만 사용

---

## 공통 규칙 참조 (CEO-DIRECTIVES.md v3.1)

### 필수 준수 항목
- **D-016**: FLOW 프레임워크 (Find→Layout→Operate→Wrap up) — 모든 작업 의무
- **D-017**: 소스코드 모듈화 원칙 (agents/graphs/models/services 4개 디렉토리)
- **D-018**: 4계층 자기치유 원칙 — L1 하트비트 기반 세션 관리
- **D-019**: 서버 상호 감시 의무화 — 3서버 2분 주기 크로스 모니터링
- **D-020**: 복구 이력 DB 의무화 — recovery_logs 테이블 기록
- **D-021**: 하트비트 기반 세션 관리 — claude_exec 하트비트, 10초 감시
- **R-001**: 작업 완료 후 HANDOVER.md 업데이트 의무 (R-001)
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
- NTV2 HANDOVER: /root/aads/aads-docs/NTV2-HANDOVER.md
- 워크플로우: /root/aads/shared/rules/WORKFLOW-PIPELINE.md
- 규칙 매트릭스: /root/aads/shared/rules/RULE-MATRIX.md
