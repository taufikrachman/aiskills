# App Store Deployment

Submit to iOS App Store & Google Play Store successfully.

## Rules

### 1. iOS App Store Checklist
- [ ] App ID registered in Apple Developer (explicit bundle ID)
- [ ] Distribution certificate (not development)
- [ ] Provisioning profile (App Store distribution)
- [ ] App icon: 1024×1024px, no alpha channel
- [ ] Screenshots: 6.7" (1290×2796) and 5.5" (1242×2208) minimum
- [ ] Privacy policy URL (required)
- [ ] App description, keywords, category
- [ ] Version number: semantic (1.0.0)
- [ ] Build uploaded via Xcode Archive or Transporter

### 2. Common Rejection Reasons
- App crashes on launch (test on release build, not debug)
- Missing privacy manifest (`PrivacyInfo.xcprivacy`)
- Using private APIs
- Incomplete functionality (placeholder/coming soon screens)
- Requesting unnecessary permissions
- Poor UI (web view wrapper, low resolution)

### 3. Google Play Checklist
- [ ] Signed APK/AAB (release keystore)
- [ ] App icon: 512×512px, adaptive icon (foreground + background)
- [ ] Screenshots: phone + tablet (if supported)
- [ ] Feature graphic: 1024×500px
- [ ] Privacy policy URL
- [ ] Content rating questionnaire
- [ ] Target API level: latest - 1

### 4. Version Management
- Version code (Android): increment every release. Version: 1, 2, 3...
- Build number (iOS): increment every upload. CFBundleVersion: 1, 2, 3...
- Semantic version: MAJOR.MINOR.PATCH → 1.2.3
- Staged rollout: 10% → 50% → 100% for risky updates

### 5. CI/CD for Mobile
```yaml
# GitHub Actions for Flutter
- uses: subosito/flutter-action@v2
  with: { flutter-version: '3.24', channel: 'stable' }
- run: flutter build ipa --release --export-method app-store
- run: flutter build apk --release
```
Automate build on tag push. Upload to App Store Connect via `fastlane`.

## Anti-Patterns
- ❌ Debug build submitted to store (always release)
- ❌ Forgetting to increment version numbers
- ❌ No privacy policy link
- ❌ Testing only on simulator (test on real device)
