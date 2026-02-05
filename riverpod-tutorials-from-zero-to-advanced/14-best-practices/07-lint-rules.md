<div dir="rtl">

# Riverpod Lint Rules - كود نظيف تلقائياً 🧹✨

**المستوى**: 🟢 مبتدئ - 🟡 متوسط

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تثبت Riverpod Lint rules
- تستخدم custom_lint للتحليل
- تفهم القواعد المهمة
- تصلح الأخطاء تلقائياً
- تكتب كود Riverpod محترف

---

## 💡 ما هو Riverpod Lint؟

**riverpod_lint** هو package بيوفر **قواعد lint مخصصة** لـ Riverpod، بيساعدك:

- 🐛 **Catch Errors Early** - اكتشف الأخطاء قبل runtime
- ✅ **Best Practices** - اتبع الـ best practices تلقائياً
- 🔧 **Auto-fix** - بعض المشاكل بتتصلح تلقائياً
- 📚 **Learn** - تعلم من التحذيرات

---

## 🚀 Setup: التثبيت

### الخطوة 1: إضافة Dependencies

</div>

```yaml
# pubspec.yaml
dev_dependencies:
  # ✅ Required for riverpod_lint
  custom_lint: ^0.6.0
  riverpod_lint: ^2.3.0
```

<div dir="rtl">

### الخطوة 2: تفعيل في analysis_options.yaml

</div>

```yaml
# analysis_options.yaml
analyzer:
  plugins:
    - custom_lint  # ✅ Enable custom_lint

# Optional: Configure severity
custom_lint:
  rules:
    # Enable all riverpod_lint rules
    - riverpod_lint
```

<div dir="rtl">

### الخطوة 3: تشغيل Lint

</div>

```bash
# Run custom_lint
dart run custom_lint

# In VS Code: Problems panel will show issues automatically

# In Android Studio: Inspection results will show issues
```

<div dir="rtl">

---

## 📋 القواعد المهمة

### Rule 1: provider_dependencies

**المشكلة:** استخدام `ref.read` داخل `build()` method

</div>

```dart
// ❌ BAD - ref.read in build()
@riverpod
class Counter extends _$Counter {
  @override
  int build() {
    final initial = ref.read(initialValueProvider);  // ❌ Lint error!
    return initial;
  }
}

// ⚠️ Lint message:
// "Avoid using ref.read inside build(). Use ref.watch instead."

// ✅ GOOD - ref.watch in build()
@riverpod
class Counter extends _$Counter {
  @override
  int build() {
    final initial = ref.watch(initialValueProvider);  // ✅ Correct!
    return initial;
  }
}
```

<div dir="rtl">

**لماذا؟** `ref.read` لا يتتبع التغييرات، استخدم `ref.watch` في `build()` للتحديثات التلقائية.

---

### Rule 2: avoid_ref_inside_state_dispose

**المشكلة:** استخدام `ref` داخل `dispose()` بعد disposal

</div>

```dart
// ❌ BAD - Using ref in dispose
@riverpod
class MyNotifier extends _$MyNotifier {
  Timer? _timer;

  @override
  int build() {
    _timer = Timer.periodic(Duration(seconds: 1), (_) {
      state++;
    });

    ref.onDispose(() {
      _timer?.cancel();
      // ❌ Lint warning!
      ref.invalidateSelf();  // Dangerous after dispose!
    });

    return 0;
  }
}

// ⚠️ Lint message:
// "Avoid using ref inside onDispose. Provider may already be disposed."

// ✅ GOOD - No ref in dispose
@riverpod
class MyNotifier extends _$MyNotifier {
  Timer? _timer;

  @override
  int build() {
    _timer = Timer.periodic(Duration(seconds: 1), (_) {
      state++;
    });

    ref.onDispose(() {
      _timer?.cancel();  // ✅ Only cleanup, no ref
    });

    return 0;
  }
}
```

<div dir="rtl">

---

### Rule 3: scoped_providers_should_specify_dependencies

**المشكلة:** Provider مش محدد dependencies بتاعه

</div>

```dart
// ❌ BAD - Unclear dependencies
@riverpod
Future<User> user(UserRef ref) async {
  // Uses userId from somewhere, but not explicit
  final userId = someGlobalVariable;  // ❌ Hidden dependency!

  return await api.getUser(userId);
}

// ⚠️ Lint warning:
// "Provider dependencies should be explicit parameters."

// ✅ GOOD - Explicit dependencies
@riverpod
Future<User> user(UserRef ref, String userId) async {
  // ✅ userId is explicit parameter!
  return await api.getUser(userId);
}

// Usage:
final userAsync = ref.watch(userProvider('user123'));
```

<div dir="rtl">

---

### Rule 4: avoid_public_notifier_properties

**المشكلة:** Public properties في Notifier (يجب استخدام methods)

</div>

```dart
// ❌ BAD - Public property
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  // ❌ Lint warning!
  int get doubled => state * 2;
}

// ⚠️ Lint message:
// "Avoid public getters/properties on Notifier. Expose via provider instead."

// ✅ GOOD - Separate provider
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() => state++;
}

// ✅ Separate provider for computed value
@riverpod
int doubledCounter(DoubledCounterRef ref) {
  final count = ref.watch(counterProvider);
  return count * 2;
}
```

<div dir="rtl">

---

### Rule 5: avoid_manual_providers_as_generated_provider_dependency

**المشكلة:** Mix بين classic و generated providers

</div>

