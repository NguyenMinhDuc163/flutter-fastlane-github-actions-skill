# CI/CD Secrets

## Android

### `GOOGLE_PLAY_SERVICE_ACCOUNT_JSON`

**Value**

```text
{
  
}

```

**Cach lay**

```bash
pbcopy < android/fastlane/play-store-credentials.json
```

### `ANDROID_KEYSTORE_BASE64`

**Value**

```text

```

**Cach lay**

```bash
store_file=$(awk -F= '/^storeFile=/{gsub(/^[ \t]+|[ \t]+$/, "", $2); print $2}' android/key.properties)
case "$store_file" in
  /*) keystore_path="$store_file" ;;
  *) keystore_path="android/app/$store_file" ;;
esac
base64 < "$keystore_path" | tr -d '\n' | pbcopy
```

### `ANDROID_KEY_PROPERTIES_BASE64`

**Value**

```text

```

**Cach lay**

```bash
base64 < android/key.properties | tr -d '\n' | pbcopy
```

## iOS

### `APP_STORE_CONNECT_KEY_ID`

**Value**

```text
```

**Cach lay**

```bash
App Store Connect
→ Users and Access
→ Integrations
→ Generate API Key

```

### `APP_STORE_CONNECT_ISSUER_ID`

**Value**

```text
```

**Cach lay**

```bash
App Store Connect
→ Users and Access
→ Integrations
→ Generate API Key
```

### `APP_STORE_CONNECT_API_KEY_P8`

**Value**

```text

```

**Cach lay**
App Store Connect
→ Users and Access
→ Integrations
→ Generate API Key
```bash
pbcopy < /path/to/AuthKey_XXXXXXXXXX.p8
```

### `IOS_DISTRIBUTION_CERTIFICATE_PASSWORD`

**Value**

```text
```

**Cach lay**

```bash
tren macos vao keychain access => export certificate ra file .p12 => khi export se co password
```

### `IOS_DISTRIBUTION_CERTIFICATE_P12_BASE64`

**Value**

```text
```

**Cach lay**

```bash
base64 -i path/to/certificate.p12 | tr -d '\n' | pbcopy
```

### `IOS_APPSTORE_PROVISIONING_PROFILE_BASE64`

**Value**

```text
vào https://developer.apple.com/account/resources/profiles/list tạo 1 provisioning profile mới với bundle id là com..., sau đó tải về và chạy lệnh dưới để lấy base64
```

**Cach lay**

```bash
base64 -i path/to/profile.mobileprovision | tr -d '\n' | pbcopy
```
