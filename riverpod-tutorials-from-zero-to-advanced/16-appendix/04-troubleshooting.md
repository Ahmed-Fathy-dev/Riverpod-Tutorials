<div dir="rtl">

# حل المشاكل الشائعة (Troubleshooting)

**المستوى**: 📚 مرجعي

## 🔴 أخطاء شائعة وحلولها

### Error: "ProviderNotFoundException"

**الرسالة:**
```
ProviderNotFoundException: No provider found for counterProvider
```

**السبب:**
التطبيق غير مُغلَّف بـ ProviderScope.

**الحل:**
```dart
void main() {
  runApp(
    const ProviderScope(  // ✅ Add this
      child: MyApp(),
    ),
  );
}
```

---

### Error: "Cannot use ref.watch inside an event handler"

**الرسالة:**
```
Cannot use ref.watch inside an event handler. Consider using ref.read instead.
```

**السبب:**
استخدام ref.watch خارج build method.

**الحل:**
```dart
// ❌ WRONG
void _onPressed() {
  final counter = ref.watch(counterProvider);  // Error!
}

// ✅ CORRECT
void _onPressed() {
  final counter = ref.read(counterProvider);
}
```

---

### Error: "Bad state: No ProviderScope found"

**الرسالة:**
```
Bad state: No ProviderScope found
```

**السبب:**
في الاختبارات، لم تُضف ProviderScope للويدجت.

**الحل:**
```dart
testWidgets('test', (tester) async {
  await tester.pumpWidget(
    const ProviderScope(  // ✅ Wrap in ProviderScope
      child: MyWidget(),
    ),
  );

  expect(find.text('Hello'), findsOneWidget);
});
```

---

### Error: "StateError (Bad state: No element)"

**الرسالة:**
```
Bad state: No element
```

**السبب:**
محاولة الوصول إلى .value! على AsyncValue وهو loading أو error.

**الحل:**
```dart
// ❌ WRONG - Can crash
final user = ref.watch(userProvider).value!;

// ✅ CORRECT - Use .when()
final userAsync = ref.watch(userProvider);
return userAsync.when(
  data: (user) => UserCard(user),
  loading: () => const CircularProgressIndicator(),
  error: (error, _) => ErrorMessage(error),
);

// ✅ CORRECT - Check first
final userAsync = ref.watch(userProvider);
if (userAsync.hasValue) {
  return UserCard(userAsync.value!);
}
return const CircularProgressIndicator();
```

---

### Error: "Circular dependency detected"

**الرسالة:**
```
Circular dependency detected: providerA -> providerB -> providerA
```

**السبب:**
providerA يعتمد على providerB والعكس صحيح.

**الحل:**
```dart
// ❌ WRONG - Circular dependency
@riverpod
int providerA(ProviderARef ref) {
  return ref.watch(providerBProvider) + 1;
}

@riverpod
int providerB(ProviderBRef ref) {
  return ref.watch(providerAProvider) + 1;  // Circular!
}

// ✅ CORRECT - Extract shared state
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;
}

@riverpod
int doubleCounter(DoubleCounterRef ref) {
  return ref.watch(counterProvider) * 2;
}

@riverpod
int tripleCounter(TripleCounterRef ref) {
  return ref.watch(counterProvider) * 3;
}
```

---

### Error: "The provider XXX was already disposed"

**الرسالة:**
```
The provider was already disposed
```

**السبب:**
محاولة الوصول إلى provider بعد dispose.

**الحل:**
```dart
// ✅ Use mounted check in StatefulWidget
if (mounted) {
  ref.read(counterProvider.notifier).increment();
}

// ✅ Or use ref.listen with fireImmediately: false
ref.listen(
  counterProvider,
  (previous, next) {
    // Handle changes
  },
  fireImmediately: false,  // Don't fire if already disposed
);
```

---

## 🟡 مشاكل Code Generation

### المشكلة: "build_runner لا يُولِّد الكود"

**الأعراض:**
- ملفات .g.dart غير موجودة
- أخطاء "Undefined class _$Counter"

**الحلول:**

**1. تأكد من تشغيل build_runner:**
```bash
dart run build_runner watch --delete-conflicting-outputs
```

**2. تأكد من وجود part directive:**
```dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'counter.g.dart';  // ✅ Must have this!

@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;
}
```

**3. تأكد من dependencies في pubspec.yaml:**
```yaml
dependencies:
  flutter_riverpod: ^2.5.0
  riverpod_annotation: ^2.3.0

dev_dependencies:
  build_runner: ^2.4.0
  riverpod_generator: ^2.3.0
```

**4. نظف الـ cache:**
```bash
dart run build_runner clean
flutter clean
flutter pub get
dart run build_runner build --delete-conflicting-outputs
```

---

### المشكلة: "Conflicts with existing file"

**الرسالة:**
```
Conflict: Multiple outputs would be written to counter.g.dart
```

