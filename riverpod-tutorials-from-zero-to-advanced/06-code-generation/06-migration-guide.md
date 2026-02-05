<div dir="rtl">

# Migration Guide - دليل التحويل الكامل

**المستوى**: 🟡 متوسط

## 📌 المفاهيم الأساسية

في الملف ده هنتعلم:
- إزاي تحول مشروع موجود من Classic لـ Code Generation
- الخطوات الصحيحة للـ migration
- أمثلة تحويل لكل نوع provider
- المشاكل الشائعة وحلولها

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تحول أي مشروع من classic لـ generated
- تتجنب المشاكل الشائعة
- تعمل migration تدريجي وآمن

---

## 📋 الخطة الكاملة

### المرحلة 1: الإعداد (Setup)
### المرحلة 2: التحويل التدريجي (Gradual Migration)
### المرحلة 3: التنظيف (Cleanup)

---

## 🔧 المرحلة 1: الإعداد

### Step 1: إضافة الـ Dependencies

</div>

```bash
# Add code generation packages
flutter pub add riverpod_annotation
flutter pub add dev:riverpod_generator
flutter pub add dev:build_runner
```

<div dir="rtl">

### Step 2: تشغيل Build Runner في الـ Background

</div>

```bash
# في terminal منفصل، شغل watch mode
flutter pub run build_runner watch --delete-conflicting-outputs
```

<div dir="rtl">

**ملاحظة:** خليه شغال طول فترة الـ migration!

---

## 🔄 المرحلة 2: التحويل التدريجي

**نصيحة مهمة:** **متحولش كل حاجة مرة واحدة!** حول provider واحد في كل مرة وجرب.

### Pattern 1: تحويل Simple Provider

</div>

```dart
// ========================================
// BEFORE (Classic)
// ========================================
// lib/providers/message_provider.dart

final messageProvider = Provider<String>((ref) {
  return 'Hello World';
});

// ========================================
// AFTER (Generated)
// ========================================
// lib/providers/message_provider.dart

import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'message_provider.g.dart';  // Add this!

@riverpod
String message(MessageRef ref) {  // Changed signature
  return 'Hello World';
}

// Generated provider name: messageProvider (same!)
```

<div dir="rtl">

**الخطوات:**
1. ✅ إضافة imports
2. ✅ إضافة `part` directive
3. ✅ حذف `final xxxProvider =`
4. ✅ إضافة `@riverpod` annotation
5. ✅ تغيير الـ signature: `Type functionName(FunctionNameRef ref)`
6. ✅ شغل build_runner (أو هو شغال فعلاً)

### Pattern 2: تحويل FutureProvider

</div>

```dart
// ========================================
// BEFORE
// ========================================
final userProvider = FutureProvider<User>((ref) async {
  final response = await http.get(Uri.parse('https://api.example.com/user'));
  return User.fromJson(jsonDecode(response.body));
});

// ========================================
// AFTER
// ========================================
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'user_provider.g.dart';

@riverpod
Future<User> user(UserRef ref) async {
  final response = await http.get(Uri.parse('https://api.example.com/user'));
  return User.fromJson(jsonDecode(response.body));
}
```

<div dir="rtl">

**ملاحظة:** الاستخدام في الـ UI **ما اتغيرش**:

</div>

```dart
// Same usage!
final userAsync = ref.watch(userProvider);
```

<div dir="rtl">

### Pattern 3: تحويل StreamProvider

</div>

```dart
// ========================================
// BEFORE
// ========================================
final messagesProvider = StreamProvider<List<Message>>((ref) {
  return FirebaseFirestore.instance
      .collection('messages')
      .snapshots()
      .map((snapshot) => snapshot.docs.map((doc) => Message.fromDoc(doc)).toList());
});

// ========================================
// AFTER
// ========================================
@riverpod
Stream<List<Message>> messages(MessagesRef ref) {
  return FirebaseFirestore.instance
      .collection('messages')
      .snapshots()
      .map((snapshot) => snapshot.docs.map((doc) => Message.fromDoc(doc)).toList());
}
```

<div dir="rtl">

### Pattern 4: تحويل Provider with .autoDispose

</div>

```dart
// ========================================
// BEFORE
// ========================================
final messagesProvider = StreamProvider.autoDispose<List<Message>>((ref) {
  return chatService.messagesStream();
});

// ========================================
// AFTER
// ========================================
// AutoDispose is DEFAULT in code generation!
@riverpod
Stream<List<Message>> messages(MessagesRef ref) {
  return chatService.messagesStream();
}

// If you DON'T want autoDispose:
@Riverpod(keepAlive: true)
Stream<List<Message>> messages(MessagesRef ref) {
  return chatService.messagesStream();
}
```

<div dir="rtl">

**مهم جداً:** Code generation بيعمل **autoDispose by default**!

### Pattern 5: تحويل Provider.family

</div>

