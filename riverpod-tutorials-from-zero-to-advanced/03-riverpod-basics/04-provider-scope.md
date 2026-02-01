<div dir="rtl">

# ProviderScope بالتفصيل

**المستوى**: 🟡 متوسط

## 📌 المفاهيم الأساسية

في الملف ده هنتكلم عن:
- إيه هو ProviderScope وليه مهم
- إزاي ProviderScope بيشتغل
- Multiple scopes و الاستخدامات
- Overriding providers

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تفهم دور ProviderScope
- تستخدم multiple scopes
- تعمل override للـ providers
- تحل مشاكل الـ scoping

---

## 🎭 إيه هو ProviderScope؟

حل ProviderScope هو الـ **root container** اللي بيحتفظ بكل الـ providers في تطبيقك.

### التشبيه البسيط

</div>

```
ProviderScope زي:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📦 مخزن كبير (warehouse)
   └─ فيه كل الـ providers
   └─ بيدير حياتهم (lifecycle)
   └─ بيسمح للـ widgets تقراهم
```

<div dir="rtl">

### الاستخدام الأساسي

</div>

```dart
void main() {
  runApp(
    ProviderScope( // The root container
      child: MyApp(),
    ),
  );
}
```

<div dir="rtl">

**بدون ProviderScope:**

</div>

```dart
// ❌ This will throw an error!
void main() {
  runApp(MyApp()); // No ProviderScope!
}

class MyApp extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider); // ERROR: No ProviderScope found!
    return Text('$count');
  }
}
```

<div dir="rtl">

---

## 🏗️ إزاي ProviderScope بيشتغل؟

### البنية الداخلية

</div>

```
App Structure:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

ProviderScope
   │
   ├─ Provider 1 (counterProvider)
   ├─ Provider 2 (userProvider)
   ├─ Provider 3 (themeProvider)
   │
   └─ Widget Tree
       ├─ MaterialApp
       │   └─ HomePage
       │       ├─ CounterDisplay (reads counterProvider)
       │       └─ UserProfile (reads userProvider)
       └─ ...
```

<div dir="rtl">

### الـ Lifecycle

</div>

```dart
void main() {
  runApp(
    ProviderScope( // 1. Creates container
      child: MyApp(),
    ),
  );
}

// When app starts:
// 2. ProviderScope initializes
// 3. Providers are created on-demand (lazy)
// 4. Widgets can access via ref

// When app closes:
// 5. ProviderScope disposes all providers
// 6. Cleanup happens automatically
```

<div dir="rtl">

---

## 🎨 Multiple ProviderScopes

ممكن يكون عندك أكتر من ProviderScope في نفس التطبيق.

### حالة 1: Testing

</div>

```dart
// In tests
void main() {
  test('counter increments', () {
    // Create isolated scope for test
    final container = ProviderContainer();

    expect(container.read(counterProvider), 0);

    container.read(counterProvider.notifier).state++;

    expect(container.read(counterProvider), 1);

    // Cleanup
    container.dispose();
  });
}
```

<div dir="rtl">

### حالة 2: Multiple Windows (Desktop)

</div>

```dart
// Desktop app with multiple windows
void main() {
  // Window 1
  runApp(
    ProviderScope(
      child: MyApp(windowId: 1),
    ),
  );

  // Window 2 - separate scope
  runApp(
    ProviderScope(
      child: MyApp(windowId: 2),
    ),
  );
}

// Each window has its own state!
```

<div dir="rtl">

### حالة 3: Nested Scopes (Override)

</div>

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: ProviderScope( // Nested scope
        overrides: [
          // Override specific providers
          themeProvider.overrideWith((ref) => ThemeMode.dark),
        ],
        child: HomePage(),
      ),
    );
  }
}
```

<div dir="rtl">

---

## 🔄 Overriding Providers

حل Override مفيد جداً للـ testing ولـ dependency injection.

### مثال 1: Override في Testing

</div>

```dart
// Original provider
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() => state++;
}

// Test with override
void main() {
  test('displays counter value', () async {
    final container = ProviderContainer(
      overrides: [
        // Override with specific value
        counterProvider.overrideWith((ref) => 42),
      ],
    );

    expect(container.read(counterProvider), 42);

    container.dispose();
  });
}
```

<div dir="rtl">

### مثال 2: Override للـ Dependencies

</div>

```dart
// Real API service
@riverpod
class ApiService extends _$ApiService {
  @override
  ApiService build() {
    return RealApiService();
  }
}

