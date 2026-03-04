# AADS iOS QA Setup Guide

## 필수 요구사항

- Mac mini M2+ (macOS 14+, Xcode 16+)
- iOS Simulator 또는 실기기 (USB/Wi-Fi)
- WebDriverAgent 빌드·설치

## 연결 방법

### Option A: Mac mini 전용 서버 ($400~$500 1회)

1. Mac mini에 Xcode, Node.js, Appium, xcuitest 드라이버 설치

   ```bash
   # Mac mini에서 실행
   xcode-select --install
   brew install node
   npm install -g appium
   appium driver install xcuitest
   appium driver install uiautomator2
   ```

2. SSH 터널 설정:
   ```bash
   ssh -L 4724:localhost:4723 user@mac-mini-ip
   ```

3. AADS `.env`에 iOS Appium URL 추가:
   ```env
   IOS_APPIUM_URL=http://localhost:4724
   IOS_PLATFORM_VERSION=17.0
   IOS_DEVICE_NAME=iPhone 15
   ```

4. MobileQAService로 iOS 연결:
   ```python
   from app.services.mobile_qa import MobileQAService
   svc = MobileQAService(appium_url="http://localhost:4724")
   driver = svc.connect_ios("17.0", "iPhone 15", None, 8100)
   ```

### Option B: 클라우드 Mac ($25~$100/월)

1. MacStadium 또는 AWS EC2 Mac 인스턴스 생성
2. Xcode + Appium + WebDriverAgent 설치

   ```bash
   # 클라우드 Mac에서 실행
   xcode-select --install
   brew install node
   npm install -g appium
   appium driver install xcuitest
   ```

3. VPN 또는 SSH 터널로 연결:
   ```bash
   ssh -L 4724:localhost:4723 user@cloud-mac-ip
   ```

4. 동일하게 `IOS_APPIUM_URL` 설정:
   ```env
   IOS_APPIUM_URL=http://localhost:4724
   ```

### Option C: GitHub Actions macOS Runner

CI/CD 파이프라인에서만 iOS 테스트 (무료 분 한도 내):

```yaml
# .github/workflows/ios-qa.yml
name: iOS QA

on:
  push:
    branches: [main]

jobs:
  ios-qa:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install Appium
        run: |
          npm install -g appium
          appium driver install xcuitest

      - name: Start Appium
        run: |
          appium --address 0.0.0.0 --port 4723 --relaxed-security &
          sleep 3

      - name: Run iOS QA
        env:
          IOS_APPIUM_URL: http://localhost:4723
          GOOGLE_API_KEY: ${{ secrets.GOOGLE_API_KEY }}
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          cd aads-server
          pip install -r requirements.txt
          python -c "
          from app.services.mobile_qa import MobileQAService
          svc = MobileQAService(appium_url='http://localhost:4723')
          driver = svc.connect_ios('17.0', 'iPhone 15 Simulator', None, 8100)
          print('iOS connected:', driver.session_id)
          svc.close(driver)
          "
```

## 연결 후 검증

```bash
# Appium 서버 상태 확인
curl http://localhost:4724/status
# → {"value":{"ready":true,"message":"The server is ready to accept new connections",...}}

# Python 연결 테스트
python3 -c "
from app.services.mobile_qa import MobileQAService
svc = MobileQAService(appium_url='http://localhost:4724')
driver = svc.connect_ios('17.0', 'iPhone 15', None, 8100)
print('iOS connected:', driver.session_id)
svc.close(driver)
"
```

## API 사용 예시

### iOS 스크린샷 감리

```bash
# 스크린샷 base64 변환
SCREENSHOT_B64=$(base64 -i /path/to/screenshot.png)

# 감리 API 호출
curl -s -X POST https://aads.newtalk.kr/api/v1/mobile-qa/audit-screen \
  -H "Content-Type: application/json" \
  -d "{\"screenshot_base64\":\"$SCREENSHOT_B64\",\"platform\":\"ios\"}"
```

### Full iOS QA (Mac 연결 시)

```bash
curl -s -X POST https://aads.newtalk.kr/api/v1/mobile-qa/full-qa \
  -H "Content-Type: application/json" \
  -d '{
    "apk_url": "https://example.com/app.ipa",
    "package_name": "com.example.app",
    "activity_name": ".",
    "scenarios": [
      {"action": "tap", "target": "login_button"},
      {"action": "screenshot", "path": "/tmp/ios_login.png"}
    ],
    "platform": "ios"
  }'
```

## 환경변수 참조

| 변수명 | 설명 | 예시 |
|--------|------|------|
| `IOS_APPIUM_URL` | iOS Appium 서버 URL | `http://localhost:4724` |
| `IOS_PLATFORM_VERSION` | iOS 버전 | `17.0` |
| `IOS_DEVICE_NAME` | 기기 이름 | `iPhone 15` |
| `ANDROID_EMULATOR_HOST` | Android 에뮬레이터 호스트 | `localhost` |
| `ANDROID_EMULATOR_PORT` | Android ADB 포트 | `5555` |
| `APPIUM_URL` | Android Appium URL | `http://localhost:4723` |

## 문제 해결

### WebDriverAgent 빌드 실패

```bash
# Xcode 개발자 인증서 설정 필요
open -a Xcode
# Preferences > Accounts > Apple ID 로그인 후 팀 선택
```

### Simulator not found

```bash
# 사용 가능한 시뮬레이터 목록 확인
xcrun simctl list devices available
```

### Port already in use

```bash
lsof -ti:4723 | xargs kill -9
appium --address 0.0.0.0 --port 4723 --relaxed-security
```