```dart
// ❌ BAD - Generated provider depending on manual provider
final manualProvider = Provider<String>((ref) => 'Hello');  // Classic

@riverpod
String message(MessageRef ref) {
  // ❌ Lint warning!
  final greeting = ref.watch(manualProvider);
  return '$greeting World';
}

// ⚠️ Lint message:
// "Avoid mixing manual and generated providers. Prefer consistent approach."

// ✅ GOOD - Use generated providers consistently
@riverpod
String greeting(GreetingRef ref) => 'Hello';  // Generated

@riverpod
String message(MessageRef ref) {
  final greeting = ref.watch(greetingProvider);  // ✅ Both generated
  return '$greeting World';
}
```

<div dir="rtl">

---

## 🔧 Auto-Fix Features

بعض القواعد عندها **Quick Fixes** تلقائية!

### مثال: Convert ref.read to ref.watch

</div>

```dart
// ❌ Before auto-fix
@riverpod
class Counter extends _$Counter {
  @override
  int build() {
    final initial = ref.read(initialValueProvider);  // ❌ Error
    return initial;
  }
}

// في VS Code:
// 1. ضع cursor على ref.read
// 2. اضغط Ctrl+. (or Cmd+. on Mac)
// 3. اختار "Convert to ref.watch"

// ✅ After auto-fix
@riverpod
class Counter extends _$Counter {
  @override
  int build() {
    final initial = ref.watch(initialValueProvider);  // ✅ Fixed!
    return initial;
  }
}
```

<div dir="rtl">

---

## ⚙️ تخصيص القواعد

يمكنك تفعيل/تعطيل قواعد معينة:

</div>

```yaml
# analysis_options.yaml
custom_lint:
  rules:
    # Enable specific rules only
    - provider_dependencies
    - avoid_ref_inside_state_dispose

    # Disable a rule
    - provider_dependencies: false
```

<div dir="rtl">

### تخصيص Severity

</div>

```yaml
# analysis_options.yaml
custom_lint:
  rules:
    - provider_dependencies:
        severity: error  # error, warning, info

    - avoid_public_notifier_properties:
        severity: warning
```

<div dir="rtl">

---

## 📊 قائمة القواعد الكاملة

| Rule | Description | Severity | Auto-fix |
|------|-------------|----------|----------|
| `provider_dependencies` | Use ref.watch in build() | Error | ✅ Yes |
| `avoid_ref_inside_state_dispose` | No ref in onDispose | Warning | ❌ No |
| `scoped_providers_should_specify_dependencies` | Explicit dependencies | Info | ❌ No |
| `avoid_public_notifier_properties` | No public getters | Warning | ❌ No |
| `avoid_manual_providers_as_generated_provider_dependency` | Consistent style | Info | ❌ No |
| `provider_parameters` | Proper parameter types | Error | ❌ No |
| `notifier_extends` | Extend correct base class | Error | ❌ No |

---

## 🎯 Best Practices

### 1. Run Lint قبل Commit

</div>

```bash
# في pre-commit hook
#!/bin/bash
dart run custom_lint

if [ $? -ne 0 ]; then
  echo "❌ Lint errors found. Please fix before committing."
  exit 1
fi
```

<div dir="rtl">

### 2. Enable في CI/CD

</div>

```yaml
# .github/workflows/ci.yml
name: CI

on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: subosito/flutter-action@v2
      - run: flutter pub get
      - run: dart run custom_lint
```

<div dir="rtl">

### 3. استخدم في IDE

**VS Code:**
- Extension: "Dart"
- يعرض errors في Problems panel تلقائياً

**Android Studio:**
- Dart plugin installed
- Inspection results window

---

## 🐛 Troubleshooting

### Problem: Lint لا يعمل

**Solution:**

</div>

```bash
# 1. Clean و re-install
flutter clean
flutter pub get

# 2. Delete analysis cache
rm -rf .dart_tool/

# 3. Re-run
dart run custom_lint
```

<div dir="rtl">

### Problem: Too many warnings

**Solution:** ابدأ بتصليح Errors أولاً، ثم Warnings

</div>

```yaml
# Temporarily show only errors
custom_lint:
  rules:
    - provider_dependencies:
        severity: error

    # Disable warnings temporarily
    - avoid_public_notifier_properties: false
```

<div dir="rtl">

---

## 🎓 الخلاصة

### Riverpod Lint في سطر واحد:
> **riverpod_lint = مساعدك الشخصي لكتابة كود Riverpod محترف!**

### الفوائد:
- ✅ Catch errors early
- ✅ Learn best practices
- ✅ Auto-fix common issues
- ✅ Consistent code style
- ✅ Better code quality

### Setup سريع:
1. Add `custom_lint` & `riverpod_lint` to `pubspec.yaml`
2. Enable في `analysis_options.yaml`
3. Run `dart run custom_lint`
4. Fix issues (many auto-fixable!)

---

## 🔗 مصادر إضافية

### Official Documentation:
- [riverpod_lint Package](https://pub.dev/packages/riverpod_lint)
- [custom_lint Package](https://pub.dev/packages/custom_lint)
- [Riverpod Best Practices](https://riverpod.dev/docs/concepts/best_practices)

---

## ✅ تأكد إنك فهمت

- [ ] عارف كيف تثبت riverpod_lint؟
- [ ] تقدر تشغل custom_lint؟
- [ ] فاهم القواعد المهمة؟
- [ ] تعرف تستخدم auto-fix؟
- [ ] تقدر تخصص القواعد؟

---

**🧹 Riverpod Lint = كود نظيف و احترافي بدون مجهود!**

استخدمه في كل مشاريعك Riverpod! 💪

</div>