// User repository depends on API
@riverpod
class UserRepository extends _$UserRepository {
  @override
  UserRepository build() {
    final api = ref.watch(apiServiceProvider);
    return UserRepository(api);
  }
}

// Test with mock
void main() {
  test('user repository fetches user', () async {
    final container = ProviderContainer(
      overrides: [
        // Override API with mock
        apiServiceProvider.overrideWith((ref) => MockApiService()),
      ],
    );

    final repository = container.read(userRepositoryProvider);
    final user = await repository.getUser();

    expect(user.name, 'Test User');

    container.dispose();
  });
}
```

<div dir="rtl">

### مثال 3: Override في UI

</div>

```dart
class SettingsPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ProviderScope(
      overrides: [
        // Preview mode - override with test data
        userProvider.overrideWith((ref) {
          return User(
            name: 'Preview User',
            email: 'preview@example.com',
          );
        }),
      ],
      child: UserProfile(),
    );
  }
}
```

<div dir="rtl">

---

## 🎯 Scoping Patterns

### Pattern 1: Feature-based Scoping

</div>

```dart
class ShoppingApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ProviderScope(
      child: MaterialApp(
        routes: {
          '/': (_) => HomePage(),
          '/cart': (_) => ProviderScope(
            // Cart has its own scope
            overrides: [
              cartProvider.overrideWith((ref) => CartNotifier()),
            ],
            child: CartPage(),
          ),
          '/checkout': (_) => CheckoutPage(),
        },
      ),
    );
  }
}
```

<div dir="rtl">

### Pattern 2: User-based Scoping

</div>

```dart
class MultiUserApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ProviderScope(
      child: MaterialApp(
        home: UserSelector(
          onUserSelected: (userId) {
            Navigator.push(
              context,
              MaterialPageRoute(
                builder: (_) => ProviderScope(
                  overrides: [
                    // Each user has isolated state
                    currentUserIdProvider.overrideWith((ref) => userId),
                  ],
                  child: UserDashboard(),
                ),
              ),
            );
          },
        ),
      ),
    );
  }
}
```

<div dir="rtl">

---

## 🔍 ProviderContainer (Manual Container)

للاستخدامات المتقدمة، ممكن تعمل container manually.

### الاستخدام الأساسي

</div>

```dart
// Create container
final container = ProviderContainer();

// Read providers
final count = container.read(counterProvider);

// Listen to changes
container.listen(
  counterProvider,
  (previous, next) {
    print('Counter changed: $previous -> $next');
  },
);

// Update state
container.read(counterProvider.notifier).state++;

// Cleanup
container.dispose();
```

<div dir="rtl">

### حالة 1: Background Services

</div>

```dart
class BackgroundService {
  late final ProviderContainer container;

  void start() {
    container = ProviderContainer();

    // Listen to location updates
    container.listen(
      locationProvider,
      (previous, next) {
        _sendToServer(next);
      },
    );
  }

  void stop() {
    container.dispose();
  }
}
```

<div dir="rtl">

### حالة 2: Isolates

</div>

```dart
// Main isolate
void main() {
  runApp(ProviderScope(child: MyApp()));
}

// Background isolate
void backgroundIsolate(SendPort sendPort) {
  final container = ProviderContainer();

  // Do work with providers
  final result = container.read(heavyComputationProvider);

  sendPort.send(result);

  container.dispose();
}
```

<div dir="rtl">

---

## ⚠️ مشاكل شائعة والحلول

### مشكلة 1: No ProviderScope Found

</div>

```dart
// ❌ Error
void main() {
  runApp(MyApp()); // Missing ProviderScope!
}

class MyApp extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider); // ERROR!
    return Text('$count');
  }
}

// ✅ Solution
void main() {
  runApp(
    ProviderScope( // Add this!
      child: MyApp(),
    ),
  );
}
```

<div dir="rtl">

### مشكلة 2: Disposed Container

</div>

```dart
// ❌ Error
final container = ProviderContainer();
container.dispose();

final count = container.read(counterProvider); // ERROR: Container disposed!

// ✅ Solution: Don't use after dispose
final container = ProviderContainer();
final count = container.read(counterProvider); // OK

container.dispose(); // Dispose when done
```

<div dir="rtl">

### مشكلة 3: Override Not Working

</div>

```dart
// ❌ Wrong: Override after child created
final container = ProviderContainer();
final widget = MyWidget();

container.updateOverrides([
  counterProvider.overrideWith((ref) => 10),
]); // Too late!

// ✅ Correct: Override during creation
final container = ProviderContainer(
  overrides: [
    counterProvider.overrideWith((ref) => 10),
  ],
);

