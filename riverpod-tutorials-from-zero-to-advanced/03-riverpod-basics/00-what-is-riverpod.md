<div dir="rtl">

# إيه هو Riverpod؟

**المستوى**: 🟢 مبتدئ

## 📌 المفاهيم الأساسية

في الملف ده هنتكلم عن:
- تعريف Riverpod الدقيق
- الفلسفة وراء Riverpod
- المكونات الأساسية
- الفرق بين Riverpod والحلول التانية

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تفهم إيه هو Riverpod بالظبط
- تعرف الفلسفة الأساسية
- تتعرف على المكونات الرئيسية
- تفهم ليه Riverpod مختلف

---

## 🎭 التعريف البسيط

حل Riverpod هو **reactive caching and data-binding framework** لـ Flutter.

خليني أبسطهالك:

</div>

```dart
// Riverpod = مدير ذكي للـ state في تطبيقك

// Instead of:
int counter = 0; // Where to put this?
                 // How to share it?
                 // When to update UI?

// With Riverpod:
final counterProvider = StateProvider<int>((ref) => 0);

// Now:
// ✅ Counter موجود في مكان واحد
// ✅ أي Widget يقدر يقراه
// ✅ UI تتحدث تلقائياً لما يتغير
// ✅ Type-safe و compile-time checked
```

<div dir="rtl">

---

## 📖 التعريف التفصيلي

حل Riverpod هو framework لإدارة الحالة (State Management) بيوفر:

### 1. Reactive Programming

البيانات بتتحدث تلقائياً لما الـ dependencies تتغير:

</div>

```dart
// Provider يعتمد على provider تاني
final userProvider = StateProvider<User?>((ref) => null);

final greetingProvider = Provider<String>((ref) {
  final user = ref.watch(userProvider);

  // لما user يتغير، greeting يتحدث تلقائياً!
  return user != null ? 'Hello ${user.name}' : 'Hello Guest';
});

// The greeting updates automatically when user changes!
```

<div dir="rtl">

### 2. Dependency Injection

حل Riverpod بيوفر DI نظيف وسهل:

</div>

```dart
// Define dependencies
final databaseProvider = Provider<Database>((ref) {
  return DatabaseImpl();
});

final apiProvider = Provider<ApiService>((ref) {
  return ApiService();
});

// Use dependencies
final userRepositoryProvider = Provider<UserRepository>((ref) {
  final database = ref.watch(databaseProvider);
  final api = ref.watch(apiProvider);

  return UserRepository(database, api);
});

// Riverpod handles all the wiring!
```

<div dir="rtl">

### 3. Caching & Lifecycle Management

حل Riverpod بيدير الذاكرة والـ lifecycle تلقائياً:

</div>

```dart
// Auto-dispose when not used
final chatMessagesProvider = StreamProvider.autoDispose<List<Message>>((ref) {
  return chatService.messagesStream();
});

// When no widget watches this:
// ✅ Stream automatically cancelled
// ✅ Memory freed
// ✅ No memory leaks!
```

<div dir="rtl">

### 4. Compile-time Safety

كل الأخطاء بتتمسك وقت الكتابة، مش وقت التشغيل:

</div>

```dart
// ✅ This won't compile if provider doesn't exist
final user = ref.watch(userProvider);

// ✅ Type mismatch caught at compile time
final count = ref.watch(userProvider); // Error: User is not int

// ❌ Compare with Provider:
final user = context.watch<User>(); // Runtime error if not found!
```

<div dir="rtl">

---

## 🏗️ المكونات الأساسية

حل Riverpod عنده 3 مكونات رئيسية:

### 1. Provider (المزوّد)

ده المكان اللي بتحط فيه الـ State:

</div>

```dart
// Provider = Container للـ state
final nameProvider = Provider<String>((ref) {
  return 'Ahmed';
});

final counterProvider = StateProvider<int>((ref) {
  return 0;
});

final userProvider = FutureProvider<User>((ref) async {
  return await api.getUser();
});
```

<div dir="rtl">

**Provider زي:**
- Variable بس global و reactive
- Function بترجع قيمة
- Cache للبيانات
- Singleton لكن أذكى

### 2. Ref (المرجع)

ده اللي بتستخدمه عشان تقرأ Providers:

</div>

```dart
// Ref = Your connection to providers
final greetingProvider = Provider<String>((ref) {
  // ref.watch = read and rebuild on change
  final name = ref.watch(nameProvider);

  return 'Hello $name';
});

class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // ref.watch in widgets
    final greeting = ref.watch(greetingProvider);

    return Text(greeting);
  }
}
```

<div dir="rtl">

