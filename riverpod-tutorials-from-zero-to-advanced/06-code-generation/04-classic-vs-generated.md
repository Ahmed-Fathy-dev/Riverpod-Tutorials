<div dir="rtl">

# Classic vs Code Generation - المقارنة التفصيلية

**المستوى**: 🟡 متوسط

## 📌 المفاهيم الأساسية

في الملف ده هنقارن بين:
- Classic Syntax (Manual)
- Code Generation Syntax (Auto-generated)
- نفس الأمثلة بالطريقتين
- متى تستخدم كل طريقة

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تفهم الفرق الحقيقي بين الطريقتين
- تختار الطريقة المناسبة لمشروعك
- تقرأ وتفهم كود Riverpod بالطريقتين

---

## 1️⃣ مقارنة شاملة: نفس المثال بالطريقتين

### مثال 1: Simple Provider

</div>

```dart
// ========================================
// CLASSIC SYNTAX (Manual)
// ========================================
final messageProvider = Provider<String>((ref) {
  return 'Hello World';
});

// Usage
final message = ref.watch(messageProvider);

// ========================================
// CODE GENERATION (Auto-generated)
// ========================================
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'message.g.dart';

@riverpod
String message(MessageRef ref) {
  return 'Hello World';
}

// Usage - SAME!
final message = ref.watch(messageProvider);
```

<div dir="rtl">

**الفرق:**
- Classic: انت بتكتب الـ provider declaration يدوياً
- Generated: بتكتب function بس، والـ provider بيتولد تلقائياً

### مثال 2: FutureProvider

</div>

```dart
// ========================================
// CLASSIC SYNTAX
// ========================================
final userProvider = FutureProvider<User>((ref) async {
  final response = await http.get(Uri.parse('https://api.example.com/user'));
  return User.fromJson(jsonDecode(response.body));
});

// ========================================
// CODE GENERATION
// ========================================
@riverpod
Future<User> user(UserRef ref) async {
  final response = await http.get(Uri.parse('https://api.example.com/user'));
  return User.fromJson(jsonDecode(response.body));
}

// Usage - SAME for both!
final userAsync = ref.watch(userProvider);
```

<div dir="rtl">

### مثال 3: Provider with Parameters

</div>

```dart
// ========================================
// CLASSIC SYNTAX - Need .family!
// ========================================
final todoProvider = FutureProvider.family<Todo, String>((ref, id) async {
  return await api.getTodo(id);
});

// Usage
final todo = ref.watch(todoProvider('123'));

// ========================================
// CODE GENERATION - No .family needed!
// ========================================
@riverpod
Future<Todo> todo(TodoRef ref, String id) async {
  return await api.getTodo(id);
}

// Usage - SAME!
final todo = ref.watch(todoProvider('123'));
```

<div dir="rtl">

**الفرق الكبير:**
- Classic: محتاج تستخدم `.family` modifier
- Generated: Parameters مباشرة في الـ function!

### مثال 4: NotifierProvider (Complex State)

</div>

```dart
// ========================================
// CLASSIC SYNTAX
// ========================================

// Step 1: Define Notifier class
class CounterNotifier extends Notifier<int> {
  @override
  int build() => 0;

  void increment() => state++;
  void decrement() => state--;
}

// Step 2: Manually declare provider
final counterProvider = NotifierProvider<CounterNotifier, int>(
  () => CounterNotifier(),
);

// ========================================
// CODE GENERATION
// ========================================

import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'counter.g.dart';

@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() => state++;
  void decrement() => state--;
}

// Provider is auto-generated in counter.g.dart!

// ========================================
// Usage - SAME for both!
// ========================================
final count = ref.watch(counterProvider);
ref.read(counterProvider.notifier).increment();
```

<div dir="rtl">

**الفرق:**
- Classic: كتبنا **10 سطور** (Class + Provider declaration)
- Generated: كتبنا **7 سطور** (Class بس)
- **توفير 30% من الكود!**

### مثال 5: AsyncNotifierProvider

</div>

```dart
// ========================================
// CLASSIC SYNTAX
// ========================================

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

// ========================================
// CODE GENERATION
// ========================================

@riverpod
class Todos extends _$Todos {
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

// Provider auto-generated!
```

<div dir="rtl">

---

## 2️⃣ AutoDispose: الفرق الأساسي

