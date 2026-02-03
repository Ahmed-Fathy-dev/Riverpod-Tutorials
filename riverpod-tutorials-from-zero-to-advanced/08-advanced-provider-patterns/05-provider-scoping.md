<div dir="rtl">

# Provider Scoping - Overriding Providers

**المستوى**: 🔴 متقدم

## 📌 الخلاصة

**Provider Scoping** = تغيير سلوك provider في جزء معين من الـ app.

```dart
ProviderScope(
  overrides: [
    // Override userProvider for this subtree
    userProvider.overrideWith((ref) => User.mock()),
  ],
  child: MyWidget(),
)
```

---

## 🎯 Use Cases

### 1. Testing

```dart
void main() {
  testWidgets('displays user name', (tester) async {
    await tester.pumpWidget(
      ProviderScope(
        overrides: [
          // Mock user for testing
          userProvider.overrideWith((ref) => User(name: 'Test User')),
        ],
        child: MaterialApp(home: UserProfileScreen()),
      ),
    );

    expect(find.text('Test User'), findsOneWidget);
  });
}
```

### 2. Feature Flags

```dart
@riverpod
bool featureEnabled(FeatureEnabledRef ref) => false; // Default: disabled

void main() {
  runApp(
    ProviderScope(
      overrides: [
        // Enable feature for beta users
        if (isBetaUser)
          featureEnabledProvider.overrideWith((ref) => true),
      ],
      child: MyApp(),
    ),
  );
}
```

### 3. Multi-Tenancy

```dart
@riverpod
class TenantConfig extends _$TenantConfig {
  @override
  Config build() => Config.default();
}

// Different config per tenant
ProviderScope(
  overrides: [
    tenantConfigProvider.overrideWith((ref) => Config.forTenant('acme-corp')),
  ],
  child: TenantApp(),
)
```

### 4. Previews/Storybook

```dart
// In your storybook/preview
class UserCardPreview extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ProviderScope(
      overrides: [
        userProvider.overrideWith((ref) => User.sample()),
      ],
      child: UserCard(),
    );
  }
}
```

---

## 🔧 Override Types

### 1. overrideWith

```dart
// Override with custom logic
ProviderScope(
  overrides: [
    userProvider.overrideWith((ref) => User.mock()),
  ],
  child: MyApp(),
)
```

### 2. overrideWithValue

```dart
// Override with specific value
final mockUser = User(name: 'Mock');

ProviderScope(
  overrides: [
    userProvider.overrideWithValue(mockUser),
  ],
  child: MyApp(),
)
```

### 3. overrideWith for NotifierProvider

```dart
// Override notifier
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() => state++;
}

// Mock counter for testing
class MockCounter extends Counter {
  @override
  int build() => 100; // Start at 100

  @override
  void increment() {
    state += 10; // Increment by 10
  }
}

// In test
ProviderScope(
  overrides: [
    counterProvider.overrideWith(() => MockCounter()),
  ],
  child: CounterApp(),
)
```

---

## 📋 Best Practices

### 1. Use in Tests

```dart
// ✅ GOOD - Override for deterministic tests
testWidgets('test', (tester) async {
  await tester.pumpWidget(
    ProviderScope(
      overrides: [
        apiProvider.overrideWith((ref) => MockApi()),
      ],
      child: MyApp(),
    ),
  );
});
```

### 2. Avoid in Production Code

```dart
// ❌ BAD - Don't override in regular app code
Widget build(BuildContext context) {
  return ProviderScope(
    overrides: [userProvider.overrideWith(...)],
    child: ...,
  );
}

// ✅ GOOD - Use parameters or state instead
@riverpod
User user(UserRef ref, {bool useMock = false}) {
  if (useMock) return User.mock();
  return User.real();
}
```

### 3. Document Overrides

```dart
/// Test helper for user-related widgets.
///
/// Provides a mock user with predictable data.
ProviderScope testUserScope({required Widget child}) {
  return ProviderScope(
    overrides: [
      userProvider.overrideWith((ref) => User.test()),
    ],
    child: child,
  );
}
```

---

## 🎓 Complete Test Example

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

// Mock API
class MockTodosApi implements TodosApi {
  @override
  Future<List<Todo>> getTodos() async {
    return [
      Todo(id: '1', title: 'Test Todo 1'),
      Todo(id: '2', title: 'Test Todo 2'),
    ];
  }

  @override
  Future<void> addTodo(String title) async {
    // Mock - do nothing
  }
}

void main() {
  testWidgets('displays todos', (tester) async {
    await tester.pumpWidget(
      ProviderScope(
        overrides: [
          // Override API with mock
          todosApiProvider.overrideWith((ref) => MockTodosApi()),
        ],
        child: const MaterialApp(home: TodosScreen()),
      ),
    );

    // Wait for async data
    await tester.pumpAndSettle();

    // Verify
    expect(find.text('Test Todo 1'), findsOneWidget);
    expect(find.text('Test Todo 2'), findsOneWidget);
  });

  testWidgets('adds todo', (tester) async {
    final mockApi = MockTodosApi();

    await tester.pumpWidget(
      ProviderScope(
        overrides: [
          todosApiProvider.overrideWithValue(mockApi),
        ],
        child: const MaterialApp(home: TodosScreen()),
      ),
    );

    // Tap add button
    await tester.tap(find.byIcon(Icons.add));
    await tester.pumpAndSettle();

    // Enter title
    await tester.enterText(find.byType(TextField), 'New Todo');
    await tester.tap(find.text('Add'));
    await tester.pumpAndSettle();

    // Verify (in real test, mock API would track calls)
  });
}
```

---

## 📚 المصادر

- [Testing | Riverpod](https://riverpod.dev/docs/how_to/testing)
- [ProviderScope | API Reference](https://pub.dev/documentation/flutter_riverpod/latest/flutter_riverpod/ProviderScope-class.html)

</div>
