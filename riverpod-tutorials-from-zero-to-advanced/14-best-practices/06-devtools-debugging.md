<div dir="rtl">

# DevTools & Debugging - دليل التصحيح الاحترافي 🐛🔍

**المستوى**: 🟡 متوسط - 🔴 متقدم

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تستخدم Riverpod DevTools للتصحيح
- تتبع state changes في real-time
- تستخدم Time-travel debugging
- تحلل provider dependencies
- تحل المشاكل بسرعة وكفاءة

---

## 📌 ما هو Riverpod DevTools؟

**Riverpod DevTools** هو أداة debugging قوية مدمجة مع Flutter DevTools، بتساعدك:

- 👁️ **State Inspector** - شوف كل الـ providers والـ state بتاعهم
- 🕐 **Time-travel** - ارجع للـ state القديم
- 📊 **Dependency Graph** - شوف العلاقات بين الـ providers
- 🔔 **Live Updates** - شوف التغييرات live
- 📝 **State History** - تتبع كل التغييرات

---

## 🚀 Setup: تثبيت DevTools

### الخطوة 1: إضافة Package

</div>

```yaml
# pubspec.yaml
dependencies:
  flutter_riverpod: ^2.5.0  # or latest

dev_dependencies:
  # ✅ Add this for DevTools integration
  riverpod_devtools_tracker: ^0.1.0
```

<div dir="rtl">

### الخطوة 2: Initialize في الـ App

</div>

```dart
// main.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

// ✅ Import DevTools tracker
import 'package:riverpod_devtools_tracker/riverpod_devtools_tracker.dart';

void main() {
  runApp(
    ProviderScope(
      // ✅ Add observers for DevTools
      observers: [
        if (kDebugMode) // Only in debug mode
          RiverpodDevToolsTracker(),
      ],
      child: MyApp(),
    ),
  );
}
```

<div dir="rtl">

### الخطوة 3: تشغيل DevTools

</div>

```bash
# 1. تشغيل التطبيق
flutter run

# 2. فتح DevTools (في terminal آخر)
flutter pub global activate devtools
flutter pub global run devtools

# أو اضغط على الرابط في الـ console:
# http://127.0.0.1:9100/
```

<div dir="rtl">

---

## 🔍 State Inspector Tab

### استخدام State Inspector

**State Inspector** بيعرض كل الـ providers النشطة والـ state بتاعهم.

#### المميزات:

1. **Provider List** - كل الـ providers في التطبيق
2. **Current Value** - الـ state الحالي
3. **Dependencies** - الـ providers اللي يعتمد عليها
4. **Listeners** - عدد الـ widgets اللي بتستمع له

### مثال: مراقبة Counter

</div>

```dart
// counter_provider.dart
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() => state++;
  void decrement() => state--;
}

// في DevTools State Inspector، هتشوف:
//
// ┌─────────────────────────────────┐
// │ counterProvider                 │
// ├─────────────────────────────────┤
// │ Type: NotifierProvider          │
// │ State: 5                        │
// │ Listeners: 2                    │
// │ Dependencies: []                │
// └─────────────────────────────────┘
```

<div dir="rtl">

### مثال: Provider مع Dependencies

</div>

```dart
// user_provider.dart
@riverpod
Future<User> user(UserRef ref, String userId) async {
  return await api.getUser(userId);
}

// stats_provider.dart
@riverpod
Future<Stats> userStats(UserStatsRef ref, String userId) async {
  // ✅ This depends on user provider
  final user = await ref.watch(userProvider(userId).future);

  return await api.getStats(user.id);
}

// في DevTools State Inspector:
//
// ┌─────────────────────────────────┐
// │ userStatsProvider('user123')    │
// ├─────────────────────────────────┤
// │ Type: FutureProvider            │
// │ State: AsyncData(Stats(...))    │
// │ Listeners: 1                    │
// │ Dependencies:                   │
// │   ↳ userProvider('user123')     │
// └─────────────────────────────────┘
```

<div dir="rtl">

---

## ⏰ Time-Travel Debugging

**Time-travel debugging** يخليك ترجع للـ state القديم وتشوف إيه اللي حصل!

### كيفية الاستخدام:

1. افتح **State Inspector** في DevTools
2. اختار الـ provider اللي عايز تتبعه
3. هتلاقي **Timeline** tab
4. اضغط على أي نقطة في الـ timeline للرجوع لها

### مثال: Todo App Timeline

</div>

