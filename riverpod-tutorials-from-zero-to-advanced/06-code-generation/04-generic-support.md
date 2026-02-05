<div dir="rtl">

# Generic Support في Code Generation 🎯✨

**المستوى**: 🟡 متوسط

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تستخدم Generic types مع @riverpod
- تنشئ providers تقبل أي نوع من الـ data
- تفهم الـ type inference
- تتجنب الأخطاء الشائعة مع Generics

---

## 💡 ما هو Generic Support؟

**Generic Support** في Riverpod 3.0 يعني إنك تقدر تستخدم **Generic types** مع الـ @riverpod annotation بدون مشاكل!

### قبل Riverpod 3.0:

</div>

```dart
// ❌ Riverpod 2.x - Generics كان صعب
// مش موجود دعم كامل للـ generics في code generation
```

<div dir="rtl">

### Riverpod 3.0:

</div>

```dart
// ✅ Riverpod 3.0 - Full generic support!
@riverpod
Future<List<T>> fetchList<T>(FetchListRef ref) async {
  // Generic provider works perfectly!
  return await api.getList<T>();
}
```

<div dir="rtl">

---

## 🎨 Basic Generic Providers

### مثال 1: Generic Data Fetcher

</div>

```dart
// ✅ Generic provider for any data type
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'data_fetcher.g.dart';

@riverpod
Future<T> fetchData<T>(
  FetchDataRef ref,
  String endpoint,
  T Function(Map<String, dynamic>) fromJson,
) async {
  final response = await api.get(endpoint);
  return fromJson(response.data);
}

// استخدام:
class User {
  final String name;
  User(this.name);

  factory User.fromJson(Map<String, dynamic> json) {
    return User(json['name']);
  }
}

class Product {
  final String title;
  Product(this.title);

  factory Product.fromJson(Map<String, dynamic> json) {
    return Product(json['title']);
  }
}

// في الـ UI:
class UserWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // ✅ Type-safe! Returns Future<User>
    final userAsync = ref.watch(
      fetchDataProvider<User>('/user', User.fromJson),
    );

    return userAsync.when(
      data: (user) => Text(user.name),
      loading: () => CircularProgressIndicator(),
      error: (e, s) => Text('Error'),
    );
  }
}

class ProductWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // ✅ Type-safe! Returns Future<Product>
    final productAsync = ref.watch(
      fetchDataProvider<Product>('/product', Product.fromJson),
    );

    return productAsync.when(
      data: (product) => Text(product.title),
      loading: () => CircularProgressIndicator(),
      error: (e, s) => Text('Error'),
    );
  }
}
```

<div dir="rtl">

### مثال 2: Generic List Provider

</div>

```dart
// ✅ Generic list fetcher
@riverpod
Future<List<T>> fetchList<T>(
  FetchListRef ref,
  String endpoint,
  T Function(Map<String, dynamic>) fromJson,
) async {
  final response = await api.get(endpoint);
  final List<dynamic> data = response.data;

  return data.map((item) => fromJson(item)).toList();
}

// استخدام:
class TodosWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // ✅ Returns Future<List<Todo>>
    final todosAsync = ref.watch(
      fetchListProvider<Todo>('/todos', Todo.fromJson),
    );

    return todosAsync.when(
      data: (todos) => ListView(
        children: todos.map((todo) => TodoTile(todo)).toList(),
      ),
      loading: () => CircularProgressIndicator(),
      error: (e, s) => Text('Error'),
    );
  }
}
```

<div dir="rtl">

---

## 🔥 Advanced Generic Patterns

### Pattern 1: Generic Notifier

</div>

