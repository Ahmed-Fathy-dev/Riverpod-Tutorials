<div dir="rtl">

# Setup Guide - دليل الإعداد الكامل

**المستوى**: 🟢 مبتدئ

## 📌 المفاهيم الأساسية

في الملف ده هنتعلم:
- إزاي نضيف الـ dependencies المطلوبة
- إعداد build_runner خطوة بخطوة
- كتابة أول provider بـ code generation
- تشغيل الـ code generator
- حل المشاكل الشائعة في الـ setup

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تضيف code generation لأي مشروع Flutter
- تعمل run للـ build_runner بشكل صحيح
- تكتب وتولد أول provider ليك
- تفهم الـ generated files

---

## 📦 Step 1: إضافة الـ Dependencies

### الـ Packages المطلوبة:

Code generation في Riverpod محتاج **4 packages:**

| Package | النوع | الاستخدام |
|---------|-------|----------|
| `flutter_riverpod` | Runtime | الـ Riverpod library نفسها |
| `riverpod_annotation` | Runtime | الـ annotations (@riverpod) |
| `riverpod_generator` | Dev | الـ code generator |
| `build_runner` | Dev | بيشغل الـ generator |

### إضافة الـ Packages:

</div>

```bash
# Step 1: Add runtime dependencies
flutter pub add flutter_riverpod
flutter pub add riverpod_annotation

# Step 2: Add dev dependencies (for code generation)
flutter pub add dev:riverpod_generator
flutter pub add dev:build_runner
```

<div dir="rtl">

### شرح كل Package:

**1. flutter_riverpod:**
- الـ library الأساسية
- فيها ConsumerWidget, ref, providers
- **Runtime dependency** - بتتحمل مع الـ app

**2. riverpod_annotation:**
- فيها الـ `@riverpod` annotation
- فيها الـ `Ref` type
- **Runtime dependency** - بتتحمل مع الـ app

**3. riverpod_generator:**
- بيولد الـ provider code
- **Dev dependency فقط** - مش بتتحمل مع الـ app
- بتشتغل وقت الـ development بس

**4. build_runner:**
- الـ tool اللي بيشغل الـ generators
- **Dev dependency فقط**
- زي compiler - بيشتغل قبل ما تعمل build

---

## ✅ Step 2: تأكد من الـ pubspec.yaml

بعد ما تضيف الـ packages، الـ `pubspec.yaml` المفروض يكون كده:

</div>

```yaml
name: my_app
description: A Flutter app with Riverpod code generation

environment:
  sdk: '>=3.0.0 <4.0.0'

dependencies:
  flutter:
    sdk: flutter

  # Riverpod runtime dependencies
  flutter_riverpod: ^2.6.1      # Or latest version
  riverpod_annotation: ^2.6.1   # Or latest version

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^2.0.0

  # Code generation dependencies
  build_runner: ^2.4.0          # Or latest version
  riverpod_generator: ^2.6.2    # Or latest version
```

<div dir="rtl">

