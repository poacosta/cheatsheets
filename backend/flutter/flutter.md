# Flutter Mobile Development Workflow

## Quick Commands

### Project Setup & Build

```bash
# Flutter build commands
flutter build ios
flutter build ipa
flutter build apk
flutter build appbundle  # For Google Play Store

# Development commands
flutter run
flutter run -d ios
flutter run -d android
flutter hot-reload  # r in terminal during run
flutter hot-restart  # R in terminal during run
```

### Asset & Icon Management

```bash
# Generate app icons
flutter pub run flutter_launcher_icons:main

# Add assets to pubspec.yaml first
flutter:
  assets:
    - assets/images/
    - assets/icons/

# Get dependencies
flutter pub get
flutter pub upgrade
```

### iOS Development

```bash
# Open iOS project in Xcode
open ios/Runner.xcworkspace

# iOS simulator management
xcrun simctl list devices
xcrun simctl boot "iPhone 14 Pro"

# iOS build for device
flutter build ios --release
```

## Common Patterns

### Project Structure

```
lib/
├── main.dart
├── models/
├── screens/
├── widgets/
├── services/
├── utils/
└── constants/

assets/
├── images/
├── icons/
└── fonts/

test/
├── unit/
├── widget/
└── integration/
```

### Pubspec.yaml Configuration

```yaml
dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.2
  http: ^0.13.5
  shared_preferences: ^2.0.15
  provider: ^6.0.3

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_launcher_icons: ^0.13.1
  flutter_lints: ^2.0.0

flutter_icons:
  android: "launcher_icon"
  ios: true
  image_path: "assets/icon/icon.png"
  min_sdk_android: 21
```

### State Management Patterns

```dart
// Provider pattern
class CounterModel extends ChangeNotifier {
  int _count = 0;
  int get count => _count;

  void increment() {
    _count++;
    notifyListeners();
  }
}

// BLoC pattern
class CounterBloc extends Bloc<CounterEvent, CounterState> {
  CounterBloc() : super(CounterInitial()) {
    on<CounterIncremented>((event, emit) {
      emit(CounterState(count: state.count + 1));
    });
  }
}

// Riverpod pattern
final counterProvider = StateNotifierProvider<CounterNotifier, int>((ref) {
  return CounterNotifier();
});
```

## Personal Gotchas

### Dart SDK Path Management

```bash
# Homebrew installation path
export PATH="$PATH:/opt/homebrew/opt/dart/libexec"

# Verify installation
dart --version
flutter doctor
```

### iOS Development Setup

- Always use `Runner.xcworkspace`, not `Runner.xcodeproj`
- Update iOS deployment target in Xcode for newer features
- Configure signing & capabilities in Xcode project settings
- Test on physical device for accurate performance metrics

### Android Configuration

```gradle
// android/app/build.gradle
android {
    compileSdkVersion 33

    defaultConfig {
        minSdkVersion 21
        targetSdkVersion 33
        versionCode flutterVersionCode.toInteger()
        versionName flutterVersionName
    }
}
```

### Common Build Issues

```bash
# Clean build cache
flutter clean
flutter pub get

# Reset iOS simulator
xcrun simctl erase all

# Fix iOS keychain issues
security delete-keychain login.keychain
security create-keychain login.keychain
```

## Performance Notes

### Build Optimization

```bash
# Release builds
flutter build apk --release --split-per-abi
flutter build ios --release --no-codesign

# Profile mode for performance testing
flutter run --profile
flutter build apk --profile
```

### Widget Performance

```dart
// Use const constructors
const Text('Static text');
const Icon(Icons.star);

// ListView.builder for large lists
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return ListTile(title: Text(items[index]));
  },
);

// RepaintBoundary for complex widgets
RepaintBoundary(
  child: ComplexCustomWidget(),
);
```

### Memory Management

```dart
// Dispose controllers
class _MyWidgetState extends State<MyWidget> {
  late TextEditingController _controller;

  @override
  void initState() {
    super.initState();
    _controller = TextEditingController();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}
```

## Architecture Considerations

### Clean Architecture Pattern

```
lib/
├── core/
│   ├── error/
│   ├── network/
│   └── utils/
├── features/
│   └── feature_name/
│       ├── data/
│       ├── domain/
│       └── presentation/
└── injection_container.dart
```

### Platform-Specific Code

```dart
// Method channels for native functionality
class NativeBridge {
  static const platform = MethodChannel('com.example/native');

  static Future<String> getNativeData() async {
    try {
      final result = await platform.invokeMethod('getData');
      return result;
    } on PlatformException catch (e) {
      throw 'Failed to get native data: ${e.message}';
    }
  }
}
```

### Testing Strategy

```dart
// Unit tests
test('Counter increments', () {
  final counter = Counter();
  counter.increment();
  expect(counter.value, 1);
});

// Widget tests
testWidgets('Counter increments smoke test', (WidgetTester tester) async {
  await tester.pumpWidget(MyApp());
  expect(find.text('0'), findsOneWidget);

  await tester.tap(find.byIcon(Icons.add));
  await tester.pump();

  expect(find.text('1'), findsOneWidget);
});

// Integration tests
void main() {
  group('App E2E', () {
    testWidgets('Full user journey', (tester) async {
      await tester.pumpAndSettle();
      // Test complete user flows
    });
  });
}
```

## Development Workflow

### Hot Reload Best Practices

- Preserve app state during development
- Use `hot restart` for state-related changes
- Leverage `flutter inspector` for widget debugging

### Debugging Tools

```bash
# Flutter inspector
flutter run --debug
# Open DevTools in browser

# Performance profiling
flutter run --profile
flutter run --trace-startup

# Network debugging
flutter run --debug --dart-define=FLUTTER_WEB_USE_SKIA=true
```

### CI/CD Integration

```yaml
# GitHub Actions example
- uses: subosito/flutter-action@v2
  with:
    flutter-version: "3.10.0"
- run: flutter pub get
- run: flutter test
- run: flutter build apk --release
```

## Context Links

- [Flutter Documentation](https://docs.flutter.dev/)
- [Dart Language Tour](https://dart.dev/guides/language/language-tour)
- [Flutter Performance Best Practices](https://docs.flutter.dev/development/ui/widgets/performance)
- [Platform Integration Guide](https://docs.flutter.dev/development/platform-integration)
