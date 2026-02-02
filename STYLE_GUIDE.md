# Style Guide - دليل الكتابة

## ⚠️ القاعدة الذهبية الأهم

### 🚫 ممنوع الهبد والتخمين!

**القاعدة:** لو مش متأكد 100% من أي معلومة تقنية - **ما تكتبهاش من خيالك!**

#### الخطوات الصحيحة:

1. **🔍 ارجع للمصادر الرسمية أولاً**
   - لو بتكتب عن Provider package → اقرأ من [GitHub الرسمي](https://github.com/rrousselGit/provider) أو [pub.dev](https://pub.dev/packages/provider)
   - لو بتكتب عن Riverpod → اقرأ من [riverpod.dev](https://riverpod.dev)
   - لو بتكتب عن Flutter → اقرأ من [docs.flutter.dev](https://docs.flutter.dev)
   - لو بتكتب عن Dart → اقرأ من [dart.dev](https://dart.dev)

2. **✅ تأكد من الـ Syntax الصحيح**
   - شوف أمثلة من المصدر الرسمي
   - جرب الكود لو ممكن
   - ما تخلطش بين packages مختلفة (مثلاً Provider vs Riverpod)

3. **📝 اكتب بثقة بعد التأكد**
   - بعد ما تتأكد من المصدر الرسمي، اكتب بثقة
   - ضيف رابط المصدر في قسم "المصادر" آخر الملف

#### أمثلة للأخطاء اللي حصلت (عشان ما تتكررش):

❌ **خطأ 1:** كتابة عن StateProvider في Provider package
- **المشكلة:** StateProvider موجود في Riverpod فقط، مش في Provider package
- **الحل:** كان لازم أراجع [Provider documentation](https://pub.dev/packages/provider) الأول

❌ **خطأ 2:** كتابة إن FutureProvider بيرجع AsyncSnapshot
- **المشكلة:** FutureProvider في Provider package بيرجع `T?` مش AsyncSnapshot
- **الحل:** كان لازم أقرأ [FutureProvider API documentation](https://pub.dev/documentation/provider/latest/provider/FutureProvider-class.html)

❌ **خطأ 3:** استخدام global variable syntax للـ providers
- **المشكلة:** ده Riverpod syntax، مش Provider package syntax
- **الحل:** Provider package بيستخدم Providers كـ Widgets في الـ tree

#### الخلاصة:

```
❌ تخمين → كتابة معلومات غلط → المتعلم يتعلم غلط
✅ مصادر رسمية → كتابة معلومات صح → المتعلم يتعلم صح
```

**ملحوظة مهمة:** لو محتاج تستخدم WebFetch أو WebSearch عشان تتأكد من معلومة - **استخدمهم قبل ما تكتب!**

---

## 📌 القواعد الأساسية

### 1. RTL Direction (اتجاه الكتابة من اليمين)

**القاعدة الذهبية:** كل سطر لازم يبدأ بنص عربي - مش رقم، مش رمز، مش مسافة.

#### ✅ صح:

```markdown
<div dir="rtl">

في المثال ده هنشوف إزاي نستخدم Provider:

</div>

```dart
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;
}
```

<div dir="rtl">

الكود ده بيعمل provider بسيط للعداد.
```

#### ❌ غلط:

```markdown
<div dir="rtl">

1. في المثال ده...  ❌ (بدأ برقم)
- في المثال ده...   ❌ (بدأ بشرطة)
  في المثال ده...   ❌ (بدأ بمسافة)
```

#### الحل:

```markdown
<div dir="rtl">

**أولاً:** في المثال ده...  ✅
**النقطة الأولى:** في المثال ده...  ✅
في المثال ده:  ✅
- النقطة الأولى: في المثال ده...  ✅
```

---

### 2. المصطلحات التقنية

**القاعدة:** متترجمش المصطلحات التقنية - اكتبها بالإنجليزي مع شرح بسيط بالعربي.

#### ✅ صح:

```markdown
في الملف ده هنتكلم عن **State Management** - وده مصطلح بيعني إزاي نتحكم في بيانات التطبيق ونشاركها بين الـ widgets.

الـ **Provider** ده زي الحاوية (container) اللي بتحفظ data وتشاركها مع الـ widgets اللي محتاجاها.

**Immutability** معناها إن البيانات ما بتتغيرش - بنعمل نسخة جديدة بدل ما نعدل القديمة.
```

#### ❌ غلط:

```markdown
في الملف ده هنتكلم عن إدارة الحالة  ❌

المزود ده زي الحاوية...  ❌

الثبات معناها...  ❌
```

#### قائمة المصطلحات الشائعة:

| المصطلح | الاستخدام الصحيح | ❌ تجنب |
|---------|------------------|---------|
| State Management | State Management (إدارة بيانات التطبيق) | إدارة الحالة |
| Provider | Provider | مزود / موفر |
| Widget | Widget | عنصر واجهة |
| State | State | حالة |
| Immutability | Immutability (البيانات اللي ما بتتغيرش) | الثبات |
| Build Context | BuildContext | سياق البناء |
| Lifecycle | Lifecycle (دورة الحياة) | دورة الحياة |
| Dependency Injection | Dependency Injection (DI) | حقن التبعيات |
| Cache | Cache | ذاكرة تخزين مؤقت |

**ملحوظة:** لو المصطلح معروف ومفهوم (زي "function" أو "class")، استخدمه عادي بدون شرح.

---

### 3. مستوى الأمثلة حسب القسم

**القاعدة:** كل قسم له مستوى معين - ما تستخدمش مفاهيم لسه ما اتشرحتش.

#### Section 00-02: مستوى تمهيدي

**المسموح:**
- `setState`
- مفاهيم عامة عن State Management
- أمثلة بسيطة جداً
- شرح نظري للمقارنات

**الممنوع:**
- تفاصيل Providers
- Code generation (`@riverpod`)
- Notifier classes
- أي implementation details

**مثال صح:**

```dart
// Section 02: مقارنة بسيطة
// BLoC approach (مفهوم عام)
// تبعت event → تستقبل state جديد

// Riverpod approach (مفهوم عام)
// تقرأ provider → يتحدث تلقائياً
```

**مثال غلط:**

```dart
// ❌ Section 02 - تفاصيل كتير!
@riverpod
class TodosList extends _$TodosList {
  @override
  Future<List<Todo>> build() async {
    return await api.getTodos();
  }
}
```

#### Section 03-05: مستوى مبتدئ (Classic Syntax - Riverpod 3)

**المسموح:**
- Classic Provider syntax فقط (بدون @riverpod)
- Provider, StateProvider, FutureProvider, StreamProvider
- **NotifierProvider, AsyncNotifierProvider, StreamNotifierProvider** ✅
- **Notifier<T>, AsyncNotifier<T>, StreamNotifier<T> classes** ✅
- `ref.watch`, `ref.read`
- ConsumerWidget
- أمثلة بسيطة ومتوسطة (Counter, Todo, Shopping Cart)

**الممنوع:**
- Code generation (`@riverpod`) ❌ (مخصوص لـ Section 06+)
- `build_runner`, `riverpod_generator` ❌
- `_$` generated classes ❌
- Family modifier (حتى Section 07+)
- AutoDispose details (إلا لو شرح نظري بسيط)
- StateNotifier (Legacy - Riverpod 2.x) ❌

**ملحوظة مهمة:**
- الهدف: تعليم Riverpod 3 patterns بالـ classic syntax
- NotifierProvider هو الطريقة الصحيحة في Riverpod 3 (مش StateNotifier)
- الفرق الوحيد عن Section 07+: هنا بنكتب manual، هناك بنستخدم code generation

#### Section 06: Code Generation Introduction

**المسموح:**
- شرح build_runner setup
- مقارنة classic vs code generation
- Migration من classic لـ code generation
- أول أمثلة بسيطة بالـ `@riverpod`

#### Section 07+: مستوى متوسط ومتقدم (Code Generation)

**المسموح:**
- كل الـ modifiers
- Notifier و AsyncNotifier
- Advanced patterns
- Performance optimization
- Complex examples

---

### 4. Riverpod Syntax Progression (CRITICAL!)

**القاعدة الأهم:** في تسلسل محدد لتعليم Riverpod - لازم نلتزم بيه!

#### 🔵 Phase 1: Classic Syntax (Sections 00-05) - Riverpod 3 Patterns

**المسموح في Sections 00-05:**

```dart
// Provider - for read-only/computed values
final nameProvider = Provider<String>((ref) {
  return 'Ahmed';
});

final doubledProvider = Provider<int>((ref) {
  final count = ref.watch(counterProvider);
  return count * 2;
});

// StateProvider - for simple primitive mutable state
final counterProvider = StateProvider<int>((ref) => 0);

// Usage
ref.read(counterProvider.notifier).state = 5;

// NotifierProvider - for complex synchronous state (Riverpod 3) ✅
class CounterNotifier extends Notifier<int> {
  @override
  int build() => 0;

  void increment() => state++;
  void decrement() => state--;
  void reset() => state = 0;
}

final counterNotifierProvider = NotifierProvider<CounterNotifier, int>(
  () => CounterNotifier(),
);

// AsyncNotifierProvider - for complex async state (Riverpod 3) ✅
class TodosNotifier extends AsyncNotifier<List<Todo>> {
  @override
  Future<List<Todo>> build() async {
    return await api.getTodos();
  }

  Future<void> addTodo(String title) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      await api.addTodo(title);
      return await api.getTodos();
    });
  }
}

final todosProvider = AsyncNotifierProvider<TodosNotifier, List<Todo>>(
  () => TodosNotifier(),
);

// FutureProvider - for one-time async data (no methods needed)
final userProvider = FutureProvider<User>((ref) async {
  return await api.getUser();
});

// StreamProvider - for continuous data streams
final messagesProvider = StreamProvider<Message>((ref) {
  return chatService.messages();
});
```

**الهدف:** المتعلم يفهم Riverpod 3 patterns (Notifier, AsyncNotifier) بدون تعقيد code generation.

---

#### 🟢 Phase 2: Code Generation Introduction (Section 06)

**في Section 06 بس - نشرح الانتقال:**

```dart
// Before: Classic syntax (Manual NotifierProvider)
class CounterNotifier extends Notifier<int> {
  @override
  int build() => 0;

  void increment() => state++;
  void decrement() => state--;
}

final counterProvider = NotifierProvider<CounterNotifier, int>(
  () => CounterNotifier(),
);

// After: Code generation (Same logic, auto-generated provider)
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() => state++;
  void decrement() => state--;
}
```

**نشرح:**
- إزاي نعمل setup لـ build_runner و riverpod_generator
- الفرق بين الطريقتين (manual vs code generation)
- مميزات code generation (less boilerplate, auto-generated, family support)
- نفس الـ logic والـ patterns - بس code generation بيولد الـ provider تلقائياً

---

#### 🟡 Phase 3: Modern Riverpod 3 (Sections 07+)

**المسموح في Sections 07+:**

```dart
// Simple provider (read-only)
@riverpod
int doubled(DoubledRef ref) {
  final count = ref.watch(counterProvider);
  return count * 2;
}

// Notifier - for synchronous mutable state
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() => state++;
  void decrement() => state--;
  void reset() => state = 0;
}

// AsyncNotifier - for asynchronous mutable state
@riverpod
class Todos extends _$Todos {
  @override
  Future<List<Todo>> build() async {
    return await api.getTodos();
  }

  Future<void> addTodo(Todo todo) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      await api.addTodo(todo);
      return await api.getTodos();
    });
  }
}

// FutureProvider with parameters (Family)
@riverpod
Future<Product> product(ProductRef ref, String id) async {
  return await api.getProduct(id);
}

// StreamProvider
@riverpod
Stream<Message> messages(MessagesRef ref) {
  return chatService.messages();
}
```

**الهدف:** استخدام أحدث وأفضل practices في Riverpod 3.

---

#### 🔴 FORBIDDEN - StateNotifier (Legacy!)

**ممنوع تماماً** في كل الأقسام (ما عدا Migration Guide):

```dart
// ❌ StateNotifier - THIS IS LEGACY! DO NOT USE!
class CounterNotifier extends StateNotifier<int> {
  CounterNotifier() : super(0);
  void increment() => state = state + 1;
}

final counterProvider = StateNotifierProvider<CounterNotifier, int>((ref) {
  return CounterNotifier();
});
```

**ليه ممنوع؟**
- StateNotifier كان في Riverpod 2.x
- Riverpod 3 عنده **Notifier** و **AsyncNotifier** أفضل
- الـ official docs بتقول استخدم Notifier بدلها

**الاستثناء الوحيد:** Section 13 (Migration Guides) - نشرح إزاي تعمل migrate من StateNotifier لـ Notifier.

---

#### 📋 Summary - متى تستخدم إيه؟

| القسم | Syntax المسموح | الهدف |
|------|----------------|-------|
| **00-02** | مفاهيم نظرية، pseudo-code | فهم State Management |
| **03-05** | Classic syntax (Provider, StateProvider, NotifierProvider, AsyncNotifierProvider) | تعلم Riverpod 3 patterns بدون code generation |
| **06** | Classic + Code Generation (المقارنة) | الانتقال من manual لـ code generation |
| **07+** | Code Generation (@riverpod with Notifier, AsyncNotifier) | Modern Riverpod 3 مع code generation |
| **13** | Migration: StateNotifier → Notifier | Legacy migration فقط |

---

#### ⚠️ ملحوظات مهمة جداً

**ملحوظة 1:** لو Section 00 فيه Quick Start، لازم يكون **classic syntax** - مش code generation! Quick start لازم يكون بسيط بدون build_runner setup.

**ملحوظة 2:** في Section 02 (Comparisons)، الملفات بتقارن بين **حلول State Management المختلفة** (Provider package, BLoC, Riverpod) - مش تعليم تفصيلي.
  - **Provider** يعني: Provider package (ChangeNotifier, ChangeNotifierProvider)
  - **Riverpod** يعني: الحل الجديد (NotifierProvider, AsyncNotifierProvider)

**ملحوظة 3:** Riverpod 3 عنده طريقتين لكتابة نفس الـ logic:
  - **Classic syntax (Sections 03-05):** Manual NotifierProvider declarations
  - **Code generation (Sections 06+):** @riverpod with auto-generated providers
  - **نفس الـ patterns** - بس الطريقة مختلفة!

**ملحوظة 4:** StateProvider **مقبول** للأمثلة البسيطة جداً (primitives). لكن NotifierProvider هو الـ **recommended way** في Riverpod 3 للـ state management.

---

#### ✅ قاعدة ذهبية

> **قبل ما تكتب أي مثال، اسأل نفسك:**
> - القسم ده رقم كام؟
> - المتعلم وصل لـ code generation ولا لسه؟
> - هل المثال ده مناسب لمستوى القسم؟
>
> **لو مش متأكد → استخدم الـ syntax الأبسط!**

---

### 5. أسلوب الكتابة

**القاعدة:** اكتب بالعامية المصرية كإنك بتشرح لصاحبك.

#### ✅ صح:

```markdown
دلوقتي هنشوف إزاي نعمل provider بسيط.

المثال ده بيوضح الفكرة بشكل أبسط.

لو حصل error، الـ widget هيعرض رسالة للمستخدم.

خد بالك إن الـ state لازم يكون immutable.
```

#### ❌ غلط:

```markdown
الآن سنرى كيفية إنشاء provider بسيط.  ❌ (فصحى)

هذا المثال يوضح الفكرة بشكل أبسط.  ❌ (فصحى)
```

#### كلمات شائعة:

| ✅ استخدم | ❌ تجنب |
|----------|---------|
| دلوقتي | الآن |
| هنشوف | سنرى |
| إزاي | كيف / كيفية |
| لو | إذا |
| عشان | لكي / من أجل |
| ممكن | يمكن |
| خد بالك | انتبه |
| لازم | يجب |

---

### 6. تنظيم الملف

**البنية المطلوبة:**

```markdown
<div dir="rtl">

# عنوان الملف

**المستوى**: 🟢 مبتدئ / 🟡 متوسط / 🔴 متقدم

## 📌 المفاهيم الأساسية

في الملف ده هنتكلم عن:
- نقطة 1
- نقطة 2
- نقطة 3

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- هدف 1
- هدف 2
- هدف 3

---

## 🔍 القسم الأول

شرح القسم بالعربي...

</div>

```dart
// Code example with English comments only
@riverpod
class Example extends _$Example {
  @override
  int build() => 0;
}
```

<div dir="rtl">

شرح الكود بالعربي...

---

## 📝 ملخص

نقاط رئيسية:
- نقطة 1
- نقطة 2

---

## 🔗 الخطوة الجاية

في الملف الجاي هنتكلم عن...

---

## 📚 المصادر

- [Link 1](url)
- [Link 2](url)

</div>
```

---

### 7. Code Comments

**القاعدة:** كل الـ comments في الكود بالإنجليزي فقط.

#### ✅ صح:

```dart
@riverpod
class Counter extends _$Counter {
  @override
  int build() {
    // Initialize counter at 0
    return 0;
  }

  void increment() {
    // Increment the counter
    state++;
  }
}
```

#### ❌ غلط:

```dart
@riverpod
class Counter extends _$Counter {
  @override
  int build() {
    // نبدأ العداد من 0  ❌
    return 0;
  }
}
```

---

### 8. Tables والقوائم

**القاعدة:** الجداول والقوائم لازم يكونوا داخل `<div dir="rtl">`.

#### ✅ صح:

```markdown
<div dir="rtl">

| الميزة | Riverpod | BLoC |
|-------|----------|------|
| سهولة التعلم | ✅ سهل | 🟡 متوسط |
| Performance | 🟢 ممتاز | 🟢 ممتاز |

</div>
```

---

### 9. Emojis

**القاعدة:** استخدم emojis بس في:
- عناوين الأقسام (📌, 🎯, 🔍)
- تحديد المستوى (🟢, 🟡, 🔴)
- الأيقونات الوظيفية (✅, ❌, ⚠️)

**ممنوع** emojis في:
- الكود
- الشرح العادي
- المصطلحات التقنية

---

## 📋 Checklist قبل الـ Commit

قبل ما تعمل commit لأي ملف، تأكد من:

**العامة:**
- [ ] كل سطر نص بيبدأ بحرف عربي (RTL)
- [ ] المصطلحات التقنية بالإنجليزي مع شرح
- [ ] الكتابة بالعامية المصرية
- [ ] Code comments بالإنجليزي فقط
- [ ] البنية منظمة حسب Template
- [ ] في قسم "الخطوة الجاية"
- [ ] في قسم "المصادر"

**حسب رقم القسم:**
- [ ] Section 00-02: مفاهيم نظرية فقط (لا implementation details)
- [ ] Section 03-05: Classic syntax (NotifierProvider ✅, لا @riverpod ❌)
- [ ] Section 06: Classic + Code Generation (المقارنة بين الطريقتين)
- [ ] Section 07+: Code Generation (@riverpod + Notifier/AsyncNotifier)
- [ ] لا StateNotifier (Legacy) في أي مكان (إلا Migration Guide)
- [ ] لا `_$` generated classes في Section 03-05
- [ ] مستوى الأمثلة مناسب للقسم

---

## 🔄 تحديث الملفات القديمة

عند مراجعة ملف قديم:

1. **افتح الملف**
2. **حدد رقم القسم** (Section number)
3. **راجع كل قاعدة** من القواعد فوق
4. **صلح المشاكل حسب القسم**:

   **لو Section 00-02:**
   - امسح أي Riverpod implementation details
   - استخدم pseudo-code أو مفاهيم نظرية
   - ترجمات → مصطلحات إنجليزي
   - RTL issues → اتجاه صحيح

   **لو Section 03-05:**
   - `@riverpod` → Classic syntax (Manual NotifierProvider)
   - StateNotifier (Legacy) → Notifier + NotifierProvider ✅
   - `_$` generated classes → Manual provider declarations
   - Keep Notifier, AsyncNotifier, StreamNotifier ✅
   - Keep NotifierProvider, AsyncNotifierProvider, StreamNotifierProvider ✅
   - StateProvider: مقبول للـ primitives البسيطة
   - امسح أي build_runner/code generation setup
   - ترجمات → مصطلحات إنجليزي
   - RTL issues → اتجاه صحيح

   **لو Section 06:**
   - اعرض الطريقتين (Classic + Code Generation)
   - اشرح المقارنة والانتقال
   - StateNotifier → Notifier

   **لو Section 07+:**
   - StateNotifier → Notifier
   - Classic syntax → Code generation
   - استخدم `@riverpod` مع Notifier/AsyncNotifier
   - ترجمات → مصطلحات إنجليزي
   - RTL issues → اتجاه صحيح

5. **Commit** بـ message واضح عن التعديلات

---

## ✅ أمثلة كاملة

### مثال: Section 02 (Comparison - مبتدئ)

```markdown
<div dir="rtl">

# المقارنة بين Riverpod و BLoC

**المستوى**: 🟡 متوسط

## 📌 المفاهيم الأساسية

في الملف ده هنقارن بين Riverpod و BLoC من حيث:
- الأسلوب (Pattern)
- سهولة الاستخدام
- Performance
- Use cases

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تفهم الفرق الأساسي بين النهجين
- تختار الحل المناسب لمشروعك
- تعرف مميزات وعيوب كل حل

---

## 🔍 الفرق الأساسي

### Riverpod - Reactive Pattern

الفكرة الأساسية في Riverpod إنك بتقرأ (read) الـ data، والـ UI بتتحدث تلقائياً لما الـ data تتغير.

**مثال بسيط جداً:**

</div>

```dart
// Riverpod: Direct state access
// The widget rebuilds automatically when state changes
final count = ref.watch(counterProvider);
```

<div dir="rtl">

### BLoC - Event-Driven Pattern

الفكرة الأساسية في BLoC إنك بتبعت Events، والـ BLoC بيرد عليك بـ States جديدة.

**مثال بسيط جداً:**

</div>

```dart
// BLoC: Send events, receive states
bloc.add(IncrementEvent());
// BLoC processes event and returns new state
```

<div dir="rtl">

**الفرق بإختصار:**
- **Riverpod:** اقرأ → تحديث تلقائي
- **BLoC:** ابعت event → استقبل state

---

## 📊 المقارنة

| الجانب | Riverpod | BLoC |
|-------|----------|------|
| **التعلم** | أسهل للمبتدئين | Learning curve أعلى |
| **الكود** | أقل boilerplate | Boilerplate أكثر |
| **Pattern** | Reactive | Event-Driven |
| **Best for** | معظم التطبيقات | Enterprise apps |

---

## 💡 متى تستخدم إيه؟

### استخدم Riverpod لو:
- تطبيق جديد
- عايز حل سريع وبسيط
- الـ team مش كلهم خبرة

### استخدم BLoC لو:
- محتاج audit trail (تسجيل كل action)
- عندك requirements معقدة للـ state flow
- الـ team متمكن من Reactive programming

---

## 📝 ملخص

**Riverpod:**
- ✅ أبسط وأسرع
- ✅ Boilerplate أقل
- 🟡 Reactive pattern

**BLoC:**
- ✅ Event-driven (كل action موثق)
- ✅ Testable جداً
- 🟡 Boilerplate أكثر

**الخلاصة:** للمشاريع العادية → Riverpod، للـ Enterprise → BLoC

---

## 🔗 الخطوة الجاية

في الملف الجاي هنتعلم إزاي نختار بين كل الحلول المتاحة.

---

## 📚 المصادر

- [Riverpod Documentation](https://riverpod.dev)
- [BLoC Documentation](https://bloclibrary.dev)

</div>
```

---

## 🔍 Fact-Checking Process (عملية التحقق من المعلومات)

### متى تستخدم WebFetch/WebSearch؟

**استخدم WebFetch أو WebSearch في الحالات دي:**

1. **قبل ما تكتب عن أي package أو library**
   - مثال: لو هتكتب عن Provider package → اعمل WebFetch للـ GitHub repo والـ pub.dev documentation
   - مثال: لو هتكتب عن flutter_hooks → اقرأ الـ README والـ examples

2. **لما تكون مش متأكد من الـ API**
   - مثال: FutureProvider بيرجع إيه؟ → WebFetch لصفحة الـ API documentation
   - مثال: إيه الـ parameters المتاحة؟ → شوف الـ constructor documentation

3. **لما تيجي تكتب code examples**
   - شوف الأمثلة الرسمية من الـ repository
   - ما تخترعش syntax من خيالك

4. **لما تكتب عن features جديدة**
   - التاريخ الحالي: 2026-02-02
   - استخدم السنة الصح في البحث (مثلاً: "Riverpod 3.0 2026")

### المصادر الرسمية لكل موضوع:

| الموضوع | المصدر الرسمي | رابط مباشر |
|---------|---------------|-------------|
| **Riverpod** | riverpod.dev | https://riverpod.dev/docs/introduction/getting_started |
| **Provider package** | GitHub + pub.dev | https://github.com/rrousselGit/provider |
| **Flutter** | docs.flutter.dev | https://docs.flutter.dev |
| **Dart** | dart.dev | https://dart.dev/guides |
| **BLoC** | bloclibrary.dev | https://bloclibrary.dev |
| **flutter_hooks** | pub.dev | https://pub.dev/packages/flutter_hooks |
| **GetX** | pub.dev + GitHub | https://pub.dev/packages/get |
| **MobX** | pub.dev + mobx.dart.dev | https://mobx.netlify.app |

### الخطوات العملية:

```
1. حدد الموضوع اللي هتكتب عنه
   ↓
2. استخدم WebFetch للمصدر الرسمي
   ↓
3. اقرأ الـ API documentation والـ examples
   ↓
4. تأكد من الـ syntax الصحيح
   ↓
5. اكتب المحتوى بثقة
   ↓
6. ضيف المصدر في قسم "المصادر" آخر الملف
```

### Red Flags (علامات تحذيرية):

🚩 **لو لقيت نفسك بتقول:**
- "على ما أفتكر..."
- "ممكن يكون..."
- "في الغالب..."
- "تقريباً..."

➡️ **يبقى STOP وارجع للمصدر الرسمي!**

### Quality Checklist قبل ما تكتب:

- [ ] قرأت المصدر الرسمي؟
- [ ] شفت أمثلة code حقيقية؟
- [ ] متأكد من الـ syntax 100%؟
- [ ] عارف الفرق بين packages مشابهة (مثلاً Provider vs Riverpod)؟
- [ ] الـ return types صح؟
- [ ] الـ parameters names صح؟

**ملحوظة:** لو عملت كل ده، المحتوى هيكون عالي الجودة والمتعلم هيستفيد فعلاً! 🎯

---

**ملحوظة نهائية:** الـ Style Guide ده living document - هنحدثه لما نلاقي patterns جديدة أو improvements.
