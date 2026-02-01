<div dir="rtl">

# تثبيت وإعداد Riverpod

**المستوى**: 🟢 مبتدئ

## 📌 المفاهيم الأساسية

في الملف ده هنتكلم عن:
- إزاي تثبت Riverpod في مشروعك
- الـ packages المختلفة ومتى تستخدم كل واحد
- إعداد Code Generation (اختياري لكن موصى به)
- أول setup للتطبيق

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تثبت Riverpod في أي مشروع
- تختار الـ package المناسب
- تعد Code Generation
- تجهز تطبيقك لاستخدام Riverpod

---

## 📦 الـ Packages المتاحة

حل Riverpod عنده 3 packages رئيسية:

### 1. flutter_riverpod (الأساسي - مطلوب دائماً)

**الاستخدام:** كل مشروع Flutter

</div>

```yaml
dependencies:
  flutter_riverpod: ^2.5.0
```

<div dir="rtl">

**يحتوي على:**
- كل الـ core functionality
- Providers بكل أنواعهم
- ConsumerWidget
- ref object

**متى تستخدمه:** دايماً! ده الـ package الأساسي.

### 2. riverpod_annotation + riverpod_generator (Code Generation)

**الاستخدام:** موصى به بشدة لـ Riverpod 3

</div>

```yaml
dependencies:
  flutter_riverpod: ^2.5.0
  riverpod_annotation: ^2.3.0

dev_dependencies:
  riverpod_generator: ^2.4.0
  build_runner: ^2.4.0
  custom_lint: ^0.6.0
  riverpod_lint: ^2.3.0
```

<div dir="rtl">

**يحتوي على:**
- Annotations (@riverpod)
- Code generator
- Less boilerplate
- Better type safety

**متى تستخدمه:** في كل المشاريع الجديدة (Riverpod 3 style)

### 3. hooks_riverpod (Advanced - اختياري)

**الاستخدام:** لو عايز تستخدم Flutter Hooks

</div>

```yaml
dependencies:
  hooks_riverpod: ^2.5.0
  flutter_hooks: ^0.20.0
```

<div dir="rtl">

**يحتوي على:**
- كل حاجة في flutter_riverpod
- Integration مع Flutter Hooks
- HookConsumerWidget

**متى تستخدمه:** لو محتاج Flutter Hooks (advanced users)

---

## 🚀 Setup خطوة بخطوة

### الطريقة 1: Basic Setup (بدون Code Generation)

مناسبة للمبتدئين أو المشاريع الصغيرة جداً.

#### الخطوة 1: أضف الـ dependency

</div>

```yaml
# pubspec.yaml
name: my_app
description: My Riverpod app

environment:
  sdk: '>=3.0.0 <4.0.0'

dependencies:
  flutter:
    sdk: flutter
  flutter_riverpod: ^2.5.0
```

<div dir="rtl">

#### الخطوة 2: Install

</div>

```bash
flutter pub get
```

<div dir="rtl">

#### الخطوة 3: Wrap app مع ProviderScope

</div>

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

void main() {
  runApp(
    // Wrap entire app
    ProviderScope(
      child: MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Riverpod App',
      home: HomePage(),
    );
  }
}
```

<div dir="rtl">

**خلاص! Ready to use** ✅

### الطريقة 2: مع Code Generation (موصى به)

مناسبة لكل المشاريع - أقل boilerplate وأكثر type safety.

#### الخطوة 1: أضف Dependencies

</div>

```yaml
# pubspec.yaml
name: my_app
description: My Riverpod 3 app

environment:
  sdk: '>=3.0.0 <4.0.0'

dependencies:
  flutter:
    sdk: flutter

  # Riverpod
  flutter_riverpod: ^2.5.0
  riverpod_annotation: ^2.3.0

dev_dependencies:
  flutter_test:
    sdk: flutter

  # Code generation
  riverpod_generator: ^2.4.0
  build_runner: ^2.4.0

  # Linting (optional but recommended)
  custom_lint: ^0.6.0
  riverpod_lint: ^2.3.0
```

<div dir="rtl">

#### الخطوة 2: Install

</div>

```bash
flutter pub get
```

<div dir="rtl">

#### الخطوة 3: إعداد analysis_options.yaml (اختياري)

</div>

```yaml
# analysis_options.yaml
include: package:flutter_lints/flutter.yaml

