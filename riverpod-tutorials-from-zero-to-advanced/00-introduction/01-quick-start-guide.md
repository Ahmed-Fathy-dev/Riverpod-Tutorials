<div dir="rtl">

# دليل البدء السريع (5 دقائق) 🚀

**المستوى**: 🟢 مبتدئ

## 🎯 الهدف

في هذا الدليل السريع، ستنشئ أول تطبيق Riverpod كامل وعملي في **أقل من 5 دقائق**!

## 📌 ما ستتعلمه

- تثبيت Riverpod في مشروع Flutter
- إنشاء Provider بسيط
- قراءة State من Provider
- تعديل State
- عرض التحديثات في UI

## 🔧 التثبيت

### 1. أنشئ مشروع Flutter جديد

</div>

```bash
flutter create riverpod_quick_start
cd riverpod_quick_start
```

<div dir="rtl">

### 2. أضف Riverpod للمشروع

افتح ملف `pubspec.yaml` وأضف:

</div>

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_riverpod: ^3.0.0
```

<div dir="rtl">

ثم نفذ الأمر:

</div>

```bash
flutter pub get
```

<div dir="rtl">

## 💻 كود التطبيق الكامل

افتح ملف `lib/main.dart` واستبدل محتواه بالكود التالي:

</div>

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

// Step 1: Create a StateProvider for the counter
// This provider holds an integer value that starts at 0
final counterProvider = StateProvider<int>((ref) {
  return 0; // Initial value
});

// Step 2: Wrap the app with ProviderScope
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

// Step 3: Use ConsumerWidget to read the provider
// ConsumerWidget automatically rebuilds when the provider value changes
class CounterPage extends ConsumerWidget {
  const CounterPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Step 4: Watch the provider to get the current value
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
                    // Step 5: Update the state
                    ref.read(counterProvider.notifier).state--;
                  },
                  child: const Icon(Icons.remove),
                ),
                const SizedBox(width: 20),
                // Reset button
                ElevatedButton.icon(
                  onPressed: () {
                    // Reset to initial value
                    ref.read(counterProvider.notifier).state = 0;
                  },
                  icon: const Icon(Icons.refresh),
                  label: const Text('Reset'),
                  style: ElevatedButton.styleFrom(
                    padding: const EdgeInsets.symmetric(
                      horizontal: 20,
                      vertical: 15,
                    ),
                  ),
                ),
                const SizedBox(width: 20),
                // Increment button
                FloatingActionButton(
                  heroTag: 'increment',
                  onPressed: () {
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

## ▶️ تشغيل التطبيق

</div>

```bash
flutter run
```

<div dir="rtl">

## 🎉 تهانينا!

الآن لديك تطبيق Riverpod يعمل بالكامل! جرّب الأزرار:
- ➕ زر الإضافة يزيد العداد
- ➖ زر الطرح ينقص العداد
- 🔄 زر Reset يعيد العداد للصفر

---

## 📖 فهم الكود خطوة بخطوة

### الخطوة 1: إنشاء Provider

</div>

```dart
final counterProvider = StateProvider<int>((ref) {
  return 0; // Initial value
});
```

<div dir="rtl">

- `StateProvider`: نوع من أنواع Providers للـ state البسيط الذي يمكن تعديله
- `<int>`: نوع البيانات (عدد صحيح)
- `(ref)`: كائن للوصول إلى providers أخرى
- `return 0`: القيمة الابتدائية

### الخطوة 2: إضافة ProviderScope

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

- `ProviderScope`: **إلزامي** في جذر التطبيق
- يحتوي على جميع Providers
- بدونه لن يعمل Riverpod

### الخطوة 3: استخدام ConsumerWidget

</div>

```dart
class CounterPage extends ConsumerWidget {
  const CounterPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // ...
  }
}
```

<div dir="rtl">

- `ConsumerWidget`: بديل لـ `StatelessWidget`
- يوفر `WidgetRef ref` في دالة `build`
- يُعيد البناء تلقائياً عند تغيير الـ state

**الفرق الوحيد**:
- `StatelessWidget`: `build(BuildContext context)`
- `ConsumerWidget`: `build(BuildContext context, WidgetRef ref)`

### الخطوة 4: مراقبة Provider (Watch)

</div>

```dart
final count = ref.watch(counterProvider);
```

<div dir="rtl">

- `ref.watch()`: يراقب التغييرات في Provider
- عند تغيير القيمة، Widget يُعاد بناؤه تلقائياً
- استخدمه داخل `build` method

### الخطوة 5: تعديل State

</div>

```dart
// Read the notifier and update state
ref.read(counterProvider.notifier).state++;
```

<div dir="rtl">

- `ref.read()`: قراءة Provider بدون مراقبة
- `.notifier`: الحصول على object للتحكم في State
- `.state`: الوصول لقيمة State وتعديلها
- استخدمه في Event Handlers (onPressed, onTap...)

---

## ⚡ نصائح سريعة

### ✅ **افعل**

</div>

```dart
// Watch inside build method
@override
Widget build(BuildContext context, WidgetRef ref) {
  final count = ref.watch(counterProvider);
  return Text('$count');
}

