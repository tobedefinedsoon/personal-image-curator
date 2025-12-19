# OneDrive Photo Cleaner - Brand Guidelines

## App Name
**Primary**: Swipe & Clean for OneDrive

**Alternatives** (if primary unavailable):
- Photo Curator for OneDrive
- OneDrive Declutter
- Clear Drive
- Swipe Clear

---

## Brand Identity

### Color Palette

#### Primary Colors
- **OneDrive Blue**: `#0078D4`
  - Usage: Primary buttons, navigation, key UI elements
  - Microsoft's official OneDrive brand color

- **OneDrive Blue Light**: `#50A0E0`
  - Usage: Highlights, gradients, hover states

- **OneDrive Blue Dark**: `#005A9E`
  - Usage: Depth, shadows, dark mode accents

#### Liquid Glass Accents (iOS 26)
- **Glass White**: `#FFFFFF` at 15-30% opacity
  - Usage: Translucent card surfaces, overlays

- **Shimmer Highlight**: `#E3F2FD` at 40% opacity
  - Usage: Glass reflections, specular highlights

- **Shadow Tint**: `#000000` at 8% opacity
  - Usage: Depth behind glass elements

#### Supporting Colors
- **Success Green**: `#107C10`
  - Usage: "Keep" action, positive metrics, success states

- **Alert Red**: `#D13438`
  - Usage: "Delete" action, warnings, destructive actions

- **Neutral Gray**: `#605E5C`
  - Usage: Secondary text, disabled states

- **Background (Light Mode)**: `#F3F2F1`
  - Usage: App background, light mode base

- **Background (Dark Mode)**: `#1B1A19`
  - Usage: App background, dark mode base

---

## Typography

### Font Families
- **Display**: SF Pro Display
  - Usage: Large headings, hero text, onboarding
  - Rounded variant for friendly, approachable feel

- **Body**: SF Pro Text
  - Usage: Body copy, descriptions, labels

- **Metrics/Numbers**: SF Pro Rounded
  - Usage: Statistics, counts, data visualization
  - Makes numbers more friendly and readable

### Font Weights
- **Regular (400)**: Body text, descriptions
- **Medium (500)**: Buttons, tabs, secondary headings
- **Semibold (600)**: Section headings, card titles
- **Bold (700)**: Large metrics, emphasis, hero numbers

### Type Scale
```
Hero:       34pt / Bold
Heading 1:  28pt / Semibold
Heading 2:  22pt / Semibold
Heading 3:  17pt / Medium
Body:       17pt / Regular
Subhead:    15pt / Regular
Caption:    13pt / Regular
Footnote:   12pt / Regular
```

---

## Logo & App Icon

### Concept
A circular icon featuring:
1. **Base layer**: OneDrive cloud symbol (simplified)
2. **Middle layer**: Swipe gesture indicator (curved arrow or finger swipe)
3. **Top layer**: Liquid Glass overlay with subtle shimmer

### Design Direction
- **Shape**: Rounded square (iOS standard app icon shape)
- **Gradient**: `#0078D4` → `#50A0E0` (top to bottom)
- **Effect**: Layered translucency (3 distinct layers visible)
- **Motion**: Icon responds to device tilt with parallax effect (iOS 26 feature)

### Icon Sizes Required
- 1024×1024 (App Store)
- 180×180 (iPhone @3x)
- 120×120 (iPhone @2x)
- 167×167 (iPad Pro @2x)
- 152×152 (iPad @2x)
- 76×76 (iPad @1x)

---

## UI Design Principles

### iOS 26 Liquid Glass Implementation

#### Core Technique
```swift
// Primary approach for iOS 26
UIVisualEffectView with .systemUltraThinMaterial

// SwiftUI equivalent
.background(.ultraThinMaterial)
```

#### Glass Card Properties
- **Blur**: 30-40% opacity background blur
- **Translucency**: Allow content behind to show through
- **Specular Highlights**: Subtle white shimmer following device motion
- **Refraction**: Slight content distortion at edges
- **Depth**: Multiple layers with varying translucency levels

#### Implementation Details
- Use `.containerRelativeFrame()` for responsive layouts
- Implement `GeometryReader` for parallax effects
- Apply `MotionManager` for device tilt tracking
- Layer multiple blur effects for depth (back = more blur, front = less blur)

### iOS 18-25 Fallback

When iOS 26 features unavailable:
- Replace `.ultraThinMaterial` with `.regularMaterial`
- Use solid color overlays at 90% opacity
- Remove motion-based shimmer effects
- Maintain color palette but use traditional shadows
- Keep card-based layout with standard depth shadows

