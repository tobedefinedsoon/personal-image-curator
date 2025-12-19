# OneDrive Photo Cleaner - Implementation Plan

## Overview

Building a native iOS app (SwiftUI) that helps users clean their OneDrive photos with a Tinder-style swipe interface. The app features iOS 26's Liquid Glass design with graceful fallback to iOS 18+, Microsoft OneDrive branding, freemium monetization (25 swipes/day, 1 CHF/month unlimited), and Firebase analytics.

---

## Technical Stack

### Platform & Language
- **Platform**: iOS 26+ (with fallback to iOS 18+)
- **Language**: Swift 5.10+
- **UI Framework**: SwiftUI
- **Minimum Deployment**: iOS 18.0
- **Architecture**: MVVM (Model-View-ViewModel)
- **Dependency Manager**: Swift Package Manager

### Key Dependencies
```swift
// Swift Package Manager dependencies
dependencies: [
    .package(url: "https://github.com/AzureAD/microsoft-authentication-library-for-objc", from: "1.3.0"),
    .package(url: "https://github.com/firebase/firebase-ios-sdk", from: "10.0.0"),
    .package(url: "https://github.com/onevcat/Kingfisher", from: "7.0.0")
]

// Native frameworks (no package needed)
- StoreKit 2 (subscriptions)
- CoreData (local storage)
- CoreMotion (device tilt for Liquid Glass)
```

---

## Project Structure

```
OneDrivePhotoCleaner/
├── App/
│   ├── OneDrivePhotoCleanerApp.swift       # App entry point, Firebase init
│   └── AppDelegate.swift                    # App lifecycle hooks
│
├── Models/
│   ├── Photo.swift                          # Photo metadata from OneDrive
│   ├── SwipeAction.swift                    # Keep/delete decision enum
│   ├── UserStats.swift                      # Metrics & progress model
│   ├── SubscriptionStatus.swift             # Subscription state model
│   └── CoreData/
│       └── SwipedPhoto.xcdatamodeld        # CoreData schema
│
├── ViewModels/
│   ├── OnboardingViewModel.swift            # Onboarding flow logic
│   ├── AuthViewModel.swift                  # Microsoft authentication
│   ├── PhotoStackViewModel.swift            # Photo loading & swipe logic
│   ├── MetricsViewModel.swift               # Stats calculation
│   └── SubscriptionViewModel.swift          # Purchase & entitlement logic
│
├── Views/
│   ├── Onboarding/
│   │   ├── OnboardingView.swift            # Carousel container
│   │   └── OnboardingPageView.swift        # Individual page view
│   │
│   ├── Auth/
│   │   └── AuthenticationView.swift        # Microsoft sign-in screen
│   │
│   ├── Main/
│   │   ├── PhotoCardView.swift             # Single card with Liquid Glass
│   │   ├── PhotoStackView.swift            # Card stack container
│   │   ├── SwipeActionOverlayView.swift    # Keep/delete indicators
│   │   ├── UndoToastView.swift             # 5s undo notification
│   │   └── MainTabView.swift               # Tab bar navigation
│   │
│   ├── Metrics/
│   │   ├── StatsView.swift                 # Metrics dashboard
│   │   └── StatsCardView.swift             # Individual stat card
│   │
│   ├── Paywall/
│   │   └── SubscriptionView.swift          # Paywall & pricing
│   │
│   └── Components/
│       ├── LiquidGlassCard.swift           # Reusable glass card
│       ├── PrimaryButton.swift             # Branded button
│       └── LoadingView.swift               # Loading spinner
│
├── Services/
│   ├── OneDriveService.swift               # Microsoft Graph API client
│   ├── LocalStorageService.swift           # UserDefaults + CoreData
│   ├── AnalyticsService.swift              # Firebase Analytics wrapper
│   ├── SubscriptionService.swift           # StoreKit 2 wrapper
│   └── UndoService.swift                   # Delete queue management
│
├── Utilities/
│   ├── Constants.swift                      # App-wide constants
│   ├── Extensions/
│   │   ├── Color+Brand.swift               # Brand color extensions
│   │   ├── View+LiquidGlass.swift          # Liquid Glass modifiers
│   │   └── String+Formatting.swift         # String helpers
│   └── LiquidGlassModifier.swift           # Custom ViewModifier
│
├── Resources/
│   ├── Assets.xcassets                      # Images, colors, icons
│   ├── GoogleService-Info.plist            # Firebase configuration
│   └── Info.plist                          # App permissions & URL schemes
│
└── Configuration/
    ├── Debug.xcconfig                       # Debug build settings
    ├── Release.xcconfig                     # Release build settings
    └── Secrets.xcconfig                     # Environment variables (git-ignored)
```

