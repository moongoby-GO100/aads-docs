# STATUS-001: 지휘통제소 종합 상황 보고서

## 보고 일시
2026-03-06 KST

## 시스템 현황 요약
| 서버 | 역할 | 상태 | 포트 |
|------|------|------|------|
| Server 68 | AADS 메인 (aads.newtalk.kr) | 운영중 | 8100, 8101, 18085 |
| Server 116 | 원격 에이전트 | 운영중 | - |
| Server 211 | QA 환경 | 대기 | - |

## API 엔드포인트 맵 (현재 운영 중)

### AADS Server (Port 8100)
| 엔드포인트 | 메서드 | 설명 |
|-----------|--------|------|
| /api/v1/health | GET | 헬스체크 |
| /api/v1/context/system | GET/POST | 시스템 메모리 |
| /api/v1/watchdog/errors | GET/POST | 에러 리포트 |
| /api/v1/approval | GET/POST | 승인 큐 |
| /api/v1/memory/log | POST | 메모리 로그 |
| /api/v1/documents | GET/POST | CEO 문서 (T-102 신규) |

### Conversations Server (Port 8101)
| 엔드포인트 | 메서드 | 설명 |
|-----------|--------|------|
| /api/v1/conversations | GET/POST | 대화 목록/저장 |

### Memory Server (Port 18085)
| 엔드포인트 | 메서드 | 설명 |
|-----------|--------|------|
| /api/v1/memory/search | GET | 메모리 검색 |
| /api/v1/memory/ceo-decisions | GET | CEO 결정사항 |

## 진행 중인 태스크
- T-095: 비용 추적 시스템
- T-101: 정적 스냅샷 시스템
- T-102: CEO 문서 자동 저장 (현재 실행 중)

## 다음 액션
- T-103: 대화창 CEO 관리 기능
- HANDOVER.md 갱신
- Docker 재배포 확인

## 생성 일시
2026-03-06 KST (현재 세션)
