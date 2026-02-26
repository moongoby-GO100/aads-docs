# Task 4.4: 프로젝트 템플릿 시스템

- **일시**: 2026-02-26
- **변경**: core/templates.py (6개 템플릿), API 3개 엔드포인트, /templates 대시보드 페이지
- **템플릿**: todo-app, weather-dashboard, blog-platform, chat-app, ecommerce, api-gateway

## 변경 사항

| 구분 | 내용 |
|------|------|
| core/templates.py | TEMPLATES 딕셔너리, get_templates(), get_template() |
| api/main.py | GET /templates, GET /templates/{id}, POST /projects/from-template/{id} |
| dashboard | app/templates/page.tsx, api.getTemplates / api.createFromTemplate, 사이드바 링크 |

## 결과

- 적용: ✅ aads-core, dashboard 소스 반영
- 배포: 로컬 실행 후 `/templates`에서 템플릿 목록 확인 및 "Use Template"으로 프로젝트 생성 가능