---

## Implementation Phases

### Phase 1: Foundation (Week 1)

**Goal**: Basic app structure, authentication, and onboarding

#### Tasks
1. **Project Setup**
   - Create Xcode project with SwiftUI template
   - Bundle ID: `com.glaglasven.onedrivecleaner`
   - Add Swift Package Manager dependencies
   - Set minimum deployment target to iOS 18.0

2. **Firebase Setup**
   - Create Firebase project: "OneDrive Photo Cleaner"
   - Add iOS app with bundle ID
   - Download and add `GoogleService-Info.plist`
   - Enable Analytics and Crashlytics
   - Initialize Firebase in `OneDrivePhotoCleanerApp.swift`

3. **Azure AD Setup**
   - Register app in Azure AD Portal
   - Configure redirect URI: `msauth.com.glaglasven.onedrivecleaner://auth`
   - Request API permissions: `Files.Read.All`, `Files.ReadWrite.All`, `User.Read`
   - Save client ID to `Secrets.xcconfig`

4. **Branding Implementation**
   - Add brand colors to `Assets.xcassets`
   - Create `Color+Brand.swift` extensions
   - Implement `LiquidGlassModifier.swift` with iOS 26 detection
   - Create reusable components (`PrimaryButton`, `LiquidGlassCard`)

5. **Onboarding Flow**
   - Build 5-page carousel with `TabView`
   - Implement page indicator with OneDrive blue
   - Add skip and continue buttons
   - Store completion flag in `UserDefaults`

6. **Microsoft Authentication**
   - Integrate MSAL library
   - Implement OAuth flow in `AuthViewModel`
   - Create `AuthenticationView` UI
   - Handle token storage in Keychain
   - Set up Firebase anonymous auth

7. **Local Storage Setup**
   - Create CoreData schema for `SwipedPhoto` entity
   - Implement `LocalStorageService` with CRUD operations
   - Set up `UserDefaults` for app state
   - Create `UserStats` model

#### Deliverables
- ✅ User can launch app and see onboarding
- ✅ User can sign in with Microsoft account
- ✅ Firebase Analytics tracks onboarding completion
- ✅ Liquid Glass visual style works on iOS 26 (degrades gracefully on iOS 18)
- ✅ App navigates from onboarding → auth → main view

---

### Phase 2: Core Swipe Feature (Week 2)

**Goal**: Photo loading, swipe interface, and delete with undo

#### Tasks
1. **OneDrive API Integration**
   - Implement `OneDriveService` class
   - Build photo fetching: `GET /me/drive/root/children`
   - Add pagination support (50 photos per batch)
   - Filter to show only image files
   - Handle API errors and token refresh

2. **Image Caching**
   - Integrate Kingfisher library
   - Configure memory and disk cache
   - Implement thumbnail loading from Microsoft Graph
   - Prefetch next 5 photos in background

3. **Photo Card UI**
   - Create `PhotoCardView` with Liquid Glass effect
   - Display photo with metadata (date, size)
   - Implement responsive layout for different screen sizes
   - Add loading state for images

4. **Card Stack Interface**
   - Build `PhotoStackView` with 3-card stack
   - Implement z-index layering and scaling
   - Add offset and opacity for depth effect
   - Connect to `PhotoStackViewModel`