```dart
@riverpod
class TodoList extends _$TodoList {
  @override
  List<Todo> build() => [];

  void addTodo(String title) {
    state = [...state, Todo(id: DateTime.now().toString(), title: title)];
  }

  void removeTodo(String id) {
    state = state.where((todo) => todo.id != id).toList();
  }
}

// Timeline في DevTools:
//
// 10:30:00 AM | [] (empty)
//             ↓
// 10:30:05 AM | [Todo("Buy milk")]
//             ↓
// 10:30:10 AM | [Todo("Buy milk"), Todo("Read book")]
//             ↓
// 10:30:15 AM | [Todo("Buy milk")]  ← اضغط هنا للرجوع!
//             ↓
// 10:30:20 AM | [Todo("Buy milk"), Todo("Exercise")]
```

<div dir="rtl">

**استخدامات:**
- 🐛 **Bug Reproduction** - ارجع للحظة حدوث الـ bug
- 🔍 **State Investigation** - شوف الـ state كان إيه في وقت معين
- 📊 **Performance Analysis** - شوف كم مرة الـ state اتغير

---

## 📊 Dependency Graph Visualization

**Dependency Graph** بيعرض العلاقات بين الـ providers بصرياً.

### مثال: E-commerce Cart System

</div>

```dart
// products_provider.dart
@riverpod
Future<List<Product>> products(ProductsRef ref) async {
  return await api.getProducts();
}

// cart_provider.dart
@riverpod
class Cart extends _$Cart {
  @override
  List<CartItem> build() => [];

  void addItem(Product product) {
    state = [...state, CartItem(product)];
  }
}

// cart_total_provider.dart
@riverpod
double cartTotal(CartTotalRef ref) {
  final items = ref.watch(cartProvider);
  return items.fold(0.0, (sum, item) => sum + item.price);
}

// tax_provider.dart
@riverpod
double tax(TaxRef ref) {
  final subtotal = ref.watch(cartTotalProvider);
  return subtotal * 0.15;  // 15% tax
}

// final_total_provider.dart
@riverpod
double finalTotal(FinalTotalRef ref) {
  final subtotal = ref.watch(cartTotalProvider);
  final tax = ref.watch(taxProvider);
  return subtotal + tax;
}

// Dependency Graph في DevTools:
//
//     ┌─────────────┐
//     │  products   │
//     └─────────────┘
//
//     ┌─────────────┐
//     │    cart     │
//     └──────┬──────┘
//            │
//            ↓
//     ┌─────────────┐
//     │ cartTotal   │
//     └──────┬──────┘
//            │
//      ┌─────┴─────┐
//      ↓           ↓
// ┌─────────┐ ┌──────────┐
// │   tax   │ │finalTotal│
// └─────────┘ └──────────┘
```

<div dir="rtl">

---

## 🔔 Live State Monitoring

### مراقبة التغييرات في Real-Time

DevTools بيعرض الـ state changes live أثناء استخدام التطبيق.

#### مثال: Chat App

</div>

```dart
@riverpod
class ChatMessages extends _$ChatMessages {
  @override
  List<Message> build(String chatId) => [];

  void addMessage(Message message) {
    state = [...state, message];
    // ✅ هتشوف التغيير فوراً في DevTools!
  }
}

// في DevTools Live Monitor:
//
// 10:30:00 | chatMessages('chat1') = []
// 10:30:05 | chatMessages('chat1') = [Message("Hi!")]
// 10:30:10 | chatMessages('chat1') = [Message("Hi!"), Message("How are you?")]
//            ↑ بيتحدث تلقائياً!
```

<div dir="rtl">

---

## 🐛 Common Debugging Scenarios

### السيناريو 1: Provider لا يتحدث

**المشكلة:** Widget مش بيتحدث رغم تغيير الـ state

**التصحيح:**

</div>

```dart
// ❌ WRONG - Using ref.read
class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.read(counterProvider);  // ❌ No rebuild!

    return Text('$count');
  }
}

// في DevTools:
// - counterProvider.state = 5 (changed!)
// - MyWidget listeners = 0 ❌ (not watching!)

// ✅ CORRECT - Using ref.watch
class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);  // ✅ Rebuilds!

    return Text('$count');
  }
}

// في DevTools:
// - counterProvider.state = 5
// - MyWidget listeners = 1 ✅ (watching!)
```

<div dir="rtl">

### السيناريو 2: Infinite Loop

**المشكلة:** الـ provider بيتحدث بشكل لا نهائي

**التصحيح:**

</div>

