<div dir="rtl">

# أول Provider ليك

**المستوى**: 🟢 مبتدئ

## 📌 المفاهيم الأساسية

في الملف ده هنتكلم عن:
- إزاي تعمل أول provider
- الأنواع المختلفة من Providers
- مع وبدون Code Generation
- Best practices للـ providers

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تعمل provider لأي نوع بيانات
- تختار النوع المناسب
- تستخدم Code Generation
- تتبع Best Practices

---

## 🎨 أنواع Providers الأساسية

قبل ما نبدأ، لازم نعرف الأنواع المتاحة:

### نظرة سريعة

| النوع | الاستخدام | مثال |
|-------|-----------|------|
| **Provider** | قيم ثابتة أو خدمات | API Service, Config |
| **StateProvider** | State بسيط (primitive types) | Counter, isDarkMode |
| **NotifierProvider** | State معقد (objects) | User, TodoList |
| **FutureProvider** | Async data (one-time) | Fetch user |
| **StreamProvider** | Async data (continuous) | Chat messages |

---

## 🚀 الطريقة 1: بدون Code Generation

### مثال 1: Provider بسيط (قيمة ثابتة)

</div>

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';

// Simple constant value
final appNameProvider = Provider<String>((ref) {
  return 'My Awesome App';
});

// Configuration object
final configProvider = Provider<AppConfig>((ref) {
  return AppConfig(
    apiUrl: 'https://api.example.com',
    timeout: Duration(seconds: 30),
  );
});

// Service (singleton)
final apiServiceProvider = Provider<ApiService>((ref) {
  final config = ref.watch(configProvider);
  return ApiService(config.apiUrl);
});
```

<div dir="rtl">

**متى تستخدمه:**
- قيم ثابتة مش بتتغير
- Services و Dependencies
- Configuration

### مثال 2: StateProvider (State بسيط)

</div>

```dart
// Simple counter
final counterProvider = StateProvider<int>((ref) {
  return 0; // Initial value
});

// Dark mode toggle
final isDarkModeProvider = StateProvider<bool>((ref) {
  return false;
});

// Selected language
final localeProvider = StateProvider<String>((ref) {
  return 'ar';
});

// Selected index (bottom navigation)
final selectedIndexProvider = StateProvider<int>((ref) {
  return 0;
});
```

<div dir="rtl">

**متى تستخدمه:**
- State بسيط (int, bool, String, enum)
- مفيش business logic معقدة
- مفيش async operations

**إزاي تستخدمه:**

</div>

```dart
class CounterPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Read value
    final count = ref.watch(counterProvider);

    return Column(
      children: [
        Text('Count: $count'),
        ElevatedButton(
          onPressed: () {
            // Update value
            ref.read(counterProvider.notifier).state++;
          },
          child: Text('Increment'),
        ),
      ],
    );
  }
}
```

<div dir="rtl">

### مثال 3: NotifierProvider (State معقد)

</div>

```dart
// State class
class TodosState {
  final List<Todo> todos;
  final bool isLoading;

  TodosState({
    required this.todos,
    this.isLoading = false,
  });

  TodosState copyWith({
    List<Todo>? todos,
    bool? isLoading,
  }) {
    return TodosState(
      todos: todos ?? this.todos,
      isLoading: isLoading ?? this.isLoading,
    );
  }
}

// Notifier
class TodosNotifier extends Notifier<TodosState> {
  @override
  TodosState build() {
    return TodosState(todos: []);
  }

  void addTodo(Todo todo) {
    state = state.copyWith(
      todos: [...state.todos, todo],
    );
  }

  void removeTodo(String id) {
    state = state.copyWith(
      todos: state.todos.where((t) => t.id != id).toList(),
    );
  }

  Future<void> loadTodos() async {
    state = state.copyWith(isLoading: true);

    final todos = await api.getTodos();

    state = state.copyWith(
      todos: todos,
      isLoading: false,
    );
  }
}

// Provider
final todosProvider = NotifierProvider<TodosNotifier, TodosState>(() {
  return TodosNotifier();
});
```

<div dir="rtl">

**متى تستخدمه:**
- State معقد (objects, lists)
- فيه business logic
- محتاج methods متعددة

### مثال 4: FutureProvider (Async one-time)

</div>

```dart
// Fetch user data once
final userProvider = FutureProvider<User>((ref) async {
  return await api.getUser();
});

