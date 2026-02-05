<div dir="rtl">

# الهجرة من Riverpod 2.x إلى Riverpod 3.0

**المستوى**: 🟡 متوسط - 🔴 متقدم

## 📌 نظرة عامة

Riverpod 3.0 جاء بتحسينات كبيرة وbreaking changes مهمة. هذا الدليل الشامل يساعدك على الهجرة بأمان وفهم كل التغييرات.

---

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تفهم كل الـ breaking changes في Riverpod 3.0
- تعرف تهاجر كودك القديم بأمان
- تتجنب الأخطاء الشائعة أثناء الهجرة
- تستخدم الـ features الجديدة بكفاءة

---

## 🚀 ما الجديد في Riverpod 3.0؟

### ✨ الميزات الجديدة:
- ✅ **ref.mounted** - Async safety check
- ✅ **Automatic retry** - إعادة محاولة تلقائية مع exponential backoff
- ✅ **Better error handling** - AsyncValue improvements
- ✅ **Enhanced DevTools** - Time-travel debugging
- ✅ **Improved testing** - ProviderContainer.test()
- ✅ **Generic type support** - Generic families مدعومة بالكامل

### ⚠️ Breaking Changes (7 تغييرات رئيسية):
1. StateNotifierProvider → Legacy
2. ref.state removed → use Notifier.state
3. ref.listenSelf removed → use Notifier.listenSelf
4. ref.future removed → use AsyncNotifier.future
5. AsyncValue.valueOrNull removed → use value/valueOrNull pattern
6. AutoDispose by default
7. ref methods throw after dispose

---

## 📋 ملخص سريع للتغييرات

| الميزة | Riverpod 2.x | Riverpod 3.0 | الحالة |
|--------|--------------|--------------|---------|
| StateNotifier | ✅ مدعوم | ⚠️ Legacy | مهمل |
| Notifier | ✅ مدعوم | ✅ مفضل | موصى به |
| ref.state | ✅ متاح | ❌ محذوف | غير متاح |
| Notifier.state | ❌ غير متاح | ✅ متاح | استخدم هذا |
| ref.listenSelf | ✅ متاح | ❌ محذوف | غير متاح |
| Notifier.listenSelf | ❌ غير متاح | ✅ متاح | استخدم هذا |
| ref.future | ✅ متاح | ❌ محذوف | غير متاح |
| AsyncNotifier.future | ❌ غير متاح | ✅ متاح | استخدم هذا |
| valueOrNull | ✅ property | ⚠️ pattern | تغير الاستخدام |
| AutoDispose | ⚠️ اختياري | ✅ افتراضي | تلقائي |
| ref after dispose | ⚠️ warning | ❌ throws | يرمي exception |

---

## 1️⃣ Breaking Change #1: StateNotifier → Notifier

### 📖 ما الذي تغير؟

**StateNotifierProvider** أصبح **legacy** (مهمل). يُفضل استخدام **NotifierProvider** و **Notifier**.

### 🤔 لماذا التغيير؟

- **Notifier** أبسط وأسهل في الاستخدام
- مدمج مباشرة مع code generation
- يدعم `ref.mounted` و الميزات الجديدة
- Performance أفضل

### ❌ قبل (Riverpod 2.x):

</div>

```dart
// ❌ Riverpod 2.x - StateNotifier (Legacy now)
import 'package:riverpod/riverpod.dart';

class Counter extends StateNotifier<int> {
  Counter() : super(0);

  void increment() => state++;
  void decrement() => state--;
}

final counterProvider = StateNotifierProvider<Counter, int>((ref) {
  return Counter();
});

// استخدام في Widget
class CounterWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);
    final counter = ref.read(counterProvider.notifier);

    return Column(
      children: [
        Text('Count: $count'),
        ElevatedButton(
          onPressed: counter.increment,
          child: Text('Increment'),
        ),
      ],
    );
  }
}
```

<div dir="rtl">

### ✅ بعد (Riverpod 3.0):

#### الطريقة 1: Classic Syntax

</div>

