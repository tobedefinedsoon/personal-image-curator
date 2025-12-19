# Swipe & Clean for OneDrive

A native iOS app that helps you declutter your OneDrive photos with a simple, Tinder-style swipe interface. Built with SwiftUI and featuring iOS 26's stunning Liquid Glass design.

![iOS 26](https://img.shields.io/badge/iOS-26%2B-blue)
![SwiftUI](https://img.shields.io/badge/SwiftUI-5.10-orange)
![License](https://img.shields.io/badge/license-MIT-green)

---

## Features

- 🎴 **Tinder-Style Swipe Interface** - Swipe right to keep, left to delete
- 🔵 **OneDrive Native** - Seamless Microsoft OneDrive integration
- ✨ **iOS 26 Liquid Glass UI** - Stunning glass effects with fallback to iOS 18+
- ↩️ **5-Second Undo** - Changed your mind? Tap undo before it's gone
- 📊 **Progress Tracking** - See how much space you've freed
- 💎 **Freemium Model** - 25 free swipes daily, unlimited for 1 CHF/month
- 🔒 **Privacy First** - All decisions stored locally, no cloud sync

---

## Screenshots

*Coming soon: Hero shot, swipe interface, stats dashboard, paywall*

---

## Requirements

### Development
- macOS Ventura (13.0) or later
- Xcode 16+ (for iOS 26 SDK)
- Swift 5.10+
- CocoaPods or Swift Package Manager
- Apple Developer account (for TestFlight & App Store)

### External Services
- Microsoft Azure AD account (free tier)
- Firebase account (free Spark plan)
- Vercel account (for privacy policy website, free tier)

### Minimum Deployment Target
- iOS 18.0+ (Liquid Glass effects require iOS 26)

---

## Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/yourusername/onedrive-photo-cleaner.git
cd onedrive-photo-cleaner
```

### 2. Install Dependencies

Dependencies are managed via Swift Package Manager and will auto-resolve on first build. No additional installation needed.

**Dependencies:**
- MSAL (Microsoft Authentication Library)
- Firebase iOS SDK (Analytics, Crashlytics)
- Kingfisher (image loading and caching)
- StoreKit 2 (native, no package)

### 3. Configure Azure AD (Microsoft Authentication)

1. Go to [Azure Portal](https://portal.azure.com)
2. Navigate to **App registrations** → **New registration**
3. **Name**: "OneDrive Photo Cleaner"
4. **Supported account types**: "Personal Microsoft accounts only"
5. **Redirect URI**: `msauth.com.glaglasven.onedrivecleaner://auth`
6. Click **Register** and copy the **Application (client) ID**
7. In **API permissions**, add:
   - Microsoft Graph: `Files.Read.All`
   - Microsoft Graph: `Files.ReadWrite.All`
   - Microsoft Graph: `User.Read`
8. Click **Grant admin consent**

9. Create `Configuration/Secrets.xcconfig` (this file is git-ignored):
   ```
   MSAL_CLIENT_ID = your-azure-ad-client-id-here
   ```

### 4. Configure Firebase

1. Go to [Firebase Console](https://console.firebase.google.com)
2. Click **Add project** → Name: "OneDrive Photo Cleaner"
3. Enable **Google Analytics** (recommended)
4. Click **Add app** → **iOS**
5. **Bundle ID**: `com.glaglasven.onedrivecleaner`
6. Download `GoogleService-Info.plist`
7. Drag the file into Xcode project root (next to `Info.plist`)
8. In Firebase Console, navigate to **Build**:
   - Enable **Analytics**
   - Enable **Crashlytics** (optional but recommended)

### 5. Configure App Store Connect (For Subscriptions)

1. Go to [App Store Connect](https://appstoreconnect.apple.com)
2. Click **My Apps** → **+** → **New App**
3. **Bundle ID**: `com.glaglasven.onedrivecleaner`
4. Navigate to **Features** → **In-App Purchases** → **Subscriptions**
5. Click **Create Subscription Group** → Name: "Unlimited Swipes"
6. Add two subscription products:

   **Monthly Subscription:**
   - Product ID: `com.glaglasven.onedrivecleaner.monthly`
   - Pricing: 1 CHF/month (select Switzerland as base territory)
   - Free trial: 3 days

   **Yearly Subscription:**
   - Product ID: `com.glaglasven.onedrivecleaner.yearly`
   - Pricing: 10 CHF/year (16% savings)
   - No free trial

7. Wait for products to show **Ready to Submit** status

### 6. Build and Run

```bash
# Open project in Xcode
open OneDrivePhotoCleaner.xcodeproj

# Or if using workspace
open OneDrivePhotoCleaner.xcworkspace

# Select target device (iOS 26 Simulator or physical iPhone)
# Press Cmd+R to build and run
```

---

## Project Structure

```
OneDrivePhotoCleaner/
├── App/                         # App entry point
│   ├── OneDrivePhotoCleanerApp.swift
│   └── AppDelegate.swift
├── Models/                      # Data models
│   ├── Photo.swift
│   ├── SwipeAction.swift
│   ├── UserStats.swift
│   └── SubscriptionStatus.swift
├── ViewModels/                  # MVVM view models
│   ├── OnboardingViewModel.swift
│   ├── AuthViewModel.swift
│   ├── PhotoStackViewModel.swift
│   ├── MetricsViewModel.swift
│   └── SubscriptionViewModel.swift
├── Views/                       # SwiftUI views
│   ├── Onboarding/             # Welcome carousel
│   ├── Auth/                   # Microsoft sign-in
│   ├── Main/                   # Photo swipe interface
│   ├── Metrics/                # Stats dashboard
│   ├── Paywall/                # Subscription screen
│   └── Components/             # Reusable UI components
├── Services/                    # Business logic
│   ├── OneDriveService.swift   # Microsoft Graph API client
│   ├── LocalStorageService.swift
│   ├── AnalyticsService.swift  # Firebase wrapper
│   ├── SubscriptionService.swift
│   └── UndoService.swift       # Delete queue manager
├── Utilities/                   # Helpers & extensions
│   ├── Constants.swift
│   ├── Extensions/
│   └── LiquidGlassModifier.swift
├── Resources/
│   ├── Assets.xcassets
│   ├── GoogleService-Info.plist
│   └── Info.plist
└── Configuration/
    ├── Debug.xcconfig
    ├── Release.xcconfig
    └── Secrets.xcconfig         # Git-ignored
```

---

## Architecture

### Design Pattern: MVVM (Model-View-ViewModel)

- **Models**: Pure data structures (Photo, UserStats, etc.)
- **Views**: SwiftUI views that observe ViewModels
- **ViewModels**: Business logic, API calls, state management

### Key Services

#### OneDriveService
- Microsoft Graph API integration
- Token management (via MSAL)
- Photo fetching, thumbnail loading, deletion

#### LocalStorageService
- CoreData for swiped photo history
- UserDefaults for app state
- Swipe count and subscription status

#### SubscriptionService
- StoreKit 2 wrapper
- Product fetching
- Purchase flow
- Entitlement checking

#### AnalyticsService
- Firebase Analytics wrapper
- Event tracking (swipes, purchases, errors)
- User properties

#### UndoService
- Delete queue management
- 5-second timer with cancellation
- Handles app backgrounding

---

## Development Workflow

### Branch Strategy

- `main` - Production-ready code
- `develop` - Integration branch for features
- `feature/*` - Feature branches (e.g., `feature/paywall`)
- `bugfix/*` - Bug fix branches
- `release/*` - Release preparation branches

### Commit Convention

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: add undo toast notification
fix: resolve token refresh issue
docs: update README with Firebase setup
style: apply Liquid Glass to stats cards
refactor: extract swipe gesture logic
test: add unit tests for UndoService
```

### Testing

```bash
# Run unit tests
Cmd + U in Xcode

# Run UI tests
Select "OneDrivePhotoCleanerUITests" scheme → Cmd + U

# Run specific test
Cmd + Click on test method → "Run {test name}"
```

### Linting

```bash
# Install SwiftLint
brew install swiftlint

# Run linter
swiftlint

# Auto-fix (where possible)
swiftlint --fix
```

---

## Deployment

### TestFlight Beta

1. **Archive the build**:
   - Product → Archive in Xcode
   - Ensure "Any iOS Device" is selected as target

2. **Upload to App Store Connect**:
   - Click "Distribute App"
   - Select "App Store Connect"
   - Wait for processing (10-30 minutes)

3. **Add testers**:
   - In App Store Connect → TestFlight
   - Add internal testers (up to 100)
   - Create external test group (up to 10,000)
   - Share public link

4. **Collect feedback**:
   - Monitor crash reports in Xcode Organizer
   - Review feedback in App Store Connect
   - Fix critical bugs before public release

### App Store Release

1. **Prepare assets**:
   - App icon: 1024×1024 PNG
   - Screenshots: 6.7", 6.5", 5.5" (required sizes)
   - App preview video (optional, recommended)
   - Privacy policy URL (from Vercel deployment)
   - Support URL (from Vercel deployment)

2. **Complete App Store listing**:
   - App name: "Swipe & Clean for OneDrive"
   - Subtitle: "Declutter Your Cloud Photos"
   - Keywords: "onedrive, photo cleaner, storage, cloud, declutter"
   - Description: See `BRANDING.md` for template
   - Age rating: 4+
   - Category: Productivity

3. **Submit for review**:
   - Increment version number (e.g., 1.0 → 1.1)
   - Increment build number
   - Add build to App Store submission
   - Submit for review
   - Respond to review feedback (typically 24-48 hours)

---

## Troubleshooting

### MSAL Authentication Fails

**Symptoms**: Sign-in button does nothing or shows error

**Solutions**:
- Verify redirect URI in Azure AD matches `msauth.com.glaglasven.onedrivecleaner://auth` exactly
- Check `CFBundleURLSchemes` in `Info.plist` includes `msauth.$(PRODUCT_BUNDLE_IDENTIFIER)`
- Ensure app registration in Azure AD allows personal Microsoft accounts
- Clear app data and try again
- Check console logs in Xcode for detailed error messages

### Firebase Not Tracking Events

**Symptoms**: No events appearing in Firebase Analytics dashboard

**Solutions**:
- Verify `GoogleService-Info.plist` is in project root
- Check bundle ID in Firebase matches Xcode project: `com.glaglasven.onedrivecleaner`
- Enable debug logging:
  ```swift
  FirebaseConfiguration.shared.setLoggerLevel(.debug)
  ```
- Wait 24 hours (Analytics can take time to process)
- Use Firebase DebugView for real-time event testing

### Photos Not Loading

**Symptoms**: Empty state or loading spinner never completes

**Solutions**:
- Verify Microsoft Graph API permissions are granted in Azure AD
- Check network connectivity (test in browser: https://graph.microsoft.com/v1.0/me)
- Inspect Xcode console for API error responses
- Token may be expired → MSAL should auto-refresh, but check manually
- Try signing out and back in
- Check OneDrive has photos in root folder

### Subscription Not Working

**Symptoms**: Purchase button does nothing or shows error

**Solutions**:
- Ensure subscription products in App Store Connect show "Ready to Submit"
- Use sandbox test account (Settings → App Store → Sandbox Account)
- Clear sandbox account and create new one
- Check StoreKit configuration file is added to project (if using local testing)
- View StoreKit logs: Window → Devices and Simulators → Open Console
- Wait 24 hours after creating products (App Store sync)

### Liquid Glass Not Appearing

**Symptoms**: Cards show solid colors instead of translucent glass

**Solutions**:
- Verify running iOS 26+ (check `UIDevice.current.systemVersion`)
- Test on physical device (Simulator may not render correctly)
- Check `LiquidGlassModifier` is applied to views
- Fallback to `.regularMaterial` is intentional for iOS 18-25
- Ensure accessibility "Reduce Transparency" is off

---

## Contributing

We welcome contributions! Please follow these steps:

1. **Fork the repository**
2. **Create a feature branch**:
   ```bash
   git checkout -b feature/amazing-feature
   ```
3. **Make your changes** and commit:
   ```bash
   git commit -m "feat: add amazing feature"
   ```
4. **Push to your fork**:
   ```bash
   git push origin feature/amazing-feature
   ```
5. **Open a Pull Request** with:
   - Clear description of changes
   - Screenshots (for UI changes)
   - Test plan
   - Link to related issue (if applicable)

### Code Style

- Follow Swift API Design Guidelines
- Use SwiftLint for consistency
- Write unit tests for business logic
- Add inline comments for complex logic
- Keep functions under 50 lines where possible

---

## Documentation

- **BRANDING.md** - Brand guidelines, colors, typography, UI principles
- **PLAN.md** - Technical architecture, API specs, implementation phases
- **LICENSE** - MIT License
- **CHANGELOG.md** - Version history and release notes (coming soon)

---

## Privacy & Security

### Data We Collect
- Microsoft account email (for authentication)
- OneDrive file metadata (name, size, date)
- Anonymous usage analytics via Firebase

### Data We Don't Collect
- Photo content (images are loaded from OneDrive, not stored)
- Personal information beyond email
- Location data
- Device identifiers

### Data Storage
- Swiped photo decisions: Stored locally in CoreData
- User stats: Stored locally in UserDefaults
- No cloud sync of user data

See full privacy policy at: [Privacy Policy URL]

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## Support

### Get Help
- **Email**: support@glaglasven.com
- **GitHub Issues**: [Report a bug](https://github.com/yourusername/onedrive-photo-cleaner/issues)
- **FAQ**: [Support Page URL]

### FAQs

**Q: Can I recover deleted photos?**
A: Photos are permanently deleted from OneDrive after the 5-second undo window. Make sure you want to delete before swiping!

**Q: Does this work with OneDrive for Business?**
A: Currently supports personal Microsoft accounts only. Business account support coming soon.

**Q: How do I cancel my subscription?**
A: Open iOS Settings → [Your Name] → Subscriptions → Swipe & Clean → Cancel Subscription.

**Q: Why can't I see all my photos?**
A: The app only shows photos in your OneDrive root folder. Photos in subfolders coming in future updates.

**Q: Is my data safe?**
A: Yes! We use Microsoft's secure OAuth for authentication and never store your photo content. All swipe decisions are stored locally on your device.

---

## Roadmap

### Phase 1 (Week 1) - CURRENT
- [x] Onboarding and authentication
- [x] Basic project structure
- [ ] Microsoft Graph API integration

### Phase 2 (Week 2)
- [ ] Photo swipe interface
- [ ] Delete with undo
- [ ] Image caching

### Phase 3 (Week 3)
- [ ] Swipe limits & paywall
- [ ] Subscription integration
- [ ] Metrics dashboard

### Phase 4 (Week 4)
- [ ] Polish & accessibility
- [ ] TestFlight beta
- [ ] App Store submission

### Future Features
- Subfolder support
- OneDrive for Business accounts
- Batch operations ("delete all screenshots")
- AI-powered suggestions (duplicates, blurry photos)
- iCloud sync for multi-device usage
- iPad and Mac Catalyst versions

---

## Acknowledgments

- **Apple** - For SwiftUI and iOS 26's Liquid Glass design system
- **Microsoft** - For OneDrive and Microsoft Graph API
- **Firebase** - For Analytics and Crashlytics
- **Kingfisher** - For image loading and caching

---

## Links

- **App Store**: [Coming soon]
- **Website**: [Coming soon]
- **Privacy Policy**: [Privacy Policy URL]
- **Terms of Service**: [Terms URL]
- **Support**: [Support URL]

---

**Built with ❤️ using SwiftUI**

*Last updated: December 19, 2025*