final widget = MyWidget(); // Now it works
```

<div dir="rtl">

---

## 🎨 Advanced: Observers

ممكن تراقب كل التغييرات في الـ providers.

### إنشاء Observer

</div>

```dart
class MyObserver extends ProviderObserver {
  @override
  void didAddProvider(
    ProviderBase provider,
    Object? value,
    ProviderContainer container,
  ) {
    print('Provider added: ${provider.name}');
  }

  @override
  void didUpdateProvider(
    ProviderBase provider,
    Object? previousValue,
    Object? newValue,
    ProviderContainer container,
  ) {
    print('Provider updated: ${provider.name}');
    print('  Old: $previousValue');
    print('  New: $newValue');
  }

  @override
  void didDisposeProvider(
    ProviderBase provider,
    ProviderContainer container,
  ) {
    print('Provider disposed: ${provider.name}');
  }

  @override
  void providerDidFail(
    ProviderBase provider,
    Object error,
    StackTrace stackTrace,
    ProviderContainer container,
  ) {
    print('Provider failed: ${provider.name}');
    print('Error: $error');
  }
}
```

<div dir="rtl">

### استخدام Observer

</div>

```dart
void main() {
  runApp(
    ProviderScope(
      observers: [
        MyObserver(), // Add observer
      ],
      child: MyApp(),
    ),
  );
}

// Now all provider changes are logged!
```

<div dir="rtl">

### مثال: Analytics Observer

</div>

```dart
class AnalyticsObserver extends ProviderObserver {
  @override
  void didUpdateProvider(
    ProviderBase provider,
    Object? previousValue,
    Object? newValue,
    ProviderContainer container,
  ) {
    // Log to analytics
    analytics.logEvent('provider_updated', {
      'provider': provider.name,
      'value': newValue.toString(),
    });
  }
}

void main() {
  runApp(
    ProviderScope(
      observers: [AnalyticsObserver()],
      child: MyApp(),
    ),
  );
}
```

<div dir="rtl">

---

## 📊 ملخص: ProviderScope

</div>

```
ما هو ProviderScope؟
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
- Container لكل الـ providers
- Root of Riverpod tree
- يدير lifecycle
- يسمح بـ overrides

متى تستخدم Multiple Scopes؟
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Testing (isolated tests)
✅ Multiple windows (desktop)
✅ Feature isolation
✅ Override providers

Observers:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Logging
✅ Analytics
✅ Debugging
✅ Monitoring
```

<div dir="rtl">

---

## 🎯 Best Practices

### ممارسة 1: واحد Root Scope

</div>

```dart
// ✅ GOOD: One root scope
void main() {
  runApp(
    ProviderScope(
      child: MyApp(),
    ),
  );
}
```

<div dir="rtl">

### ممارسة 2: استخدم Observers للـ Debugging

</div>

```dart
void main() {
  runApp(
    ProviderScope(
      observers: [
        if (kDebugMode) DebugObserver(), // Only in debug
      ],
      child: MyApp(),
    ),
  );
}
```

<div dir="rtl">

### ممارسة 3: Dispose Containers

</div>

```dart
class MyService {
  late final ProviderContainer _container;

  void start() {
    _container = ProviderContainer();
  }

  void stop() {
    _container.dispose(); // Always dispose!
  }
}
```

<div dir="rtl">

### ممارسة 4: Override للـ Testing فقط

</div>

```dart
// ✅ GOOD: Override in tests
test('my test', () {
  final container = ProviderContainer(
    overrides: [/*...*/],
  );
});

// ⚠️ CAREFUL: Override in production
// Only if you really need it
```

<div dir="rtl">

---

## 🔗 الخطوة الجاية

دلوقتي بعد ما فهمت ProviderScope، وقت:
- **تطبيق عملي كامل** (الملف الجاي)
- **Advanced topics**
- **Real-world examples**

---

## 📚 المصادر

- [ProviderScope Documentation](https://riverpod.dev/docs/concepts/provider_scope)
- [ProviderContainer](https://riverpod.dev/docs/concepts/provider_container)
- [ProviderObserver](https://riverpod.dev/docs/concepts/provider_observer)

---

## ✅ تأكد إنك فهمت

- [ ] تعرف دور ProviderScope؟
- [ ] فاهم Multiple Scopes؟
- [ ] تقدر تعمل Override؟
- [ ] تعرف تستخدم Observers؟
- [ ] جاهز للتطبيق العملي؟

**يلا نعمل تطبيق كامل! 🚀**

</div>
