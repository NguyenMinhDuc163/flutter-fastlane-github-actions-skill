# Flutter Fastlane GitHub Actions Skill

Dự án này chứa bộ **agent skill** và cấu hình mẫu để hỗ trợ thiết lập CI/CD cho ứng dụng **Flutter** bằng **Fastlane** và **GitHub Actions**.

Mục tiêu chính là giúp bạn build và phát hành app Flutter mà không cần phụ thuộc vào máy cá nhân:

- Build Android trên GitHub Actions runner Linux.
- Build iOS trên GitHub Actions runner macOS.
- Dùng Fastlane để tự động hóa build, ký app và upload.
- Upload Android lên Google Play Console.
- Upload iOS lên TestFlight / App Store Connect.
- Cung cấp skill để agent có thể hiểu, tạo, sửa và debug pipeline Flutter release.

## Dự án này dùng để làm gì?

Khi bạn có một app Flutter và muốn triển khai CI/CD, agent có thể dùng skill trong repo này để:

1. Tạo hoặc cập nhật cấu hình Fastlane cho Android.
2. Tạo hoặc cập nhật cấu hình Fastlane cho iOS.
3. Hướng dẫn cấu hình GitHub Actions workflows.
4. Hướng dẫn tạo GitHub Secrets cần thiết.
5. Kiểm tra các file cấu hình release.
6. Debug lỗi upload Google Play hoặc TestFlight.
7. Tách riêng build Android và iOS để tận dụng đúng runner:
   - Android: `ubuntu-latest`
   - iOS: `macos-latest`

## Cấu trúc thư mục

```text
.
├── README.md
├── android/
│   ├── Gemfile
│   ├── key.properties
│   ├── app/
│   │   └── your_files_key.jks
│   └── fastlane/
│       ├── Appfile
│       ├── Fastfile
│       └── README.md
├── ios/
│   ├── Gemfile
│   └── fastlane/
│       ├── Appfile
│       ├── Fastfile
│       └── README.md
└── .codex/
    └── skills/
        ├── flutter-android-fastlane-google-play/
        │   └── SKILL.md
        ├── flutter-ios-fastlane-testflight/
        │   └── SKILL.md
        └── flutter-mobile-store-github-actions/
            └── SKILL.md
```

## Các skill có trong dự án

### 1. `flutter-mobile-store-github-actions`

Skill tổng quát cho Flutter CI/CD bằng GitHub Actions.

Dùng khi bạn muốn agent hỗ trợ:

- Thiết lập workflow build Android và iOS.
- Tạo release theo tag hoặc chạy thủ công bằng `workflow_dispatch`.
- Dùng Linux runner cho Android.
- Dùng macOS runner cho iOS.
- Kết hợp GitHub Actions với Fastlane.

### 2. `flutter-android-fastlane-google-play`

Skill chuyên cho Android release lên Google Play.

Dùng khi bạn muốn agent hỗ trợ:

- Cấu hình `android/Gemfile`.
- Cấu hình `android/fastlane/Appfile`.
- Cấu hình `android/fastlane/Fastfile`.
- Build Android App Bundle `.aab`.
- Upload lên Google Play track như `internal`, `alpha`, `beta`, hoặc `production`.
- Debug lỗi service account, signing key, versionCode, Play Console API.

### 3. `flutter-ios-fastlane-testflight`

Skill chuyên cho iOS release lên TestFlight / App Store Connect.

Dùng khi bạn muốn agent hỗ trợ:

- Cấu hình `ios/Gemfile`.
- Cấu hình `ios/fastlane/Appfile`.
- Cấu hình `ios/fastlane/Fastfile`.
- Build iOS app bằng macOS runner.
- Upload IPA lên TestFlight.
- Cấu hình App Store Connect API Key.
- Làm việc với signing certificate, provisioning profile hoặc Fastlane Match.

## Yêu cầu trước khi sử dụng

Bạn cần có:

- Một project Flutter hợp lệ.
- Repository GitHub.
- Quyền cấu hình GitHub Actions Secrets.
- Tài khoản Google Play Console nếu phát hành Android.
- Tài khoản Apple Developer nếu phát hành iOS.
- App đã được tạo trên Google Play Console / App Store Connect.

Trên máy local hoặc trong CI cần có:

- Flutter SDK.
- Ruby.
- Bundler.
- Fastlane.
- CocoaPods cho iOS.

## Cách sử dụng skill trong agent

Sao chép thư mục skill vào nơi agent có thể đọc skill, ví dụ:

```text
.codex/skills/
```

Sau đó yêu cầu agent bằng ngôn ngữ tự nhiên, ví dụ:

```text
Hãy dùng skill flutter-mobile-store-github-actions để setup CI/CD cho app Flutter của tôi.
```

Hoặc:

```text
Hãy cấu hình Android Fastlane để upload app Flutter lên Google Play internal testing.
```

