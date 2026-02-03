<div dir="rtl">

# قاموس المصطلحات (Glossary)

**المستوى**: 📚 مرجعي

## 🔤 المصطلحات الأساسية

### Provider
**البروفايدر**: كائن يحمل قطعة من الحالة (state) ويُمكِّن الويدجتات من الوصول إليها والاستماع للتغييرات.

```dart
@riverpod
int counter(CounterRef ref) => 0;
```

---

### Ref
**المرجع**: كائن يُستخدم للتفاعل مع البروفايدرات الأخرى.

- **ref.watch()**: للاستماع للتغييرات (في build فقط)
- **ref.read()**: للقراءة مرة واحدة (في methods)
- **ref.listen()**: للاستماع وتنفيذ side effects

---

### Notifier
**المُبلِّغ**: كلاس يحمل حالة قابلة للتعديل ويوفر methods للتعامل معها.

```dart
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() => state++;
}
```

---

### AsyncNotifier
**المُبلِّغ غير المتزامن**: نسخة من Notifier للحالات التي تحتاج عمليات async.

```dart
@riverpod
class Todos extends _$Todos {
  @override
  Future<List<Todo>> build() async {
    return await api.getTodos();
  }
}
```

---

### AsyncValue
**القيمة غير المتزامنة**: wrapper للبيانات الآتية من عمليات async، يحمل الحالة (loading/data/error).

```dart
AsyncValue<User> user = AsyncValue.loading();
AsyncValue<User> user = AsyncValue.data(User(...));
AsyncValue<User> user = AsyncValue.error(error, stackTrace);
```

---

### Family
**العائلة**: بروفايدر يأخذ parameters لإنشاء instances متعددة.

```dart
// Riverpod 3 - No .family modifier needed!
@riverpod
Future<Product> product(ProductRef ref, String id) async {
  return await api.getProduct(id);
}
```

---

### AutoDispose
**التخلص التلقائي**: آلية تتخلص من البروفايدر تلقائياً عندما لا يكون هناك مستمعين له.

```dart
@riverpod  // Default: AutoDispose
Future<Data> data(DataRef ref) async { ... }

@Riverpod(keepAlive: true)  // Disable AutoDispose
Future<Config> config(ConfigRef ref) async { ... }
```

---

### KeepAlive
**الإبقاء حياً**: تعطيل AutoDispose لإبقاء البروفايدر حياً دائماً.

```dart
// Option 1: Annotation
@Riverpod(keepAlive: true)
Future<Config> config(ConfigRef ref) async { ... }

// Option 2: ref.keepAlive()
@riverpod
Future<Data> data(DataRef ref) async {
  final link = ref.keepAlive();
  Timer(Duration(minutes: 5), link.close);
  return await api.getData();
}
```

---

### ProviderScope
**نطاق البروفايدر**: ويدجت يُغلِّف التطبيق ويُمكِّن استخدام البروفايدرات.

```dart
void main() {
  runApp(
    const ProviderScope(
      child: MyApp(),
    ),
  );
}
```

---

### ProviderContainer
**حاوية البروفايدر**: نسخة من ProviderScope للاستخدام خارج الويدجتات (مثل الاختبارات).

```dart
final container = ProviderContainer.test();
final value = container.read(counterProvider);
```

---

### ConsumerWidget
**ويدجت المستهلك**: StatelessWidget يمكنه الوصول إلى ref.

```dart
class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final counter = ref.watch(counterProvider);
    return Text('$counter');
  }
}
```

---

### ConsumerStatefulWidget
**ويدجت المستهلك ذو الحالة**: StatefulWidget يمكنه الوصول إلى ref.

```dart
class MyWidget extends ConsumerStatefulWidget {
  @override
  ConsumerState<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends ConsumerState<MyWidget> {
  @override
  Widget build(BuildContext context) {
    final counter = ref.watch(counterProvider);
    return Text('$counter');
  }
}
```

---

### Code Generation
**توليد الكود**: استخدام build_runner لتوليد كود البروفايدرات تلقائياً.

```bash
dart run build_runner watch
```

```dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'counter.g.dart';

@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() => state++;
}
```

---

### Select
**الاختيار**: تحسين الأداء بالاستماع لجزء محدد من الحالة فقط.

```dart
// Rebuilds only when name changes
final name = ref.watch(userProvider.select((user) => user.name));
```

---

### Invalidate
**الإبطال**: جعل البروفايدر غير صالح، سيُعاد بناؤه عند القراءة التالية.