**الحل:**
```bash
# استخدم --delete-conflicting-outputs
dart run build_runner build --delete-conflicting-outputs
```

---

### المشكلة: "part 'file.g.dart' has no corresponding part of directive"

**السبب:**
اسم الملف في part directive لا يطابق اسم الملف الفعلي.

**الحل:**
```dart
// File: products_provider.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'products_provider.g.dart';  // ✅ Must match file name

@riverpod
class Products extends _$Products {
  // ...
}
```

---

## 🟠 مشاكل الأداء

### المشكلة: "الويدجت يُعاد بناؤه كثيراً"

**التشخيص:**
استخدم Flutter DevTools لمراقبة rebuilds.

**الحلول:**

**1. استخدم .select:**
```dart
// ❌ Rebuilds on any user change
final user = ref.watch(userProvider);
return Text(user.name);

// ✅ Rebuilds only when name changes
final name = ref.watch(userProvider.select((u) => u.name));
return Text(name);
```

**2. قسّم الويدجتات:**
```dart
// ❌ Large widget watches everything
class MyScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final user = ref.watch(userProvider);
    final products = ref.watch(productsProvider);
    // Rebuilds entire screen!
    return Column(
      children: [
        UserCard(user),
        ProductsList(products),
      ],
    );
  }
}

// ✅ Split into smaller widgets
class MyScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return const Column(
      children: [
        UserCardWidget(),     // Watches user only
        ProductsListWidget(), // Watches products only
      ],
    );
  }
}

class UserCardWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final user = ref.watch(userProvider);
    return UserCard(user);
  }
}
```

**3. استخدم const constructors:**
```dart
return const CircularProgressIndicator();  // ✅ const
return CircularProgressIndicator();        // ❌ rebuilds every time
```

---

### المشكلة: "Memory leak مع Family providers"

**السبب:**
استخدام keepAlive: true مع Family.

**الحل:**
```dart
// ❌ WRONG - Memory leak!
@Riverpod(keepAlive: true)
Future<Product> product(ProductRef ref, String id) async {
  // Each ID creates a new instance that never disposes!
  return await api.getProduct(id);
}

// ✅ CORRECT - Use AutoDispose
@riverpod  // Default: AutoDispose
Future<Product> product(ProductRef ref, String id) async {
  return await api.getProduct(id);
}

// ✅ OR use selective keepAlive
@riverpod
Future<Product> product(ProductRef ref, String id) async {
  final link = ref.keepAlive();

  // Dispose after 2 minutes of inactivity
  Timer? timer;
  ref.onCancel(() {
    timer = Timer(const Duration(minutes: 2), link.close);
  });
  ref.onResume(() {
    timer?.cancel();
  });
  ref.onDispose(() {
    timer?.cancel();
  });

  return await api.getProduct(id);
}
```

---

## 🔵 مشاكل الاختبارات

### المشكلة: "Provider not disposed في الاختبارات"

**الرسالة:**
```
Warning: Provider was not disposed
```

**السبب (قديم):**
في Riverpod 2.x كان يجب dispose يدوياً.

**الحل (Riverpod 3):**
استخدم **ProviderContainer.test()** - يتخلص تلقائياً!

```dart
// ✅ Riverpod 3 - Auto dispose
test('counter increments', () {
  final container = ProviderContainer.test();  // ✅ No manual dispose!

  expect(container.read(counterProvider), 0);
  container.read(counterProvider.notifier).increment();
  expect(container.read(counterProvider), 1);
});

// ❌ Old way (Riverpod 2.x)
test('counter increments', () {
  final container = ProviderContainer();
  addTearDown(container.dispose);  // Manual dispose

  // ...
});
```

---

### المشكلة: "Mock لا يعمل في الاختبارات"

**السبب:**
Override خاطئ أو mock غير صحيح.

**الحل:**
```dart
// ✅ CORRECT - Mock the dependency, not the provider
class MockTodosApi extends Mock implements TodosApi {}

test('loads todos', () async {
  final mockApi = MockTodosApi();

  // Setup mock
  when(() => mockApi.getTodos()).thenAnswer(
    (_) async => [Todo(id: '1', title: 'Test')],
  );

  final container = ProviderContainer.test(
    overrides: [
      todosApiProvider.overrideWithValue(mockApi),  // ✅ Mock API
    ],
  );

  final todos = await container.read(todosProvider.future);

  expect(todos.length, 1);
  verify(() => mockApi.getTodos()).called(1);
});

// ❌ WRONG - Don't mock the provider itself
test('loads todos', () async {
  final container = ProviderContainer.test(
    overrides: [
      todosProvider.overrideWith(() => [/* mock data */]),  // ❌ Bad practice
    ],
  );
  // This bypasses all logic!
});
```

---

## 🟢 مشاكل متنوعة

### المشكلة: "لا يمكن استخدام ref خارج Widget"

**السبب:**
محاولة الوصول إلى ref في كلاس عادي.

**الحل:**