```dart
// ✅ Riverpod 3.0 - Notifier (Classic Syntax)
import 'package:riverpod/riverpod.dart';

class Counter extends Notifier<int> {
  @override
  int build() => 0;  // Initial state

  void increment() => state++;
  void decrement() => state--;
}

final counterProvider = NotifierProvider<Counter, int>(Counter.new);

// استخدام في Widget (نفس الطريقة!)
class CounterWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);
    final counter = ref.read(counterProvider.notifier);

    return Column(
      children: [
        Text('Count: $count'),
        ElevatedButton(
          onPressed: counter.increment,
          child: Text('Increment'),
        ),
      ],
    );
  }
}
```

<div dir="rtl">

#### الطريقة 2: Code Generation (مفضلة)

</div>

```dart
// ✅ Riverpod 3.0 - Code Generation (Recommended)
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'counter.g.dart';

@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;  // Initial state

  void increment() => state++;
  void decrement() => state--;
}

// استخدام في Widget
class CounterWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);
    // Notice: no .notifier needed with code generation!

    return Column(
      children: [
        Text('Count: $count'),
        ElevatedButton(
          onPressed: () => ref.read(counterProvider.notifier).increment(),
          child: Text('Increment'),
        ),
      ],
    );
  }
}
```

<div dir="rtl">

### 🔧 خطوات الهجرة:

1. **استبدل** `StateNotifier<T>` بـ `Notifier<T>`
2. **استبدل** `StateNotifierProvider<Notifier, State>` بـ `NotifierProvider<Notifier, State>`
3. **غير** constructor إلى `build()` method
4. **اختياري**: استخدم code generation للحصول على تجربة أفضل

### ⚠️ أخطاء شائعة:

</div>

```dart
// ❌ خطأ: نسيت تغيير constructor إلى build()
class Counter extends Notifier<int> {
  Counter() : super(0);  // ❌ Wrong! No constructor in Notifier

  void increment() => state++;
}

// ✅ صحيح: استخدم build() method
class Counter extends Notifier<int> {
  @override
  int build() => 0;  // ✅ Correct!

  void increment() => state++;
}

// ❌ خطأ: استخدام provider type خطأ
final counterProvider = StateNotifierProvider<Counter, int>(Counter.new);
//                       ↑ Wrong! This is legacy

// ✅ صحيح: استخدم NotifierProvider
final counterProvider = NotifierProvider<Counter, int>(Counter.new);
//                       ↑ Correct!
```

<div dir="rtl">

### 💡 نصيحة: استخدام Legacy Support

إذا كان عندك كود كبير ومش عايز تهاجر دلوقتي، ممكن تستخدم `riverpod_legacy`:

</div>

```dart
// ⚠️ مؤقت: استخدام legacy support
import 'package:riverpod_legacy/riverpod_legacy.dart';

// StateNotifier still works, but not recommended
class Counter extends StateNotifier<int> {
  Counter() : super(0);
  void increment() => state++;
}

final counterProvider = StateNotifierProvider<Counter, int>((ref) {
  return Counter();
});
```

<div dir="rtl">

**لكن**: يُفضل الهجرة الكاملة لـ Notifier في أقرب وقت!

---

## 2️⃣ Breaking Change #2: ref.state → Notifier.state

### 📖 ما الذي تغير؟

**ref.state** تم حذفه. استخدم `state` property مباشرة داخل الـ Notifier.

### 🤔 لماذا التغيير؟

- أبسط وأوضح
- يقلل من الـ confusion
- أفضل type safety

### ❌ قبل (Riverpod 2.x):

</div>

```dart
// ❌ Riverpod 2.x - باستخدام ref.state
class TodoList extends StateNotifier<List<Todo>> {
  TodoList() : super([]);

  void addTodo(Todo todo) {
    // باستخدام ref.state (في بعض الحالات)
    state = [...state, todo];
  }
}
```

<div dir="rtl">

### ✅ بعد (Riverpod 3.0):

</div>

```dart
// ✅ Riverpod 3.0 - استخدم state مباشرة
class TodoList extends Notifier<List<Todo>> {
  @override
  List<Todo> build() => [];

  void addTodo(Todo todo) {
    // استخدم state مباشرة (أبسط!)
    state = [...state, todo];
  }
}
```

