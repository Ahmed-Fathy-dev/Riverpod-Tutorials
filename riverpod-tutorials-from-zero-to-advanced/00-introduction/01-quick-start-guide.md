<div dir="rtl">

# دليل البدء السريع (5 دقائق) 🚀

**المستوى**: 🟢 مبتدئ

## 🎯 الهدف

في الدليل السريع ده، هنعمل أول تطبيق Riverpod كامل وشغال في **أقل من 5 دقايق**!

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
```

<div dir="rtl">

**ملحوظة:** في Quick Start ده هنستخدم الطريقة الكلاسيكية (Classic Syntax) عشان تكون بسيطة. في Section 06 هنتعلم طريقة أحدث باستخدام Code Generation.

بعدين نفذ الأمر ده عشان تحمل الـ packages:

</div>

```bash
flutter pub get
```

<div dir="rtl">

## 💻 كود التطبيق الكامل

### الخطوة 3: اعمل الـ main.dart

افتح ملف `lib/main.dart` واستبدل كل اللي فيه بالكود ده:

</div>

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

// Create a simple counter provider
// StateProvider is perfect for simple state that can be modified
final counterProvider = StateProvider<int>((ref) => 0);

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
                    // Modify the state using .notifier.state
                    ref.read(counterProvider.notifier).state--;
                  },
                  child: const Icon(Icons.remove),
                ),
                const SizedBox(width: 20),
                // Reset button
                ElevatedButton.icon(
                  onPressed: () {
                    // Reset the state to 0
                    ref.read(counterProvider.notifier).state = 0;
                  },
                  icon: const Icon(Icons.refresh),
                  label: const Text('Reset'),
                ),
                const SizedBox(width: 20),
                // Increment button
                FloatingActionButton(
                  heroTag: 'increment',
                  onPressed: () {
                    // Modify the state using .notifier.state
                    ref.read(counterProvider.notifier).state++;
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

### الخطوة 4: شغل التطبيق

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
final counterProvider = StateProvider<int>((ref) => 0);
```

<div dir="rtl">

**إيه اللي بيحصل هنا:**
- **`StateProvider<int>`**: نوع provider للـ state البسيط اللي ممكن يتعدل
- **`(ref) => 0`**: دالة بترجع القيمة الأولية (Initial state) = 0
- **`final counterProvider`**: المتغير اللي هنستخدمه عشان نوصل للـ provider

**ليه StateProvider؟**
- بسيط جداً للـ state الأساسي (زي الأرقام، Strings، Booleans)
- بيسمحلك تقرأ وتعدل القيمة بسهولة
- مثالي للمبتدئين

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

**مهم:** ProviderScope لازم يكون wrapper للـ app كله - ده شرط أساسي!

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

**ملحوظة:** ممكن تستخدم `Consumer` widget لو مش عايز تحول الـ class كله.

### الجزء 4: Reading vs Watching

</div>

```dart
// For reading and rebuilding on changes
final count = ref.watch(counterProvider);

// For modifying the state (no rebuild)
ref.read(counterProvider.notifier).state = 5;
```

<div dir="rtl">

**الفرق:**
- **`ref.watch()`**: للقراءة + rebuild تلقائي لما القيمة تتغير
- **`ref.read()`**: للقراءة أو التعديل مرة واحدة (في الـ event handlers)

**قاعدة ذهبية:**
- في الـ `build` method → استخدم `watch`
- في الـ button handlers → استخدم `read`

### الجزء 5: Modifying State

</div>

```dart
// Increment
ref.read(counterProvider.notifier).state++;

// Set to specific value
ref.read(counterProvider.notifier).state = 10;

// Decrement
ref.read(counterProvider.notifier).state--;
```

<div dir="rtl">

**إيه اللي بيحصل هنا:**
- **`.notifier`**: بيجيبلك الـ StateController
- **`.state`**: القيمة الحالية
- لما تعدل `.state`، كل الـ widgets اللي بتعمل watch بتتحدث تلقائياً

---

## 🔄 الخطوات التانية

دلوقتي بعد ما عملت أول تطبيق، جرب:

### تجربة 1: ضيف زرار Double

</div>

```dart
ElevatedButton(
  onPressed: () {
    // Double the current value
    ref.read(counterProvider.notifier).state *= 2;
  },
  child: const Text('Double'),
),
```