### Microsoft Fluent Design Integration

#### Where Fluent Applies
- **Acrylic surfaces**: Similar to Liquid Glass, use for cards and overlays
- **Depth**: Layer elements with subtle shadows
- **Motion**: Physics-based animations (spring curves)
- **Light**: Subtle gradients and highlights

#### Where iOS Guidelines Override
- Use iOS standard navigation patterns (not Fluent's)
- Follow iOS tab bar conventions
- Use iOS system fonts (not Fluent's Segoe UI)
- Adopt iOS alert/modal styles

### Layout Standards

#### Spacing System
```
XXS:  4pt   (icon padding, tight spacing)
XS:   8pt   (element padding)
S:    12pt  (small gaps)
M:    16pt  (default spacing)
L:    24pt  (section spacing)
XL:   32pt  (major sections)
XXL:  48pt  (screen margins)
```

#### Corner Radius
- **Cards**: 16pt
- **Buttons**: 12pt
- **Input fields**: 10pt
- **Small elements** (badges, chips): 8pt

#### Shadows
- **Elevation 1** (subtle): `#000000` at 5% opacity, 2pt offset, 4pt blur
- **Elevation 2** (medium): `#000000` at 8% opacity, 4pt offset, 8pt blur
- **Elevation 3** (high): `#000000` at 12% opacity, 8pt offset, 16pt blur

---

## Animation Guidelines

### Timing Functions
- **Standard easing**: `easeInOut` for most transitions
- **Spring**: Use for interactive elements (swipe cards, buttons)
  - Damping: 0.8
  - Response: 0.3
- **Linear**: Only for continuous animations (loading spinners)

### Duration Standards
- **Micro**: 0.15s (hover, tap feedback)
- **Short**: 0.3s (button press, small transitions)
- **Medium**: 0.5s (card swipe, navigation)
- **Long**: 0.8s (page transitions, major animations)

### Swipe Gesture Animation
```swift
// Card swipe physics
.animation(.spring(response: 0.3, dampingFraction: 0.8), value: offset)

// Rotation during swipe
let rotation = offset.width / 20  // 1° per 20pt

// Scale next card as top card leaves
let nextCardScale = 0.95 + (abs(offset.width) / 1000 * 0.05)
```

### Haptic Feedback
- **Light**: Card reaches swipe threshold
- **Medium**: Card dismissed (keep or delete)
- **Heavy**: Error, limit reached
- **Success**: Subscription completed

---

## Accessibility

### Color Contrast
All text must meet WCAG 2.1 AA standards:
- **Normal text**: 4.5:1 minimum contrast ratio
- **Large text** (18pt+): 3:1 minimum contrast ratio
- **Interactive elements**: 3:1 minimum contrast ratio

### High Contrast Mode
When user enables high contrast:
- Increase Liquid Glass opacity from 30% to 80%
- Add 2pt borders to all interactive elements
- Use solid colors instead of gradients
- Increase shadow darkness

### Dynamic Type Support
- Support all iOS Dynamic Type sizes (XS to XXXL)
- Test layouts at 200% text size
- Allow text to reflow, never truncate critical info
- Maintain minimum 44pt tap target size

### VoiceOver Labels
- **Cards**: "Photo from {date}, swipe right to keep, left to delete"
- **Stats**: "You've freed {size} gigabytes by deleting {count} photos"
- **Buttons**: Clear action labels, avoid icon-only buttons
- **Progress**: Announce remaining free swipes

### Reduce Motion
When user enables reduce motion:
- Disable device tilt parallax effects
- Replace slide/swipe animations with fade transitions
- Remove shimmer/shimmy effects
- Keep all functionality, change only animation style

---

## Component Styling

### Primary Button
```
Background: OneDrive Blue (#0078D4)
Text: White (#FFFFFF)
Height: 50pt
Corner radius: 12pt
Font: SF Pro Text Medium 17pt
Shadow: Elevation 1
Active state: Darken 10%
Disabled: 40% opacity
```

### Secondary Button
```
Background: Clear
Border: 2pt OneDrive Blue (#0078D4)
Text: OneDrive Blue (#0078D4)
Height: 50pt
Corner radius: 12pt
Font: SF Pro Text Medium 17pt
Active state: Fill with OneDrive Blue Light (#50A0E0)
```

### Card (Liquid Glass)
```
Background: .ultraThinMaterial (iOS 26) or .regularMaterial (iOS 18-25)
Corner radius: 16pt
Shadow: Elevation 2
Padding: 16pt
Border: 1pt white at 10% opacity (iOS 26 only)
```

### Toast Notification
```
Background: Liquid Glass with red tint (#D13438 at 20%)
Height: 60pt
Corner radius: 12pt
Position: Bottom, 16pt margin
Text: White (#FFFFFF), SF Pro Text Medium 15pt
Button: "UNDO" in white, bold
```

---

## Voice & Tone

### Brand Voice
- **Friendly**: Conversational, approachable language
- **Helpful**: Clear instructions, no jargon
- **Efficient**: Concise copy, respect user's time
- **Trustworthy**: Transparent about data handling

### Messaging Examples

#### Onboarding
- "Let's clean up your OneDrive" (not "Start cleaning process")
- "Swipe right to keep photos you love" (not "Retain desired items")
- "Free up space without the hassle" (not "Optimize storage utilization")

#### Actions
- "Keep" / "Delete" (not "Retain" / "Remove")
- "You've freed 4.2 GB!" (not "4.2 GB has been deallocated")
- "Oops! Tap undo to bring it back" (not "Deletion can be reversed")

#### Errors
- "Couldn't connect to OneDrive. Check your internet?" (not "Network error: ERR_CONNECTION_FAILED")
- "Let's try signing in again" (not "Authentication required")

---

## App Store Assets

### Screenshots Copy

**Screenshot 1** (Hero):
- Visual: Photo card with swipe gesture
- Text: "Clean up your OneDrive in seconds"

**Screenshot 2** (Feature):
- Visual: Card stack showing multiple photos
- Text: "Swipe right to keep, left to delete"

**Screenshot 3** (Undo):
- Visual: Undo toast at bottom
- Text: "5-second undo. No stress."

**Screenshot 4** (Stats):
- Visual: Metrics dashboard
- Text: "Watch your free space grow"

**Screenshot 5** (Subscription):
- Visual: Clean paywall screen
- Text: "25 free swipes daily, unlimited for 1 CHF/month"

### App Description Template
```
Tired of a cluttered OneDrive? Swipe & Clean makes decluttering effortless.

⚡ FAST & SIMPLE
Swipe right to keep photos, left to delete. No complicated menus, just quick decisions.

↩️ SAFE DELETING
5-second undo on every delete. Made a mistake? Just tap undo.

📊 TRACK YOUR PROGRESS
See how much space you've freed and celebrate your progress.

🔵 ONEDRIVE NATIVE
Built specifically for OneDrive with Microsoft's design language.

💎 FREEMIUM
- 25 free swipes every day
- Unlimited swipes for just 1 CHF/month
- No ads, ever

✨ BEAUTIFUL DESIGN
Features iOS 26's stunning Liquid Glass interface with smooth animations.

---

Your OneDrive, decluttered. Download now.
```

### Keywords (for App Store Connect)
```
onedrive, photo cleaner, storage, cloud storage, declutter, organize photos,
delete photos, free space, photo manager, microsoft onedrive, photo organizer,
cleanup, disk cleaner, cloud photos, photo curator
```

---

## Design Resources

### Tools Recommended
- **Design**: Figma or Sketch
- **Icon creation**: SF Symbols app (for system icons)
- **Color management**: ColorSlurp (macOS) for extracting brand colors
- **Prototyping**: Figma or Principle for animations

### SwiftUI Color Extensions
```swift
extension Color {
    static let oneDriveBlue = Color(hex: "0078D4")
    static let oneDriveBlueLight = Color(hex: "50A0E0")
    static let oneDriveBlueDark = Color(hex: "005A9E")
    static let successGreen = Color(hex: "107C10")
    static let alertRed = Color(hex: "D13438")
    static let neutralGray = Color(hex: "605E5C")
    static let backgroundLight = Color(hex: "F3F2F1")
    static let backgroundDark = Color(hex: "1B1A19")
}
```

---

## Brand Checklist

Before launch, verify:
- [ ] All colors match brand palette exactly
- [ ] App icon includes Liquid Glass effect (iOS 26)
- [ ] Typography uses SF Pro family consistently
- [ ] Animations use spring physics for swipe gestures
- [ ] Accessibility: VoiceOver labels on all interactive elements
- [ ] Accessibility: Tested at 200% Dynamic Type
- [ ] Accessibility: High contrast mode support
- [ ] Accessibility: Reduce motion alternative animations
- [ ] Dark mode: All screens adapt correctly
- [ ] Voice & tone: Copy is friendly and clear
- [ ] App Store: Screenshots match brand guidelines
- [ ] App Store: Description follows template structure

---

**Last Updated**: December 19, 2025
**Version**: 1.0
