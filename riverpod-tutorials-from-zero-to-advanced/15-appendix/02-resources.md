<div dir="rtl">

# المصادر والروابط (Resources)

**المستوى**: 📚 مرجعي

## 📖 التوثيق الرسمي

### [Riverpod Official Documentation](https://riverpod.dev)
التوثيق الرسمي الكامل لـ Riverpod - أفضل مصدر للمعلومات الدقيقة والمحدثة.

### [Riverpod API Reference](https://pub.dev/documentation/riverpod/latest/)
مرجع API الكامل لجميع الكلاسات والدوال في Riverpod.

### [Riverpod GitHub Repository](https://github.com/rrousselGit/riverpod)
الكود المصدري والـ issues والـ discussions.

---

## 📦 الحزم المطلوبة

### Core Packages

```yaml
dependencies:
  flutter_riverpod: ^2.5.0
  riverpod_annotation: ^2.3.0

dev_dependencies:
  riverpod_generator: ^2.3.0
  build_runner: ^2.4.0
  custom_lint: ^0.6.0
  riverpod_lint: ^2.3.0
```

### Testing Packages

```yaml
dev_dependencies:
  mocktail: ^1.0.0
```

---

## 🎓 دروس ومقالات

### Official Guides

- [Getting Started](https://riverpod.dev/docs/introduction/getting_started)
- [Migration from Provider](https://riverpod.dev/docs/from_provider/motivation)
- [Essential Concepts](https://riverpod.dev/docs/concepts/providers)

### Community Articles

- [Riverpod 2.0 - The Ultimate Guide](https://codewithandrea.com/articles/flutter-state-management-riverpod/)
- [AsyncNotifier Deep Dive](https://codewithandrea.com/articles/async-notifier-riverpod/)
- [Testing with Riverpod](https://codewithandrea.com/articles/flutter-test-riverpod/)

---

## 🎥 فيديوهات تعليمية

### Official YouTube Channel

- [Riverpod YouTube Playlist](https://www.youtube.com/playlist?list=PLjxrf2q8roU23XGwz3Km7sQZFTdB996iG)

### Community Channels

- **Code with Andrea** - شروحات شاملة لـ Riverpod
- **Reso Coder** - أمثلة عملية
- **Flutter Explained** - مفاهيم متقدمة

---

## 🛠️ أدوات مساعدة

### VS Code Extensions

- **Flutter Riverpod Snippets** - اختصارات سريعة
- **Dart** - دعم Dart
- **Flutter** - دعم Flutter

### Code Generation

```bash
# Watch mode - rebuilds automatically
dart run build_runner watch --delete-conflicting-outputs

# One-time build
dart run build_runner build --delete-conflicting-outputs
```

### Linting

```yaml
# analysis_options.yaml
analyzer:
  plugins:
    - custom_lint

custom_lint:
  rules:
    - riverpod_final_provider: true
    - provider_dependencies: true
```

---

## 👥 المجتمع

### Discord

- [Riverpod Discord Server](https://discord.gg/Bbumvej) - مناقشات مباشرة ومساعدة

### Stack Overflow

- Tag: `[flutter-riverpod]`
- ابحث عن الأسئلة والأجوبة الموجودة قبل طرح سؤال جديد

### GitHub Discussions

- [Riverpod Discussions](https://github.com/rrousselGit/riverpod/discussions) - مناقشات طويلة واقتراحات

---

## 📱 أمثلة تطبيقات حقيقية

### Official Examples

- [Counter App](https://github.com/rrousselGit/riverpod/tree/master/examples/counter)
- [Todo App](https://github.com/rrousselGit/riverpod/tree/master/examples/todos)
- [Marvel API](https://github.com/rrousselGit/riverpod/tree/master/examples/marvel)

### Community Examples

- [Grocery Shopping App](https://github.com/bizz84/starter_architecture_flutter_firebase)
- [E-commerce App](https://github.com/bizz84/flutter-firebase-masterclass)

---

## 📚 كتب إلكترونية

### [Flutter & Firebase Masterclass](https://codewithandrea.com/courses/flutter-firebase-masterclass/)
دورة شاملة تستخدم Riverpod في تطبيق e-commerce كامل.

### [Flutter in Action](https://www.manning.com/books/flutter-in-action)
كتاب يغطي state management بما فيه Riverpod.

---

## 🔧 Best Practices Repositories

### [Very Good Ventures - Architecture](https://github.com/VeryGoodOpenSource/very_good_cli)
قوالب ومعايير لبنية المشاريع.

### [Reso Coder - Clean Architecture](https://github.com/ResoCoder/flutter-tdd-clean-architecture-course)
تطبيق Clean Architecture مع state management.

---

## 🌐 مواقع مفيدة

### [pub.dev](https://pub.dev)
البحث عن packages Flutter.

### [Flutter Documentation](https://docs.flutter.dev)
التوثيق الرسمي لـ Flutter.

### [Dart Documentation](https://dart.dev/guides)
دليل لغة Dart.

---

## 📊 مقارنات

### [State Management Comparison](https://docs.flutter.dev/development/data-and-backend/state-mgmt/options)
مقارنة رسمية من Flutter لحلول state management المختلفة.

### [Riverpod vs Provider](https://riverpod.dev/docs/from_provider/motivation)
لماذا تم إنشاء Riverpod كتطوير لـ Provider.

---

## 🔄 Migration Guides

### From Provider
- [Official Migration Guide](https://riverpod.dev/docs/from_provider/motivation)
- [Code Examples](https://riverpod.dev/docs/from_provider/provider_vs_riverpod)

### From Bloc
- [Bloc to Riverpod Migration](https://codewithandrea.com/articles/flutter-state-management-riverpod/)

### From GetX
- Community articles on Reddit and Medium

---

## 🐛 Debugging Tools

### Flutter DevTools
- [DevTools Guide](https://docs.flutter.dev/development/tools/devtools/overview)
- Provider Inspector للـ Riverpod

### Logging

```dart
class LoggerObserver extends ProviderObserver {
  @override
  void didUpdateProvider(
    ProviderBase provider,
    Object? previousValue,
    Object? newValue,
    ProviderContainer container,
  ) {
    print('''
{
  "provider": "${provider.name ?? provider.runtimeType}",
  "newValue": "$newValue"
}''');
  }
}

void main() {
  runApp(
    ProviderScope(
      observers: [LoggerObserver()],
      child: MyApp(),
    ),
  );
}
```

---

## 📋 Cheat Sheets

### Quick Reference

```dart
// Basic Provider
@riverpod
int counter(CounterRef ref) => 0;

// Notifier
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;
  void increment() => state++;
}

// AsyncNotifier
@riverpod
class Todos extends _$Todos {
  @override
  Future<List<Todo>> build() async {
    return await api.getTodos();
  }
}

// Family (with parameters)
@riverpod
Future<Product> product(ProductRef ref, String id) async {
  return await api.getProduct(id);
}

// Watch (in build)
final counter = ref.watch(counterProvider);

// Read (in methods)
final counter = ref.read(counterProvider);

// Listen (side effects)
ref.listen(counterProvider, (previous, next) {
  print('Counter changed from $previous to $next');
});
```

---

## 🔐 Security Resources

### [OWASP Mobile Security](https://owasp.org/www-project-mobile-security/)
أفضل الممارسات للأمان في التطبيقات.

### [Flutter Security Best Practices](https://docs.flutter.dev/security)
نصائح أمان من Flutter الرسمي.

---

## 💡 نصيحة نهائية

**ابدأ من التوثيق الرسمي دائماً** - riverpod.dev هو أفضل مصدر محدث ودقيق.

عند مواجهة مشكلة:
1. ابحث في التوثيق الرسمي
2. ابحث في GitHub Issues
3. ابحث في Stack Overflow
4. اسأل في Discord
5. افتح issue جديد في GitHub

</div>
