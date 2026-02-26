# GitHub PAT + CI/CD Push 점검

**일시**: 2026-02-26 20:20 KST

## 점검 결과

| 항목 | 상태 |
|------|------|
| .github/workflows 디렉터리 | ✅ 존재 |
| test.yml | ✅ 존재 |
| deploy.yml | ✅ 존재 |
| dashboard.yml | ✅ 존재 |
| git push (aads-core) | ❌ 실패 — workflow 스코프 필요 |

## push 실패 사유

```
! [remote rejected] main -> main (refusing to allow a Personal Access Token to create or update workflow `.github/workflows/dashboard.yml` without `workflow` scope)
```

현재 PAT(Classic)에 **workflow** 스코프가 없어, `.github/workflows/*.yml` 변경 시 push가 거부됨.

## PAT workflow 스코프 조치 방법

1. **GitHub** → **Settings** → **Developer settings** → **Personal access tokens** (Classic)
2. 사용 중인 토큰 선택 → **Edit**
3. **workflow** 체크박스 활성화 → **Update token**
4. 서버에서 credential 갱신 후 재 push:
   ```bash
   git config --global credential.helper store
   # 다음 push 시 새 토큰 입력 또는 URL에 반영
   ```

**Fine-grained token 사용 시:** Repository permissions → **Workflows** → **Read and write**

## 후속

- [ ] PAT에 workflow 스코프 추가 후 `git push origin main` 재시도
- [ ] push 성공 시 GitHub Actions에서 워크플로우 실행 여부 확인