<div dir="rtl">

**ملاحظة**: في Notifier، استخدم `state` property مباشرة (مش `ref.state`).

---

## 3️⃣ Breaking Change #3: ref.listenSelf → Notifier.listenSelf

### 📖 ما الذي تغير؟

**ref.listenSelf** تم حذفه. استخدم `listenSelf` method مباشرة من الـ Notifier.

### ❌ قبل (Riverpod 2.x):

</div>

```dart
// ❌ Riverpod 2.x - ref.listenSelf
class Counter extends StateNotifier<int> {
  Counter(this.ref) : super(0) {
    ref.listenSelf((previous, next) {
      print('Counter changed: $previous → $next');
    });
  }

  final Ref ref;
  void increment() => state++;
}
```

<div dir="rtl">

### ✅ بعد (Riverpod 3.0):

</div>

```dart
// ✅ Riverpod 3.0 - listenSelf method
class Counter extends Notifier<int> {
  @override
  int build() {
    // استخدم listenSelf مباشرة
    listenSelf((previous, next) {
      print('Counter changed: $previous → $next');
    });

    return 0;
  }

  void increment() => state++;
}
```

<div dir="rtl">

### 🔧 خطوات الهجرة:

1. **احذف** `ref.listenSelf`
2. **استخدم** `listenSelf` method مباشرة
3. **انقل** الـ listener logic لـ `build()` method

### 💡 مثال متقدم: مع side effects

</div>

```dart
// ✅ مثال: logging كل التغييرات
@riverpod
class TodoList extends _$TodoList {
  @override
  List<Todo> build() {
    // Log every state change
    listenSelf((previous, next) {
      if (previous != null) {
        print('Todos changed: ${previous.length} → ${next.length}');

        // Side effect: save to local storage
        if (next.length > previous.length) {
          _saveToLocalStorage(next);
        }
      }
    });

    return [];
  }

  void addTodo(Todo todo) {
    state = [...state, todo];
  }

  Future<void> _saveToLocalStorage(List<Todo> todos) async {
    await storage.save('todos', todos);
  }
}
```

<div dir="rtl">

---

## 4️⃣ Breaking Change #4: ref.future → AsyncNotifier.future

### 📖 ما الذي تغير؟

في **AsyncNotifier**، استخدم `future` property مباشرة (مش `ref.future`).

### ❌ قبل (Riverpod 2.x):

</div>

```dart
// ❌ Riverpod 2.x - ref.future (في بعض السياقات)
final userProvider = FutureProvider<User>((ref) async {
  final userId = ref.watch(userIdProvider);

  // الوصول للـ future
  // (كان معقد في بعض الحالات)
  return await api.getUser(userId);
});
```

<div dir="rtl">

### ✅ بعد (Riverpod 3.0):

</div>

```dart
// ✅ Riverpod 3.0 - استخدم future property مباشرة
@riverpod
class User extends _$User {
  @override
  Future<UserData> build(String userId) async {
    return await api.getUser(userId);
  }

  // الوصول للـ future مباشرة
  Future<void> refresh() async {
    state = const AsyncValue.loading();

    // استخدم future property
    state = await AsyncValue.guard(() => future);
  }
}
```

<div dir="rtl">

### 💡 مثال عملي: Cancel والrefresh

</div>

```dart
// ✅ مثال: إلغاء الـ request القديم
@riverpod
class SearchResults extends _$SearchResults {
  CancelToken? _cancelToken;

  @override
  Future<List<Result>> build(String query) async {
    // إلغاء الـ request السابق
    _cancelToken?.cancel();
    _cancelToken = CancelToken();

    ref.onDispose(() {
      _cancelToken?.cancel();
    });

    return await api.search(query, cancelToken: _cancelToken);
  }

  // Refresh باستخدام future
  Future<void> refresh() async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(future);
  }
}
```

<div dir="rtl">

---

## 5️⃣ Breaking Change #5: AsyncValue.valueOrNull Changes

### 📖 ما الذي تغير؟

**AsyncValue.valueOrNull** تغيرت طريقة استخدامها. الآن استخدم pattern matching أو properties.

### ❌ قبل (Riverpod 2.x):