```dart
// ========================================
// BEFORE
// ========================================
final todoProvider = FutureProvider.family<Todo, String>((ref, id) async {
  return await api.getTodo(id);
});

// Usage
final todo = ref.watch(todoProvider('123'));

// ========================================
// AFTER
// ========================================
@riverpod
Future<Todo> todo(TodoRef ref, String id) async {
  return await api.getTodo(id);
}

// Usage - SAME!
final todo = ref.watch(todoProvider('123'));
```

<div dir="rtl">

**ملاحظة:** مفيش `.family` - الـ parameters مباشرة!

### Pattern 6: تحويل NotifierProvider

</div>

```dart
// ========================================
// BEFORE
// ========================================
class CounterNotifier extends Notifier<int> {
  @override
  int build() => 0;

  void increment() => state++;
  void decrement() => state--;
}

final counterProvider = NotifierProvider<CounterNotifier, int>(
  () => CounterNotifier(),
);

// ========================================
// AFTER
// ========================================
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'counter.g.dart';

@riverpod
class Counter extends _$Counter {  // Extends _$ClassName
  @override
  int build() => 0;

  void increment() => state++;
  void decrement() => state--;
}

// Provider auto-generated!
```

<div dir="rtl">

**الخطوات:**
1. ✅ إضافة imports و part directive
2. ✅ إضافة `@riverpod` على الـ class
3. ✅ تغيير extends: من `Notifier<T>` لـ `_$ClassName`
4. ✅ حذف الـ provider declaration

**ملاحظة:** اسم الـ class بيتغير! `CounterNotifier` → `Counter`

لكن اسم الـ provider **نفسه**: `counterProvider` ✅

### Pattern 7: تحويل AsyncNotifierProvider

</div>

```dart
// ========================================
// BEFORE
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
// AFTER
// ========================================
@riverpod
class Todos extends _$Todos {  // Note: Extends _$Todos
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
```

<div dir="rtl">

**الكود الداخلي نفسه!** بس الـ declaration مختلف.

### Pattern 8: تحويل Provider مع Dependencies

</div>

```dart
// ========================================
// BEFORE
// ========================================
final userProvider = FutureProvider<User>((ref) async {
  return await authService.getCurrentUser();
});

final userPostsProvider = FutureProvider<List<Post>>((ref) async {
  final user = await ref.watch(userProvider.future);
  return await api.getUserPosts(user.id);
});

// ========================================
// AFTER
// ========================================
@riverpod
Future<User> user(UserRef ref) async {
  return await authService.getCurrentUser();
}

@riverpod
Future<List<Post>> userPosts(UserPostsRef ref) async {
  final user = await ref.watch(userProvider.future);  // SAME!
  return await api.getUserPosts(user.id);
}
```

<div dir="rtl">

**Dependencies بتشتغل نفس الطريقة!** ✅

---

## 📝 مثال كامل: Shopping Cart Migration

### قبل التحويل

</div>

```dart
// lib/providers/cart_provider.dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../models/cart_item.dart';
import '../models/product.dart';

class CartNotifier extends Notifier<List<CartItem>> {
  @override
  List<CartItem> build() => [];

  void addItem(Product product) {
    final existingIndex = state.indexWhere((item) => item.productId == product.id);

    if (existingIndex != -1) {
      state = [
        for (int i = 0; i < state.length; i++)
          if (i == existingIndex)
            state[i].copyWith(quantity: state[i].quantity + 1)
          else
            state[i],
      ];
    } else {
      state = [...state, CartItem.fromProduct(product)];
    }
  }

  void removeItem(String productId) {
    state = state.where((item) => item.productId != productId).toList();
  }

  double get totalPrice {
    return state.fold(0.0, (sum, item) => sum + (item.price * item.quantity));
  }
}

final cartProvider = NotifierProvider<CartNotifier, List<CartItem>>(
  () => CartNotifier(),
);
```

<div dir="rtl">

### بعد التحويل

</div>

```dart
// lib/providers/cart_provider.dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';  // ✅ Added
import '../models/cart_item.dart';
import '../models/product.dart';

part 'cart_provider.g.dart';  // ✅ Added

@riverpod  // ✅ Added
class Cart extends _$Cart {  // ✅ Changed: Notifier → _$Cart
  @override
  List<CartItem> build() => [];

  void addItem(Product product) {
    final existingIndex = state.indexWhere((item) => item.productId == product.id);

    if (existingIndex != -1) {
      state = [
        for (int i = 0; i < state.length; i++)
          if (i == existingIndex)
            state[i].copyWith(quantity: state[i].quantity + 1)
          else
            state[i],
      ];
    } else {
      state = [...state, CartItem.fromProduct(product)];
    }
  }

  void removeItem(String productId) {
    state = state.where((item) => item.productId != productId).toList();
  }

  double get totalPrice {
    return state.fold(0.0, (sum, item) => sum + (item.price * item.quantity));
  }
}

// ✅ Removed: Provider declaration
// Provider now auto-generated as: cartProvider
```