Hoặc:

```text
Hãy cấu hình iOS Fastlane để upload app Flutter lên TestFlight bằng GitHub Actions macOS runner.
```

Agent sẽ đọc file `SKILL.md` tương ứng và thực hiện theo checklist trong skill.

## Cấu hình Android Fastlane

File Android Fastlane hiện nằm tại:

```text
android/fastlane/Fastfile
```

Lane mẫu:

```ruby
default_platform(:android)

platform :android do
  desc "Deploy a new beta build to Google Play"
  lane :beta do
    gradle(task: "clean assembleRelease")
    upload_to_play_store(track: 'beta')
  end
end
```

Bạn có thể chạy từ thư mục `android/`:

```bash
bundle install
bundle exec fastlane android beta
```

> Lưu ý: chỉ chạy lane upload khi bạn đã cấu hình đầy đủ signing key và Google Play service account.

## Cấu hình iOS Fastlane

File iOS Fastlane hiện nằm tại:

```text
ios/fastlane/Fastfile
```

Lane mẫu:

```ruby
default_platform(:ios)

platform :ios do
  desc "Push a new beta build to TestFlight"
  lane :beta do
    increment_build_number(xcodeproj: "Runner.xcodeproj")
    build_app(workspace: "Runner.xcworkspace", scheme: "Runner")
    upload_to_testflight
  end
end
```

Bạn có thể chạy từ thư mục `ios/`:

```bash
bundle install
bundle exec fastlane ios beta
```

> Lưu ý: iOS cần chạy trên macOS và phải có cấu hình signing hợp lệ.

## GitHub Secrets cần cấu hình

### Android

Các secret thường cần cho Android release:

```text
ANDROID_KEYSTORE_BASE64
ANDROID_KEYSTORE_PASSWORD
ANDROID_KEY_ALIAS
ANDROID_KEY_PASSWORD
GOOGLE_PLAY_SERVICE_ACCOUNT_JSON
```

Ý nghĩa:

- `ANDROID_KEYSTORE_BASE64`: file keystore được encode base64.
- `ANDROID_KEYSTORE_PASSWORD`: mật khẩu keystore.
- `ANDROID_KEY_ALIAS`: alias của key dùng để ký app.
- `ANDROID_KEY_PASSWORD`: mật khẩu của key alias.
- `GOOGLE_PLAY_SERVICE_ACCOUNT_JSON`: nội dung JSON của Google Play service account.

### iOS

Các secret thường cần cho iOS release:

```text
APP_STORE_CONNECT_API_KEY_KEY_ID
APP_STORE_CONNECT_API_KEY_ISSUER_ID
APP_STORE_CONNECT_API_KEY_KEY
APPLE_TEAM_ID
MATCH_PASSWORD
MATCH_GIT_URL
```

Ý nghĩa:

- `APP_STORE_CONNECT_API_KEY_KEY_ID`: Key ID trong App Store Connect.
- `APP_STORE_CONNECT_API_KEY_ISSUER_ID`: Issuer ID trong App Store Connect.
- `APP_STORE_CONNECT_API_KEY_KEY`: nội dung private key `.p8`.
- `APPLE_TEAM_ID`: Apple Developer Team ID.
- `MATCH_PASSWORD`: mật khẩu dùng cho Fastlane Match nếu có dùng Match.
- `MATCH_GIT_URL`: repository lưu certificate/profile nếu có dùng Match.

## Gợi ý GitHub Actions workflow

Android nên chạy trên Linux:

```yaml
name: Android Release

on:
  workflow_dispatch:

jobs:
  android:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: subosito/flutter-action@v2
        with:
          channel: stable

      - uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.3'
          bundler-cache: true
          working-directory: android

      - name: Install Flutter dependencies
        run: flutter pub get

      - name: Deploy Android
        working-directory: android
        env:
          GOOGLE_PLAY_SERVICE_ACCOUNT_JSON: ${{ secrets.GOOGLE_PLAY_SERVICE_ACCOUNT_JSON }}
          ANDROID_KEYSTORE_BASE64: ${{ secrets.ANDROID_KEYSTORE_BASE64 }}
          ANDROID_KEYSTORE_PASSWORD: ${{ secrets.ANDROID_KEYSTORE_PASSWORD }}
          ANDROID_KEY_ALIAS: ${{ secrets.ANDROID_KEY_ALIAS }}
          ANDROID_KEY_PASSWORD: ${{ secrets.ANDROID_KEY_PASSWORD }}
        run: bundle exec fastlane android beta
```

iOS nên chạy trên macOS:

```yaml
name: iOS Release

on:
  workflow_dispatch:

jobs:
  ios:
    runs-on: macos-latest

    steps:
      - uses: actions/checkout@v4

      - uses: subosito/flutter-action@v2
        with:
          channel: stable

      - uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.3'
          bundler-cache: true
          working-directory: ios

      - name: Install Flutter dependencies
        run: flutter pub get

      - name: Install CocoaPods
        working-directory: ios
        run: pod install

      - name: Deploy iOS
        working-directory: ios
        env:
          APP_STORE_CONNECT_API_KEY_KEY_ID: ${{ secrets.APP_STORE_CONNECT_API_KEY_KEY_ID }}
          APP_STORE_CONNECT_API_KEY_ISSUER_ID: ${{ secrets.APP_STORE_CONNECT_API_KEY_ISSUER_ID }}
          APP_STORE_CONNECT_API_KEY_KEY: ${{ secrets.APP_STORE_CONNECT_API_KEY_KEY }}
          APPLE_TEAM_ID: ${{ secrets.APPLE_TEAM_ID }}
          MATCH_PASSWORD: ${{ secrets.MATCH_PASSWORD }}
          MATCH_GIT_URL: ${{ secrets.MATCH_GIT_URL }}
        run: bundle exec fastlane ios beta
```

## Quy trình sử dụng đề xuất

### Bước 1: Chuẩn bị Flutter app

Đảm bảo app Flutter build được ở local:

```bash
flutter pub get
flutter test
flutter build apk --debug
```

### Bước 2: Cấu hình Android signing

Tạo keystore release và cấu hình `android/key.properties`.

Không commit các file sau nếu chúng chứa secret thật:

```text
android/key.properties
*.jks
*.keystore
android/fastlane/play-store-credentials.json
```

### Bước 3: Cấu hình Google Play service account

Trong Google Play Console:

1. Tạo hoặc liên kết Google Cloud project.
2. Bật Google Play Android Developer API.
3. Tạo service account.
4. Tạo JSON key.
5. Mời service account vào Google Play Console.
6. Cấp quyền upload release cho app.

Sau đó lưu nội dung JSON vào GitHub Secret:

```text
GOOGLE_PLAY_SERVICE_ACCOUNT_JSON
```

### Bước 4: Cấu hình iOS signing

Bạn có thể dùng một trong hai hướng:

1. Fastlane Match để quản lý certificate và provisioning profile.
2. Cấu hình certificate/profile trực tiếp trong CI.

Với CI/CD lâu dài, nên dùng Fastlane Match.

### Bước 5: Tạo GitHub Actions workflows

Tạo các file:

```text
.github/workflows/android-release.yml
.github/workflows/ios-release.yml
```

Android dùng `ubuntu-latest`, iOS dùng `macos-latest`.

### Bước 6: Chạy thử bằng workflow thủ công

Vào GitHub repository:

```text
Actions → Android Release → Run workflow
Actions → iOS Release → Run workflow
```

Nên upload lên track test trước:

- Android: `internal`
- iOS: TestFlight

Không nên deploy production ngay từ lần đầu.

## Lưu ý bảo mật

Không commit các file chứa secret:

```text
*.jks
*.keystore
android/key.properties
android/fastlane/play-store-credentials.json
*.p8
*.mobileprovision
*.cer
*.p12
```

Nên lưu secret trong:

```text
GitHub Repository Settings → Secrets and variables → Actions
```

Nếu lỡ commit secret, bạn cần:

1. Revoke hoặc rotate secret đó.
2. Xóa secret khỏi git history.
3. Không chỉ xóa bằng commit mới, vì secret vẫn còn trong lịch sử git.

## Những lệnh kiểm tra an toàn

Kiểm tra Ruby syntax của Fastlane file:

```bash
ruby -c android/fastlane/Fastfile
ruby -c ios/fastlane/Fastfile
```

Liệt kê lanes Android:

```bash
cd android
bundle exec fastlane lanes
```

Liệt kê lanes iOS:

```bash
cd ios
bundle exec fastlane lanes
```

Kiểm tra Flutter:

```bash
flutter doctor
flutter pub get
flutter test
```

## Ghi chú quan trọng

- Không chạy upload production nếu chưa có xác nhận rõ ràng.
- Nên dùng `internal` hoặc `beta` track cho Android khi thử nghiệm.
- Nên dùng TestFlight cho iOS trước khi submit review.
- iOS bắt buộc cần macOS runner để build.
- Android có thể build tốt trên Linux runner nên tiết kiệm chi phí hơn.
- Fastlane giúp thống nhất quy trình build/upload giữa local và CI.

## Trạng thái hiện tại

Repo hiện đã có cấu hình Fastlane cơ bản cho:

- Android beta upload lên Google Play.
- iOS beta upload lên TestFlight.
- Các skill agent trong `.codex/skills/` để hướng dẫn setup và bảo trì pipeline.

Bạn có thể tiếp tục mở rộng dự án bằng cách thêm:

- Workflow GitHub Actions thật trong `.github/workflows/`.
- Template cấu hình CI/CD.
- Script tự động tạo secret file trong CI.
- Tài liệu debug lỗi Google Play và App Store Connect.