**Ref عنده:**
- `ref.watch()` - قراءة مع rebuild
- `ref.read()` - قراءة بدون rebuild
- `ref.listen()` - استماع للتغييرات

### 3. ProviderScope (النطاق)

ده الـ root اللي بيلف التطبيق:

</div>

```dart
void main() {
  runApp(
    ProviderScope( // Root of Riverpod tree
      child: MyApp(),
    ),
  );
}

// All providers available here and below
```

<div dir="rtl">

**ProviderScope زي:**
- Container لكل Providers
- Dependency injection container
- State management root

---

## 🎨 الفلسفة الأساسية

حل Riverpod مبني على 5 مبادئ:

### مبدأ 1: Explicitness (الوضوح)

كل حاجة واضحة ومكتوبة:

</div>

```dart
// ✅ Explicit: يمكنك رؤية كل dependencies
final profileProvider = Provider<Profile>((ref) {
  final user = ref.watch(userProvider);      // Dependency واضح
  final settings = ref.watch(settingsProvider); // Dependency واضح

  return Profile(user, settings);
});

// ❌ Implicit (في حلول تانية)
// Dependencies مخفية، صعب تتبعها
```

<div dir="rtl">

### مبدأ 2: Immutability (الثبات)

الـ State بيتبدل، مش بيتعدل:

</div>

```dart
// ✅ Immutable approach
final todosProvider = NotifierProvider<TodosNotifier, List<Todo>>(() {
  return TodosNotifier();
});

class TodosNotifier extends Notifier<List<Todo>> {
  @override
  List<Todo> build() {
    return []; // Initial state
  }

  void addTodo(Todo todo) {
    state = [...state, todo]; // Replace, don't mutate
  }

  void removeTodo(String id) {
    state = state.where((t) => t.id != id).toList(); // New list
  }
}
```

<div dir="rtl">

**ليه ده مهم:**
- مفيش side effects غير متوقعة
- Testing أسهل
- Time-travel debugging ممكن
- Predictable behavior

### مبدأ 3: Composability (التركيب)

ممكن تبني providers من providers تانية:

</div>

```dart
// Simple providers
final firstNameProvider = StateProvider<String>((ref) => 'Ahmed');
final lastNameProvider = StateProvider<String>((ref) => 'Mohamed');

// Composed provider
final fullNameProvider = Provider<String>((ref) {
  final firstName = ref.watch(firstNameProvider);
  final lastName = ref.watch(lastNameProvider);

  return '$firstName $lastName';
});

// Another composed provider
final greetingProvider = Provider<String>((ref) {
  final fullName = ref.watch(fullNameProvider);

  return 'Hello $fullName!';
});

// Composition = Building blocks
```

<div dir="rtl">

### مبدأ 4: Testability (قابلية الاختبار)

كل provider قابل للاختبار بسهولة:

</div>

```dart
// Testing is SUPER easy
test('full name combines first and last', () {
  final container = ProviderContainer(
    overrides: [
      firstNameProvider.overrideWith((ref) => 'Test'),
      lastNameProvider.overrideWith((ref) => 'User'),
    ],
  );

  expect(container.read(fullNameProvider), 'Test User');

  container.dispose();
});
```

<div dir="rtl">

### مبدأ 5: Performance (الأداء)

فقط الأجزاء المتأثرة بتعمل rebuild:

</div>

```dart
// User provider changed
final userProvider = StateProvider<User>((ref) => User());

// These rebuild
class UserName extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final user = ref.watch(userProvider);
    return Text(user.name); // Rebuilds when user changes
  }
}

// This doesn't rebuild
class StaticText extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Text('Static'); // Never rebuilds
  }
}

// Selective rebuilds = Better performance
```

<div dir="rtl">

---

## 🔄 إزاي Riverpod بيشتغل؟

خليني أوريك الـ flow:

### الخطوة 1: Define Provider

</div>

```dart
final counterProvider = StateProvider<int>((ref) => 0);
```

<div dir="rtl">

### الخطوة 2: Widget watches Provider

</div>

```dart
class CounterDisplay extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);
    return Text('Count: $count');
  }
}
```

<div dir="rtl">

### الخطوة 3: State Changes

</div>

```dart
// User clicks button
ElevatedButton(
  onPressed: () {
    ref.read(counterProvider.notifier).state++;
  },
  child: Text('Increment'),
);
```

<div dir="rtl">

### الخطوة 4: Riverpod Updates

</div>

```
User clicks button
      ↓
counterProvider.state changes (0 → 1)
      ↓
Riverpod notifies all listeners
      ↓
CounterDisplay widget rebuilds
      ↓
UI shows new value (1)
```

<div dir="rtl">

**السحر:** كل ده تلقائي!