// Fetch configuration
final remoteConfigProvider = FutureProvider<RemoteConfig>((ref) async {
  final response = await http.get(Uri.parse('https://api.example.com/config'));
  return RemoteConfig.fromJson(jsonDecode(response.body));
});
```

<div dir="rtl">

**متى تستخدمه:**
- Async operations (API calls)
- البيانات بتتجاب مرة واحدة
- Loading state مطلوب

**إزاي تستخدمه:**

</div>

```dart
class UserProfile extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider);

    return userAsync.when(
      data: (user) => Text('Hello ${user.name}'),
      loading: () => CircularProgressIndicator(),
      error: (error, stack) => Text('Error: $error'),
    );
  }
}
```

<div dir="rtl">

### مثال 5: StreamProvider (Async continuous)

</div>

```dart
// Real-time chat messages
final messagesProvider = StreamProvider<List<Message>>((ref) {
  return chatService.messagesStream();
});

// User location updates
final locationProvider = StreamProvider<Location>((ref) {
  return locationService.locationStream();
});

// With auto-dispose (recommended)
final chatMessagesProvider = StreamProvider.autoDispose<List<Message>>((ref) {
  return chatService.messagesStream();
});
```

<div dir="rtl">

**متى تستخدمه:**
- Real-time data
- Streams من Firebase, WebSockets, etc.
- البيانات بتتحدث باستمرار

---

## 🎯 الطريقة 2: مع Code Generation (موصى به)

حل Code Generation بيقلل الـ boilerplate ويديك type safety أفضل.

### Setup سريع

</div>

```dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

// Required for code generation
part 'my_providers.g.dart';
```

<div dir="rtl">

### مثال 1: Provider بسيط

</div>

```dart
// Before (without codegen)
final appNameProvider = Provider<String>((ref) {
  return 'My App';
});

// After (with codegen)
@riverpod
String appName(AppNameRef ref) {
  return 'My App';
}

// Generated: appNameProvider
```

<div dir="rtl">

### مثال 2: StateProvider → Notifier

</div>

```dart
// Before (StateProvider)
final counterProvider = StateProvider<int>((ref) => 0);

// After (with codegen) - Better!
@riverpod
class Counter extends _$Counter {
  @override
  int build() {
    return 0; // Initial value
  }

  void increment() {
    state++;
  }

  void decrement() {
    state--;
  }

  void reset() {
    state = 0;
  }
}

// Generated: counterProvider
// Use: ref.watch(counterProvider)
// Methods: ref.read(counterProvider.notifier).increment()
```

<div dir="rtl">

### مثال 3: FutureProvider

</div>

```dart
// Before (without codegen)
final userProvider = FutureProvider<User>((ref) async {
  return await api.getUser();
});

// After (with codegen)
@riverpod
Future<User> user(UserRef ref) async {
  return await api.getUser();
}

// Generated: userProvider
```

<div dir="rtl">

### مثال 4: StreamProvider

</div>

```dart
// Before (without codegen)
final messagesProvider = StreamProvider.autoDispose<List<Message>>((ref) {
  return chatService.messagesStream();
});

// After (with codegen)
@riverpod
Stream<List<Message>> messages(MessagesRef ref) {
  return chatService.messagesStream();
}

// Generated: messagesProvider (with autoDispose by default!)
```

<div dir="rtl">

### مثال 5: Notifier معقد

</div>

```dart
@riverpod
class Todos extends _$Todos {
  @override
  FutureOr<List<Todo>> build() async {
    // Load initial data
    return await _repository.getTodos();
  }

  Future<void> addTodo(String title) async {
    // Set to loading
    state = AsyncValue.loading();

    // Add todo
    final newTodo = await _repository.addTodo(title);

    // Update state
    state = AsyncValue.data([
      ...state.value ?? [],
      newTodo,
    ]);
  }

  Future<void> removeTodo(String id) async {
    state = AsyncValue.data(
      state.value?.where((t) => t.id != id).toList() ?? [],
    );

    await _repository.deleteTodo(id);
  }

  Future<void> toggleTodo(String id) async {
    final todos = state.value ?? [];
    final index = todos.indexWhere((t) => t.id == id);

    if (index != -1) {
      final todo = todos[index];
      final updated = todo.copyWith(isCompleted: !todo.isCompleted);

      state = AsyncValue.data([
        ...todos.sublist(0, index),
        updated,
        ...todos.sublist(index + 1),
      ]);

      await _repository.updateTodo(updated);
    }
  }
}