analyzer:
  plugins:
    - custom_lint

linter:
  rules:
    # Recommended rules
    avoid_print: true
    prefer_const_constructors: true
    prefer_const_literals_to_create_immutables: true
```

<div dir="rtl">

#### الخطوة 4: Setup main.dart

</div>

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

void main() {
  runApp(
    ProviderScope(
      child: MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Riverpod 3 App',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: HomePage(),
    );
  }
}

class HomePage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Riverpod App'),
      ),
      body: Center(
        child: Text('Ready to go!'),
      ),
    );
  }
}
```

<div dir="rtl">

#### الخطوة 5: أول Provider باستخدام Code Generation

</div>

```dart
// lib/providers/counter_provider.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

// This is required for code generation
part 'counter_provider.g.dart';

@riverpod
class Counter extends _$Counter {
  @override
  int build() {
    return 0; // Initial value
  }

  void increment() {
    state++;
  }

  void decrement() {
    state--;
  }
}
```

<div dir="rtl">

#### الخطوة 6: Generate Code

</div>

```bash
# One-time generation
flutter pub run build_runner build

# Watch mode (recommended during development)
flutter pub run build_runner watch

# Clean and rebuild
flutter pub run build_runner build --delete-conflicting-outputs
```

<div dir="rtl">

**الآن ready للاستخدام!** ✅

---

## 🔧 Build Runner Commands

خليني أفصلهالك:

### أمر 1: build (بناء لمرة واحدة)

</div>

```bash
flutter pub run build_runner build
```

<div dir="rtl">

**متى تستخدمه:**
- أول مرة بعد إضافة providers جديدة
- قبل الـ commit في git
- قبل الـ production build

### أمر 2: watch (المراقبة المستمرة)

</div>

```bash
flutter pub run build_runner watch
```

<div dir="rtl">

**متى تستخدمه:**
- أثناء الـ development
- بيراقب التغييرات ويعمل generate تلقائياً
- أفضل تجربة developer

**نصيحة:** شغله في terminal منفصل وخليه شغال طول الوقت.

### أمر 3: build مع delete-conflicting-outputs

</div>

```bash
flutter pub run build_runner build --delete-conflicting-outputs
```

<div dir="rtl">

**متى تستخدمه:**
- لو حصلت conflicts في الـ generated files
- بعد تغييرات كبيرة في الكود
- لما build العادي يفشل

---

## 📁 هيكل المشروع الموصى به

</div>

```
my_app/
├── lib/
│   ├── main.dart
│   ├── app.dart
│   │
│   ├── features/
│   │   ├── auth/
│   │   │   ├── providers/
│   │   │   │   ├── auth_provider.dart
│   │   │   │   └── auth_provider.g.dart  # Generated
│   │   │   ├── screens/
│   │   │   │   └── login_page.dart
│   │   │   └── models/
│   │   │       └── user.dart
│   │   │
│   │   └── home/
│   │       ├── providers/
│   │       ├── screens/
│   │       └── widgets/
│   │
│   ├── shared/
│   │   ├── providers/
│   │   │   ├── theme_provider.dart
│   │   │   └── locale_provider.dart
│   │   └── widgets/
│   │
│   └── core/
│       ├── services/
│       ├── repositories/
│       └── models/
│
├── test/
│   └── providers/
│       └── auth_provider_test.dart
│
└── pubspec.yaml
```

<div dir="rtl">

---

## ⚙️ إعدادات إضافية (Optional)

### 1. Git: تجاهل Generated Files

</div>

```gitignore
# .gitignore

# Generated files
*.g.dart
*.freezed.dart

# Build
build/
.dart_tool/
```

<div dir="rtl">

**ملاحظة:** بعض الناس بيفضلوا يعملوا commit للـ generated files. الاختيار ليك.

**الموصى به:** متعملش commit عشان:
- بيتغيروا كتير
- Git diffs بتبقى كبيرة
- كل developer يقدر يعملهم generate

### 2: VS Code Extensions

</div>

```json
// .vscode/extensions.json
{
  "recommendations": [
    "Dart-Code.dart-code",
    "Dart-Code.flutter",
    "usernamehw.errorlens"
  ]
}
```