**ملاحظة مهمة:** الأرقام دي قد تتغير - استخدم أحدث version متاح على [pub.dev](https://pub.dev/packages/flutter_riverpod).

---

## 📝 Step 3: كتابة أول Provider

خليني أكتب أول provider بالـ code generation:

### إنشاء الملف:

إنشاء ملف جديد: `lib/providers/counter_provider.dart`

</div>

```dart
// lib/providers/counter_provider.dart

// Step 1: Import riverpod_annotation
import 'package:riverpod_annotation/riverpod_annotation.dart';

// Step 2: Add part directive (اسم الملف + .g.dart)
part 'counter_provider.g.dart';

// Step 3: Annotate with @riverpod
@riverpod
class Counter extends _$Counter {
  @override
  int build() {
    // Initial state
    return 0;
  }

  // Methods to modify state
  void increment() {
    state++;
  }

  void decrement() {
    state--;
  }

  void reset() {
    state = 0;
  }
}
```

<div dir="rtl">

### شرح الكود:

**السطر 1:** `import 'package:riverpod_annotation/riverpod_annotation.dart';`
- بنستورد الـ annotations اللي هنستخدمها

**السطر 2:** `part 'counter_provider.g.dart';`
- **مهم جداً!** ده بيقول للـ Dart إن فيه ملف generated اسمه `counter_provider.g.dart`
- الاسم لازم يكون: **نفس اسم الملف + .g.dart**
- لو الملف اسمه `my_counter.dart` → يبقى `part 'my_counter.g.dart';`

**السطر 3:** `@riverpod`
- الـ annotation اللي بتقول للـ generator: "ولد provider من الكلاس ده"

**السطر 4:** `class Counter extends _$Counter`
- `_$Counter` ده class هيتولد تلقائياً في الـ `.g.dart` file
- الـ `_$` prefix معناه "generated class"

---

## ⚙️ Step 4: تشغيل الـ Code Generator

دلوقتي عندنا الكود، محتاجين نشغل الـ generator عشان يولد الـ provider.

### Option 1: One-time Generation (مرة واحدة)

</div>

```bash
flutter pub run build_runner build --delete-conflicting-outputs
```

<div dir="rtl">

**الأمر ده بيعمل:**
- بيشغل الـ build_runner
- بيولد كل الـ `.g.dart` files
- `--delete-conflicting-outputs`: بيمسح أي generated files قديمة لو فيه conflicts

**متى تستخدمه:**
- أول مرة بتشغل الـ generator
- قبل ما تعمل build للـ production

### Option 2: Watch Mode (التشغيل المستمر) - الأفضل!

</div>

```bash
flutter pub run build_runner watch --delete-conflicting-outputs
```

<div dir="rtl">

**الأمر ده بيعمل:**
- بيشغل الـ generator في الـ background
- **بيراقب** أي تغييرات في الملفات
- لو غيرت أي حاجة → بيولد الـ code تلقائياً!
- **ما بيوقفش** إلا لما توقفه انت (Ctrl+C)

**متى تستخدمه:**
- وقت الـ development
- عشان يولد الـ code تلقائياً مع كل تغيير
- **ده الأمر اللي هتستخدمه 90% من الوقت!**

### Output المتوقع:

</div>

```bash
[INFO] Generating build script...
[INFO] Generating build script completed, took 412ms

[INFO] Initializing inputs
[INFO] Reading cached asset graph...
[INFO] Checking for updates since last build...
[INFO] Running build...
[INFO] 1.2s elapsed, 0/3 actions completed.
[INFO] 3.5s elapsed, 1/3 actions completed.
[INFO] Running build completed, took 4.1s

[INFO] Caching finalized dependency graph...
[INFO] Caching finalized dependency graph completed, took 52ms

[SUCCESS] Build completed successfully!
```

<div dir="rtl">

---

## 📂 Step 5: فهم الـ Generated Files

بعد ما تشغل الـ build_runner، هيتولد ملف جديد:

### الملفات قبل Generation:

</div>

```
lib/
  providers/
    counter_provider.dart    ✅ (الملف اللي كتبناه)
```

<div dir="rtl">

### الملفات بعد Generation:

</div>

```
lib/
  providers/
    counter_provider.dart       ✅ (الملف الأصلي)
    counter_provider.g.dart     🆕 (Generated!)
```

<div dir="rtl">

### محتوى الـ Generated File:

</div>

```dart
// GENERATED CODE - DO NOT MODIFY BY HAND
// ⚠️ لا تعدل هذا الملف يدوياً!

part of 'counter_provider.dart';

// **************************************************************************
// RiverpodGenerator
// **************************************************************************

String _$counterHash() => r'abc123def456...';

/// See also [Counter].
@ProviderFor(Counter)
final counterProvider = AutoDisposeNotifierProvider<Counter, int>.internal(
  Counter.new,
  name: r'counterProvider',
  debugGetCreateSourceHash: _$counterHash,
  dependencies: null,
  allTransitiveDependencies: null,
);

typedef _$Counter = AutoDisposeNotifier<int>;
```

<div dir="rtl">

**ملاحظات مهمة:**
- الملف ده **generated تلقائياً** - ما تعدلهوش يدوياً!
- فيه الـ `counterProvider` اللي هتستخدمه في الـ UI
- `AutoDisposeNotifierProvider` - auto-dispose by default!
- الـ `_$Counter` base class موجود هنا

---

## 🎨 Step 6: استخدام الـ Provider في الـ UI

دلوقتي الـ provider جاهز! خليني أستخدمه:

</div>

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'providers/counter_provider.dart';

void main() {
  runApp(
    // Wrap app with ProviderScope
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
      home: CounterPage(),
    );
  }
}