<div dir="rtl">

### تجربة 2: غير الـ Initial Value

</div>

```dart
// Start from 10 instead of 0
final counterProvider = StateProvider<int>((ref) => 10);
```

<div dir="rtl">

### تجربة 3: ضيف Validation

</div>

```dart
FloatingActionButton(
  onPressed: () {
    final current = ref.read(counterProvider);
    if (current < 100) {  // Don't go above 100
      ref.read(counterProvider.notifier).state++;
    }
  },
  child: const Icon(Icons.add),
),
```

<div dir="rtl">

---

## ⚠️ مشاكل شائعة والحلول

### مشكلة 1: "Could not find ProviderScope"

**السبب:** نسيت تحط ProviderScope في الـ main

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

### مشكلة 2: الـ UI مش بتتحدث

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

### مشكلة 3: "The argument type 'WidgetRef' can't be assigned"

**السبب:** استخدمت `StatelessWidget` بدل `ConsumerWidget`

**الحل:**

</div>

```dart
// ✅ Correct
class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // ...
  }
}

// ❌ Wrong
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // No ref available!
  }
}
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
    onPressed: () => ref.read(counterProvider.notifier).state++,
    child: Text('$count'),
  );
}
```

<div dir="rtl">

**ليه؟**
- `watch` بيخلي الـ widget يتحدث لما القيمة تتغير
- `read` بيقرأ القيمة مرة واحدة (أو يعدل عليها) بدون rebuild

### نصيحة 2: StateProvider للبيانات البسيطة فقط

**StateProvider مثالي لـ:**
- ✅ Counter
- ✅ Toggle (true/false)
- ✅ Current tab index
- ✅ Text field value

**لما البيانات تبقى معقدة، استخدم حاجة تانية:**
- 🟡 List معقدة → استخدم Notifier (Section 06+)
- 🟡 Async data → استخدم FutureProvider/AsyncNotifier
- 🟡 Business logic → استخدم Notifier class

### نصيحة 3: Classic vs Code Generation

**في Quick Start ده استخدمنا Classic Syntax عشان:**
- ✅ أبسط للمبتدئين
- ✅ ما يحتاجش build_runner setup
- ✅ الكود واضح ومباشر

**في Section 06 هنتعلم Code Generation:**
- ✅ Type safety أفضل
- ✅ Less boilerplate
- ✅ الطريقة المفضلة في المشاريع الكبيرة

---

## 📝 ملخص

**اللي عملناه النهاردة:**
1. ✅ نصبنا Riverpod 3 (flutter_riverpod فقط)
2. ✅ عملنا أول Provider باستخدام `StateProvider`
3. ✅ قرينا الـ state باستخدام `ref.watch()`
4. ✅ عدلنا الـ state باستخدام `.notifier.state`
5. ✅ فهمنا الفرق بين `watch` و `read`

**الـ concepts المهمة:**
- **Provider**: الحاوية اللي بتحفظ وتشارك الـ state
- **StateProvider**: نوع provider للـ state البسيط
- **ref.watch()**: للقراءة + rebuild
- **ref.read()**: للقراءة/التعديل في الـ events
- **ProviderScope**: لازم يكون في root الـ app
- **ConsumerWidget**: بدل StatelessWidget عشان توصل للـ ref

---

## 🔗 الخطوة الجاية

دلوقتي بعد ما عرفت الأساسيات:

**لو عايز تفهم أكتر:**
- روح على `02-what-is-state-management.md` عشان تفهم State Management بعمق
- أو ابدأ من Section 01 عشان تتعلم المبادئ الأساسية

**لو عايز تطبق أكتر:**
- جرب التجارب اللي فوق (Double, Validation, etc.)
- حاول تعمل تطبيق Todo list بسيط
- جرب تضيف providers تانية (مثلاً: theme mode, username)

**افتكر:** ده كان quick start - في تفاصيل كتير هنتعلمها في الأقسام الجاية!

---

## 📚 المصادر

- [Riverpod Official Documentation](https://riverpod.dev)
- [StateProvider Guide](https://riverpod.dev/docs/providers/state_provider)
- [Getting Started with Riverpod](https://riverpod.dev/docs/getting_started)

</div>