</div>

```dart
// ❌ Riverpod 2.x - valueOrNull property
class UserWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider);

    // استخدام valueOrNull كـ property
    final user = userAsync.valueOrNull;

    if (user != null) {
      return Text(user.name);
    }

    return CircularProgressIndicator();
  }
}
```

<div dir="rtl">

### ✅ بعد (Riverpod 3.0):

#### الطريقة 1: استخدام when/map

</div>

```dart
// ✅ Riverpod 3.0 - استخدم when (موصى به)
class UserWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider);

    return userAsync.when(
      data: (user) => Text(user.name),
      loading: () => CircularProgressIndicator(),
      error: (error, stack) => Text('Error: $error'),
    );
  }
}
```

<div dir="rtl">

#### الطريقة 2: Pattern matching (Dart 3+)

</div>

```dart
// ✅ Riverpod 3.0 - Pattern matching
class UserWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider);

    return switch (userAsync) {
      AsyncData(:final value) => Text(value.name),
      AsyncLoading() => CircularProgressIndicator(),
      AsyncError(:final error) => Text('Error: $error'),
    };
  }
}
```

<div dir="rtl">

#### الطريقة 3: hasValue check

</div>

```dart
// ✅ Riverpod 3.0 - hasValue check
class UserWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider);

    // Check if value exists
    if (userAsync.hasValue) {
      final user = userAsync.value;  // Safe!
      return Text(user.name);
    }

    if (userAsync.isLoading) {
      return CircularProgressIndicator();
    }

    return Text('Error: ${userAsync.error}');
  }
}
```

<div dir="rtl">

### 🔧 خطوات الهجرة:

1. **استبدل** `valueOrNull` بـ `when()` method (الأفضل)
2. **أو استخدم** pattern matching (Dart 3+)
3. **أو استخدم** `hasValue` + `value`

### ⚠️ أخطاء شائعة:

</div>

```dart
// ❌ خطأ: استخدام value بدون check
final userAsync = ref.watch(userProvider);
final user = userAsync.value;  // ❌ قد يرمي exception!

// ✅ صحيح: استخدم when أو hasValue
// الطريقة 1: when
return userAsync.when(
  data: (user) => Text(user.name),
  loading: () => CircularProgressIndicator(),
  error: (error, _) => Text('Error'),
);

// الطريقة 2: hasValue check
if (userAsync.hasValue) {
  final user = userAsync.value;  // ✅ Safe!
  return Text(user.name);
}
```

<div dir="rtl">

---

## 6️⃣ Breaking Change #6: AutoDispose by Default

### 📖 ما الذي تغير؟

في **Riverpod 3.0**، الـ providers بتبقى **autoDispose** افتراضياً! (مش زي قبل)

### 🤔 لماذا التغيير؟

- **Memory safety**: تجنب memory leaks
- **Best practice**: معظم الـ providers يجب أن تكون autoDispose
- **Simpler API**: مفيش حاجة اسمها `.autoDispose` دلوقتي

### ❌ قبل (Riverpod 2.x):

</div>

```dart
// ❌ Riverpod 2.x - لازم تضيف .autoDispose يدوياً
final userProvider = FutureProvider.autoDispose<User>((ref) async {
  return await api.getUser();
});

// بدون autoDispose = يفضل في الذاكرة!
final settingsProvider = StateProvider<Settings>((ref) {
  return Settings();
});
```

<div dir="rtl">

### ✅ بعد (Riverpod 3.0):

</div>

```dart
// ✅ Riverpod 3.0 - AutoDispose افتراضياً!
@riverpod
Future<User> user(Ref ref) async {
  // AutoDispose automatically! No need to specify
  return await api.getUser();
}

// إذا عايز تمنع AutoDispose، استخدم keepAlive
@riverpod
class Settings extends _$Settings {
  @override
  SettingsData build() {
    // Keep this provider alive
    ref.keepAlive();
    return SettingsData();
  }
}
```

<div dir="rtl">

### 💡 متى تستخدم keepAlive؟

</div>

