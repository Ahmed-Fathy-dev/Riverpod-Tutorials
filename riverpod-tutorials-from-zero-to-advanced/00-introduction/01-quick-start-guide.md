<div dir="rtl">

# دليل البدء السريع (10 دقائق) 🚀

**المستوى**: 🟢 مبتدئ

## 🎯 الهدف

في الدليل السريع ده، هنعمل أول تطبيق Riverpod كامل وشغال في **أقل من 10 دقايق**!

## 📌 هتتعلم إيه

- إزاي تنصب Riverpod 3 في مشروع Flutter
- إزاي تعمل Provider بسيط
- إزاي تقرأ State من Provider
- إزاي تعدل على State
- إزاي تعرض التحديثات في UI

## 🔧 التنصيب

### الخطوة 1: اعمل مشروع Flutter جديد

</div>

```bash
flutter create riverpod_quick_start
cd riverpod_quick_start
```

<div dir="rtl">

### الخطوة 2: ضيف Riverpod للمشروع

افتح ملف `pubspec.yaml` وضيف:

</div>

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_riverpod: ^3.0.0
  riverpod_annotation: ^3.0.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  build_runner: ^2.4.0
  riverpod_generator: ^3.0.0
  riverpod_lint: ^3.0.0
  custom_lint: ^0.6.0
```

<div dir="rtl">

بعدين نفذ الأمر ده عشان تحمل الـ packages:

</div>

```bash
flutter pub get
```

<div dir="rtl">

## 💻 كود التطبيق الكامل

### الخطوة 3: اعمل ملف الـ Provider

اعمل ملف جديد اسمه `lib/counter_provider.dart`:

</div>

```dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

// IMPORTANT: This line is required for code generation
// The generated file will be named counter_provider.g.dart
part 'counter_provider.g.dart';

// Create a simple counter provider using @riverpod annotation
@riverpod
class Counter extends _$Counter {
  // The build method returns the initial state
  @override
  int build() {
    return 0; // Counter starts at 0
  }

  // Method to increment the counter
  void increment() {
    state++;
  }

  // Method to decrement the counter
  void decrement() {
    state--;
  }