5. **Swipe Gesture**
   - Implement `DragGesture` with threshold detection (100pt)
   - Add card rotation during swipe
   - Show keep/delete overlay based on direction
   - Animate card dismissal with spring physics
   - Add haptic feedback on threshold cross

6. **Delete Queue & Undo**
   - Create `UndoService` with timer management
   - Build `UndoToastView` at bottom of screen
   - Implement 5-second countdown with cancellation
   - Handle app backgrounding (persist queue to UserDefaults)
   - Execute actual deletion via OneDrive API after timer

7. **Swipe Tracking**
   - Save swiped photos to CoreData
   - Filter already-swiped photos from API results
   - Update `UserStats` after each swipe
   - Track analytics events (swipe_right, swipe_left, undo_delete)

#### Deliverables
- ✅ User can swipe through OneDrive photos
- ✅ Left swipe triggers delete with 5-second undo
- ✅ Right swipe logs keep action
- ✅ Photos load smoothly with caching
- ✅ Swipe history persists across app restarts
- ✅ Delete queue survives app backgrounding

---

### Phase 3: Monetization & Metrics (Week 3)

**Goal**: Swipe limits, paywall, subscriptions, and stats dashboard

#### Tasks
1. **Swipe Limit System**
   - Implement daily limit logic (25 free swipes)
   - Add reset at midnight logic
   - Store swipe count and reset date in `UserDefaults`
   - Block swipes after limit reached

2. **Subscription Products**
   - Set up App Store Connect account
   - Create subscription group
   - Add products:
     - Monthly: `com.glaglasven.onedrivecleaner.monthly` (1 CHF)
     - Yearly: `com.glaglasven.onedrivecleaner.yearly` (10 CHF)
   - Configure 3-day free trial for monthly subscription

3. **StoreKit 2 Integration**
   - Create `SubscriptionService` wrapper
   - Implement product fetching
   - Build purchase flow with error handling
   - Add entitlement checking on app launch
   - Implement restore purchases functionality

4. **Paywall UI**
   - Design `SubscriptionView` with benefits list
   - Show pricing with local currency formatting
   - Add subscription buttons (monthly/yearly)
   - Display restore purchases link
   - Include terms of service and privacy policy links

5. **Metrics Dashboard**
   - Create `StatsView` with Liquid Glass cards
   - Calculate total space freed
   - Count photos kept vs deleted
   - Track daily streak (consecutive days of usage)
   - Compute efficiency score (% deleted)

6. **Data Visualization**
   - Integrate Swift Charts for activity graph
   - Show 7-day swipe history
   - Add share functionality for stats
   - Create shareable image with metrics

7. **Tab Navigation**
   - Build `MainTabView` with 3 tabs:
     - Photos (main swipe interface)
     - Stats (metrics dashboard)
     - Settings (subscription, legal links, sign out)

#### Deliverables
- ✅ Free users limited to 25 swipes/day
- ✅ Paywall appears after reaching limit
- ✅ Users can purchase subscriptions (1 CHF/month or 10 CHF/year)
- ✅ Subscribers have unlimited swipes
- ✅ Stats page shows space freed, photos deleted, streak
- ✅ Users can share their progress on social media

---

### Phase 4: Polish & Launch Prep (Week 4)

**Goal**: Testing, error handling, and App Store submission

#### Tasks
1. **Error Handling**
   - Handle network errors gracefully
   - Show retry UI for failed API calls
   - Manage token expiration and refresh
   - Add empty states (no photos, no internet)
   - Display user-friendly error messages

2. **Loading States**
   - Add skeletons for loading photos
   - Show progress indicator during authentication
   - Display spinner while fetching subscription status
   - Implement pull-to-refresh on photo stack

3. **Dark Mode Support**
   - Test all screens in dark mode
   - Adjust Liquid Glass opacity for dark backgrounds
   - Verify color contrast in both modes
   - Update brand colors for dark mode variants

4. **Accessibility Audit**
   - Add VoiceOver labels to all interactive elements
   - Test with Dynamic Type (up to 200%)
   - Verify minimum 44pt tap targets
   - Implement high contrast mode support
   - Test with reduce motion enabled

