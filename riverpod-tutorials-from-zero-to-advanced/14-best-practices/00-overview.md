<div dir="rtl">

# Best Practices - نظرة عامة

**المستوى**: 🟡 متوسط - 🔴 متقدم

## 🎯 الهدف

دليل شامل لأفضل الممارسات في Riverpod لكتابة كود نظيف، قابل للصيانة، وعالي الأداء.

---

## 📊 ملخص سريع

| الممارسة | الأهمية | متى تستخدمها |
|----------|---------|---------------|
| **Feature-based Structure** | ⭐⭐⭐⭐⭐ | مشاريع متوسطة وكبيرة |
| **Naming Conventions** | ⭐⭐⭐⭐ | دائماً - في كل مشروع |
| **تجنب الأخطاء الشائعة** | ⭐⭐⭐⭐⭐ | دائماً - للجميع |
| **Performance Optimization** | ⭐⭐⭐⭐ | عند مشاكل الأداء |
| **Security Best Practices** | ⭐⭐⭐⭐⭐ | دائماً - خاصة في production |

---

## 📚 المحتوى التفصيلي

### 01. تنظيم الكود (Code Organization)

**ما ستتعلمه:**
- Feature-based structure vs Layer-based
- كيف تنظم الـ providers
- أين تضع الملفات المشتركة

**مثال سريع:**
```dart
lib/
├── features/          ← كل feature في مجلد مستقل
│   ├── auth/
│   └── products/
├── core/             ← الأساسيات المشتركة
└── shared/           ← الـ widgets والـ utils المشتركة
```

**📄 التفاصيل:** [01-code-organization.md](./01-code-organization.md)

---

### 02. قواعد التسمية (Naming Conventions)

**ما ستتعلمه:**
- كيف تسمي الـ providers
- قواعد تسمية الـ methods
- أسماء الملفات

**مثال سريع:**
```dart
// ✅ GOOD - Clear names
@riverpod
Future<User> currentUser(CurrentUserRef ref) { ... }

@riverpod
List<Product> filteredProducts(FilteredProductsRef ref) { ... }

// ❌ BAD - Vague names
@riverpod
Future<User> user(UserRef ref) { ... }  // Which user?
```

**📄 التفاصيل:** [02-naming-conventions.md](./02-naming-conventions.md)

---

### 03. الأخطاء الشائعة (Common Pitfalls)

**ما ستتعلمه:**
- 5 أخطاء شائعة يقع فيها الجميع
- كيف تتجنبها
- الحل الصحيح لكل خطأ

**أهم الأخطاء:**
1. ❌ استخدام `ref.watch` في methods
2. ❌ نسيان `.notifier`
3. ❌ عدم معالجة AsyncValue states
4. ❌ Circular dependencies
5. ❌ Memory leaks مع families

**📄 التفاصيل:** [03-common-pitfalls.md](./03-common-pitfalls.md)

---

### 04. تحسين الأداء (Performance Tips)

**ما ستتعلمه:**
- استخدام `.select()` لتقليل rebuilds
- AutoDispose best practices
- Cache strategies
- Optimization techniques

**مثال سريع:**
```dart
// ❌ Rebuilds on any user change
final user = ref.watch(userProvider);
return Text(user.name);

// ✅ Rebuilds only when name changes
final name = ref.watch(userProvider.select((u) => u.name));
return Text(name);
```

**📄 التفاصيل:** [04-performance-tips.md](./04-performance-tips.md)

---

### 05. الأمان (Security)

**ما ستتعلمه:**
- حماية البيانات الحساسة
- التعامل الآمن مع الـ API keys
- Validation للـ user input
- Best practices للأمان

**أهم النقاط:**
- 🔒 لا تضع secrets في الكود
- 🔒 استخدم environment variables
- 🔒 validate كل user input
- 🔒 امسح البيانات الحساسة عند logout

**📄 التفاصيل:** [05-security.md](./05-security.md)

---

### 06. DevTools & Debugging (🆕 Riverpod 3.0)

**ما ستتعلمه:**
- استخدام Riverpod DevTools
- State Inspector و Time-travel debugging
- Dependency Graph visualization
- Custom observers للـ logging

**مثال سريع:**
```dart
// Setup DevTools
runApp(
  ProviderScope(
    observers: [
      if (kDebugMode) RiverpodDevToolsTracker(),
    ],
    child: MyApp(),
  ),
);
```

**📄 التفاصيل:** [06-devtools-debugging.md](./06-devtools-debugging.md)

---

### 07. Lint Rules (🆕 Riverpod 3.0)

**ما ستتعلمه:**
- تثبيت riverpod_lint
- القواعد المهمة
- Auto-fix features
- تخصيص القواعد

**مثال سريع:**
```yaml
# pubspec.yaml
dev_dependencies:
  custom_lint: ^0.6.0
  riverpod_lint: ^2.3.0

# analysis_options.yaml
analyzer:
  plugins:
    - custom_lint
```

**📄 التفاصيل:** [07-lint-rules.md](./07-lint-rules.md)

---

## 🎯 من أين تبدأ؟

### إذا كنت مبتدئ:
1. ابدأ بـ **Common Pitfalls** - تجنب الأخطاء الشائعة
2. ثم **Naming Conventions** - اكتب كود واضح
3. ثم **Code Organization** - نظم مشروعك

### إذا كنت متوسط:
1. ابدأ بـ **Performance Tips** - حسّن الأداء
2. ثم **Code Organization** - استخدم Clean Architecture
3. ثم **Security** - احمِ تطبيقك

### إذا كنت محترف:
اقرأ كل شيء وطبقه! 🚀

---

## 💡 نصيحة ذهبية

> "الكود النظيف ليس رفاهية - هو استثمار في مستقبل مشروعك"

**لا تحاول تطبيق كل شيء دفعة واحدة!**
- طبق best practice واحدة كل مرة
- تعلم من الأخطاء
- حسّن تدريجياً

---

## 🔗 الخطوة الجاية

بعد ما تخلص هذا القسم:
- راجع الـ **Appendix** (القسم 16) - Glossary, FAQ, Troubleshooting
- ابدأ مشروعك الخاص!

</div>