// Use ConsumerWidget to access providers
class CounterPage extends ConsumerWidget {
  const CounterPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Watch the counter value
    final count = ref.watch(counterProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('Counter with Code Generation')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text(
              'Counter Value:',
              style: TextStyle(fontSize: 20),
            ),
            Text(
              '$count',
              style: const TextStyle(
                fontSize: 48,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 20),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                // Decrement button
                ElevatedButton(
                  onPressed: () => ref.read(counterProvider.notifier).decrement(),
                  child: const Icon(Icons.remove),
                ),
                const SizedBox(width: 10),
                // Reset button
                ElevatedButton(
                  onPressed: () => ref.read(counterProvider.notifier).reset(),
                  child: const Icon(Icons.refresh),
                ),
                const SizedBox(width: 10),
                // Increment button
                ElevatedButton(
                  onPressed: () => ref.read(counterProvider.notifier).increment(),
                  child: const Icon(Icons.add),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

<div dir="rtl">

**شرح الاستخدام:**

1. **`ref.watch(counterProvider)`:**
   - بيسمع لتغييرات الـ state
   - لما الـ state يتغير → الـ widget يعمل rebuild

2. **`ref.read(counterProvider.notifier)`:**
   - بيوصلك للـ Counter class نفسها
   - عشان تقدر تنادي methods زي `increment()`, `decrement()`

---

## ✅ Checklist: تأكد إنك عملت كل حاجة

- [ ] ضفت الـ 4 dependencies (flutter_riverpod, riverpod_annotation, riverpod_generator, build_runner)
- [ ] كتبت provider مع `@riverpod` annotation
- [ ] ضفت `part 'filename.g.dart';` في أول الملف
- [ ] شغلت `flutter pub run build_runner watch`
- [ ] اتولد الـ `.g.dart` file بنجاح
- [ ] لفيت الـ app بـ `ProviderScope`
- [ ] استخدمت `ConsumerWidget` أو `Consumer` للوصول للـ provider
- [ ] استخدمت `ref.watch()` للقراءة و `ref.read().notifier` للتعديل

---

## ⚠️ مشاكل شائعة وحلولها

### مشكلة 1: "The part directive is missing"

</div>

```
Error: Could not find part 'counter_provider.g.dart'
```

<div dir="rtl">

**الحل:**
- تأكد إنك ضفت: `part 'filename.g.dart';`
- الاسم لازم يطابق اسم الملف بالضبط
- شغل build_runner

### مشكلة 2: "Undefined name '_$Counter'"

</div>

```
Error: The class '_$Counter' isn't defined
```

<div dir="rtl">

**الحل:**
- الـ `.g.dart` file لسه ما اتولدش
- شغل: `flutter pub run build_runner build --delete-conflicting-outputs`

### مشكلة 3: "Conflicting outputs"

</div>

```
[SEVERE] Conflicting outputs were detected...
```

<div dir="rtl">

**الحل:**
- استخدم `--delete-conflicting-outputs` flag:

</div>

```bash
flutter pub run build_runner build --delete-conflicting-outputs
```

<div dir="rtl">

### مشكلة 4: Build Runner بيوقف فجأة

**الحل:**
- أوقف الـ build_runner (Ctrl+C)
- نضف الـ cache:

</div>

```bash
flutter pub run build_runner clean
```

<div dir="rtl">

- شغله تاني:

</div>

```bash
flutter pub run build_runner watch --delete-conflicting-outputs
```

<div dir="rtl">

---

## 💡 نصائح مهمة

### نصيحة 1: استخدم Watch Mode دايماً
- **بدل ما تشغل `build` كل مرة**، شغل `watch` مرة واحدة في terminal منفصل
- هيوفر عليك وقت كتير!

### نصيحة 2: أضف .g.dart للـ .gitignore؟
**لأ!** الـ `.g.dart` files لازم تكون في الـ git عشان:
- الفريق يقدر يعمل build بدون ما يشغل generator
- الـ CI/CD يشتغل أسرع

### نصيحة 3: Organize الـ Providers
- حط كل الـ providers في folder: `lib/providers/`
- أو استخدم feature-based structure:

</div>

```
lib/
  features/
    auth/
      providers/
        auth_provider.dart
        auth_provider.g.dart
    todos/
      providers/
        todos_provider.dart
        todos_provider.g.dart
```

<div dir="rtl">

---

## 🔗 الخطوة الجاية

في الملف الجاي هنتعلم **الـ Syntax الأساسي** بالتفصيل:
- كل أنواع الـ providers مع @riverpod
- Parameters و Family
- keepAlive vs AutoDispose
- Dependencies between providers

جاهز؟ يلا نكمل! 🚀

---

## 📚 المصادر

المعلومات في الملف ده مأخوذة من المصادر الرسمية:
- [Riverpod Code Generation Setup](https://riverpod.dev/docs/concepts/about_code_generation)
- [riverpod_generator Package](https://pub.dev/packages/riverpod_generator)
- [How to Auto-Generate Providers with Riverpod Generator](https://codewithandrea.com/articles/flutter-riverpod-generator/)

</div>
