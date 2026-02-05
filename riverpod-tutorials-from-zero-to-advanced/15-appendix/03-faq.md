<div dir="rtl">

# الأسئلة الشائعة (FAQ)

**المستوى**: 📚 مرجعي

## ❓ أسئلة عامة

### لماذا Riverpod وليس Provider؟

**الجواب:**
Riverpod هو تطوير كامل لـ Provider، يحل جميع مشاكله:

- ✅ **لا يعتمد على BuildContext** - يمكن استخدامه في أي مكان
- ✅ **Compile-time safety** - الأخطاء تظهر أثناء الكتابة
- ✅ **أسهل في الاختبار** - لا حاجة لـ widget tree
- ✅ **أفضل أداء** - تحسينات في الـ rebuild
- ✅ **Code generation** - أقل boilerplate

---

### هل يجب استخدام Code Generation؟

**الجواب:**
**نعم، بشدة!** Code generation هو الطريقة الموصى بها في Riverpod 2+.

**المميزات:**
- أقل boilerplate code
- Type safety أفضل
- أخطاء أقل
- Syntax أنظف

**الاستثناءات:**
- مشاريع قديمة جداً
- عدم إمكانية استخدام build_runner

---

### متى أستخدم Notifier ومتى AsyncNotifier؟

**الجواب:**

**استخدم Notifier** عندما:
- الحالة synchronous (مثل counter، form state)
- لا تحتاج async operations في build

```dart
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;  // Synchronous
}
```

**استخدم AsyncNotifier** عندما:
- الحالة تحتاج async initialization
- تحمل البيانات من API

```dart
@riverpod
class Todos extends _$Todos {
  @override
  Future<List<Todo>> build() async {  // Asynchronous
    return await api.getTodos();
  }
}
```

---

### ما الفرق بين ref.watch و ref.read؟

**الجواب:**

**ref.watch:**
- يستمع للتغييرات
- يُعيد بناء الويدجت/البروفايدر عند التغيير
- **فقط في build method**

```dart
@override
Widget build(BuildContext context, WidgetRef ref) {
  final counter = ref.watch(counterProvider);  // ✅ في build
  return Text('$counter');
}
```

**ref.read:**
- يقرأ القيمة مرة واحدة
- لا يستمع للتغييرات
- **في event handlers و methods**

```dart
void _onButtonPressed() {
  ref.read(counterProvider.notifier).increment();  // ✅ في method
}
```

---

### متى أستخدم AutoDispose ومتى keepAlive؟

**الجواب:**

**AutoDispose (الافتراضي):**
```dart
@riverpod  // AutoDispose by default
Future<Data> data(DataRef ref) async { ... }
```

**استخدمه عندما:**
- البيانات يمكن إعادة تحميلها بسهولة
- البيانات خاصة بصفحة معينة
- تستخدم Family (parameters)

**keepAlive:**
```dart
@Riverpod(keepAlive: true)
Future<Config> config(ConfigRef ref) async { ... }
```

**استخدمه عندما:**
- البيانات تُحمل مرة واحدة فقط (مثل app config)
- تحميل البيانات مكلف
- البيانات global للتطبيق كله

---

### كيف أتعامل مع Loading و Error في AsyncValue؟

**الجواب:**

**الطريقة الأولى - .when():**
```dart
final todosAsync = ref.watch(todosProvider);

return todosAsync.when(
  data: (todos) => TodosList(todos),
  loading: () => const CircularProgressIndicator(),
  error: (error, stack) => Text('Error: $error'),
);
```

**الطريقة الثانية - Pattern Matching (Dart 3):**
```dart
return switch (todosAsync) {
  AsyncData(:final value) => TodosList(value),
  AsyncError(:final error) => Text('Error: $error'),
  _ => const CircularProgressIndicator(),
};
```

**الطريقة الثالثة - Properties:**
```dart
if (todosAsync.isLoading) {
  return const CircularProgressIndicator();
}

if (todosAsync.hasError) {
  return Text('Error: ${todosAsync.error}');
}

return TodosList(todosAsync.value!);
```

---

### كيف أحدث البيانات (Refresh)؟

**الجواب:**

**الطريقة الأولى - ref.invalidate:**
```dart
// Mark as invalid, rebuilds when read next time
ref.invalidate(todosProvider);
```

**الطريقة الثانية - ref.refresh:**
```dart
// Invalidate + read immediately
ref.refresh(todosProvider);
```

**الطريقة الثالثة - .future في RefreshIndicator:**
```dart
RefreshIndicator(
  onRefresh: () => ref.refresh(todosProvider.future),
  child: ListView(...),
)
```

---

### كيف أستخدم Family في Riverpod 3؟

**الجواب:**

في Riverpod 3 **لا تحتاج .family modifier!**

```dart
// ✅ Riverpod 3 - Just add parameters
@riverpod
Future<Product> product(ProductRef ref, String id) async {
  return await api.getProduct(id);
}

// Usage
final product = ref.watch(productProvider('123'));
```