### Classic Syntax

</div>

```dart
// Default: NO AutoDispose
final messagesProvider = StreamProvider<List<Message>>((ref) {
  return chatService.messagesStream();
});
// This provider lives FOREVER (memory leak risk!)

// Need to add .autoDispose explicitly
final messagesProvider = StreamProvider.autoDispose<List<Message>>((ref) {
  return chatService.messagesStream();
});
// Now it disposes automatically
```

<div dir="rtl">

### Code Generation

</div>

```dart
// Default: YES AutoDispose! ✅
@riverpod
Stream<List<Message>> messages(MessagesRef ref) {
  return chatService.messagesStream();
}
// Automatically disposes when no longer used!

// Want to keep alive? Add parameter
@Riverpod(keepAlive: true)
Stream<List<Message>> messages(MessagesRef ref) {
  return chatService.messagesStream();
}
// Now it lives forever
```

<div dir="rtl">

**الخلاصة:**
- **Classic:** Default = NO AutoDispose (memory leak risk!)
- **Generated:** Default = YES AutoDispose (safer!)

---

## 3️⃣ جدول المقارنة الشامل

| Feature | Classic Syntax | Code Generation |
|---------|---------------|-----------------|
| **Setup** | بس dependencies | Dependencies + build_runner |
| **Boilerplate** | أكتر (manual declarations) | أقل (auto-generated) |
| **Learning Curve** | أسهل للمبتدئين | محتاج فهم code generation |
| **Type Safety** | جيد | أفضل (generated types) |
| **AutoDispose Default** | ❌ لأ | ✅ أيوة |
| **Parameters** | محتاج `.family` | مباشرة في الـ function |
| **Provider Declaration** | يدوي | تلقائي |
| **IDE Support** | جيد | ممتاز (generated types) |
| **Build Time** | أسرع (no generation) | أبطأ (need build_runner) |
| **Recommended by Riverpod** | Supported | ⭐ Recommended |

---

## 4️⃣ مميزات وعيوب كل طريقة

### Classic Syntax

#### ✅ المميزات:
1. **أبسط للمبتدئين** - ما فيش build_runner complexity
2. **Build time أسرع** - no code generation
3. **أسهل للتعلم** - تشوف كل حاجة واضحة
4. **مفيد للفهم** - تفهم إزاي Riverpod بيشتغل من جوا

#### ❌ العيوب:
1. **Boilerplate أكتر** - محتاج تكتب provider declarations يدوياً
2. **AutoDispose مش default** - ممكن تنسى وتعمل memory leaks
3. **Family معقد** - لما عندك parameters كتير
4. **Refactoring أصعب** - لو غيرت أسماء محتاج تغير في أماكن كتير

### Code Generation

#### ✅ المميزات:
1. **Boilerplate أقل بكتير** - 30% less code
2. **AutoDispose by default** - أأمن للـ memory
3. **Parameters بدون family** - syntax أبسط بكتير
4. **Type safety أفضل** - generated types دقيقة
5. **Refactoring أسهل** - الـ generator بيحدث كل حاجة تلقائياً
6. **Recommended officially** - Riverpod بيشجع عليها

#### ❌ العيوب:
1. **Setup أعقد** - محتاج build_runner setup
2. **Build time أبطأ** - محتاج تشغل generator
3. **Learning curve** - محتاج تفهم code generation
4. **Generated files** - ملفات زيادة في الـ project

---

## 5️⃣ أمثلة Side-by-Side

### Example: Shopping Cart

</div>

```dart
// ========================================
// CLASSIC SYNTAX (15 lines)
// ========================================
class CartNotifier extends Notifier<List<CartItem>> {
  @override
  List<CartItem> build() => [];

  void addItem(Product product) {
    state = [...state, CartItem.fromProduct(product)];
  }

  void removeItem(String id) {
    state = state.where((item) => item.id != id).toList();
  }
}

final cartProvider = NotifierProvider<CartNotifier, List<CartItem>>(
  () => CartNotifier(),
);

// ========================================
// CODE GENERATION (12 lines - 20% less!)
// ========================================
@riverpod
class Cart extends _$Cart {
  @override
  List<CartItem> build() => [];

  void addItem(Product product) {
    state = [...state, CartItem.fromProduct(product)];
  }

  void removeItem(String id) {
    state = state.where((item) => item.id != id).toList();
  }
}
// Provider auto-generated!
```

