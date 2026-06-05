# Mobile UI Polish

Platform-native feel: haptics, animations, safe areas, keyboard handling.

## Rules

### 1. Platform Conventions
- iOS: navigation bar at top, "Back" gesture, Cupertino modals, SF Symbols.
- Android: material design, back button (hardware), ripple effects.
- Respect platform defaults. Don't force iOS patterns on Android.

### 2. Safe Areas & Insets
```dart
// Flutter
SafeArea(child: content)
MediaQuery.of(context).padding.top  // Status bar height

// React Native
import { useSafeAreaInsets } from 'react-native-safe-area-context';
const { top, bottom } = useSafeAreaInsets();
```
Always account for notch, dynamic island, home indicator.

### 3. Keyboard Handling
```dart
// Flutter
Scaffold(resizeToAvoidBottomInset: true)
// React Native
<KeyboardAvoidingView behavior="padding">
```
- Scroll to focused input when keyboard opens.
- Dismiss keyboard on scroll/drag.
- Don't let keyboard cover the submit button.

### 4. Haptics & Feedback
- Button press: light haptic feedback.
- Success/error: medium haptic.
- Swipe to delete: confirmation haptic.
- Use platform haptics packages. Don't overuse.

### 5. Animations
- Duration: 200-300ms for micro-interactions, 400ms for transitions.
- Easing: ease-out for appearing, ease-in for disappearing.
- Stagger children animations (50ms delay between items).
- Use `Animated` (RN) / `AnimationController` (Flutter) over setState loops.

## Anti-Patterns
- ❌ Ignoring safe areas (content under notch)
- ❌ No keyboard handling (input hidden by keyboard)
- ❌ Custom animations that don't match platform feel
- ❌ Animations longer than 500ms (feels sluggish)