**مع multiple parameters:**
```dart
@riverpod
Future<List<Product>> products(
  ProductsRef ref,
  String category,
  double minPrice,
  double maxPrice,
) async {
  return await api.getProducts(
    category: category,
    minPrice: minPrice,
    maxPrice: maxPrice,
  );
}

// Usage
final products = ref.watch(productsProvider('Electronics', 10.0, 100.0));
```

---

## 🐛 أسئلة عن الأخطاء

### لماذا أحصل على "ProviderNotFoundException"؟

**الجواب:**

**السبب:** ProviderScope غير موجود في widget tree.

**الحل:**
```dart
void main() {
  runApp(
    const ProviderScope(  // ✅ Wrap your app
      child: MyApp(),
    ),
  );
}
```

---

### لماذا أحصل على "Cannot use ref.watch inside an event handler"؟

**الجواب:**

**السبب:** استخدام ref.watch خارج build method.

```dart
// ❌ WRONG
void _onButtonPressed() {
  final counter = ref.watch(counterProvider);  // ❌ Error!
}

// ✅ CORRECT
void _onButtonPressed() {
  final counter = ref.read(counterProvider);  // ✅ Use ref.read
}
```

---

### لماذا الويدجت لا يُعاد بناؤه عند تغيير الحالة؟

**الجواب:**

**أسباب محتملة:**

1. **نسيت .notifier:**
```dart
// ❌ WRONG - This doesn't work
ref.read(counterProvider).increment();

// ✅ CORRECT
ref.read(counterProvider.notifier).increment();
```

2. **استخدمت ref.read بدلاً من ref.watch:**
```dart
// ❌ WRONG - Doesn't listen
final counter = ref.read(counterProvider);

// ✅ CORRECT
final counter = ref.watch(counterProvider);
```

3. **لم تُحدث state في Notifier:**
```dart
// ❌ WRONG
void increment() {
  _count++;  // Local variable, not state!
}

// ✅ CORRECT
void increment() {
  state++;  // Updates state
}
```

---

### لماذا أحصل على "Bad state: No ProviderScope found"؟

**الجواب:**

**السبب:** الاختبار لا يحتوي على ProviderScope.

```dart
// ❌ WRONG
testWidgets('test', (tester) async {
  await tester.pumpWidget(MyWidget());
});

// ✅ CORRECT
testWidgets('test', (tester) async {
  await tester.pumpWidget(
    const ProviderScope(
      child: MyWidget(),
    ),
  );
});
```

---

## 🎯 أسئلة عن الأداء

### كيف أحسّن الأداء؟

**الجواب:**

**1. استخدم .select للقراءة الجزئية:**
```dart
// ❌ Rebuilds on any user change
final user = ref.watch(userProvider);
return Text(user.name);

// ✅ Rebuilds only when name changes
final name = ref.watch(userProvider.select((u) => u.name));
return Text(name);
```

**2. استخدم AutoDispose:**
```dart
@riverpod  // Default AutoDispose
Future<Data> data(DataRef ref) async { ... }
```

**3. Batch Updates:**
```dart
// ✅ GOOD - Single update
state = state.copyWith(name: 'Ahmed', age: 25);

// ❌ BAD - Multiple updates
state = state.copyWith(name: 'Ahmed');
state = state.copyWith(age: 25);
```

---

### هل Riverpod أسرع من Provider/Bloc/GetX؟

**الجواب:**

**نعم، في معظم الحالات:**
- Rebuild optimization أفضل
- Dependency tracking أكثر دقة
- Less boilerplate = less code to run

**لكن الفرق عملياً:**
- في التطبيقات الصغيرة: لا يوجد فرق ملحوظ
- في التطبيقات الكبيرة: Riverpod أسرع قليلاً

**الأهم:** الـ API أفضل والكود أنظف!

---

## 🧪 أسئلة عن الاختبارات

### كيف أختبر provider يعتمد على provider آخر؟

**الجواب:**

استخدم **overrides** لتوفير mock للـ dependency:

```dart
test('loads user orders', () async {
  final mockApi = MockApi();
  when(() => mockApi.getOrders(any())).thenAnswer(
    (_) async => [Order(id: '1')],
  );

  final container = ProviderContainer.test(
    overrides: [
      apiProvider.overrideWithValue(mockApi),  // Mock dependency
    ],
  );

  final orders = await container.read(userOrdersProvider.future);
  expect(orders.length, 1);
});
```

---

### هل يجب أن أختبر كل provider؟

**الجواب:**

**لا!** اختبر فقط:

1. **Business logic** في Notifiers
2. **Computed values** (derived providers)
3. **Critical flows** (auth، checkout، etc.)

**لا تختبر:**
- Simple getters
- Direct API wrappers بدون logic
- UI-only providers (theme، etc.)

---

### كيف أختبر AsyncNotifier؟

**الجواب:**