// Read inside event handlers
onPressed: () {
  ref.read(counterProvider.notifier).state++;
}
```

<div dir="rtl">

### ❌ **لا تفعل**

</div>

```dart
// ❌ DON'T watch in event handlers
onPressed: () {
  final count = ref.watch(counterProvider); // Wrong!
  print(count);
}

// ❌ DON'T read in build method (if you need to react to changes)
@override
Widget build(BuildContext context, WidgetRef ref) {
  final count = ref.read(counterProvider); // Won't rebuild!
  return Text('$count');
}
```

<div dir="rtl">

### 💡 **القاعدة الذهبية**

- **`ref.watch`** في `build` → للعرض والمراقبة
- **`ref.read`** في event handlers → للتعديل

---

## 🔍 مقارنة سريعة

### قبل Riverpod (StatefulWidget)

</div>

```dart
class CounterPage extends StatefulWidget {
  const CounterPage({super.key});

  @override
  State<CounterPage> createState() => _CounterPageState();
}

class _CounterPageState extends State<CounterPage> {
  int count = 0; // State lives in widget

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Text('$count'),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          setState(() {
            count++; // Update state
          });
        },
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

<div dir="rtl">

### بعد Riverpod (ConsumerWidget)

</div>

```dart
// State lives outside widget
final counterProvider = StateProvider<int>((ref) => 0);

class CounterPage extends ConsumerWidget {
  const CounterPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);

    return Scaffold(
      body: Center(
        child: Text('$count'),
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

### ✨ المميزات

| الميزة | StatefulWidget | Riverpod |
|--------|----------------|----------|
| **مكان الـ State** | داخل Widget | خارج Widget (عام) |
| **إعادة الاستخدام** | صعبة | سهلة جداً |
| **Testing** | معقد | سهل جداً |
| **الوصول من أي مكان** | لا | نعم |
| **Hot Reload** | يفقد State أحياناً | يحفظ State |
| **الأداء** | جيد | ممتاز (rebuilds أقل) |

---

## 🎯 ماذا بعد؟

الآن بعد أن أنشأت أول تطبيق Riverpod، حان الوقت لفهم الأساسيات بشكل أعمق:

### المسار المقترح:

1. **اقرأ**: `02-what-is-state-management.md`
   - لفهم **لماذا** نحتاج State Management

2. **القسم 01**: State Management Fundamentals
   - الأساسيات النظرية

3. **القسم 02**: State Management Comparison
   - مقارنة Riverpod بالحلول الأخرى

4. **القسم 03**: Riverpod Basics
   - تعمق في Riverpod

---

## 🆘 مشاكل شائعة وحلولها

### المشكلة 1: `ProviderScope not found`

</div>

```dart
// ❌ Error
void main() {
  runApp(const MyApp());
}

// ✅ Solution
void main() {
  runApp(
    const ProviderScope(
      child: MyApp(),
    ),
  );
}
```

<div dir="rtl">

### المشكلة 2: Widget لا يُعاد بناؤه

</div>

```dart
// ❌ Wrong - Using StatelessWidget
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // Can't access ref here!
  }
}

// ✅ Solution - Use ConsumerWidget
class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final value = ref.watch(myProvider);
    return Text('$value');
  }
}
```

<div dir="rtl">

### المشكلة 3: القراءة في Build Method

</div>

```dart
// ❌ Wrong - Won't rebuild on changes
@override
Widget build(BuildContext context, WidgetRef ref) {
  final count = ref.read(counterProvider);
  return Text('$count');
}

// ✅ Solution - Use watch
@override
Widget build(BuildContext context, WidgetRef ref) {
  final count = ref.watch(counterProvider);
  return Text('$count');
}
```

<div dir="rtl">

---

## 📚 المصادر

- [Riverpod Official Docs - Getting Started](https://riverpod.dev/docs/introduction/getting_started)
- [StateProvider Documentation](https://riverpod.dev/docs/providers/state_provider)
- [ConsumerWidget Documentation](https://riverpod.dev/docs/concepts/reading)

---

## ✅ قائمة التحقق

تأكد أنك فهمت:

- [ ] كيفية تثبيت Riverpod
- [ ] إنشاء Provider بسيط
- [ ] استخدام `ProviderScope`
- [ ] الفرق بين `ConsumerWidget` و `StatelessWidget`
- [ ] متى تستخدم `ref.watch`
- [ ] متى تستخدم `ref.read`
- [ ] كيفية تعديل State

**إذا فهمت كل النقاط، أنت جاهز للانتقال للقسم التالي!** 🎉

</div>