```dart
// ✅ استخدم keepAlive للـ global state
@riverpod
class AppSettings extends _$AppSettings {
  @override
  Settings build() {
    // Keep settings in memory (global state)
    ref.keepAlive();
    return Settings();
  }
}

// ✅ استخدم keepAlive للـ expensive computations
@riverpod
Future<LargeData> expensiveComputation(Ref ref) async {
  // Keep result cached
  ref.keepAlive();

  // This is expensive, keep it in memory
  return await computeExpensiveData();
}

// ✅ استخدم keepAlive مع شرط
@riverpod
class ShoppingCart extends _$ShoppingCart {
  @override
  List<Item> build() {
    // Keep alive only if cart is not empty
    ref.listenSelf((previous, next) {
      if (next.isEmpty) {
        ref.invalidateSelf();  // Can dispose now
      }
    });

    if (state.isNotEmpty) {
      ref.keepAlive();
    }

    return [];
  }
}
```

<div dir="rtl">

### 🔧 خطوات الهجرة:

1. **احذف** كل `.autoDispose` من الكود (تلقائياً دلوقتي!)
2. **أضف** `ref.keepAlive()` للـ providers اللي عايزها تفضل في الذاكرة
3. **راجع** كل provider وقرر: autoDispose (default) أو keepAlive؟

---

## 7️⃣ Breaking Change #7: ref Methods Throw After Dispose

### 📖 ما الذي تغير؟

في **Riverpod 3.0**، استخدام `ref` methods بعد dispose بيرمي **exception** (مش بس warning)!

### 🤔 لماذا التغيير؟

- **Safety**: يمنع race conditions
- **Explicit**: يجبرك تتعامل مع الحالة دي
- **Best practice**: مفروض مايكونش في ref access بعد dispose

### ❌ قبل (Riverpod 2.x):

</div>

```dart
// ❌ Riverpod 2.x - warning فقط
@riverpod
Future<User> user(Ref ref) async {
  final user = await api.getUser();

  // قد يحدث dispose هنا

  // ⚠️ Warning فقط (لكن الكود يشتغل)
  ref.read(cacheProvider).save(user);

  return user;
}
```

<div dir="rtl">

### ✅ بعد (Riverpod 3.0):

</div>

```dart
// ✅ Riverpod 3.0 - استخدم ref.mounted
@riverpod
Future<User> user(Ref ref) async {
  final user = await api.getUser();

  // ✅ Check if still mounted
  if (!ref.mounted) {
    return user;  // Provider disposed, exit early
  }

  // Safe to use ref now
  ref.read(cacheProvider).save(user);

  return user;
}
```

<div dir="rtl">

### 💡 مثال: Error handling مع ref.mounted

</div>

```dart
// ✅ مثال شامل: ref.mounted في error handling
@riverpod
class UserProfile extends _$UserProfile {
  @override
  Future<User> build() async {
    return await api.getUser();
  }

  Future<void> updateProfile(User updatedUser) async {
    state = const AsyncValue.loading();

    try {
      // محاولة التحديث
      await api.updateUser(updatedUser);

      // ✅ Check before continuing
      if (!ref.mounted) return;

      // جلب البيانات المحدثة
      final freshUser = await api.getUser();

      // ✅ Check again
      if (!ref.mounted) return;

      state = AsyncValue.data(freshUser);

    } catch (error, stackTrace) {
      // ✅ Check in catch too
      if (ref.mounted) {
        state = AsyncValue.error(error, stackTrace);
      }
    }
  }
}
```

<div dir="rtl">

### ⚠️ أخطاء شائعة:

</div>