```dart
// ✅ Generic state management
@riverpod
class DataManager<T> extends _$DataManager<T> {
  @override
  List<T> build() => [];

  void add(T item) {
    state = [...state, item];
  }

  void remove(T item) {
    state = state.where((i) => i != item).toList();
  }

  void clear() {
    state = [];
  }
}

// استخدام:
class StringListWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // ✅ Type-safe! List<String>
    final strings = ref.watch(dataManagerProvider<String>());

    return Column(
      children: [
        ...strings.map((s) => Text(s)),
        ElevatedButton(
          onPressed: () {
            ref.read(dataManagerProvider<String>().notifier).add('New item');
          },
          child: Text('Add'),
        ),
      ],
    );
  }
}

class IntListWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // ✅ Type-safe! List<int>
    final numbers = ref.watch(dataManagerProvider<int>());

    return Column(
      children: [
        ...numbers.map((n) => Text('$n')),
        ElevatedButton(
          onPressed: () {
            ref.read(dataManagerProvider<int>().notifier).add(42);
          },
          child: Text('Add'),
        ),
      ],
    );
  }
}
```

<div dir="rtl">

### Pattern 2: Generic with Constraints

</div>

```dart
// ✅ Generic with type constraints
abstract class Identifiable {
  String get id;
}

class User implements Identifiable {
  @override
  final String id;
  final String name;

  User(this.id, this.name);
}

class Product implements Identifiable {
  @override
  final String id;
  final String title;

  Product(this.id, this.title);
}

@riverpod
class Repository<T extends Identifiable> extends _$Repository<T> {
  @override
  Map<String, T> build() => {};

  void add(T item) {
    state = {...state, item.id: item};
  }

  T? getById(String id) {
    return state[id];
  }

  void remove(String id) {
    final newState = Map<String, T>.from(state);
    newState.remove(id);
    state = newState;
  }
}

// استخدام:
class UserRepositoryWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final users = ref.watch(repositoryProvider<User>());

    return Column(
      children: [
        ...users.values.map((user) => UserTile(user)),
        ElevatedButton(
          onPressed: () {
            ref.read(repositoryProvider<User>().notifier).add(
              User('1', 'Alice'),
            );
          },
          child: Text('Add User'),
        ),
      ],
    );
  }
}
```

<div dir="rtl">

---

## 🎯 Type Inference

### Automatic Type Inference

Riverpod 3.0 بيستنتج الـ types تلقائياً في معظم الحالات:

</div>

```dart
// ✅ Type inference works!
@riverpod
Future<List<User>> users(UsersRef ref) async {
  // Return type is inferred
  return await api.getUsers();
}

// في الـ UI:
final usersAsync = ref.watch(usersProvider);
// Type is automatically: AsyncValue<List<User>> ✅
```

<div dir="rtl">

### Explicit Type Parameters

في بعض الحالات، لازم تحدد الـ type بشكل صريح:

</div>

```dart
// ✅ Explicit type parameter needed
@riverpod
Future<T> parse<T>(
  ParseRef ref,
  String json,
  T Function(Map<String, dynamic>) fromJson,
) async {
  final data = jsonDecode(json);
  return fromJson(data);  // ✅ Type is T
}

// استخدام:
final user = await ref.read(
  parseProvider<User>(jsonString, User.fromJson).future,
);
// Must specify <User> explicitly ✅
```

<div dir="rtl">

---

## ⚠️ Common Pitfalls (أخطاء شائعة)

### خطأ 1: نسيت Type Parameter

</div>

```dart
// ❌ WRONG - Missing type parameter
final data = ref.watch(fetchDataProvider('/user', User.fromJson));
// Error: Cannot infer type argument for 'fetchDataProvider'

// ✅ CORRECT - Specify type
final data = ref.watch(
  fetchDataProvider<User>('/user', User.fromJson),
);
```

<div dir="rtl">

### خطأ 2: Type Mismatch

</div>

```dart
// ❌ WRONG - Type mismatch
@riverpod
Future<List<User>> users(UsersRef ref) async {
  return await api.getData();  // ❌ Returns List<dynamic>
}

// ✅ CORRECT - Proper casting
@riverpod
Future<List<User>> users(UsersRef ref) async {
  final List<dynamic> data = await api.getData();
  return data.map((item) => User.fromJson(item)).toList();
}
```