<div dir="rtl">

### 3. Analysis Options المتقدمة

</div>

```yaml
# analysis_options.yaml
include: package:flutter_lints/flutter.yaml

analyzer:
  plugins:
    - custom_lint

  errors:
    # Make riverpod_lint warnings into errors
    riverpod_unsynchronized_generator: error
    provider_dependencies: error

  exclude:
    - "**/*.g.dart"
    - "**/*.freezed.dart"

linter:
  rules:
    # Riverpod specific
    avoid_public_notifier_properties: true

    # General good practices
    prefer_const_constructors: true
    prefer_const_literals_to_create_immutables: true
    avoid_print: true
    sized_box_for_whitespace: true
```

<div dir="rtl">

---

## ✅ Verification (تأكد إن كل حاجة شغالة)

### اختبار 1: Counter App بسيط

</div>

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'main.g.dart';

@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() => state++;
}

void main() {
  runApp(
    ProviderScope(
      child: MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: CounterPage(),
    );
  }
}

class CounterPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);

    return Scaffold(
      appBar: AppBar(title: Text('Counter')),
      body: Center(
        child: Text(
          '$count',
          style: TextStyle(fontSize: 48),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => ref.read(counterProvider.notifier).increment(),
        child: Icon(Icons.add),
      ),
    );
  }
}
```

<div dir="rtl">

### اختبار 2: Run

</div>

```bash
# 1. Generate code
flutter pub run build_runner build

# 2. Run app
flutter run
```

<div dir="rtl">

**لو التطبيق اشتغل والـ counter بيزيد - Setup ناجح! ✅**

---

## ⚠️ مشاكل شائعة والحلول

### مشكلة 1: part directive مش موجود

</div>

```
Error: Part directive missing
```

<div dir="rtl">

**الحل:**

</div>

```dart
// Add this line at top of file
part 'filename.g.dart';

// Example
part 'counter_provider.g.dart';
```

<div dir="rtl">

### مشكلة 2: Generated file مش موجود

</div>

```
Error: main.g.dart not found
```

<div dir="rtl">

**الحل:**

</div>

```bash
# Run build runner
flutter pub run build_runner build
```

<div dir="rtl">

### مشكلة 3: Conflicting outputs

</div>

```
Error: Conflicting outputs
```

<div dir="rtl">

**الحل:**

</div>

```bash
flutter pub run build_runner build --delete-conflicting-outputs
```

<div dir="rtl">

### مشكلة 4: ProviderScope مش موجود

</div>

```
Error: ProviderScope not wrapped
```

<div dir="rtl">

**الحل:**

</div>

```dart
void main() {
  runApp(
    ProviderScope( // Don't forget this!
      child: MyApp(),
    ),
  );
}
```

<div dir="rtl">

---

## 📊 ملخص: أي Setup أختار؟

| السيناريو | Setup الموصى به |
|-----------|------------------|
| **مبتدئ تماماً** | Basic (بدون codegen) |
| **مشروع جديد** | مع Code Generation ✅ |
| **مشروع كبير** | مع Code Generation ✅ |
| **Solo developer** | مع Code Generation ✅ |
| **Team project** | مع Code Generation ✅ |
| **Prototype سريع** | Basic (بدون codegen) |

**التوصية العامة:** استخدم Code Generation إلا لو:
- بتتعلم Riverpod لأول مرة (ابدأ basic)
- Demo سريع جداً (< ساعة)

---

## 🔗 الخطوة الجاية

دلوقتي بعد ما Setup خلص، وقت:
- **أول Provider ليك** (الملف الجاي)
- **إزاي تقرأ Providers**
- **ProviderScope بالتفصيل**

---

## 📚 المصادر

- [Riverpod Installation](https://riverpod.dev/docs/introduction/getting_started)
- [Code Generation Setup](https://riverpod.dev/docs/concepts/about_code_generation)
- [Build Runner](https://pub.dev/packages/build_runner)

---

## ✅ Checklist

- [ ] ثبّت flutter_riverpod
- [ ] (Optional) ثبّت riverpod_generator + build_runner
- [ ] لفّيت الـ app بـ ProviderScope
- [ ] جربت Counter example
- [ ] كل حاجة شغالة

**جاهز لأول Provider؟** 🎯

</div>