5. **Device Testing**
   - Test on iOS 18, 19, 25, and 26
   - Verify on iPhone SE, iPhone 15, iPhone 15 Pro Max
   - Test iPad support (if applicable)
   - Check performance on older devices

6. **App Store Assets**
   - Design app icon (1024×1024) with Liquid Glass effect
   - Create screenshots for all required sizes:
     - 6.7" (iPhone 15 Pro Max)
     - 6.5" (iPhone 14 Pro Max)
     - 5.5" (iPhone 8 Plus)
   - Write app description following brand voice
   - Select keywords for App Store optimization
   - Record app preview video (optional)

7. **Legal & Privacy**
   - Create privacy policy (see Privacy Policy Website section)
   - Write terms of service with subscription terms
   - Add support page with FAQ
   - Deploy Next.js site to Vercel
   - Update `Constants.swift` with legal URLs

8. **TestFlight Beta**
   - Archive and upload build to App Store Connect
   - Set up internal testing group
   - Invite 10-20 beta testers
   - Collect feedback and fix critical bugs
   - Prepare for public beta (optional)

9. **App Store Submission**
   - Complete App Store Connect listing
   - Upload final build
   - Submit for review
   - Monitor review status
   - Respond to any rejection feedback

#### Deliverables
- ✅ App handles all error cases gracefully
- ✅ Dark mode fully supported
- ✅ Accessibility requirements met (VoiceOver, Dynamic Type, high contrast)
- ✅ Tested on iOS 18-26 and multiple device sizes
- ✅ App Store listing complete with screenshots and description
- ✅ Privacy policy and terms of service live on Vercel
- ✅ TestFlight beta deployed with feedback collected
- ✅ App submitted to Apple for review

---

## Core Feature Specifications

### 1. Onboarding Carousel

**Implementation**:
```swift
TabView {
    OnboardingPageView(
        icon: "cloud.fill",
        title: "Welcome to Swipe & Clean",
        description: "Declutter your OneDrive in minutes"
    )
    OnboardingPageView(
        icon: "hand.point.right.fill",
        title: "Swipe right to keep",
        description: "Photos you love stay safe"
    )
    // ... more pages
}
.tabViewStyle(.page)
.indexViewStyle(.page(backgroundDisplayMode: .always))
```

**Pages**:
1. Welcome + app logo
2. Swipe right demo
3. Swipe left demo
4. Metrics preview
5. Get started (auth button)

---

### 2. Microsoft Authentication

**MSAL Configuration**:
```swift
let config = MSALPublicClientApplicationConfig(
    clientId: AppConstants.msalClientId,
    redirectUri: AppConstants.msalRedirectUri,
    authority: try MSALAuthority(url: URL(string: "https://login.microsoftonline.com/common")!)
)

let scopes = ["Files.Read.All", "Files.ReadWrite.All", "User.Read"]
```

**Auth Flow**:
1. User taps "Sign in with Microsoft"
2. Present MSAL web view
3. User authenticates with Microsoft credentials
4. Receive access token and store in Keychain
5. Fetch user profile from `/me` endpoint
6. Navigate to main photo stack

**Token Refresh**:
- MSAL handles automatic refresh
- Catch 401 errors and trigger silent refresh
- Fall back to interactive auth if silent fails

---

### 3. Photo Loading Strategy

**API Endpoint**:
```
GET https://graph.microsoft.com/v1.0/me/drive/root/children
Query parameters:
  - $filter=file ne null
  - $top=50
  - $select=id,name,size,createdDateTime,@microsoft.graph.downloadUrl
  - $orderby=createdDateTime desc
```

**Caching Strategy**:
1. Request large thumbnail: `/me/drive/items/{id}/thumbnails/0/large` (800×800)
2. Use Kingfisher for automatic memory + disk caching
3. Prefetch next 5 photos in background using `ImagePrefetcher`
4. Clear cache weekly or when over 500MB