**استخدم Dependency Injection:**
```dart
// ❌ WRONG - Can't access ref here
class ProductService {
  List<Product> getProducts() {
    // How to get ref here???
  }
}

// ✅ CORRECT - Pass dependencies
class ProductService {
  final ProductsRepository repository;

  ProductService(this.repository);  // DI

  Future<List<Product>> getProducts() {
    return repository.getProducts();
  }
}

// Provide via Riverpod
@riverpod
ProductService productService(ProductServiceRef ref) {
  final repository = ref.watch(productsRepositoryProvider);
  return ProductService(repository);
}
```

---

### المشكلة: "AsyncValue.guard لا يمسك الأخطاء"

**السبب:**
الخطأ يحدث خارج guard.

**الحل:**
```dart
// ❌ WRONG - Error happens before guard
Future<void> addTodo(String title) async {
  validateTitle(title);  // ❌ Error thrown here, not caught!

  state = await AsyncValue.guard(() async {
    return await api.addTodo(title);
  });
}

// ✅ CORRECT - Put validation inside guard
Future<void> addTodo(String title) async {
  state = await AsyncValue.guard(() async {
    validateTitle(title);  // ✅ Now caught
    return await api.addTodo(title);
  });
}
```

---

### المشكلة: "Hot Reload لا يُحدث الـ providers"

**السبب:**
Providers لا يُعاد بناؤها مع Hot Reload.

**الحل:**

**1. استخدم Hot Restart بدلاً من Hot Reload:**
- Hot Reload (⚡) - لتحديثات UI سريعة
- Hot Restart (🔄) - لإعادة بناء كل شيء

**2. أو استخدم ref.invalidate أثناء التطوير:**
```dart
// في ويدجت Dev Tools
ElevatedButton(
  onPressed: () => ref.invalidate(todosProvider),
  child: const Text('Refresh Provider'),
)
```

---

### المشكلة: "Provider يُعاد بناؤه مرتين عند التحديث"

**السبب:**
Dependency chain تُحدَّث مرتين.

**الحل:**

**تحقق من dependencies:**
```dart
// ❌ WRONG - Unnecessary rebuild
@riverpod
Future<List<Order>> userOrders(UserOrdersRef ref) async {
  final userId = ref.watch(userIdProvider);
  final user = await ref.watch(userProvider.future);  // Already watches userId!
  return await api.getUserOrders(user.id);
}

// ✅ CORRECT - Watch only what you need
@riverpod
Future<List<Order>> userOrders(UserOrdersRef ref) async {
  final user = await ref.watch(userProvider.future);
  return await api.getUserOrders(user.id);
}
```

---

## 🛠️ أدوات التشخيص

### استخدام ProviderObserver للتتبع

```dart
class LoggerObserver extends ProviderObserver {
  @override
  void didUpdateProvider(
    ProviderBase provider,
    Object? previousValue,
    Object? newValue,
    ProviderContainer container,
  ) {
    print('Provider: ${provider.name ?? provider.runtimeType}');
    print('Previous: $previousValue');
    print('New: $newValue');
    print('---');
  }

  @override
  void didAddProvider(
    ProviderBase provider,
    Object? value,
    ProviderContainer container,
  ) {
    print('Created: ${provider.name ?? provider.runtimeType}');
  }

  @override
  void didDisposeProvider(
    ProviderBase provider,
    ProviderContainer container,
  ) {
    print('Disposed: ${provider.name ?? provider.runtimeType}');
  }
}

void main() {
  runApp(
    ProviderScope(
      observers: [LoggerObserver()],  // Add observer
      child: const MyApp(),
    ),
  );
}
```

---

### استخدام Flutter DevTools

1. افتح DevTools
2. اذهب إلى Widget Inspector
3. ابحث عن ProviderScope
4. راقب rebuilds والـ state

---

## 📞 طلب المساعدة

عند طلب المساعدة في Discord/GitHub:

**1. وفر معلومات النسخ:**
```bash
flutter --version
flutter pub deps | grep riverpod
```

**2. وفر الكود:**
- الـ provider
- كيف تستخدمه
- الخطأ الكامل

**3. وفر خطوات إعادة المشكلة:**
- ما الذي فعلته؟
- ما كانت النتيجة المتوقعة؟
- ما الذي حدث بدلاً من ذلك؟

**4. جرّب Minimal Reproducible Example:**
- أزل الكود غير الضروري
- اصنع مثال بسيط يُظهر المشكلة

---

## ✅ نصائح لتجنب المشاكل

1. **اقرأ رسائل الأخطاء بعناية** - غالباً تخبرك بالحل!
2. **استخدم Linting** - riverpod_lint يكشف الأخطاء مبكراً
3. **تابع التوثيق الرسمي** - دائماً محدث
4. **ابدأ بسيط ثم تقدم** - لا تستخدم features متقدمة بدون فهم الأساسيات
5. **اختبر مبكراً وباستمرار** - اكتشف المشاكل قبل أن تكبر

</div>