```dart
ref.invalidate(todosProvider);  // Mark as invalid
```

---

### Refresh
**التحديث**: إبطال + قراءة فورية.

```dart
ref.refresh(todosProvider);  // Invalidate + read immediately
```

---

### Override
**التجاوز**: استبدال قيمة بروفايدر (للاختبارات غالباً).

```dart
ProviderScope(
  overrides: [
    counterProvider.overrideWith((ref) => 100),
  ],
  child: MyApp(),
)
```

---

### Dependency
**التبعية**: بروفايدر يعتمد على بروفايدر آخر.

```dart
@riverpod
Future<Weather> weather(WeatherRef ref) async {
  final city = ref.watch(cityProvider);  // Dependency
  return await api.getWeather(city);
}
```

---

### State
**الحالة**: البيانات التي يحملها البروفايدر.

```dart
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() {
    state++;  // Update state
  }
}
```

---

### Build Method
**دالة البناء**: الدالة التي تُحدد القيمة الأولية للبروفايدر.

```dart
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;  // Build method
}
```

---

### Lifecycle Hooks
**خطافات دورة الحياة**: callbacks تُستدعى في مراحل مختلفة من حياة البروفايدر.

```dart
@riverpod
Future<Data> data(DataRef ref) async {
  ref.onDispose(() {
    // Cleanup
  });

  ref.onCancel(() {
    // Called when last listener is removed
  });

  ref.onResume(() {
    // Called when a new listener is added after onCancel
  });

  return await api.getData();
}
```

---

### .notifier
**المُبلِّغ**: الوصول إلى methods الخاصة بالـ Notifier.

```dart
// Read state
final count = ref.watch(counterProvider);

// Call methods
ref.read(counterProvider.notifier).increment();
```

---

### .future
**المستقبل**: الوصول إلى Future من AsyncValue.

```dart
@riverpod
Future<List<Order>> userOrders(UserOrdersRef ref) async {
  final user = await ref.watch(userProvider.future);
  return await api.getUserOrders(user.id);
}
```

---

### Pattern Matching
**مطابقة الأنماط**: استخدام Dart 3 switch expressions مع AsyncValue.

```dart
return switch (asyncValue) {
  AsyncData(:final value) => Text('Hello ${value.name}'),
  AsyncError(:final error) => Text('Error: $error'),
  _ => const CircularProgressIndicator(),
};
```

---

### Repository Pattern
**نمط المستودع**: فصل منطق الوصول للبيانات عن منطق العمل.

```dart
abstract class ProductsRepository {
  Future<List<Product>> getProducts();
}

class ProductsRepositoryImpl implements ProductsRepository {
  @override
  Future<List<Product>> getProducts() async {
    return await api.getProducts();
  }
}
```

---

### Clean Architecture
**العمارة النظيفة**: تقسيم التطبيق إلى طبقات (Presentation, Domain, Data).

```
lib/
├── features/
│   └── products/
│       ├── data/         # Data layer
│       ├── domain/       # Business logic
│       └── presentation/ # UI
```

---

### Dependency Injection
**حقن التبعيات**: توفير التبعيات للكلاسات من الخارج (Riverpod يعمل كـ DI container).

```dart
@riverpod
ProductsRepository productsRepository(ProductsRepositoryRef ref) {
  final api = ref.watch(apiProvider);
  return ProductsRepositoryImpl(api);
}
```

---

## 🔗 مصطلحات إضافية

### Guard
استخدام `AsyncValue.guard()` للتعامل مع الأخطاء تلقائياً:

```dart
state = await AsyncValue.guard(() async {
  return await api.getData();
});
```

---

### whenData
تطبيق transformation على البيانات داخل AsyncValue:

```dart
final doubled = asyncValue.whenData((value) => value * 2);
```

---

### unwrapPrevious
الحصول على القيمة السابقة أثناء التحديث:

```dart
final previousValue = asyncValue.unwrapPrevious();
```

---

### isRefreshing
البروفايدر يُحدَّث مع الاحتفاظ بالبيانات القديمة:

```dart
if (asyncValue.isRefreshing && asyncValue.hasValue) {
  // Show loading indicator while keeping old data visible
}
```

---

### isReloading
البروفايدر يُعاد بناؤه بسبب تغيير dependency:

```dart
if (asyncValue.isReloading) {
  // Dependency changed, reloading...
}
```

</div>