  // Method to reset the counter
  void reset() {
    state = 0;
  }
}
```

<div dir="rtl">

### الخطوة 4: اعمل Code Generation

دلوقتي لازم نعمل generate للكود. نفذ الأمر ده:

</div>

```bash
dart run build_runner build
```

<div dir="rtl">

الأمر ده هيعمل ملف `counter_provider.g.dart` جنب ملف `counter_provider.dart`. ده الملف اللي فيه الكود المُولّد تلقائياً.

**ملحوظة:** لو عايز الـ generator يشتغل تلقائياً كل ما تعدل على الكود، استخدم:

</div>

```bash
dart run build_runner watch
```

<div dir="rtl">

### الخطوة 5: اعمل ملف الـ main

افتح ملف `lib/main.dart` واستبدل كل اللي فيه بالكود ده:

</div>

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'counter_provider.dart';

// Wrap the app with ProviderScope
// ProviderScope is required at the root of your app
void main() {
  runApp(
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
      title: 'Riverpod Quick Start',
      theme: ThemeData(
        primarySwatch: Colors.blue,
        useMaterial3: true,
      ),
      home: const CounterPage(),
    );
  }
}

// Use ConsumerWidget to read the provider
// ConsumerWidget automatically rebuilds when the provider value changes
class CounterPage extends ConsumerWidget {
  const CounterPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Watch the provider to get the current value
    // The widget will rebuild when the value changes
    final count = ref.watch(counterProvider);

    return Scaffold(
      appBar: AppBar(
        title: const Text('Riverpod Counter'),
        centerTitle: true,
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text(
              'You have pushed the button this many times:',
              style: TextStyle(fontSize: 18),
            ),
            const SizedBox(height: 20),
            Text(
              '$count',
              style: const TextStyle(
                fontSize: 72,
                fontWeight: FontWeight.bold,
                color: Colors.blue,
              ),
            ),
            const SizedBox(height: 40),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                // Decrement button
                FloatingActionButton(
                  heroTag: 'decrement',
                  onPressed: () {
                    // Call the decrement method
                    ref.read(counterProvider.notifier).decrement();
                  },
                  child: const Icon(Icons.remove),
                ),
                const SizedBox(width: 20),
                // Reset button
                ElevatedButton.icon(
                  onPressed: () {
                    // Call the reset method
                    ref.read(counterProvider.notifier).reset();
                  },
                  icon: const Icon(Icons.refresh),
                  label: const Text('Reset'),
                ),
                const SizedBox(width: 20),
                // Increment button
                FloatingActionButton(
                  heroTag: 'increment',
                  onPressed: () {
                    // Call the increment method
                    ref.read(counterProvider.notifier).increment();
                  },
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

### الخطوة 6: شغل التطبيق

</div>

```bash
flutter run
```

<div dir="rtl">

## 🎉 مبروك!

دلوقتي عندك أول تطبيق Riverpod 3 شغال! جرب:
- اضغط على زرار **+** عشان تزود العداد
- اضغط على زرار **-** عشان تقلل العداد
- اضغط على زرار **Reset** عشان ترجع للصفر

---

## 📖 فهم الكود

خليني أشرحلك كل جزء:

### الجزء 1: Provider Definition

</div>

```dart
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() => state++;
  void decrement() => state--;
  void reset() => state = 0;
}
```

<div dir="rtl">

**إيه اللي بيحصل هنا:**
- **`@riverpod`**: annotation بتقول للـ generator إنشئ provider من الـ class ده
- **`Counter extends _$Counter`**: بنرث من generated base class
- **`build()`**: بترجع الـ initial state (القيمة الأولية)
- **Methods**: كل method بتعدل على `state` - الـ UI بتتحدث تلقائياً

### الجزء 2: ProviderScope

</div>

```dart
void main() {
  runApp(
    const ProviderScope(
      child: MyApp(),
    ),
  );
}
```

<div dir="rtl">

**إيه اللي بيحصل هنا:**
- **`ProviderScope`**: لازم يكون في root الـ app
- بيعمل container لكل الـ providers
- بدونه، الـ providers مش هتشتغل

### الجزء 3: ConsumerWidget

</div>

```dart
class CounterPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);
    // ...
  }
}
```

<div dir="rtl">

**إيه اللي بيحصل هنا:**
- **`ConsumerWidget`**: بدل `StatelessWidget`
- بيديك `ref` - ده مفتاحك للـ providers
- **`ref.watch(counterProvider)`**: بتقرأ القيمة وتتابع التغييرات
- الـ widget بيعمل rebuild تلقائي لما القيمة تتغير

### الجزء 4: Reading vs Watching

</div>

```dart
// For reading and rebuilding on changes
final count = ref.watch(counterProvider);

// For calling methods (no rebuild)
ref.read(counterProvider.notifier).increment();
```

<div dir="rtl">

**الفرق:**
- **`ref.watch()`**: للقراءة + rebuild تلقائي
- **`ref.read()`**: للقراءة مرة واحدة (في الـ event handlers)

---

## ⚙️ Code Generation - إزاي بيشتغل؟

لما تكتب:

</div>

```dart
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;
}
```

<div dir="rtl">

الـ `build_runner` بيولّد كود زي ده تلقائياً:

</div>

```dart
// counter_provider.g.dart (generated automatically)
final counterProvider = CounterProvider();

class CounterProvider extends AutoDisposeNotifierProvider<Counter, int> {
  // Generated implementation...
}

