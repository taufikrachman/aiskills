# Flutter App Builder

Build production Flutter apps with clean architecture and offline-first patterns.

## Rules

### 1. Project Structure
```
lib/
├── main.dart
├── app/
│   ├── app.dart              # MaterialApp + theme + routes
│   └── router.dart           # Navigation config
├── core/
│   ├── constants/            # API URLs, keys, limits
│   ├── theme/                # Colors, typography, spacing
│   └── utils/                # Formatters, validators, helpers
├── data/
│   ├── datasources/          # API client, DB helper, secure storage
│   ├── models/               # Data models with fromJson/toJson
│   └── repositories/         # Data access logic
├── features/
│   ├── auth/                 # Login, PIN, register
│   ├── pos/                  # POS screen
│   └── products/             # Product management
└── services/                 # Sync, image upload, notifications
```

### 2. State Management
- Provider for app-wide state (auth, sync, theme).
- Provider + ChangeNotifier for feature state (cart, product list).
- SetState for local UI state only (text field, checkbox).

### 3. Offline-First Architecture
```dart
// 1. Save to local DB first
await db.insert('transactions', transaction.toJson());
// 2. Queue sync event
await syncService.queueSyncEvent(action: create, data: transaction);
// 3. Sync in background (runs on connectivity change + periodic timer)
```
- SQLite (sqflite) for local storage.
- FlutterSecureStorage for tokens, PIN, device ID.
- Sync service: push pending → pull changes.
- Retry with exponential backoff on failure.

### 4. Platform-Specific Code
```dart
import 'dart:io' show Platform;
if (Platform.isIOS) { /* iOS-specific */ }
if (Platform.isAndroid) { /* Android-specific */ }
```
- iOS: Cupertino widgets for dialogs, date pickers, switches.
- Android: Material widgets. Match platform feel.
- Feature detection over platform check when possible.

### 5. Performance
- `const` constructors for static widgets (rebuild optimization).
- `ListView.builder` for long lists (virtualization).
- `RepaintBoundary` for isolating repaint areas.
- Image caching: `cached_network_image` package.
- Avoid rebuilds: `Consumer<T>(builder: (_, value, child) => child!)` for static children.

## Anti-Patterns
- ❌ Business logic in widgets (extract to providers/services)
- ❌ setState for everything (use Provider for shared state)
- ❌ Hardcoded strings in UI (use AppLocalizations or constants)
- ❌ Assuming always-on internet (always save locally first)