```dart
// ❌ WRONG - Infinite loop
@riverpod
Future<Data> data(DataRef ref) async {
  final timestamp = DateTime.now();  // ❌ Always different!

  return await api.getData(timestamp);
}

// في DevTools Timeline:
// 10:30:00.000 | Loading...
// 10:30:00.100 | Data loaded
// 10:30:00.101 | Loading...  ← Loop starts!
// 10:30:00.201 | Data loaded
// 10:30:00.202 | Loading...
// ... (forever!)

// ✅ CORRECT - Cache timestamp
@riverpod
Future<Data> data(DataRef ref) async {
  // Use family parameter instead
  return await api.getData();
}
```

<div dir="rtl">

### السيناريو 3: Memory Leak

**المشكلة:** Providers مش بيتعملهم dispose

**التصحيح:**

</div>

```dart
// ❌ WRONG - keepAlive prevents disposal
@riverpod
Future<User> user(UserRef ref, String userId) async {
  ref.keepAlive();  // ❌ Never disposed!

  return await api.getUser(userId);
}

// في DevTools:
// Active Providers:
// - userProvider('user1')  ← Still alive!
// - userProvider('user2')  ← Still alive!
// - userProvider('user3')  ← Still alive!
// ... (100+ providers!) 💥 Memory leak!

// ✅ CORRECT - Let it auto-dispose
@riverpod
Future<User> user(UserRef ref, String userId) async {
  // No keepAlive - disposes automatically
  return await api.getUser(userId);
}

// في DevTools:
// Active Providers:
// - userProvider('current_user')  ← Only the one in use
```

<div dir="rtl">

---

## 📝 Custom Observers

يمكنك إضافة **custom observers** لتتبع الـ state changes.

### مثال: Logging Observer

</div>

```dart
// logging_observer.dart
class LoggingObserver extends ProviderObserver {
  @override
  void didAddProvider(
    ProviderBase provider,
    Object? value,
    ProviderContainer container,
  ) {
    print('✅ Provider added: ${provider.name ?? provider.runtimeType}');
  }

  @override
  void didUpdateProvider(
    ProviderBase provider,
    Object? previousValue,
    Object? newValue,
    ProviderContainer container,
  ) {
    print('🔄 Provider updated: ${provider.name ?? provider.runtimeType}');
    print('   Old: $previousValue');
    print('   New: $newValue');
  }

  @override
  void didDisposeProvider(
    ProviderBase provider,
    ProviderContainer container,
  ) {
    print('🗑️ Provider disposed: ${provider.name ?? provider.runtimeType}');
  }

  @override
  void providerDidFail(
    ProviderBase provider,
    Object error,
    StackTrace stackTrace,
    ProviderContainer container,
  ) {
    print('❌ Provider failed: ${provider.name ?? provider.runtimeType}');
    print('   Error: $error');
  }
}

// main.dart
void main() {
  runApp(
    ProviderScope(
      observers: [
        LoggingObserver(),  // ✅ Add custom observer
        if (kDebugMode) RiverpodDevToolsTracker(),
      ],
      child: MyApp(),
    ),
  );
}

// Console output:
// ✅ Provider added: counterProvider
// 🔄 Provider updated: counterProvider
//    Old: 0
//    New: 1
// 🔄 Provider updated: counterProvider
//    Old: 1
//    New: 2
```

<div dir="rtl">

### مثال: Analytics Observer

</div>

```dart
// analytics_observer.dart
class AnalyticsObserver extends ProviderObserver {
  final Analytics analytics;

  AnalyticsObserver(this.analytics);

  @override
  void didUpdateProvider(
    ProviderBase provider,
    Object? previousValue,
    Object? newValue,
    ProviderContainer container,
  ) {
    // Log to analytics
    analytics.logEvent(
      name: 'provider_updated',
      parameters: {
        'provider': provider.name ?? provider.runtimeType.toString(),
        'timestamp': DateTime.now().toIso8601String(),
      },
    );
  }

  @override
  void providerDidFail(
    ProviderBase provider,
    Object error,
    StackTrace stackTrace,
    ProviderContainer container,
  ) {
    // Log errors to crash reporting
    analytics.logError(
      error: error,
      stackTrace: stackTrace,
      context: {'provider': provider.name ?? provider.runtimeType.toString()},
    );
  }
}
```

<div dir="rtl">

---

## 🎯 Best Practices للـ Debugging

### 1. استخدم Provider Names

</div>