```dart
// ❌ خطأ: نسيت ref.mounted check
@riverpod
Future<Data> data(Ref ref) async {
  final result = await api.getData();

  // ❌ قد يكون الـ provider اتعمله dispose!
  ref.read(cacheProvider).save(result);  // Exception!

  return result;
}

// ✅ صحيح: استخدم ref.mounted
@riverpod
Future<Data> data(Ref ref) async {
  final result = await api.getData();

  // ✅ Check first
  if (ref.mounted) {
    ref.read(cacheProvider).save(result);
  }

  return result;
}

// ❌ خطأ: ref.mounted بعد await بدون check
@riverpod
Future<void> complexOperation(Ref ref) async {
  await step1();
  ref.read(provider1);  // ❌ قد يرمي exception

  await step2();
  ref.read(provider2);  // ❌ قد يرمي exception

  await step3();
  ref.read(provider3);  // ❌ قد يرمي exception
}

// ✅ صحيح: check بعد كل await
@riverpod
Future<void> complexOperation(Ref ref) async {
  await step1();
  if (!ref.mounted) return;
  ref.read(provider1);  // ✅ Safe

  await step2();
  if (!ref.mounted) return;
  ref.read(provider2);  // ✅ Safe

  await step3();
  if (!ref.mounted) return;
  ref.read(provider3);  // ✅ Safe
}
```

<div dir="rtl">

---

## 🛠️ أدوات الهجرة التلقائية

### 1. Riverpod Lint (موصى به)

</div>

```yaml
# pubspec.yaml
dev_dependencies:
  custom_lint: ^0.5.0
  riverpod_lint: ^2.3.0

# analysis_options.yaml
analyzer:
  plugins:
    - custom_lint
```

<div dir="rtl">

**الميزات:**
- ✅ يكتشف استخدام legacy APIs
- ✅ يقترح fixes تلقائية
- ✅ يحذرك من أخطاء شائعة

### 2. Dart Fix

</div>

```bash
# تشغيل dart fix للحصول على اقتراحات
dart fix --dry-run

# تطبيق الـ fixes تلقائياً
dart fix --apply
```

<div dir="rtl">

### 3. Manual Search & Replace

</div>

```bash
# البحث عن StateNotifier usage
grep -r "StateNotifier" lib/

# البحث عن StateNotifierProvider
grep -r "StateNotifierProvider" lib/

# البحث عن ref.state
grep -r "ref\.state" lib/

# البحث عن ref.listenSelf
grep -r "ref\.listenSelf" lib/
```

<div dir="rtl">

---

## ✅ خطة الهجرة الموصى بها

### المرحلة 1: التحضير (يوم 1)

1. **Backup**: اعمل backup للكود
2. **Update dependencies**: حدث riverpod للإصدار 3.0
3. **Add lint rules**: ضيف riverpod_lint
4. **Run analysis**: شغل `dart analyze`

</div>

```bash
# Update to Riverpod 3.0
flutter pub upgrade riverpod

# Add riverpod_lint
flutter pub add --dev riverpod_lint custom_lint

# Run analysis
dart analyze
```

<div dir="rtl">

### المرحلة 2: الهجرة الأساسية (أيام 2-3)

1. **StateNotifier → Notifier**
   - ابحث عن كل `StateNotifier`
   - حولها لـ `Notifier`
   - غير `StateNotifierProvider` لـ `NotifierProvider`

2. **ref.state → state**
   - ابحث عن `ref.state`
   - استبدلها بـ `state`

3. **ref.listenSelf → listenSelf**
   - ابحث عن `ref.listenSelf`
   - انقلها لـ `build()` method

### المرحلة 3: التحسينات (أيام 4-5)

1. **أضف ref.mounted checks**
   - لكل async operation طويل
   - قبل استخدام ref بعد await

2. **راجع AutoDispose**
   - احذف `.autoDispose` (تلقائي دلوقتي)
   - أضف `keepAlive` حيث تحتاج

3. **حدث AsyncValue usage**
   - استخدم `when()` بدل `valueOrNull`
   - أو pattern matching

### المرحلة 4: الاختبار (يوم 6)

1. **شغل الـ tests**
2. **اختبر يدوياً**
3. **راجع الـ warnings**

---

## 🧪 Testing بعد الهجرة

### تأكد من الآتي:

</div>

