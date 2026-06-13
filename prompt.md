# Configure Existing Flutter Mobile Store CI/CD

Bạn là coding agent cấu hình CI/CD store release cho một dự án Flutter mobile mới.

Dự án này đã copy sẵn Fastlane, GitHub Actions workflow và `.codex/skills` từ một dự án Flutter cũ đã chạy production ổn định.

Nhiệm vụ: migrate cấu hình sang project hiện tại. Không thiết kế lại pipeline từ đầu.

## Bắt Buộc Đọc Trước

Trước khi sửa file, đọc đủ 3 skill này và làm theo:

```text
.codex/skills/flutter-mobile-store-github-actions/SKILL.md
.codex/skills/flutter-android-fastlane-google-play/SKILL.md
.codex/skills/flutter-ios-fastlane-testflight/SKILL.md
```

Các skill trong repo là source of truth.

## Mục Tiêu

Cấu hình lại CI/CD để:

- Android build AAB và upload Google Play.
- iOS build IPA và upload TestFlight/App Store.
- Dùng FVM nếu project có `.fvm/` hoặc `.fvmrc`.
- Không hardcode Flutter path cá nhân.
- Tận dụng tối đa template cũ.

Giữ workflow shape nếu hợp lý:

```text
mobile-store-release.yml
  ├── release_config
  ├── bump_version
  ├── reusable-android-google-play.yml
  └── reusable-ios-testflight.yml
```

## Phân Tích Project

Tự động đọc và suy ra, không hỏi nếu source code đã có:

```text
pubspec.yaml
.fvm/
.fvmrc
android/app/build.gradle
android/app/build.gradle.kts
android/app/src/main/AndroidManifest.xml
ios/Runner.xcodeproj/project.pbxproj
ios/Runner/Info.plist
ios/Podfile
.github/workflows/
android/fastlane/
ios/fastlane/
.gitignore
android/.gitignore
```

Phát hiện:

- Android `applicationId` / package name.
- iOS bundle identifier.
- App name.
- Flutter/FVM version.
- Release branch.
- Flavor nếu có.
- Existing signing config.
- Existing Fastlane lanes.

## Cập Nhật Project-Specific Config

Thay toàn bộ giá trị của project cũ bằng project hiện tại:

- Android package/applicationId/Play package.
- iOS bundle id/team id/scheme/artifact name.
- App name/artifact naming.
- Release track/status defaults.
- Workflow trigger branch.
- Workflow inputs/outputs.
- Secret mapping.
- Fastlane Appfile/Fastfile/README.
- Signing paths.

Không đổi logic lớn nếu không cần.

## GitHub Actions Rules

Đảm bảo:

- Chỉ có một job bump version.
- Bump đúng `pubspec.yaml` theo format `version: x.y.z+n`.
- Chỉ tăng build number sau `+`; không đổi marketing version nếu user không yêu cầu.
- Commit bump có `[skip ci]`.
- Store jobs checkout đúng commit SHA đã bump.
- Android và iOS chỉ depend `release_config` + `bump_version`, không depend lẫn nhau.
- Platform toggle dùng input/env, không comment workflow để skip.
- Nếu một workflow cũ đã comment hết nhưng còn nằm trong `.github/workflows/*.yml`, move ra ngoài, ví dụ `.github/workflows-disabled/`, để GitHub không scan/chạy nữa.
- Reusable workflow path chính xác.

Root workflow nên có toggle:

```yaml
env:
  RUN_IOS: "false"
  RUN_ANDROID: "true"
  ANDROID_TRACK: "internal"
  ANDROID_RELEASE_STATUS: "completed"
```

## Android Fastlane Rules

Kiểm tra và sửa:

```text
android/Gemfile
android/Gemfile.lock
android/fastlane/Appfile
android/fastlane/Fastfile
android/fastlane/README.md
```

Đảm bảo:

- `package_name(...)` khớp Android `applicationId`.
- Appfile dùng `json_key_file("fastlane/play-store-credentials.json")`.
- Build AAB, không APK.
- Fastlane lane có `doctor`, `build`, `validate`, `deploy`.
- `upload_to_play_store` có `version_name` dạng `"buildNumber (buildName)"` nếu template cũ dùng vậy.
- Không yêu cầu secret nằm trong git.

### Android Bundler Check

Nếu dùng `ruby/setup-ruby@v1` + `bundler-cache: true`, CI chạy Bundler deployment/frozen mode.

Phải kiểm tra `android/Gemfile.lock`. Nếu CI lỗi:

```text
Your lockfile has an empty CHECKSUMS entry for "rake", but can't be updated because frozen mode is set
```

Fix:

```bash
cd android
bundle lock --add-checksums
bundle config set --local path vendor/bundle
bundle config set --local deployment true
bundle install --jobs 4
```

Commit `android/Gemfile.lock`. Không commit `android/.bundle/` hoặc `android/vendor/bundle/`.

`android/.gitignore` phải ignore:

```gitignore
key.properties
**/*.keystore
**/*.jks
.bundle/
vendor/bundle/
```

## iOS Fastlane Rules

Kiểm tra và sửa:

```text
ios/Gemfile
ios/Gemfile.lock
ios/fastlane/Appfile
ios/fastlane/Fastfile
ios/Runner.xcodeproj/project.pbxproj
ios/Runner/Info.plist
```

Đảm bảo:

- `app_identifier(...)` khớp iOS bundle id.
- Team ID đúng.
- Không truyền `PRODUCT_BUNDLE_IDENTIFIER` bằng global `xcargs`.
- `MARKETING_VERSION = "$(FLUTTER_BUILD_NAME)"`.
- `CURRENT_PROJECT_VERSION = "$(FLUTTER_BUILD_NUMBER)"`.
- App Store Connect dùng `.p8` qua:

```text
APP_STORE_CONNECT_KEY_ID
APP_STORE_CONNECT_ISSUER_ID
APP_STORE_CONNECT_API_KEY_KEY_FILEPATH
```

Không dùng `APP_STORE_CONNECT_API_KEY_PATH` cho raw `.p8`.

### TestFlight Duplicate Build

Nếu upload lỗi:

```text
The bundle version must be higher than the previously uploaded version: 'N'
```

Tăng baseline trong `pubspec.yaml` lên ít nhất `N`, để workflow bump lần kế tiếp upload `N+1`.

Ví dụ App Store đã có build `21`:

```yaml
version: 1.0.0+21
```

Workflow sẽ bump lên `1.0.0+22`.

## Secrets Và Ignore

Không in secret ra log. Không commit:

```text
android/key.properties
android/fastlane/play-store-credentials*.json
*.jks
*.keystore
*.p8
*.p12
*.mobileprovision
ios/fastlane/.env
```

GitHub secrets expected:

```text
GOOGLE_PLAY_SERVICE_ACCOUNT_JSON
ANDROID_KEYSTORE_BASE64
ANDROID_KEY_PROPERTIES_BASE64
APP_STORE_CONNECT_KEY_ID
APP_STORE_CONNECT_ISSUER_ID
APP_STORE_CONNECT_API_KEY_P8
IOS_DISTRIBUTION_CERTIFICATE_P12_BASE64
IOS_DISTRIBUTION_CERTIFICATE_PASSWORD
IOS_APPSTORE_PROVISIONING_PROFILE_BASE64
```

Có thể thêm debug an toàn trong workflow, chỉ in metadata không nhạy cảm:

- Secret presence.
- File exists/byte size.
- JSON parse ok.
- Android package id match.
- iOS provisioning profile bundle id/team id match.
- Không in private key, password, `key.properties`, JSON credentials, `.p12`, `.mobileprovision`.

## Không Được Chạy Nếu Chưa Được Phép

Không chạy:

```bash
fastlane ios beta
fastlane android deploy*
flutter build ipa
flutter build appbundle
```

trừ khi user cho phép rõ ràng.

## Validation Được Phép

Sau khi sửa, chạy non-upload checks:

```bash
ruby -e 'require "yaml"; ARGV.each { |f| YAML.load_file(f); puts "#{f}: ok" }' \
  .github/workflows/mobile-store-release.yml \
  .github/workflows/reusable-ios-testflight.yml \
  .github/workflows/reusable-android-google-play.yml

ruby -c android/fastlane/Fastfile
ruby -c android/fastlane/Appfile
ruby -c ios/fastlane/Fastfile
ruby -c ios/fastlane/Appfile
plutil -lint ios/Runner/Info.plist

cd android && bundle exec fastlane lanes
cd ios && bundle exec fastlane lanes
```

Nếu `actionlint` có sẵn:

```bash
actionlint .github/workflows/*.yml
```

## Kết Quả Cần Báo Lại

Báo ngắn gọn:

- Project identifiers đã detect.
- File đã sửa.
- Secrets cần set.
- Validation đã chạy và kết quả.
- Những gì chưa chạy vì là build/upload production.