```dart
// ✅ GOOD - Named providers for easy debugging
@Riverpod(name: 'userProfile')
Future<User> user(UserRef ref, String userId) async {
  return await api.getUser(userId);
}

@Riverpod(name: 'userStats')
Future<Stats> stats(StatsRef ref, String userId) async {
  return await api.getStats(userId);
}

// في DevTools:
// - userProfile('user123')  ← واضح!
// - userStats('user123')    ← واضح!

// ❌ BAD - Generic names
@riverpod
Future<User> user(UserRef ref, String userId) async { ... }

@riverpod
Future<Stats> stats(StatsRef ref, String userId) async { ... }

// في DevTools:
// - userProvider('user123')   ← ممكن يتلخبط مع providers تانية
// - statsProvider('user123')  ← مش واضح stats إيه
```

<div dir="rtl">

### 2. أضف Logging في Development

</div>

```dart
// ✅ GOOD - Conditional logging
@riverpod
class TodoList extends _$TodoList {
  @override
  List<Todo> build() {
    if (kDebugMode) {
      print('[TodoList] Initialized');
    }
    return [];
  }

  void addTodo(String title) {
    if (kDebugMode) {
      print('[TodoList] Adding todo: $title');
    }

    state = [...state, Todo(title: title)];

    if (kDebugMode) {
      print('[TodoList] New state: ${state.length} todos');
    }
  }
}
```

<div dir="rtl">

### 3. استخدم Breakpoints بذكاء

</div>

```dart
// ✅ Set breakpoints at critical points
@riverpod
Future<Data> criticalData(CriticalDataRef ref) async {
  // 🔴 Breakpoint here - Before API call
  final result = await api.getCriticalData();

  // 🔴 Breakpoint here - After API call
  if (result == null) {
    // 🔴 Breakpoint here - Error handling
    throw Exception('No data');
  }

  // 🔴 Breakpoint here - Before return
  return result;
}
```

<div dir="rtl">

### 4. راقب الـ Performance

</div>

```dart
// ✅ Monitor expensive operations
@riverpod
Future<ProcessedData> expensiveOperation(ExpensiveOperationRef ref) async {
  final stopwatch = Stopwatch()..start();

  final data = await api.getData();
  final processed = _processData(data);  // Expensive!

  stopwatch.stop();

  if (kDebugMode && stopwatch.elapsedMilliseconds > 100) {
    print('⚠️ Slow operation: ${stopwatch.elapsedMilliseconds}ms');
  }

  return processed;
}
```

<div dir="rtl">

---

## 🔧 Debugging Tools Summary

| Tool | Use Case | Example |
|------|----------|---------|
| **State Inspector** | شوف current state | قيمة الـ counter دلوقتي |
| **Time-travel** | ارجع لـ state قديم | لحظة حدوث الـ bug |
| **Dependency Graph** | شوف provider relationships | مين يعتمد على مين |
| **Live Monitor** | شوف changes real-time | رسائل الـ chat |
| **Custom Observers** | تتبع مخصص | Logging, Analytics |
| **Breakpoints** | توقف عند نقطة | قبل API call |

---

## 🎓 الخلاصة

### DevTools في سطر واحد:
> **Riverpod DevTools = عيونك على كل حاجة في الـ state management!**

### الفوائد:
- ✅ تصحيح سريع وفعال
- ✅ فهم الـ provider dependencies
- ✅ تتبع state changes
- ✅ اكتشاف memory leaks
- ✅ تحليل الأداء

### متى تستخدمه:
- 🐛 عندك bug صعب
- 🔍 عايز تفهم كيف الـ app بيشتغل
- ⚡ عايز تحسن الأداء
- 📊 عايز تتبع الـ state changes

---

## 🔗 مصادر إضافية

### Official Documentation:
- [Riverpod DevTools | Riverpod](https://riverpod.dev/docs/devtools)
- [Flutter DevTools](https://docs.flutter.dev/tools/devtools)

### Community Resources:
- [Debugging Riverpod Apps](https://codewithandrea.com/articles/debugging-riverpod/)
- [Riverpod Best Practices](https://riverpod.dev/docs/concepts/best_practices)

---

## ✅ تأكد إنك فهمت

- [ ] عارف كيف تثبت DevTools؟
- [ ] تقدر تستخدم State Inspector؟
- [ ] تعرف تستخدم Time-travel debugging؟
- [ ] تقدر تقرأ Dependency Graph؟
- [ ] تعرف تضيف Custom Observers؟
- [ ] تقدر تصلح الـ bugs الشائعة؟

---

**🔍 DevTools = أداتك السحرية للتصحيح!**

استخدمها في كل مرة عندك مشكلة - هتوفر عليك وقت كتير! ⚡

</div>