**Filtering**:
- Query CoreData for swiped photo IDs
- Filter API results to exclude already-swiped photos
- Show "All done!" empty state when no photos remain

---

### 4. Swipe Gesture Implementation

**Gesture Code**:
```swift
@State private var offset = CGSize.zero
@State private var rotation: Double = 0

var body: some View {
    PhotoCardView(photo: currentPhoto)
        .offset(offset)
        .rotationEffect(.degrees(rotation))
        .gesture(
            DragGesture()
                .onChanged { gesture in
                    offset = gesture.translation
                    rotation = Double(gesture.translation.width / 20)
                }
                .onEnded { gesture in
                    if abs(gesture.translation.width) > 100 {
                        // Swipe threshold reached
                        if gesture.translation.width > 0 {
                            handleKeep()
                        } else {
                            handleDelete()
                        }
                    } else {
                        // Spring back to center
                        withAnimation(.spring()) {
                            offset = .zero
                            rotation = 0
                        }
                    }
                }
        )
}
```

**Threshold**: 100pt horizontal drag
**Haptics**: `.impact(.medium)` on threshold cross
**Animation**: Spring (response: 0.3, damping: 0.8)

---

### 5. Delete Queue & Undo System

**Data Structure**:
```swift
struct DeleteQueueItem: Identifiable {
    let id = UUID()
    let photo: Photo
    let timestamp: Date
    var timer: Timer?
}

class UndoService: ObservableObject {
    @Published var deleteQueue: [DeleteQueueItem] = []

    func scheduleDelete(_ photo: Photo) {
        let item = DeleteQueueItem(photo: photo, timestamp: Date())
        deleteQueue.append(item)

        Timer.publish(every: 1, on: .main, in: .common)
            .autoconnect()
            .sink { _ in
                if Date().timeIntervalSince(item.timestamp) >= 5 {
                    self.executeDelete(item)
                }
            }
    }

    func undo(_ item: DeleteQueueItem) {
        deleteQueue.removeAll { $0.id == item.id }
        item.timer?.invalidate()
    }

    func executeDelete(_ item: DeleteQueueItem) {
        // Call OneDrive API: DELETE /me/drive/items/{id}
        oneDriveService.deletePhoto(item.photo.id)
        deleteQueue.removeAll { $0.id == item.id }
    }
}
```

**Edge Cases**:
- App backgrounded: Persist queue to `UserDefaults`, resume timers on foreground
- Multiple pending deletes: Show "X photos will be deleted. UNDO ALL"
- API failure: Retry 3 times with exponential backoff, then show error

---

### 6. Swipe Limit & Paywall

**Limit Logic**:
```swift
struct SwipeLimitManager {
    @AppStorage("swipesUsedToday") private var swipesUsedToday = 0
    @AppStorage("lastResetDate") private var lastResetDate = Date()

    let dailyFreeSwipes = 25

    mutating func canSwipe(isSubscribed: Bool) -> Bool {
        if isSubscribed { return true }

        resetIfNewDay()
        return swipesUsedToday < dailyFreeSwipes
    }

    mutating func incrementSwipe() {
        resetIfNewDay()
        swipesUsedToday += 1
    }

    private mutating func resetIfNewDay() {
        if !Calendar.current.isDateInToday(lastResetDate) {
            swipesUsedToday = 0
            lastResetDate = Date()
        }
    }
}
```

**Paywall Trigger**:
- Check before each swipe: `if !swipeLimitManager.canSwipe(isSubscribed:)`
- Present `.sheet` with `SubscriptionView`
- Block interaction with photo stack using overlay

---

### 7. StoreKit 2 Subscription

**Product Fetching**:
```swift
let products = try await Product.products(for: [
    "com.glaglasven.onedrivecleaner.monthly",
    "com.glaglasven.onedrivecleaner.yearly"
])
```