<div dir="rtl">

### Example: Provider with Dependencies

</div>

```dart
// ========================================
// CLASSIC SYNTAX
// ========================================
final userProvider = FutureProvider<User>((ref) async {
  return await authService.getCurrentUser();
});

final userPostsProvider = FutureProvider<List<Post>>((ref) async {
  final user = await ref.watch(userProvider.future);
  return await api.getUserPosts(user.id);
});

// ========================================
// CODE GENERATION - SAME logic!
// ========================================
@riverpod
Future<User> user(UserRef ref) async {
  return await authService.getCurrentUser();
}

@riverpod
Future<List<Post>> userPosts(UserPostsRef ref) async {
  final user = await ref.watch(userProvider.future);
  return await api.getUserPosts(user.id);
}
```

<div dir="rtl">

---

## 6️⃣ متى تستخدم كل طريقة؟

### استخدم Classic Syntax لو:

</div>

```dart
✅ انت لسه بتتعلم Riverpod
✅ عايز تفهم الأساسيات الأول
✅ مشروع تعليمي أو prototype سريع
✅ الفريق مش comfortable مع code generation
✅ عندك legacy code موجود بـ classic syntax
```

<div dir="rtl">

### استخدم Code Generation لو:

</div>

```dart
⭐ بتبدأ مشروع جديد (الأفضل!)
⭐ عايز أقل boilerplate ممكن
⭐ محتاج providers مع parameters كتير
⭐ المشروع كبير ومحتاج scalability
⭐ عايز أفضل type safety
⭐ الفريق comfortable مع code generation
```

<div dir="rtl">

---

## 7️⃣ الانتقال بين الطريقتين

### هل ممكن أخلط بينهم؟

</div>

```dart
// ✅ نعم! ممكن تستخدم الاتنين في نفس المشروع

// Classic provider
final configProvider = Provider<AppConfig>((ref) {
  return AppConfig();
});

// Generated provider
@riverpod
Future<User> user(UserRef ref) async {
  // Can watch classic providers!
  final config = ref.watch(configProvider);
  return await api.getUser(config.apiUrl);
}

// Works perfectly! ✅
```

<div dir="rtl">

**الخلاصة:** مفيش مشكلة تخلط بينهم، لكن الأفضل تختار طريقة واحدة للمشروع كله.

---

## 8️⃣ التوصية النهائية من Riverpod

من [الـ documentation الرسمي](https://riverpod.dev/docs/concepts/about_code_generation):

> **"Code generation is the recommended way to use Riverpod."**

### ليه؟

1. **أقل أخطاء** - الـ generator بيضمن صحة الـ types
2. **أأمن** - AutoDispose by default
3. **أسهل** - parameters بدون family
4. **أسرع في الـ development** - less boilerplate
5. **المستقبل** - الـ features الجديدة هتركز على code generation

---

## 9️⃣ الخلاصة

</div>

```
للتعلم والفهم:
  → ابدأ بـ Classic Syntax (Sections 03-05)
  → افهم المفاهيم الأساسية
  ↓
للمشاريع الحقيقية:
  → استخدم Code Generation (Section 06+)
  → أقل كود، أكتر أمان، أفضل type safety
```

<div dir="rtl">

**الرحلة المثالية:**
1. اتعلم Classic Syntax ✅ (عشان تفهم الأساسيات)
2. افهم إزاي Riverpod بيشتغل ✅
3. انتقل لـ Code Generation ✅ (للمشاريع الحقيقية)
4. استمتع بأقل boilerplate! 🎉

---

## 🔗 الخطوة الجاية

في الملف الجاي هنتعلم **Migration Guide** - إزاي تحول مشروع موجود من Classic لـ Code Generation:
- خطوات التحويل
- أمثلة عملية
- Common pitfalls

جاهز؟ يلا نكمل! 🚀

---

## 📚 المصادر

- [About Code Generation | Riverpod](https://riverpod.dev/docs/concepts/about_code_generation)
- [Riverpod 2.0 Migration Guide](https://riverpod.dev/docs/migration/0.14.0_to_1.0.0)
- [Code Generation Best Practices](https://codewithandrea.com/articles/flutter-riverpod-generator/)

</div>