<div dir="rtl">

### خطأ 3: Generic في Classic Syntax

</div>

```dart
// ❌ WRONG - Generics don't work well with classic syntax
final fetchDataProvider = FutureProvider.family<T, String>((ref, endpoint) async {
  // ❌ This doesn't work as expected
});

// ✅ CORRECT - Use code generation for generics
@riverpod
Future<T> fetchData<T>(FetchDataRef ref, String endpoint) async {
  // ✅ Works perfectly!
}
```

<div dir="rtl">

---

## 🎓 Best Practices

### 1. استخدم Constraints للـ Type Safety

</div>

```dart
// ✅ GOOD - Type constraints
@riverpod
class EntityManager<T extends Entity> extends _$EntityManager<T> {
  @override
  List<T> build() => [];

  void add(T entity) {
    // ✅ Can safely access Entity properties
    print('Adding entity: ${entity.id}');
    state = [...state, entity];
  }
}

// ❌ BAD - No constraints
@riverpod
class EntityManager<T> extends _$EntityManager<T> {
  @override
  List<T> build() => [];

  void add(T entity) {
    // ❌ Cannot access properties without casting
    state = [...state, entity];
  }
}
```

<div dir="rtl">

### 2. وثق الـ Type Parameters

</div>

```dart
// ✅ GOOD - Document generic types
/// Fetches a list of items from the API.
///
/// **Type Parameter:**
/// - [T]: The type of items to fetch. Must have a `fromJson` constructor.
///
/// **Parameters:**
/// - [endpoint]: The API endpoint to fetch from
/// - [fromJson]: Function to parse JSON to type T
@riverpod
Future<List<T>> fetchList<T>(
  FetchListRef ref,
  String endpoint,
  T Function(Map<String, dynamic>) fromJson,
) async {
  // Implementation
}
```

<div dir="rtl">

### 3. استخدم Factory Constructors

</div>

```dart
// ✅ GOOD - Factory constructor pattern
class ApiResponse<T> {
  final T data;
  final int statusCode;

  ApiResponse(this.data, this.statusCode);

  factory ApiResponse.fromJson(
    Map<String, dynamic> json,
    T Function(dynamic) dataParser,
  ) {
    return ApiResponse(
      dataParser(json['data']),
      json['statusCode'],
    );
  }
}

@riverpod
Future<ApiResponse<T>> fetchApi<T>(
  FetchApiRef ref,
  String endpoint,
  T Function(dynamic) parser,
) async {
  final json = await api.get(endpoint);
  return ApiResponse.fromJson(json, parser);
}
```

<div dir="rtl">

---

## 🎯 الخلاصة

### Generic Support في سطر واحد:
> **Riverpod 3.0 يدعم Generic types بشكل كامل في code generation!**

### الفوائد:
- ✅ Type-safe reusable providers
- ✅ Less code duplication
- ✅ Better IDE support
- ✅ Compile-time type checking

### متى تستخدمه:
- ✅ عندك نفس المنطق لأنواع بيانات مختلفة
- ✅ عايز تعمل reusable data fetchers
- ✅ محتاج type-safe collections
- ✅ بتبني libraries أو packages

---

## 🔗 مصادر إضافية

### Official Documentation:
- [Code Generation | Riverpod](https://riverpod.dev/docs/concepts/about_code_generation)
- [Dart Generics](https://dart.dev/language/generics)

---

## ✅ تأكد إنك فهمت

- [ ] عارف كيف تستخدم Generic types مع @riverpod؟
- [ ] تقدر تنشئ generic providers؟
- [ ] فاهم type inference؟
- [ ] تعرف تستخدم type constraints؟
- [ ] تقدر تتجنب الأخطاء الشائعة؟

---

**🎯 Generic Support = Reusable & Type-Safe Providers!**

استخدمه لتقليل الكود المكرر وزيادة الـ type safety! 💪

</div>
