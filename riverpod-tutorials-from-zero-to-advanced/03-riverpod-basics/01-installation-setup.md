<div dir="rtl">

# تثبيت وإعداد Riverpod

**المستوى**: 🟢 مبتدئ

## 📌 المفاهيم الأساسية

في الملف ده هنتكلم عن:
- إزاي تثبت Riverpod في مشروعك
- الـ package الأساسي (flutter_riverpod)
- إعداد أول تطبيق Riverpod
- أدوات مساعدة (Linting)

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تثبت Riverpod في أي مشروع Flutter
- تعد الـ setup الأساسي
- تجهز تطبيقك لاستخدام Riverpod
- تستخدم أدوات المساعدة المتاحة

---

## 📦 Package المطلوب

في Section ده، هنستخدم **Classic Syntax** اللي بيعتمد على package واحد بس:

### flutter_riverpod (الأساسي)

ده Package الأساسي اللي بيحتوي على كل حاجة محتاجها:

**يحتوي على:**
- كل الـ Providers (Provider, StateProvider, FutureProvider, StreamProvider, NotifierProvider)
- ConsumerWidget & Consumer
- ref object (للوصول للـ providers)
- ProviderScope
- كل الـ core functionality

**ملحوظة:** في Section 06 هنتعلم طريقة تانية باستخدام Code Generation، لكن دلوقتي هنركز على الأساسيات.

---

## 🛠️ التثبيت - خطوة بخطوة

### الخطوة 1: افتح pubspec.yaml

في مشروع Flutter الموجود (أو اعمل مشروع جديد):

</div>

```bash
flutter create my_riverpod_app
cd my_riverpod_app
```

<div dir="rtl">

### الخطوة 2: ضيف flutter_riverpod

افتح ملف `pubspec.yaml` وضيف Riverpod:

</div>

```yaml
name: my_riverpod_app
description: Learning Riverpod

environment:
  sdk: '>=3.0.0 <4.0.0'

dependencies:
  flutter:
    sdk: flutter

  # Riverpod - State management
  flutter_riverpod: ^3.0.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^3.0.0

  # Optional but recommended - Riverpod linting
  custom_lint: ^0.6.0
  riverpod_lint: ^3.0.0
```

<div dir="rtl">

### الخطوة 3: حمّل الـ packages

</div>

```bash
flutter pub get
```

<div dir="rtl">

**ملحوظة:** لو حصل error في التثبيت، تأكد من:
- Flutter SDK محدّث (`flutter upgrade`)
- pubspec.yaml مكتوب صح (الـ indentation مهم!)
- الإنترنت متصل

---

## 🎬 أول Setup

بعد ما ثبتّ Riverpod، خلينا نعمل أول setup:

### الخطوة 1: افتح lib/main.dart

</div>

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

// Simple counter provider
final counterProvider = StateProvider<int>((ref) => 0);

void main() {
  runApp(
    // Wrap your app with ProviderScope
    // This is REQUIRED for Riverpod to work
    const ProviderScope(
      child: MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Riverpod Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
        useMaterial3: true,
      ),
      home: const HomePage(),
    );
  }
}