// Generated: todosProvider
```

<div dir="rtl">

---

## 🎨 إزاي تختار النوع المناسب؟

### شجرة القرار

</div>

```
                    محتاج Provider؟
                          |
        ┌─────────────────┴─────────────────┐
        |                                   |
    البيانات بتتغير؟                   ثابتة
        |                                   |
    ┌───┴───┐                          Provider
    |       |                              ✅
 Async?    Sync
    |       |
    |    ┌──┴──┐
    |    |     |
    | بسيط  معقد
    |    |     |
    | StateProvider  Notifier
    |    ✅           ✅
    |
 ┌──┴──┐
 |     |
One   Stream
time
 |     |
Future Stream
Provider Provider
   ✅      ✅
```

<div dir="rtl">

### جدول سريع

| البيانات | النوع | مثال |
|---------|-------|------|
| ثابتة | Provider | API URL, Theme |
| State بسيط | StateProvider أو Notifier | Counter, isDark |
| State معقد | Notifier | TodoList, User |
| Async مرة واحدة | Future function | Fetch user |
| Async مستمر | Stream function | Chat messages |

---

## 💡 Best Practices

### ممارسة 1: اسم واضح

</div>

```dart
// ❌ Bad
final p1 = Provider<String>((ref) => 'value');
final data = FutureProvider<User>((ref) => api.get());

// ✅ Good
final appNameProvider = Provider<String>((ref) => 'My App');
final currentUserProvider = FutureProvider<User>((ref) => api.getUser());
```

<div dir="rtl">

### ممارسة 2: استخدم autoDispose للـ Streams

</div>

```dart
// ❌ Bad - Memory leak!
final messagesProvider = StreamProvider<List<Message>>((ref) {
  return chatService.messagesStream();
});

// ✅ Good - Auto cleanup
final messagesProvider = StreamProvider.autoDispose<List<Message>>((ref) {
  return chatService.messagesStream();
});

// Or with codegen (autoDispose by default)
@riverpod
Stream<List<Message>> messages(MessagesRef ref) {
  return chatService.messagesStream();
}
```

<div dir="rtl">

### ممارسة 3: نظم Providers في ملفات منفصلة

</div>

```dart
// ❌ Bad - كل حاجة في main.dart
// main.dart (500 lines!)
final provider1 = ...;
final provider2 = ...;
// ... 20 more providers

// ✅ Good - منظم في folders
lib/
├── features/
│   ├── auth/
│   │   └── providers/
│   │       └── auth_provider.dart
│   └── todos/
│       └── providers/
│           └── todos_provider.dart
└── shared/
    └── providers/
        ├── theme_provider.dart
        └── locale_provider.dart
```

<div dir="rtl">

### ممارسة 4: استخدم const للقيم الثابتة

</div>

```dart
// ❌ Bad
final defaultTimeoutProvider = Provider<Duration>((ref) {
  return Duration(seconds: 30);
});

// ✅ Good
const _defaultTimeout = Duration(seconds: 30);

final timeoutProvider = Provider<Duration>((ref) {
  return _defaultTimeout;
});
```

<div dir="rtl">

### ممارسة 5: وثّق الـ Providers المعقدة

</div>

```dart
/// Manages the user's authentication state.
///
/// This provider:
/// - Loads user from storage on startup
/// - Handles login/logout
/// - Persists user data
/// - Auto-refreshes token
@riverpod
class Auth extends _$Auth {
  @override
  FutureOr<User?> build() async {
    return await _loadUserFromStorage();
  }

  // ... methods
}
```

<div dir="rtl">

---

## 🔧 مثال كامل: Counter App

خليني أوريك مثال كامل مع كل الخطوات:

### الخطوة 1: إنشاء الملف

</div>

```dart
// lib/features/counter/providers/counter_provider.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'counter_provider.g.dart';

@riverpod
class Counter extends _$Counter {
  @override
  int build() {
    return 0;
  }

  void increment() {
    state++;
  }

  void decrement() {
    state--;
  }

  void reset() {
    state = 0;
  }

  void incrementBy(int value) {
    state += value;
  }
}
```

<div dir="rtl">

### الخطوة 2: Generate Code

</div>

```bash
flutter pub run build_runner watch
```

<div dir="rtl">

### الخطوة 3: استخدام Provider

</div>

```dart
// lib/features/counter/screens/counter_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../providers/counter_provider.dart';

class CounterPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);

    return Scaffold(
      appBar: AppBar(
        title: Text('Counter'),
        actions: [
          IconButton(
            icon: Icon(Icons.refresh),
            onPressed: () {
              ref.read(counterProvider.notifier).reset();
            },
          ),
        ],
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(
              'Count:',
              style: TextStyle(fontSize: 24),
            ),
            Text(
              '$count',
              style: TextStyle(
                fontSize: 72,
                fontWeight: FontWeight.bold,
                color: count > 0 ? Colors.green : Colors.red,
              ),
            ),
            SizedBox(height: 32),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                FloatingActionButton(
                  heroTag: 'decrement',
                  onPressed: () {
                    ref.read(counterProvider.notifier).decrement();
                  },
                  child: Icon(Icons.remove),
                ),
                SizedBox(width: 16),
                FloatingActionButton(
                  heroTag: 'increment',
                  onPressed: () {
                    ref.read(counterProvider.notifier).increment();
                  },
                  child: Icon(Icons.add),
                ),
              ],
            ),
            SizedBox(height: 16),
            ElevatedButton(
              onPressed: () {
                ref.read(counterProvider.notifier).incrementBy(10);
              },
              child: Text('Add 10'),
            ),
          ],
        ),
      ),
    );
  }
}
```

<div dir="rtl">

---

## 🎯 تمرين: اعمل Provider بنفسك

جرب تعمل الـ providers دي:

### تمرين 1: Theme Provider

</div>

```dart
// TODO: اعمل provider للـ dark mode
// - StateProvider<bool>
// - Default: false
// - Method to toggle

@riverpod
class ThemeMode extends _$ThemeMode {
  @override
  bool build() {
    // Your code here
    return false;
  }

  void toggle() {
    // Your code here
  }
}
```

<div dir="rtl">

### تمرين 2: User Provider

</div>

```dart
// TODO: اعمل provider للـ user
// - FutureProvider<User>
// - Fetch from API

@riverpod
Future<User> currentUser(CurrentUserRef ref) async {
  // Your code here
}
```

<div dir="rtl">

### تمرين 3: Shopping Cart

</div>

```dart
// TODO: اعمل provider لـ shopping cart
// - Notifier<List<CartItem>>
// - Methods: add, remove, clear

@riverpod
class ShoppingCart extends _$ShoppingCart {
  @override
  List<CartItem> build() {
    // Your code here
  }

  void addItem(CartItem item) {
    // Your code here
  }

  void removeItem(String id) {
    // Your code here
  }

  void clear() {
    // Your code here
  }
}
```

<div dir="rtl">

**الحلول في نهاية الملف** 👇

---

## 📊 ملخص: أول Provider

</div>

```
الأنواع الأساسية:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Provider - قيم ثابتة
✅ StateProvider/Notifier - State بسيط/معقد
✅ Future function - Async one-time
✅ Stream function - Async continuous

الطريقتين:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. بدون Code Generation - للبداية
2. مع Code Generation - موصى به ✅

Best Practices:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ أسماء واضحة
✅ autoDispose للـ streams
✅ تنظيم في folders
✅ توثيق للمعقد
```

<div dir="rtl">

---

## 🔗 الخطوة الجاية

دلوقتي بعد ما عرفت تعمل providers، وقت:
- **إزاي تقرأ Providers** (الملف الجاي)
- **ref.watch vs ref.read vs ref.listen**
- **ProviderScope بالتفصيل**

---

## 📚 حلول التمارين

### حل تمرين 1: Theme Provider

</div>

```dart
@riverpod
class ThemeMode extends _$ThemeMode {
  @override
  bool build() {
    return false; // Light mode by default
  }

  void toggle() {
    state = !state;
  }

  void setDark(bool isDark) {
    state = isDark;
  }
}
```

<div dir="rtl">

### حل تمرين 2: User Provider

</div>

```dart
@riverpod
Future<User> currentUser(CurrentUserRef ref) async {
  final api = ref.watch(apiServiceProvider);
  return await api.getUser();
}
```

<div dir="rtl">

### حل تمرين 3: Shopping Cart

</div>

```dart
@riverpod
class ShoppingCart extends _$ShoppingCart {
  @override
  List<CartItem> build() {
    return [];
  }

  void addItem(CartItem item) {
    state = [...state, item];
  }

  void removeItem(String id) {
    state = state.where((item) => item.id != id).toList();
  }

  void clear() {
    state = [];
  }

  double get total {
    return state.fold(0, (sum, item) => sum + item.price);
  }
}
```

<div dir="rtl">

---

## ✅ تأكد إنك فهمت

- [ ] تعرف أنواع Providers المختلفة؟
- [ ] تقدر تختار النوع المناسب؟
- [ ] تعرف تستخدم Code Generation؟
- [ ] فاهم Best Practices؟
- [ ] عملت التمارين؟

**جاهز تتعلم إزاي تقرأ Providers؟** 📖

</div>