```dart
// ✅ Test 1: Notifier works correctly
test('Counter increments correctly', () {
  final container = ProviderContainer();

  expect(container.read(counterProvider), 0);

  container.read(counterProvider.notifier).increment();

  expect(container.read(counterProvider), 1);

  container.dispose();
});

// ✅ Test 2: AutoDispose works
test('Provider disposes when not used', () async {
  final container = ProviderContainer();

  // Read provider
  container.read(userProvider);

  // Wait for dispose
  await Future.delayed(Duration.zero);

  // Provider should be disposed (if autoDispose)
  // Check internal state or side effects

  container.dispose();
});

// ✅ Test 3: ref.mounted prevents errors
test('ref.mounted prevents usage after dispose', () async {
  final container = ProviderContainer();

  // Start async operation
  final future = container.read(asyncProvider.future);

  // Dispose immediately
  container.dispose();

  // Should not throw (ref.mounted check prevents it)
  await expectLater(future, completes);
});
```

<div dir="rtl">

---

## 📋 Checklist النهائي

قبل ما تخلص الهجرة، تأكد من:

### Code Changes:
- [ ] كل `StateNotifier` اتحول لـ `Notifier`
- [ ] كل `StateNotifierProvider` اتحول لـ `NotifierProvider`
- [ ] كل `ref.state` اتحول لـ `state`
- [ ] كل `ref.listenSelf` اتحول لـ `listenSelf`
- [ ] كل `ref.future` اتحول لـ `future` property
- [ ] كل `valueOrNull` اتحول لـ `when()` أو pattern matching
- [ ] حذفت كل `.autoDispose` (تلقائي دلوقتي)
- [ ] أضفت `ref.keepAlive()` حيث مناسب
- [ ] أضفت `ref.mounted` checks قبل async operations

### Testing:
- [ ] كل الـ tests بتعدي
- [ ] اختبرت الـ app يدوياً
- [ ] مفيش memory leaks
- [ ] مفيش crashes

### Code Quality:
- [ ] `dart analyze` مفيش errors
- [ ] `dart fix` مفيش suggestions
- [ ] Lint rules بتعدي
- [ ] Code formatted

### Documentation:
- [ ] حدثت الـ comments
- [ ] حدثت الـ README
- [ ] وثقت الـ breaking changes
- [ ] شرحت الـ migration للفريق

---

## 🎓 الخلاصة

### Breaking Changes ملخص:

| # | التغيير | الحل |
|---|---------|------|
| 1 | StateNotifier → Legacy | استخدم Notifier |
| 2 | ref.state removed | استخدم state |
| 3 | ref.listenSelf removed | استخدم listenSelf |
| 4 | ref.future removed | استخدم future property |
| 5 | valueOrNull changed | استخدم when/pattern matching |
| 6 | AutoDispose default | استخدم keepAlive عند الحاجة |
| 7 | ref throws after dispose | استخدم ref.mounted |

### الميزات الجديدة في Riverpod 3.0:

- ✅ **ref.mounted** - Async safety
- ✅ **Automatic retry** - Exponential backoff
- ✅ **Better testing** - ProviderContainer.test()
- ✅ **Enhanced DevTools** - Time-travel debugging
- ✅ **Improved performance** - Better caching
- ✅ **Generic support** - Full generic families

### نصائح عامة:

1. **خذ وقتك**: الهجرة مش سهلة، خذ وقتك
2. **اعمل backup**: دايماً اعمل backup قبل أي تغيير كبير
3. **استخدم lint**: riverpod_lint يساعدك كتير
4. **اختبر كويس**: اختبر كل حاجة بعد الهجرة
5. **وثق التغييرات**: اكتب documentation للفريق

---

## 🔗 مصادر إضافية

### Official Documentation:
- [Riverpod 3.0 Migration Guide](https://riverpod.dev/docs/migration/3_0_0)
- [Riverpod 3.0 Changelog](https://github.com/rrousselGit/riverpod/releases/tag/riverpod-3.0.0)
- [Breaking Changes Announcement](https://riverpod.dev/blog/riverpod-3-0-0)

### Community Resources:
- [Reddit Discussion](https://www.reddit.com/r/FlutterDev/comments/riverpod_3)
- [Stack Overflow Tag](https://stackoverflow.com/questions/tagged/riverpod)
- [GitHub Discussions](https://github.com/rrousselGit/riverpod/discussions)

---

## 💪 Let's Go!

الهجرة لـ Riverpod 3.0 استثمار يستحق! الـ features الجديدة والـ improvements هتخلي كودك أفضل وأأمن. 🚀

**Good luck!** 🎉

</div>