---

## 🆚 الفرق بين Riverpod والحلول التانية

### مقارنة سريعة

| الميزة | Provider | BLoC | Riverpod |
|--------|----------|------|----------|
| **Compile-time safety** | ❌ Runtime | ❌ Runtime | ✅ Compile |
| **BuildContext** | ✅ مطلوب | ✅ مطلوب | ❌ اختياري |
| **Auto disposal** | ❌ يدوي | ❌ يدوي | ✅ تلقائي |
| **Boilerplate** | ⭐⭐⭐ | ⭐ كتير | ⭐⭐⭐⭐ قليل |
| **DI** | ⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Testing** | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |

### الفرق الجوهري

</div>

```dart
// Provider (Old way)
// ❌ Runtime checks
final user = context.watch<User>();

// ❌ Need BuildContext
void logout(BuildContext context) {
  context.read<AuthNotifier>().logout();
}

// ❌ Manual disposal
@override
void dispose() {
  subscription.cancel();
  super.dispose();
}

// Riverpod (New way)
// ✅ Compile-time checks
final user = ref.watch(userProvider);

// ✅ No BuildContext needed
void logout() {
  ref.read(authProvider.notifier).logout();
}

// ✅ Auto disposal
final chatProvider = StreamProvider.autoDispose<Chat>((ref) {
  return chatService.stream();
});
// Automatic cleanup when not watched!
```

<div dir="rtl">

---

## 🎯 امتى تستخدم Riverpod؟

### استخدمه لو:

</div>

```
✅ بتبدأ مشروع جديد
✅ محتاج compile-time safety
✅ عايز dependency injection نظيف
✅ محتاج testing سهل
✅ عايز auto disposal
✅ محتاج performance optimization
✅ التطبيق هيكبر (scalability)
```

<div dir="rtl">

### متستخدموش لو:

</div>

```
❌ Demo سريع جداً (< ساعة)
❌ مشروع قديم ضخم بـ BLoC وشغال كويس
❌ الفريق كله رافض يتعلم حاجة جديدة

في الحالات دي: ممكن setState أو استمر على الحل القديم
```

<div dir="rtl">

---

## 💡 مثال بسيط كامل

خليني أوريك مثال بسيط يوضح كل حاجة:

</div>

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

// 1. Define provider
final counterProvider = StateProvider<int>((ref) => 0);

// 2. Setup ProviderScope
void main() {
  runApp(
    ProviderScope(
      child: MyApp(),
    ),
  );
}

// 3. App
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: CounterPage(),
    );
  }
}

// 4. Use provider
class CounterPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);

    return Scaffold(
      appBar: AppBar(title: Text('Riverpod Counter')),
      body: Center(
        child: Text(
          '$count',
          style: TextStyle(fontSize: 48),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          ref.read(counterProvider.notifier).state++;
        },
        child: Icon(Icons.add),
      ),
    );
  }
}
```

<div dir="rtl">

**ده كل اللي محتاجه!** بسيط، نضيف، type-safe.

---

## 📊 ملخص: إيه هو Riverpod؟

</div>

```
Riverpod هو:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Framework لإدارة الحالة في Flutter
✅ Reactive - البيانات بتتحدث تلقائياً
✅ Type-safe - الأخطاء وقت الكتابة
✅ Testable - Testing سهل جداً
✅ Performant - Rebuilds محسّنة
✅ من مطور Provider - خبرة مثبتة

Riverpod يوفر:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Compile-time safety
✅ Dependency injection
✅ Auto disposal
✅ Caching
✅ Lifecycle management
✅ Testing utilities

الفلسفة:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
- Explicit > Implicit
- Immutable > Mutable
- Composable building blocks
- Testable by design
- Performance-focused
```

<div dir="rtl">

---

## 🔗 الخطوة الجاية

دلوقتي بعد ما فهمت إيه هو Riverpod، وقت:
- **Installation & Setup** (الملف الجاي)
- **أول Provider ليك**
- **إزاي تقرأ Providers**

---

## 📚 المصادر

- [Riverpod Official Website](https://riverpod.dev)
- [Why Riverpod?](https://riverpod.dev/docs/concepts/about)
- [Riverpod GitHub](https://github.com/rrousselGit/riverpod)

---

## ✅ تأكد إنك فهمت

- [ ] تعرف إيه هو Riverpod؟
- [ ] فاهم الفلسفة الأساسية؟
- [ ] تعرف المكونات الثلاثة (Provider, Ref, ProviderScope)؟
- [ ] فاهم الفرق بينه وبين الحلول التانية؟
- [ ] جاهز تبدأ تستخدمه؟

**يلا نبدأ Setup! 🚀**

</div>