```dart
test('adds todo', () async {
  final mockApi = MockTodosApi();

  // Setup initial state
  when(() => mockApi.getTodos()).thenAnswer((_) async => []);
  when(() => mockApi.addTodo(any())).thenAnswer((_) async {});

  final container = ProviderContainer.test(
    overrides: [todosApiProvider.overrideWithValue(mockApi)],
  );

  // Wait for initial load
  await container.read(todosProvider.future);

  // Setup new state after add
  when(() => mockApi.getTodos()).thenAnswer((_) async => [
    Todo(id: '1', title: 'New Todo'),
  ]);

  // Perform action
  await container.read(todosProvider.notifier).addTodo('New Todo');

  // Verify
  final todos = await container.read(todosProvider.future);
  expect(todos.length, 1);
  verify(() => mockApi.addTodo('New Todo')).called(1);
});
```

---

## 🏗️ أسئلة عن البنية

### ما هي أفضل بنية لتطبيق Riverpod؟

**الجواب:**

**Feature-based structure:**

```
lib/
├── features/
│   ├── auth/
│   │   ├── data/
│   │   │   ├── auth_repository.dart
│   │   │   └── auth_api.dart
│   │   ├── domain/
│   │   │   ├── user.dart
│   │   │   └── auth_service.dart
│   │   ├── presentation/
│   │   │   ├── screens/
│   │   │   └── widgets/
│   │   └── providers/
│   │       └── auth_providers.dart
│   └── products/
│       ├── data/
│       ├── domain/
│       ├── presentation/
│       └── providers/
├── core/
│   ├── network/
│   ├── errors/
│   └── constants/
└── shared/
    ├── widgets/
    └── utils/
```

---

### هل يجب استخدام Repository Pattern مع Riverpod؟

**الجواب:**

**يعتمد على حجم المشروع:**

**مشاريع صغيرة:**
```dart
// Direct API call in provider - OK!
@riverpod
Future<List<Product>> products(ProductsRef ref) async {
  return await api.getProducts();
}
```

**مشاريع متوسطة/كبيرة:**
```dart
// Use Repository for abstraction
@riverpod
ProductsRepository productsRepository(ProductsRepositoryRef ref) {
  return ProductsRepositoryImpl(ref.watch(apiProvider));
}

@riverpod
Future<List<Product>> products(ProductsRef ref) async {
  final repository = ref.watch(productsRepositoryProvider);
  return await repository.getProducts();
}
```

**المميزات:**
- سهولة الاختبار
- Separation of concerns
- يمكن تبديل data sources

---

### أين أضع الـ providers؟

**الجواب:**

**الخيار الأول - مع الـ feature:**
```
features/auth/providers/auth_providers.dart
```

**الخيار الثاني - ملف منفصل لكل provider:**
```
features/auth/
├── providers/
│   ├── auth_provider.dart
│   ├── user_provider.dart
│   └── login_provider.dart
```

**الخيار الثالث - حسب الطبقة:**
```
features/auth/
├── data/
│   └── auth_repository.dart  // Repository provider here
├── domain/
│   └── auth_service.dart     // Service provider here
└── presentation/
    └── auth_controller.dart   // Controller provider here
```

**اختر ما يناسب فريقك وحجم المشروع!**

---

## 💡 نصائح عامة

### ما أفضل طريقة لتعلم Riverpod؟

**الجواب:**

1. **ابدأ بالأساسيات** - Counter app مع @riverpod
2. **اقرأ التوثيق الرسمي** - riverpod.dev
3. **اصنع تطبيق Todo بسيط** - CRUD operations
4. **أضف async operations** - API calls
5. **تعلم الاختبارات** - Unit & widget tests
6. **اصنع تطبيق حقيقي** - E-commerce أو social media

---

### هل يمكن استخدام Riverpod مع packages أخرى؟

**الجواب:**

**نعم! Riverpod يعمل مع:**

- ✅ GoRouter - للـ navigation
- ✅ Dio - للـ HTTP requests
- ✅ Hive/Isar - للـ local storage
- ✅ Firebase - للـ backend
- ✅ Freezed - للـ data classes
- ✅ Any other package!

**مثال مع GoRouter:**
```dart
@riverpod
GoRouter router(RouterRef ref) {
  final authState = ref.watch(authProvider);

  return GoRouter(
    redirect: (context, state) {
      if (authState.isUnauthenticated) {
        return '/login';
      }
      return null;
    },
    routes: [...],
  );
}
```

---

### متى لا يجب استخدام Riverpod؟

**الجواب:**

**Riverpod مناسب لـ 99% من الحالات!**

**لكن قد تفضل غيره عندما:**

- فريقك يستخدم Bloc بالفعل وسعيد به
- تطبيق قديم جداً مع Provider ولا وقت للـ migration
- تحتاج Redux DevTools specifically

**لكن بشكل عام: Riverpod خيار ممتاز للجميع!**

</div>