// Base class for Counter
abstract class _$Counter extends AutoDisposeNotifier<int> {
  // Generated implementation...
}
```

<div dir="rtl">

**الميزة:** إنت بتكتب كود أقل وأبسط، والـ generator بيعمل الباقي!

---

## 🔄 الخطوات التانية

دلوقتي بعد ما عملت أول تطبيق، جرب:

### تجربة 1: ضيف زرار Double

</div>

```dart
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() => state++;
  void decrement() => state--;
  void reset() => state = 0;

  // Add this new method
  void double() => state = state * 2;
}
```

<div dir="rtl">

بعدين ضيف الزرار في الـ UI:

</div>

```dart
ElevatedButton(
  onPressed: () => ref.read(counterProvider.notifier).double(),
  child: const Text('Double'),
),
```

<div dir="rtl">

### تجربة 2: غير الـ Initial Value

</div>

```dart
@override
int build() => 10; // Start from 10 instead of 0
```

<div dir="rtl">

### تجربة 3: ضيف Validation

</div>

```dart
void increment() {
  if (state < 100) {  // Don't go above 100
    state++;
  }
}
```

<div dir="rtl">

---

## ⚠️ مشاكل شائعة والحلول

### مشكلة 1: "part 'counter_provider.g.dart' doesn't exist"

**السبب:** ما عملتش `build_runner` بعد

**الحل:**

</div>

```bash
dart run build_runner build
```

<div dir="rtl">

### مشكلة 2: "The getter 'counterProvider' isn't defined"

**السبب:** مش عامل import للـ provider file

**الحل:**

</div>

```dart
import 'counter_provider.dart';
```

<div dir="rtl">

### مشكلة 3: الـ UI مش بتتحدث

**السبب:** استخدمت `ref.read()` بدل `ref.watch()`

**الحل:**

</div>

```dart
// ✅ Correct: rebuilds on changes
final count = ref.watch(counterProvider);

// ❌ Wrong: doesn't rebuild
final count = ref.read(counterProvider);
```

<div dir="rtl">

---

## 💡 نصائح مهمة

### نصيحة 1: استخدام watch vs read

</div>

```dart
@override
Widget build(BuildContext context, WidgetRef ref) {
  // ✅ Use watch in build method
  final count = ref.watch(counterProvider);

  return ElevatedButton(
    // ✅ Use read in event handlers
    onPressed: () => ref.read(counterProvider.notifier).increment(),
    child: Text('$count'),
  );
}
```

<div dir="rtl">

### نصيحة 2: استخدام build_runner watch أثناء التطوير

</div>

```bash
# Run this once and leave it running
dart run build_runner watch
```

<div dir="rtl">

ده هيخلي الـ code generation يحصل تلقائياً كل ما تعدل على الكود.

### نصيحة 3: Clean وRegenerate لو حصلت مشاكل

</div>

```bash
# Clean old generated files
dart run build_runner clean

# Regenerate everything
dart run build_runner build --delete-conflicting-outputs
```

<div dir="rtl">

---

## 📝 ملخص

**اللي عملناه النهاردة:**
1. ✅ نصبنا Riverpod 3 مع code generation
2. ✅ عملنا أول Provider باستخدام `@riverpod`
3. ✅ قرينا الـ state باستخدام `ref.watch()`
4. ✅ عدلنا الـ state باستخدام methods
5. ✅ فهمنا الفرق بين `watch` و `read`

**الـ concepts المهمة:**
- **Provider**: الحاوية اللي بتحفظ وتشارك الـ state
- **@riverpod**: Annotation للـ code generation
- **ref.watch()**: للقراءة + rebuild
- **ref.read()**: للقراءة في الـ events
- **ProviderScope**: لازم يكون في root الـ app

---

## 🔗 الخطوة الجاية

دلوقتي بعد ما عرفت الأساسيات:
- روح على `02-what-is-state-management.md` عشان تفهم State Management بعمق
- أو ابدأ من Section 03 لو عايز تدخل في التفاصيل

**افتكر:** ده كان quick start - في تفاصيل كتير هنتعلمها في الأقسام الجاية!

---

## 📚 المصادر

- [Riverpod Getting Started](https://riverpod.dev/docs/getting_started)
- [Code Generation Guide](https://riverpod.dev/docs/concepts/about_code_generation)
- [Riverpod Examples](https://github.com/rrousselGit/riverpod/tree/master/examples)

</div>