**Purchase Flow**:
```swift
func purchase(_ product: Product) async throws {
    let result = try await product.purchase()

    switch result {
    case .success(let verification):
        switch verification {
        case .verified(let transaction):
            // Unlock features
            await transaction.finish()
        case .unverified:
            // Handle verification failure
        }
    case .userCancelled:
        // User tapped cancel
    case .pending:
        // Waiting for approval (Ask to Buy)
    @unknown default:
        break
    }
}
```

**Entitlement Check**:
```swift
for await result in Transaction.currentEntitlements {
    if case .verified(let transaction) = result {
        if transaction.productID == "com.glaglasven.onedrivecleaner.monthly" ||
           transaction.productID == "com.glaglasven.onedrivecleaner.yearly" {
            isSubscribed = true
        }
    }
}
```

---

### 8. Firebase Analytics Events

**Key Events**:
```swift
// User journey
Analytics.logEvent("onboarding_started", parameters: nil)
Analytics.logEvent("onboarding_completed", parameters: nil)
Analytics.logEvent("auth_success", parameters: ["method": "microsoft"])

// Engagement
Analytics.logEvent("swipe_right", parameters: ["photo_id": photoId])
Analytics.logEvent("swipe_left", parameters: ["photo_id": photoId])
Analytics.logEvent("undo_delete", parameters: nil)
Analytics.logEvent("daily_limit_reached", parameters: ["swipes_used": 25])

// Monetization
Analytics.logEvent(AnalyticsEventPurchase, parameters: [
    AnalyticsParameterValue: 1.0,
    AnalyticsParameterCurrency: "CHF"
])

// Errors
Analytics.logEvent("api_error", parameters: [
    "error_code": errorCode,
    "endpoint": endpoint
])
```

**User Properties**:
```swift
Analytics.setUserProperty("free", forName: "subscription_status")
Analytics.setUserProperty("26", forName: "ios_version")
Analytics.setUserProperty("42", forName: "photos_reviewed_total")
```

---

## API Integration

### Microsoft Graph API

**Base URL**: `https://graph.microsoft.com/v1.0`

**Headers**:
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

**Endpoints**:

1. **List Photos**
   ```
   GET /me/drive/root/children
   Query: $filter=file ne null&$top=50&$select=id,name,size,createdDateTime
   ```

2. **Get Thumbnail**
   ```
   GET /me/drive/items/{id}/thumbnails/0/large
   Response: 800×800 JPEG
   ```

3. **Delete Photo**
   ```
   DELETE /me/drive/items/{id}
   Success: 204 No Content
   ```

4. **Get User Info**
   ```
   GET /me
   Response: { displayName, mail, userPrincipalName }
   ```

**Error Handling**:
- `401`: Token expired → trigger refresh
- `404`: Photo not found → remove from UI
- `429`: Rate limited → exponential backoff (2s, 4s, 8s)
- `500`: Server error → retry 3 times

---

## Privacy Policy Website (Next.js 16 + Vercel)

### Project Setup
```bash
npx create-next-app@latest privacy-policy-site --typescript --app --tailwind
cd privacy-policy-site
```

### Required Pages

1. **`/privacy-policy`**
   - Data collection disclosure
   - GDPR compliance
   - User rights (access, delete, export)

2. **`/terms-of-service`**
   - Subscription terms (1 CHF/month, 10 CHF/year)
   - Auto-renewal disclosure
   - Cancellation and refund policy
   - Governing law (Swiss)

3. **`/support`**
   - Contact email
   - FAQ (cancel subscription, recover photos, etc.)

### Deployment
```bash
vercel --prod
```

**URLs to add to `Constants.swift`**:
```swift
static let privacyPolicyURL = "https://your-site.vercel.app/privacy-policy"
static let termsOfServiceURL = "https://your-site.vercel.app/terms-of-service"
static let supportURL = "https://your-site.vercel.app/support"
```

---

## Testing Strategy

### Unit Tests
- `OneDriveService`: Mock API responses, test pagination
- `UndoService`: Test timer management, backgrounding
- `SwipeLimitManager`: Test daily reset logic
- `SubscriptionService`: Mock StoreKit transactions