class HomePage extends ConsumerWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);

    return Scaffold(
      appBar: AppBar(
        title: const Text('Riverpod Counter'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text(
              'You have pushed the button this many times:',
              style: TextStyle(fontSize: 16),
            ),
            const SizedBox(height: 20),
            Text(
              '$count',
              style: const TextStyle(
                fontSize: 48,
                fontWeight: FontWeight.bold,
                color: Colors.blue,
              ),
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          ref.read(counterProvider.notifier).state++;
        },
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

<div dir="rtl">

### الخطوة 2: شغّل التطبيق

</div>

```bash
flutter run
```

<div dir="rtl">

**لو كل حاجة تمام،** هتشوف تطبيق counter بسيط شغال! 🎉

---

## 🔍 فهم الـ Setup

خليني أشرحلك كل جزء:

### 1. ProviderScope

</div>

```dart
void main() {
  runApp(
    const ProviderScope(  // ← REQUIRED!
      child: MyApp(),
    ),
  );
}
```

<div dir="rtl">

**إيه ده؟**
- `ProviderScope` هو الـ root container لكل الـ providers
- **لازم** يكون wrapper للـ app كله
- بدونه، الـ providers مش هتشتغل

**فين مكانه؟**
- في الـ `main()` function
- wrapper للـ `MaterialApp` أو `CupertinoApp`

**ممكن يكون أكتر من واحد؟**
- أيوه! ممكن تعمل nested ProviderScopes لـ testing أو لـ override providers
- بس في الأغلب، واحد كفاية

### 2. Provider Definition

</div>

```dart
final counterProvider = StateProvider<int>((ref) => 0);
```

<div dir="rtl">

**إيه ده؟**
- `StateProvider` للـ state البسيط اللي بيتغير
- `<int>` → نوع الـ state (عدد صحيح)
- `(ref) => 0` → القيمة الأولية (Initial value)

**فين مكانه؟**
- في الـ global scope (خارج أي class)
- أو في ملف منفصل في `lib/providers/`

### 3. ConsumerWidget

</div>

```dart
class HomePage extends ConsumerWidget {  // ← استخدم ConsumerWidget بدل StatelessWidget
  const HomePage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {  // ← ref parameter
    final count = ref.watch(counterProvider);  // ← قراءة الـ provider
    // ...
  }
}
```

<div dir="rtl">

**إيه ده؟**
- `ConsumerWidget` بدل `StatelessWidget`
- بيديك `ref` في الـ build method
- بيعمل rebuild تلقائي لما الـ provider يتغير

**البدائل:**
- `Consumer` widget → لو مش عايز تحول الـ class كله
- `ConsumerStatefulWidget` → لو محتاج StatefulWidget

### 4. ref.watch vs ref.read

</div>

```dart
// Watch: للقراءة + rebuild تلقائي
final count = ref.watch(counterProvider);

// Read: للقراءة/التعديل مرة واحدة (في event handlers)
ref.read(counterProvider.notifier).state++;
```

<div dir="rtl">

**الفرق:**
- **`watch`**: بيعمل rebuild للـ widget لما القيمة تتغير
- **`read`**: بيقرأ القيمة مرة واحدة بدون rebuild

**متى تستخدم إيه؟**
- في `build` method → `watch`
- في button handlers → `read`
- في `initState` أو callbacks → `read`

---

## 🔧 Optional Tools (موصى بها)

### 1. Riverpod Lint (تحذيرات مفيدة)

الـ `riverpod_lint` package بيديك warnings لو استخدمت Riverpod غلط.

**التثبيت:**

</div>

```yaml
dev_dependencies:
  custom_lint: ^0.6.0
  riverpod_lint: ^3.0.0
```

<div dir="rtl">

**الإعداد:** اعمل ملف `analysis_options.yaml`:

</div>

```yaml
analyzer:
  plugins:
    - custom_lint
```

<div dir="rtl">

**أمثلة للـ warnings:**
- استخدمت `ref.watch` في event handler ← warning (استخدم `read`)
- استخدمت `ref.read` في `build` ← warning (استخدم `watch`)
- نسيت `ProviderScope` ← error

### 2. VS Code Extensions (اختياري)

**Flutter Riverpod Snippets:**
- Shortcuts للكود المتكرر
- مثال: `riverpod` → يعمل provider template

**كيفية التثبيت:**
1. افتح VS Code
2. Extensions (Ctrl+Shift+X)
3. ابحث عن "Flutter Riverpod Snippets"
4. Install

---

## 📁 تنظيم المشروع

للمشاريع الكبيرة، نظم الكود كده:

</div>

```
lib/
├── main.dart
├── providers/
│   ├── auth_provider.dart
│   ├── user_provider.dart
│   └── todos_provider.dart
├── models/
│   ├── user.dart
│   └── todo.dart
├── screens/
│   ├── home_screen.dart
│   ├── profile_screen.dart
│   └── todos_screen.dart
└── widgets/
    ├── todo_item.dart
    └── user_card.dart
```

<div dir="rtl">

**مثال:** ملف `providers/auth_provider.dart`

</div>

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../models/user.dart';

// Auth token
final authTokenProvider = StateProvider<String?>((ref) => null);

// Current user (depends on authToken)
final currentUserProvider = FutureProvider<User?>((ref) async {
  final token = ref.watch(authTokenProvider);

  if (token == null) {
    return null;
  }

  // Fetch user with token
  return await api.getUserWithToken(token);
});

// Is user logged in?
final isLoggedInProvider = Provider<bool>((ref) {
  final token = ref.watch(authTokenProvider);
  return token != null;
});
```

<div dir="rtl">

**استخدامه:**

</div>

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../providers/auth_provider.dart';

class ProfileScreen extends ConsumerWidget {
  const ProfileScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(currentUserProvider);

    return userAsync.when(
      data: (user) {
        if (user == null) {
          return const Text('Please login');
        }
        return Text('Hello ${user.name}');
      },
      loading: () => const CircularProgressIndicator(),
      error: (error, stack) => Text('Error: $error'),
    );
  }
}
```

<div dir="rtl">

---

## ⚠️ مشاكل شائعة والحلول

### مشكلة 1: "Could not find the correct Provider"

**السبب:** نسيت تحط `ProviderScope` في الـ main

**الحل:**

</div>

```dart
void main() {
  runApp(
    const ProviderScope(  // Don't forget this!
      child: MyApp(),
    ),
  );
}
```

<div dir="rtl">

### مشكلة 2: "The argument type 'WidgetRef' can't be assigned"

**السبب:** استخدمت `StatelessWidget` بدل `ConsumerWidget`

**الحل:**

</div>

```dart
// ✅ Correct
class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // ref is available here
  }
}

// ❌ Wrong
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // No ref!
  }
}
```

<div dir="rtl">

### مشكلة 3: "Version solving failed"

**السبب:** conflict في الـ versions

**الحل:**

</div>

```bash
# Clean the project
flutter clean

# Remove pubspec.lock
rm pubspec.lock

# Get packages again
flutter pub get
```

<div dir="rtl">

### مشكلة 4: UI مش بتتحدث

**السبب:** استخدمت `ref.read` بدل `ref.watch` في `build`

**الحل:**

</div>

```dart
@override
Widget build(BuildContext context, WidgetRef ref) {
  // ✅ Correct: rebuilds when value changes
  final count = ref.watch(counterProvider);

  // ❌ Wrong: doesn't rebuild
  final count = ref.read(counterProvider);

  return Text('$count');
}
```

<div dir="rtl">

---

## 💡 Best Practices

### 1. استخدم final للـ providers

</div>

```dart
// ✅ GOOD: final
final counterProvider = StateProvider<int>((ref) => 0);

// ❌ BAD: var or no modifier
var counterProvider = StateProvider<int>((ref) => 0);
```

<div dir="rtl">

### 2. Naming convention واضح

</div>

```dart
// ✅ GOOD: descriptive with 'Provider' suffix
final currentUserProvider = FutureProvider<User>(...)
final authTokenProvider = StateProvider<String?>(...)
final todosListProvider = NotifierProvider<TodosNotifier, List<Todo>>(...)

// ❌ BAD: unclear names
final user = FutureProvider<User>(...)
final token = StateProvider<String?>(...)
final data = NotifierProvider<TodosNotifier, List<Todo>>(...)
```

<div dir="rtl">

### 3. نظم الـ providers في ملفات منفصلة

</div>

```dart
// ✅ GOOD: organized
lib/
  providers/
    auth_provider.dart
    user_provider.dart
    todos_provider.dart

// ❌ BAD: everything in main.dart
lib/
  main.dart  (2000 lines!)
```

<div dir="rtl">

### 4. استخدم const constructors

</div>

```dart
// ✅ GOOD: const
const ProviderScope(child: MyApp())
const ConsumerWidget(...)

// ❌ BAD: missing const
ProviderScope(child: MyApp())
ConsumerWidget(...)
```

<div dir="rtl">

---

## 📝 ملخص

**خطوات التثبيت:**
1. ضيف `flutter_riverpod: ^3.0.0` في pubspec.yaml
2. نفذ `flutter pub get`
3. لف الـ app بـ `ProviderScope` في `main()`
4. استخدم `ConsumerWidget` للوصول للـ providers
5. استخدم `ref.watch` في build و `ref.read` في events

**المكونات الأساسية:**
- **ProviderScope**: الـ root container (required)
- **ConsumerWidget**: للوصول لـ `ref`
- **ref.watch()**: قراءة + rebuild
- **ref.read()**: قراءة مرة واحدة

**Optional tools:**
- `riverpod_lint`: تحذيرات مفيدة
- VS Code extensions: snippets

**Best practices:**
- استخدم `final` للـ providers
- naming convention واضح
- نظم في ملفات منفصلة
- استخدم `const` constructors

---

## 🔗 الخطوة الجاية

دلوقتي بعد ما ثبتّ Riverpod وعملت أول setup:
- روح على `02-first-provider.md` عشان تتعلم إزاي تعمل providers

**ملحوظة:** في Section 06 هنتعلم **Code Generation** - طريقة أحدث وأسهل للكتابة. لكن دلوقتي هنتعلم الأساسيات بالـ Classic Syntax الأول.

---

## 📚 المصادر

- [Riverpod Official Documentation](https://riverpod.dev)
- [Getting Started Guide](https://riverpod.dev/docs/introduction/getting_started)
- [flutter_riverpod package](https://pub.dev/packages/flutter_riverpod)
- [riverpod_lint package](https://pub.dev/packages/riverpod_lint)

---

## ✅ Checklist

قبل ما تكمل، تأكد من:

- [ ] ثبّت `flutter_riverpod`
- [ ] لفيت الـ app بـ `ProviderScope`
- [ ] جربت Counter example وشغال
- [ ] (Optional) ثبّت `riverpod_lint`
- [ ] فهمت الفرق بين `watch` و `read`

</div>