<div dir="rtl">

### الاستخدام في الـ UI - لم يتغير!

</div>

```dart
// Same code works!
class CartPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final cart = ref.watch(cartProvider);
    final notifier = ref.read(cartProvider.notifier);

    return ListView.builder(
      itemCount: cart.length,
      itemBuilder: (context, index) {
        return ListTile(
          title: Text(cart[index].name),
          trailing: IconButton(
            icon: Icon(Icons.delete),
            onPressed: () => notifier.removeItem(cart[index].productId),
          ),
        );
      },
    );
  }
}
```

<div dir="rtl">

---

## ⚠️ مشاكل شائعة وحلولها

### مشكلة 1: "Undefined name '_$ClassName'"

</div>

```
Error: The class '_$Counter' isn't defined
```

<div dir="rtl">

**السبب:** الـ `.g.dart` file لسه ما اتولدش

**الحل:**

</div>

```bash
flutter pub run build_runner build --delete-conflicting-outputs
```

<div dir="rtl">

### مشكلة 2: "The part directive is missing"

</div>

```
Error: Could not find part 'counter.g.dart'
```

<div dir="rtl">

**السبب:** نسيت تضيف `part` directive

**الحل:** إضافة بعد الـ imports مباشرة:

</div>

```dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'filename.g.dart';  // Add this!
```

<div dir="rtl">

### مشكلة 3: Provider name مختلف

</div>

```dart
// You had:
final userProfileProvider = ...

// After migration, generated as:
final userProvider = ...  // Different name!
```

<div dir="rtl">

**السبب:** اسم الـ provider بيتولد من اسم الـ function/class

**الحل:** سمّي الـ function/class نفس الاسم:

</div>

```dart
// If you want: userProfileProvider
@riverpod
Future<User> userProfile(UserProfileRef ref) async {  // Name: userProfile
  // ...
}
// Generated: userProfileProvider ✅
```

<div dir="rtl">

### مشكلة 4: AutoDispose سلوك مختلف

</div>

```dart
// Classic - default NO autoDispose
final messagesProvider = StreamProvider<List<Message>>((ref) {
  return chatService.messagesStream();
});

// Generated - default YES autoDispose!
@riverpod
Stream<List<Message>> messages(MessagesRef ref) {
  return chatService.messagesStream();
}
```

<div dir="rtl">

**إذا كنت محتاج keep-alive:**

</div>

```dart
@Riverpod(keepAlive: true)
Stream<List<Message>> messages(MessagesRef ref) {
  return chatService.messagesStream();
}
```

<div dir="rtl">

### مشكلة 5: Multiple parameters بدون names

</div>

```dart
// ❌ Wrong - no parameter names
@riverpod
Future<SearchResults> search(SearchRef ref, String, int) async {
  // ...
}

// ✅ Correct - named parameters
@riverpod
Future<SearchResults> search(SearchRef ref, String query, int limit) async {
  // ...
}
```

<div dir="rtl">

---

## ✅ Checklist للـ Migration

بعد ما تحول كل provider، تأكد من:

- [ ] الـ build_runner شغال بدون errors
- [ ] كل الـ `.g.dart` files اتولدت
- [ ] الـ app بيعمل build بدون errors
- [ ] الـ tests كلها pass
- [ ] الـ UI بيشتغل زي الأول
- [ ] مفيش memory leaks (تأكد من autoDispose)

---

## 🎯 استراتيجية الـ Migration

### خطة مقترحة:

</div>

```
Week 1: Setup & Simple Providers
  ↓
Week 2: FutureProviders & StreamProviders
  ↓
Week 3: NotifierProviders (Complex state)
  ↓
Week 4: Testing & Cleanup
```

<div dir="rtl">

### نصائح مهمة:

1. **متحولش كل حاجة مرة واحدة** - حول تدريجياً
2. **ابدأ بالـ simple providers** - بعدين الـ complex
3. **جرب بعد كل تحويل** - تأكد إن كل حاجة شغالة
4. **خلي build_runner شغال** - watch mode دايماً
5. **احتفظ بالـ tests** - جرب بعد كل خطوة

---

## 🔗 الخطوة الجاية

دلوقتي بعد ما فهمت كل حاجة عن Code Generation، وقت نستخدمه في مشاريع حقيقية!

في الـ Sections الجاية هنتعلم:
- Advanced Patterns
- Architecture Patterns
- Testing
- Real-world Examples

كل الأمثلة من دلوقتي هتستخدم **Code Generation**! 🚀

---

## 📚 المصادر

- [Riverpod Migration Guide](https://riverpod.dev/docs/migration/0.14.0_to_1.0.0)
- [Code Generation Best Practices](https://codewithandrea.com/articles/flutter-riverpod-generator/)
- [From StateNotifier to AsyncNotifier](https://riverpod.dev/docs/migration/from_state_notifier)

</div>