### UI Tests
- Onboarding flow (complete all 5 pages)
- Sign-in flow (mock MSAL web view)
- Swipe gestures (right, left, threshold detection)
- Paywall appearance after 25 swipes
- Undo functionality (tap undo within 5s)

### Manual Testing Checklist
- [ ] Onboarding on first launch
- [ ] Microsoft authentication (success and failure)
- [ ] Photo loading with slow network
- [ ] Swipe gestures in all directions
- [ ] Delete queue persists when app backgrounds
- [ ] Daily limit resets at midnight
- [ ] Paywall shows correctly after limit
- [ ] Subscription purchase flow (use sandbox)
- [ ] Restore purchases functionality
- [ ] Stats dashboard updates correctly
- [ ] Dark mode on all screens
- [ ] VoiceOver navigation
- [ ] Dynamic Type at 200%
- [ ] High contrast mode
- [ ] Reduce motion enabled

---

## Configuration Files

### `Info.plist`
```xml
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>msauth.$(PRODUCT_BUNDLE_IDENTIFIER)</string>
        </array>
    </dict>
</array>

<key>LSApplicationQueriesSchemes</key>
<array>
    <string>msauthv2</string>
    <string>msauthv3</string>
</array>

<key>NSPhotoLibraryUsageDescription</key>
<string>We don't access your local photos, only OneDrive.</string>
```

### `Constants.swift`
```swift
enum AppConstants {
    // Azure AD
    static let msalClientId = "YOUR_CLIENT_ID" // From Azure Portal
    static let msalRedirectUri = "msauth.com.glaglasven.onedrivecleaner://auth"

    // Subscription
    static let monthlyProductId = "com.glaglasven.onedrivecleaner.monthly"
    static let yearlyProductId = "com.glaglasven.onedrivecleaner.yearly"

    // Limits
    static let dailyFreeSwipes = 25
    static let undoWindowSeconds = 5
    static let prefetchCount = 5

    // Legal
    static let privacyPolicyURL = "https://your-site.vercel.app/privacy-policy"
    static let termsOfServiceURL = "https://your-site.vercel.app/terms-of-service"
    static let supportURL = "https://your-site.vercel.app/support"
}
```

### `Secrets.xcconfig` (git-ignored)
```
MSAL_CLIENT_ID = your-azure-ad-client-id
FIREBASE_API_KEY = your-firebase-api-key
```

---

## Success Metrics

### Week 1 Goals
- 100 downloads
- 60% onboarding completion
- 40% Microsoft auth success rate
- Average 15 swipes per user

### Month 1 Goals
- 1,000 total downloads
- 5% paid conversion rate
- 50 GB total space freed
- 4.0+ App Store rating

### Month 3 Goals
- 5,000 total downloads
- 10% paid conversion rate
- Featured in App Store "Productivity" category
- 1,000 monthly active users

---

## Risk Management

### Technical Risks
- **MSAL integration complexity**: Mitigate with thorough testing, fallback to web auth
- **Liquid Glass performance**: Test on iPhone SE, provide iOS 18 fallback
- **OneDrive API rate limits**: Implement exponential backoff, cache aggressively

### Business Risks
- **Low conversion rate**: A/B test paywall copy, offer discounts
- **High churn**: Add value (metrics, insights, batch operations)
- **App Store rejection**: Follow guidelines strictly, prepare appeal

### Legal Risks
- **GDPR compliance**: Clear privacy policy, honor deletion requests
- **Subscription regulations**: Auto-renewal disclosure, easy cancellation
- **OneDrive ToS**: Review Microsoft's developer terms

---

## Open Questions

1. **Company Name**: For App Store listing (default: "glaglasven")
2. **Support Email**: Required for submission (suggest: support@glaglasven.com)
3. **Custom Domain**: For privacy policy site (optional, Vercel subdomain works)
4. **Beta Testers**: Initial list for TestFlight
5. **Marketing Plan**: App Store Optimization, social media, press outreach

---

**Last Updated**: December 19, 2025
**Version**: 1.0
