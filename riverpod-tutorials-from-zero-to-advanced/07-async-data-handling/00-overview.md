<div dir="rtl">

# نظرة عامة على Async Data Handling

**المستوى**: 🟡 متوسط

## 📌 المفاهيم الأساسية

في Section ده هنتعلم:
- إزاي نتعامل مع Async Data في Riverpod بشكل احترافي
- `AsyncValue` - الأداة الأقوى للـ async state
- Pattern Matching في Riverpod 3.0
- Error Handling بشكل صحيح
- Loading States و UI Patterns
- Refresh و Refetch Strategies

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تتعامل مع كل حالات الـ async data (loading, data, error)
- تستخدم Pattern Matching الجديد (Dart 3)
- تعمل error handling محترف
- تعمل loading states جميلة
- تتعامل مع refresh و caching

---

## 📖 ليه Async Data Handling مهم؟

معظم التطبيقات الحقيقية بتتعامل مع async data:
- **API calls** - جلب data من السيرفر
- **Database** - قراءة من local database
- **Files** - قراءة ملفات
- **Streams** - Real-time updates (Firebase, WebSockets)

**المشكلة:** الـ async operations عندها 3 حالات محتاجة handling:
1. **Loading** - العملية لسه شغالة
2. **Data** - العملية خلصت بنجاح
3. **Error** - حصل خطأ

---

## 🎨 AsyncValue - البطل الخارق

`AsyncValue<T>` هو sealed class في Riverpod بيمثل كل حالات الـ async operation.

### الحالات الثلاثة:

</div>

```dart
// 1. Loading - لسه بنحمل الـ data
AsyncValue<User> userState = const AsyncLoading();

// 2. Data - الـ data جاهزة!
AsyncValue<User> userState = AsyncData(user);

// 3. Error - حصل خطأ
AsyncValue<User> userState = AsyncError(error, stackTrace);
```

<div dir="rtl">

### ليه AsyncValue أفضل من FutureBuilder/StreamBuilder؟

</div>

```dart
// ❌ FutureBuilder - Verbose and limited
FutureBuilder<User>(
  future: fetchUser(),
  builder: (context, snapshot) {
    if (snapshot.connectionState == ConnectionState.waiting) {
      return CircularProgressIndicator();
    }
    if (snapshot.hasError) {
      return Text('Error: ${snapshot.error}');
    }
    if (!snapshot.hasData) {
      return Text('No data');
    }
    final user = snapshot.data!;
    return Text('Hello ${user.name}');
  },
);

// ✅ AsyncValue - Clean and powerful
final userAsync = ref.watch(userProvider);

// Modern way (Dart 3 Pattern Matching)
return switch (userAsync) {
  AsyncData(:final value) => Text('Hello ${value.name}'),
  AsyncError(:final error) => Text('Error: $error'),
  _ => CircularProgressIndicator(),
};

// Or use .when()
return userAsync.when(
  data: (user) => Text('Hello ${user.name}'),
  loading: () => CircularProgressIndicator(),
  error: (error, stack) => Text('Error: $error'),
);
```

<div dir="rtl">

**المميزات:**
- ✅ Type-safe
- ✅ Immutable
- ✅ Built-in error handling
- ✅ Caching support
- ✅ Easy to test
- ✅ Pattern matching support (Dart 3)

---

## 🆕 ما الجديد في Riverpod 3.0؟

### Pattern Matching (Dart 3)

Riverpod 3.0 بيستفيد من Dart 3 Pattern Matching:

</div>

```dart
// Modern approach (Recommended!)
switch (asyncValue) {
  case AsyncData(:final value):
    return Text('Data: $value');
  case AsyncError(:final error):
    return Text('Error: $error');
  case AsyncLoading():
    return CircularProgressIndicator();
}

// Or even shorter with if-case
if (asyncValue case AsyncData(:final value)) {
  return Text('Data: $value');
}
```

<div dir="rtl">

**مقارنة:**

| الطريقة | متى تستخدمها |
|---------|--------------|
| **Pattern Matching** | الأفضل! Modern, type-safe, concise |
| **when()** | لما محتاج تتعامل مع الثلاث حالات |
| **map()** | لما محتاج access للـ AsyncValue نفسه |
| **maybeWhen()** | لما محتاج default case |

---

## 📦 الملفات في Section ده

1. **00-overview.md** (الملف ده)
2. **01-asyncvalue-basics.md** - كل حاجة عن AsyncValue
3. **02-pattern-matching.md** - Dart 3 patterns
4. **03-error-handling.md** - Error handling محترف
5. **04-loading-states.md** - UI patterns للـ loading
6. **05-refresh-strategies.md** - Refresh و caching

---

## 💡 Quick Examples

### مثال 1: FutureProvider مع AsyncValue

</div>

```dart
@riverpod
Future<User> user(UserRef ref) async {
  final response = await http.get(Uri.parse('https://api.example.com/user'));
  return User.fromJson(jsonDecode(response.body));
}

// In UI
class UserProfile extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider);

    return switch (userAsync) {
      AsyncData(:final value) => Text('Hello ${value.name}'),
      AsyncError() => Text('Failed to load user'),
      _ => CircularProgressIndicator(),
    };
  }
}
```

<div dir="rtl">

### مثال 2: Error Handling مع AsyncValue.guard

</div>

```dart
@riverpod
class Todos extends _$Todos {
  @override
  Future<List<Todo>> build() async {
    return await _fetchTodos();
  }

  Future<void> addTodo(String title) async {
    // Show loading
    state = const AsyncLoading();

    // AsyncValue.guard handles errors automatically!
    state = await AsyncValue.guard(() async {
      await api.addTodo(title);
      return await _fetchTodos();
    });
  }

  Future<List<Todo>> _fetchTodos() async {
    return await api.getTodos();
  }
}
```

<div dir="rtl">

---

## 🎓 Learning Path

**الترتيب المقترح للتعلم:**

</div>

```
1. AsyncValue Basics
   ↓
2. Pattern Matching (Modern way)
   ↓
3. Error Handling
   ↓
4. Loading States & UI
   ↓
5. Refresh Strategies
```

<div dir="rtl">

---

## 🔗 الخطوة الجاية

في الملف الجاي هنغوص في **AsyncValue** بالتفصيل:
- كل الـ constructors
- كل الـ methods
- أمثلة عملية
- Best practices

جاهز؟ يلا نكمل! 🚀

---

## 📚 المصادر

- [AsyncValue API Documentation](https://pub.dev/documentation/riverpod/latest/riverpod/AsyncValue-class.html)
- [Use AsyncValue rather than FutureBuilder](https://codewithandrea.com/articles/flutter-use-async-value-not-future-stream-builder/)
- [Riverpod Async Data Guide](https://riverpod.dev/docs/tutorials/first_app)

</div>
