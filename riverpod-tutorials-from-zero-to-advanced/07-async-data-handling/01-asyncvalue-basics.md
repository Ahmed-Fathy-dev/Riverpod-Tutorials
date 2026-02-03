<div dir="rtl">

# AsyncValue Basics - الدليل الشامل

**المستوى**: 🟡 متوسط

## 📌 المفاهيم الأساسية

في الملف ده هنتعلم:
- `AsyncValue<T>` - كل التفاصيل
- الـ 3 Constructors (Data, Error, Loading)
- كل الـ Properties
- كل الـ Methods
- أمثلة عملية شاملة

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تستخدم كل constructors الـ AsyncValue
- تفهم كل property ومتى تستخدمها
- تستخدم كل method بشكل صحيح
- تكتب async code محترف

---

## 🔍 ما هو AsyncValue؟

`AsyncValue<T>` هو **sealed class** في Riverpod بيمثل حالة أي عملية async.

### التعريف من الـ API الرسمي:

> "A sealed utility class that handles async data safely by guaranteeing all loading, error, and success states are managed."

**ترجمة:** AsyncValue بيضمن إنك مش هتنسى تتعامل مع أي حالة من حالات الـ async operation (loading, data, error).

---

## 🏗️ الـ 3 Constructors

### 1. AsyncData - البيانات الجاهزة

</div>

```dart
// Constructor signature
AsyncValue.data(T value)

// Usage
final userState = AsyncValue.data(
  User(id: '1', name: 'Ahmed'),
);

// Example in provider
@riverpod
class UserController extends _$UserController {
  @override
  AsyncValue<User> build() {
    // Return data directly
    return AsyncValue.data(
      User(id: '1', name: 'Initial User'),
    );
  }

  void setUser(User user) {
    state = AsyncValue.data(user);
  }
}
```

<div dir="rtl">

**متى تستخدمها:**
- لما يكون عندك data جاهزة
- بعد ما async operation تخلص بنجاح
- لما تعمل update للـ state بـ data جديدة

---

### 2. AsyncError - حصل خطأ

</div>

```dart
// Constructor signature
AsyncValue.error(Object error, StackTrace stackTrace)

// Usage
final userState = AsyncValue<User>.error(
  'User not found',
  StackTrace.current,
);

// Example in provider
@riverpod
class UserController extends _$UserController {
  @override
  AsyncValue<User> build() {
    return AsyncValue.data(User(id: '1', name: 'Ahmed'));
  }

  Future<void> loadUser(String id) async {
    try {
      final user = await api.getUser(id);
      state = AsyncValue.data(user);
    } catch (error, stackTrace) {
      // Store error state
      state = AsyncValue.error(error, stackTrace);
    }
  }
}
```

<div dir="rtl">

**متى تستخدمها:**
- لما async operation تفشل
- لما تمسك exception في try-catch
- لما تعمل error handling

**ملاحظة:** لازم تبعت **StackTrace** مع الـ error عشان debugging أسهل.

---

### 3. AsyncLoading - جاري التحميل

</div>

```dart
// Constructor signature
AsyncValue.loading({num? progress})

// Simple loading
final userState = AsyncValue<User>.loading();

// Loading with progress (optional)
final downloadState = AsyncValue<File>.loading(progress: 0.45); // 45%

// Example in provider
@riverpod
class UserController extends _$UserController {
  @override
  AsyncValue<User> build() {
    return AsyncValue.data(User(id: '1', name: 'Ahmed'));
  }

  Future<void> refreshUser() async {
    // Show loading
    state = const AsyncValue.loading();

    try {
      final user = await api.getUser('1');
      state = AsyncValue.data(user);
    } catch (error, stackTrace) {
      state = AsyncValue.error(error, stackTrace);
    }
  }

  Future<void> downloadFile() async {
    // Loading with progress
    state = const AsyncValue.loading();

    await for (final progress in api.downloadFile()) {
      state = AsyncValue.loading(progress: progress);
    }

    state = AsyncValue.data(downloadedFile);
  }
}
```

<div dir="rtl">

**متى تستخدمها:**
- قبل ما تبدأ async operation
- لما تعمل refresh للـ data
- لما عايز تعرض progress bar (optional progress parameter)

**الـ progress parameter:**
- نوعه `num?` (ممكن يكون null)
- من 0.0 إلى 1.0 (0% إلى 100%)
- مفيد للـ download/upload operations

---

## 🎨 كل الـ Properties

### 1. Core Data Properties

</div>

```dart
final userAsync = AsyncValue.data(User(id: '1', name: 'Ahmed'));

// value - القيمة الحالية (nullable)
final user = userAsync.value; // User? - قد يكون null لو loading أو error
print(user?.name); // 'Ahmed'

// hasValue - هل فيه قيمة؟
if (userAsync.hasValue) {
  print('We have data!');
}
```

<div dir="rtl">

#### value vs requireValue

</div>

```dart
// value - Returns T? (nullable)
final userAsync = AsyncValue<User>.loading();
final user = userAsync.value; // null - safe!

// requireValue - Returns T (throws if no value)
try {
  final user = userAsync.requireValue; // ❌ Throws exception!
} catch (e) {
  print('No value available');
}

// Use requireValue when:
@riverpod
Future<String> userGreeting(UserGreetingRef ref) async {
  final userAsync = await ref.watch(userProvider.future);
  final settingsAsync = await ref.watch(settingsProvider.future);

  // Safe to use requireValue in provider callbacks
  final user = userAsync.requireValue;
  final settings = settingsAsync.requireValue;

  return '${settings.greeting}, ${user.name}!';
}
```

<div dir="rtl">

**متى تستخدم كل واحدة:**

| Property | Use Case |
|----------|----------|
| **value** | في الـ UI - مش متأكد إذا فيه data ولا لأ |
| **requireValue** | في الـ Providers - لما متأكد إن الـ data موجودة |

---

### 2. Error Properties

</div>

```dart
final errorAsync = AsyncValue<User>.error(
  'User not found',
  StackTrace.current,
);

// error - الـ Error object
print(errorAsync.error); // 'User not found'

// hasError - هل فيه error؟
if (errorAsync.hasError) {
  print('An error occurred: ${errorAsync.error}');
}

// stackTrace - الـ Stack trace للـ error
print(errorAsync.stackTrace); // StackTrace object

// Usage in UI
Widget build(BuildContext context, WidgetRef ref) {
  final userAsync = ref.watch(userProvider);

  if (userAsync.hasError) {
    return Text('Error: ${userAsync.error}');
  }

  // ... rest of UI
}
```

<div dir="rtl">

---

### 3. Loading State Properties

</div>

```dart
// isLoading - هل في حالة loading؟
final loadingAsync = AsyncValue<User>.loading();
print(loadingAsync.isLoading); // true

// isRefreshing - هل بيعمل refresh بعد ما كان عندنا data؟
@riverpod
class Todos extends _$Todos {
  @override
  Future<List<Todo>> build() async {
    return await api.getTodos();
  }

  Future<void> refresh() async {
    // This sets isRefreshing to true!
    state = const AsyncValue.loading();

    state = await AsyncValue.guard(() async {
      return await api.getTodos();
    });
  }
}

// Usage in UI
Widget build(BuildContext context, WidgetRef ref) {
  final todosAsync = ref.watch(todosProvider);

  // Show pull-to-refresh indicator only when refreshing
  return RefreshIndicator(
    onRefresh: () => ref.refresh(todosProvider.future),
    child: todosAsync.when(
      data: (todos) => ListView(
        children: [
          // Show small loading indicator at top when refreshing
          if (todosAsync.isRefreshing)
            const LinearProgressIndicator(),
          ...todos.map((todo) => TodoTile(todo)),
        ],
      ),
      loading: () => const CircularProgressIndicator(),
      error: (error, _) => Text('Error: $error'),
    ),
  );
}

// isReloading - هل بيعمل reload بسبب dependency change؟
@riverpod
Future<User> user(UserRef ref, String userId) async {
  return await api.getUser(userId);
}

@riverpod
Future<List<Post>> userPosts(UserPostsRef ref, String userId) async {
  // Watch user - if userId changes, this reloads
  final user = await ref.watch(userProvider(userId).future);
  return await api.getUserPosts(user.id);
}

// isReloading = true when userId parameter changes
```

<div dir="rtl">

**الفرق بين Loading States:**

| State | Scenario | Example |
|-------|----------|---------|
| **isLoading** | أول مرة loading أو أي loading state | Initial load, refresh, reload |
| **isRefreshing** | بنعمل refresh **بعد** ما كان عندنا data قبل كده | Pull-to-refresh |
| **isReloading** | Dependency اتغيرت فأعاد الـ computation | Parameter change, watched provider changed |

---

### 4. Additional Properties

</div>

```dart
// progress - تقدم العملية (0.0 to 1.0)
final downloadAsync = AsyncValue<File>.loading(progress: 0.65);
print(downloadAsync.progress); // 0.65 (65%)

// Usage in UI
Widget build(BuildContext context, WidgetRef ref) {
  final downloadAsync = ref.watch(downloadProvider);

  if (downloadAsync.isLoading && downloadAsync.progress != null) {
    return LinearProgressIndicator(
      value: downloadAsync.progress, // 0.0 to 1.0
    );
  }

  return downloadAsync.when(
    data: (file) => Text('Downloaded: ${file.name}'),
    loading: () => const CircularProgressIndicator(),
    error: (error, _) => Text('Error: $error'),
  );
}

// retrying - هل بيعيد المحاولة بعد error؟
@riverpod
class DataController extends _$DataController {
  @override
  Future<Data> build() async {
    return await api.getData();
  }

  Future<void> retryLoad() async {
    // This sets retrying to true
    state = AsyncValue.loading(progress: null);

    state = await AsyncValue.guard(() async {
      return await api.getData();
    });
  }
}

// isFromCache - هل الـ data جاية من cache؟
// (Used with offline persistence packages)
```

<div dir="rtl">

---

## 🔧 كل الـ Methods

### 1. Pattern Matching Methods

#### when() - التعامل مع كل الحالات

الـ **method الأساسي** في AsyncValue.

</div>

```dart
// Signature
R when<R>({
  required R Function(T data) data,
  required R Function(Object error, StackTrace stackTrace) error,
  required R Function() loading,
  bool skipLoadingOnReload = false,
  bool skipLoadingOnRefresh = true,
  bool skipError = false,
})

// Basic usage
final userAsync = ref.watch(userProvider);

return userAsync.when(
  // When we have data
  data: (user) => Text('Hello ${user.name}'),

  // When there's an error
  error: (error, stackTrace) => Text('Error: $error'),

  // When loading
  loading: () => const CircularProgressIndicator(),
);

// With skip flags
return userAsync.when(
  data: (user) => UserWidget(user),
  error: (error, stack) => ErrorWidget(error),
  loading: () => const CircularProgressIndicator(),

  // Don't show loading when refreshing (already have previous data)
  skipLoadingOnRefresh: true, // Default

  // Don't show loading when reloading (dependency changed)
  skipLoadingOnReload: false, // Default
);
```

<div dir="rtl">

**الـ Skip Flags:**

| Flag | Default | Purpose |
|------|---------|---------|
| **skipLoadingOnRefresh** | `true` | لما بنعمل refresh، مش نعرض loading (نعرض الـ data القديمة) |
| **skipLoadingOnReload** | `false` | لما dependency تتغير، نعرض loading |
| **skipError** | `false` | تخطى حالة الـ error (نادر الاستخدام) |

**مثال عملي:**

</div>

```dart
// Scenario: Todo list with pull-to-refresh
@riverpod
class Todos extends _$Todos {
  @override
  Future<List<Todo>> build() async {
    return await api.getTodos();
  }

  Future<void> refresh() async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() => api.getTodos());
  }
}

// In UI
class TodoListScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final todosAsync = ref.watch(todosProvider);

    return RefreshIndicator(
      onRefresh: () => ref.read(todosProvider.notifier).refresh(),
      child: todosAsync.when(
        // Show the todo list
        data: (todos) => ListView.builder(
          itemCount: todos.length,
          itemBuilder: (context, index) => TodoTile(todos[index]),
        ),

        // Show error message
        error: (error, _) => Center(
          child: Text('Failed to load todos: $error'),
        ),

        // Show loading spinner
        loading: () => const Center(
          child: CircularProgressIndicator(),
        ),

        // ✅ Don't show loading when refreshing!
        // User will see the pull-to-refresh indicator instead
        skipLoadingOnRefresh: true,
      ),
    );
  }
}
```

<div dir="rtl">

---

#### maybeWhen() - تعامل مع حالات معينة بس

</div>

```dart
// Signature
R maybeWhen<R>({
  R Function(T data)? data,
  R Function(Object error, StackTrace stackTrace)? error,
  R Function()? loading,
  required R Function() orElse,
  bool skipLoadingOnReload = false,
  bool skipLoadingOnRefresh = true,
  bool skipError = false,
})

// Usage - Handle only data and provide default for rest
final userAsync = ref.watch(userProvider);

return userAsync.maybeWhen(
  // Only handle data case
  data: (user) => Text('Hello ${user.name}'),

  // All other cases (loading, error) go here
  orElse: () => const CircularProgressIndicator(),
);

// Another example - Handle error specifically
return userAsync.maybeWhen(
  error: (error, _) => ErrorScreen(error),

  // For data and loading, show loading
  orElse: () => const LoadingScreen(),
);

// Real example - Show previous data while refreshing
class UserProfile extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider);

    // If refreshing, show previous data with loading indicator
    if (userAsync.isRefreshing && userAsync.hasValue) {
      return Column(
        children: [
          const LinearProgressIndicator(),
          UserCard(userAsync.value!),
        ],
      );
    }

    // Otherwise use normal pattern matching
    return userAsync.maybeWhen(
      data: (user) => UserCard(user),
      orElse: () => const LoadingScreen(),
    );
  }
}
```

<div dir="rtl">

**متى تستخدم maybeWhen:**
- لما محتاج تتعامل مع حالة واحدة أو اتنين بس
- عندك default behavior للحالات التانية
- عايز custom handling للـ refresh state

---

#### map() - Transform الـ AsyncValue

</div>

```dart
// Signature
R map<R>({
  required R Function(AsyncData<T> data) data,
  required R Function(AsyncError<T> error) error,
  required R Function(AsyncLoading<T> loading) loading,
})

// Usage - Access the AsyncValue itself
final userAsync = ref.watch(userProvider);

return userAsync.map(
  // AsyncData gives us access to the whole data object
  data: (asyncData) {
    print('Has value: ${asyncData.hasValue}');
    print('Is refreshing: ${asyncData.isRefreshing}');
    return Text('Hello ${asyncData.value.name}');
  },

  // AsyncError gives us error details
  error: (asyncError) {
    print('Error: ${asyncError.error}');
    print('Stack trace: ${asyncError.stackTrace}');
    return ErrorWidget(asyncError.error);
  },

  // AsyncLoading gives us loading details
  loading: (asyncLoading) {
    print('Progress: ${asyncLoading.progress}');
    return CircularProgressIndicator(
      value: asyncLoading.progress,
    );
  },
);
```

<div dir="rtl">

**الفرق بين when() و map():**

| Method | Parameter Type | Use Case |
|--------|---------------|----------|
| **when()** | Raw values (T, error, nothing) | لما محتاج الـ data بس |
| **map()** | AsyncValue objects | لما محتاج access للـ AsyncValue properties |

---

#### maybeMap() - Transform حالات معينة

</div>

```dart
// Signature
R maybeMap<R>({
  R Function(AsyncData<T> data)? data,
  R Function(AsyncError<T> error)? error,
  R Function(AsyncLoading<T> loading)? loading,
  required R Function() orElse,
})

// Usage
final userAsync = ref.watch(userProvider);

return userAsync.maybeMap(
  // Only handle data case
  data: (asyncData) {
    // Check if refreshing
    if (asyncData.isRefreshing) {
      return Column(
        children: [
          const LinearProgressIndicator(),
          UserCard(asyncData.value),
        ],
      );
    }
    return UserCard(asyncData.value);
  },

  // Default for loading and error
  orElse: () => const LoadingScreen(),
);
```

<div dir="rtl">

---

#### whenData() - Shorthand للـ data handling

</div>

```dart
// Signature
AsyncValue<R> whenData<R>(R Function(T value) cb)

// Usage - Transform only the data
final userAsync = ref.watch(userProvider);

// Transform user to greeting
final greetingAsync = userAsync.whenData((user) {
  return 'Hello ${user.name}!';
});

// greetingAsync is AsyncValue<String>
return greetingAsync.when(
  data: (greeting) => Text(greeting),
  loading: () => const CircularProgressIndicator(),
  error: (error, _) => Text('Error: $error'),
);

// Real example - Chain transformations
@riverpod
Future<User> user(UserRef ref) async {
  return await api.getUser();
}

// Widget
class UserGreeting extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider);

    // Transform data while keeping loading/error states
    final greetingAsync = userAsync.whenData((user) {
      final hour = DateTime.now().hour;
      final greeting = hour < 12 ? 'صباح الخير' : 'مساء الخير';
      return '$greeting، ${user.name}!';
    });

    // greetingAsync inherits loading and error states from userAsync
    return greetingAsync.when(
      data: (greeting) => Text(greeting, style: TextStyle(fontSize: 24)),
      loading: () => const CircularProgressIndicator(),
      error: (error, _) => const Text('Failed to load user'),
    );
  }
}
```

<div dir="rtl">

**متى تستخدم whenData:**
- عايز تحول الـ data بس وتحتفظ بالـ loading و error states
- بتعمل data transformation في الـ UI
- عايز syntax أقصر من when()

---

### 2. Utility Methods

#### AsyncValue.guard() - Convert Future to AsyncValue

الـ method الأهم للـ error handling!

</div>

```dart
// Signature
static Future<AsyncValue<T>> guard<T>(Future<T> Function() future)

// Usage - Automatically catches errors
@riverpod
class Todos extends _$Todos {
  @override
  Future<List<Todo>> build() async {
    return await api.getTodos();
  }

  Future<void> addTodo(String title) async {
    // Show loading
    state = const AsyncValue.loading();

    // guard() automatically catches errors!
    state = await AsyncValue.guard(() async {
      await api.addTodo(title);
      return await api.getTodos();
    });

    // If success: state = AsyncData(todos)
    // If error: state = AsyncError(error, stackTrace)
    // No try-catch needed!
  }

  Future<void> deleteTodo(String id) async {
    state = const AsyncValue.loading();

    state = await AsyncValue.guard(() async {
      await api.deleteTodo(id);
      return await api.getTodos();
    });
  }
}

// Without guard (manual approach)
Future<void> addTodoManual(String title) async {
  state = const AsyncValue.loading();

  try {
    await api.addTodo(title);
    final todos = await api.getTodos();
    state = AsyncValue.data(todos);
  } catch (error, stackTrace) {
    state = AsyncValue.error(error, stackTrace);
  }
}

// With guard (cleaner!)
Future<void> addTodoWithGuard(String title) async {
  state = const AsyncValue.loading();

  state = await AsyncValue.guard(() async {
    await api.addTodo(title);
    return await api.getTodos();
  });
}
```

<div dir="rtl">

**مميزات guard():**
- ✅ يمسك كل الـ exceptions تلقائياً
- ✅ بيسجل الـ StackTrace صح
- ✅ كود أقصر وأوضح
- ✅ مفيش risk تنسى try-catch

**Best Practice:** استخدم `guard()` في كل مكان بتعدل فيه state في AsyncNotifier!

---

#### unwrapPrevious() - الرجوع للحالة الأصلية

</div>

```dart
// Signature
AsyncValue<T> unwrapPrevious()

// Context: When you set loading while having previous data
@riverpod
class Counter extends _$Counter {
  @override
  AsyncValue<int> build() {
    return const AsyncValue.data(0);
  }

  Future<void> increment() async {
    // Save previous value
    final previousState = state;

    // Set loading (but keep previous data)
    state = const AsyncValue.loading();

    // After some async work
    await Future.delayed(const Duration(seconds: 1));

    // Restore from loading to previous state
    state = previousState.unwrapPrevious();

    // Then update
    state = AsyncValue.data((state.value ?? 0) + 1);
  }
}

// Real-world example - Optimistic updates
@riverpod
class Todos extends _$Todos {
  @override
  Future<List<Todo>> build() async {
    return await api.getTodos();
  }

  Future<void> toggleTodo(String id) async {
    // Get current state
    final previousState = state;

    // Optimistically update UI
    state = previousState.whenData((todos) {
      return todos.map((todo) {
        if (todo.id == id) {
          return todo.copyWith(completed: !todo.completed);
        }
        return todo;
      }).toList();
    });

    // Try to update on server
    try {
      await api.toggleTodo(id);
    } catch (error, stackTrace) {
      // Revert to previous state on error
      state = AsyncValue.error(error, stackTrace);

      // Or restore previous data
      state = previousState;
    }
  }
}
```

<div dir="rtl">

---

#### asData / asError - Safe Upcasting

</div>

```dart
// Signature
AsyncData<T>? get asData
AsyncError<T>? get asError

// Usage - Check and cast safely
final userAsync = ref.watch(userProvider);

// Check if it's data
final asyncData = userAsync.asData;
if (asyncData != null) {
  print('We have data: ${asyncData.value}');
  print('Is refreshing: ${asyncData.isRefreshing}');
}

// Check if it's error
final asyncError = userAsync.asError;
if (asyncError != null) {
  print('Error: ${asyncError.error}');
  print('Stack: ${asyncError.stackTrace}');
}

// Real example - Custom error logging
class ErrorLogger {
  void logAsyncError(AsyncValue<dynamic> asyncValue) {
    final error = asyncValue.asError;
    if (error != null) {
      logToService(
        error: error.error,
        stackTrace: error.stackTrace,
        timestamp: DateTime.now(),
      );
    }
  }
}
```

<div dir="rtl">

---

## 📊 مقارنة شاملة: متى تستخدم أي method؟

| Method | Use Case | Code Length |
|--------|----------|-------------|
| **when()** | لما محتاج تتعامل مع الثلاث حالات | متوسط |
| **maybeWhen()** | لما محتاج تتعامل مع حالة أو اتنين بس | أقصر |
| **map()** | لما محتاج access للـ AsyncValue نفسه | متوسط |
| **maybeMap()** | لما محتاج access لـ AsyncValue لحالة معينة | أقصر |
| **whenData()** | لما عايز تحول الـ data بس | أقصر |
| **guard()** | لتحويل Future لـ AsyncValue مع error handling | الأفضل! |

---

## 🎯 Best Practices

### 1. استخدم guard() دايماً

</div>

```dart
// ✅ GOOD - Use guard()
Future<void> addTodo(String title) async {
  state = const AsyncValue.loading();
  state = await AsyncValue.guard(() async {
    await api.addTodo(title);
    return await api.getTodos();
  });
}

// ❌ BAD - Manual try-catch
Future<void> addTodo(String title) async {
  state = const AsyncValue.loading();
  try {
    await api.addTodo(title);
    final todos = await api.getTodos();
    state = AsyncValue.data(todos);
  } catch (error, stackTrace) {
    state = AsyncValue.error(error, stackTrace);
  }
}
```

<div dir="rtl">

### 2. استخدم skipLoadingOnRefresh في when()

</div>

```dart
// ✅ GOOD - Don't show loading when refreshing
return todosAsync.when(
  data: (todos) => TodoList(todos),
  loading: () => const LoadingSpinner(),
  error: (error, _) => ErrorMessage(error),
  skipLoadingOnRefresh: true, // Keep showing old data
);

// ❌ BAD - Shows loading spinner while refreshing
return todosAsync.when(
  data: (todos) => TodoList(todos),
  loading: () => const LoadingSpinner(),
  error: (error, _) => ErrorMessage(error),
  skipLoadingOnRefresh: false, // User sees spinner! Bad UX
);
```

<div dir="rtl">

### 3. استخدم whenData للـ transformations

</div>

```dart
// ✅ GOOD - Clean transformation
final greetingAsync = userAsync.whenData((user) {
  return 'Hello ${user.name}!';
});

// ❌ BAD - Verbose
final greetingAsync = userAsync.when(
  data: (user) => AsyncValue.data('Hello ${user.name}!'),
  loading: () => const AsyncValue.loading(),
  error: (error, stack) => AsyncValue.error(error, stack),
);
```

<div dir="rtl">

### 4. استخدم value في UI، requireValue في Providers

</div>

```dart
// ✅ GOOD - value in UI
Widget build(BuildContext context, WidgetRef ref) {
  final userAsync = ref.watch(userProvider);
  final user = userAsync.value; // T? - safe

  if (user != null) {
    return Text(user.name);
  }
  return const SizedBox();
}

// ✅ GOOD - requireValue in providers
@riverpod
Future<String> userGreeting(UserGreetingRef ref) async {
  final user = await ref.watch(userProvider.future);
  return 'Hello ${user.name}!'; // No null check needed
}

// ❌ BAD - requireValue in UI
Widget build(BuildContext context, WidgetRef ref) {
  final userAsync = ref.watch(userProvider);
  final user = userAsync.requireValue; // Throws if loading!
  return Text(user.name);
}
```

<div dir="rtl">

### 5. Handle isRefreshing للـ better UX

</div>

```dart
// ✅ GOOD - Show small indicator when refreshing
Widget build(BuildContext context, WidgetRef ref) {
  final todosAsync = ref.watch(todosProvider);

  return Column(
    children: [
      // Small loading bar at top when refreshing
      if (todosAsync.isRefreshing)
        const LinearProgressIndicator(),

      // Show content
      Expanded(
        child: todosAsync.when(
          data: (todos) => TodoList(todos),
          loading: () => const CircularProgressIndicator(),
          error: (error, _) => ErrorMessage(error),
          skipLoadingOnRefresh: true, // Keep showing old data
        ),
      ),
    ],
  );
}
```

<div dir="rtl">

---

## 🎓 أمثلة عملية شاملة

### مثال 1: Todo App مع كل Features

</div>

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'todos.g.dart';

// Model
class Todo {
  final String id;
  final String title;
  final bool completed;

  Todo({
    required this.id,
    required this.title,
    this.completed = false,
  });

  Todo copyWith({String? title, bool? completed}) {
    return Todo(
      id: id,
      title: title ?? this.title,
      completed: completed ?? this.completed,
    );
  }
}

// API Service
class TodosApi {
  Future<List<Todo>> getTodos() async {
    await Future.delayed(const Duration(seconds: 1));
    return [
      Todo(id: '1', title: 'Learn Riverpod'),
      Todo(id: '2', title: 'Build awesome app'),
    ];
  }

  Future<void> addTodo(String title) async {
    await Future.delayed(const Duration(milliseconds: 500));
    // API call
  }

  Future<void> toggleTodo(String id) async {
    await Future.delayed(const Duration(milliseconds: 500));
    // API call
  }

  Future<void> deleteTodo(String id) async {
    await Future.delayed(const Duration(milliseconds: 500));
    // API call
  }
}

final todosApiProvider = Provider((ref) => TodosApi());

// Todos Provider with AsyncNotifier
@riverpod
class Todos extends _$Todos {
  @override
  Future<List<Todo>> build() async {
    final api = ref.watch(todosApiProvider);
    return await api.getTodos();
  }

  // Add todo
  Future<void> addTodo(String title) async {
    state = const AsyncValue.loading();

    state = await AsyncValue.guard(() async {
      final api = ref.read(todosApiProvider);
      await api.addTodo(title);
      return await api.getTodos();
    });
  }

  // Toggle todo with optimistic update
  Future<void> toggleTodo(String id) async {
    final previousState = state;

    // Optimistically update UI
    state = state.whenData((todos) {
      return todos.map((todo) {
        if (todo.id == id) {
          return todo.copyWith(completed: !todo.completed);
        }
        return todo;
      }).toList();
    });

    // Try to update on server
    final result = await AsyncValue.guard(() async {
      final api = ref.read(todosApiProvider);
      await api.toggleTodo(id);
      return await api.getTodos();
    });

    // Update state with result (or keep optimistic if it worked)
    state = result;
  }

  // Delete todo
  Future<void> deleteTodo(String id) async {
    state = const AsyncValue.loading();

    state = await AsyncValue.guard(() async {
      final api = ref.read(todosApiProvider);
      await api.deleteTodo(id);
      return await api.getTodos();
    });
  }

  // Refresh
  Future<void> refresh() async {
    state = const AsyncValue.loading();

    state = await AsyncValue.guard(() async {
      final api = ref.read(todosApiProvider);
      return await api.getTodos();
    });
  }
}

// UI
class TodosScreen extends ConsumerWidget {
  const TodosScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final todosAsync = ref.watch(todosProvider);

    return Scaffold(
      appBar: AppBar(
        title: const Text('Todos'),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: () => ref.read(todosProvider.notifier).refresh(),
          ),
        ],
      ),
      body: Column(
        children: [
          // Show loading indicator when refreshing
          if (todosAsync.isRefreshing)
            const LinearProgressIndicator(),

          // Main content
          Expanded(
            child: todosAsync.when(
              data: (todos) {
                if (todos.isEmpty) {
                  return const Center(
                    child: Text('No todos yet!'),
                  );
                }

                return RefreshIndicator(
                  onRefresh: () async {
                    await ref.read(todosProvider.notifier).refresh();
                  },
                  child: ListView.builder(
                    itemCount: todos.length,
                    itemBuilder: (context, index) {
                      final todo = todos[index];
                      return ListTile(
                        leading: Checkbox(
                          value: todo.completed,
                          onChanged: (_) {
                            ref
                                .read(todosProvider.notifier)
                                .toggleTodo(todo.id);
                          },
                        ),
                        title: Text(
                          todo.title,
                          style: TextStyle(
                            decoration: todo.completed
                                ? TextDecoration.lineThrough
                                : null,
                          ),
                        ),
                        trailing: IconButton(
                          icon: const Icon(Icons.delete),
                          onPressed: () {
                            ref
                                .read(todosProvider.notifier)
                                .deleteTodo(todo.id);
                          },
                        ),
                      );
                    },
                  ),
                );
              },
              loading: () => const Center(
                child: CircularProgressIndicator(),
              ),
              error: (error, stackTrace) => Center(
                child: Column(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    Text('Error: $error'),
                    const SizedBox(height: 16),
                    ElevatedButton(
                      onPressed: () {
                        ref.invalidate(todosProvider);
                      },
                      child: const Text('Retry'),
                    ),
                  ],
                ),
              ),
              skipLoadingOnRefresh: true, // Keep showing data while refreshing
            ),
          ),
        ],
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () async {
          final title = await _showAddTodoDialog(context);
          if (title != null && title.isNotEmpty) {
            ref.read(todosProvider.notifier).addTodo(title);
          }
        },
        child: const Icon(Icons.add),
      ),
    );
  }

  Future<String?> _showAddTodoDialog(BuildContext context) async {
    final controller = TextEditingController();

    return showDialog<String>(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('Add Todo'),
        content: TextField(
          controller: controller,
          decoration: const InputDecoration(
            hintText: 'Enter todo title',
          ),
          autofocus: true,
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('Cancel'),
          ),
          TextButton(
            onPressed: () => Navigator.pop(context, controller.text),
            child: const Text('Add'),
          ),
        ],
      ),
    );
  }
}
```

<div dir="rtl">

### مثال 2: User Profile مع AsyncValue transformations

</div>

```dart
// User model
class User {
  final String id;
  final String name;
  final String email;
  final String avatarUrl;

  User({
    required this.id,
    required this.name,
    required this.email,
    required this.avatarUrl,
  });
}

// API
class UserApi {
  Future<User> getUser(String id) async {
    await Future.delayed(const Duration(seconds: 1));
    return User(
      id: id,
      name: 'Ahmed',
      email: 'ahmed@example.com',
      avatarUrl: 'https://i.pravatar.cc/150?u=$id',
    );
  }

  Future<void> updateUser(User user) async {
    await Future.delayed(const Duration(milliseconds: 500));
  }
}

final userApiProvider = Provider((ref) => UserApi());

// User provider
@riverpod
Future<User> user(UserRef ref, String userId) async {
  final api = ref.watch(userApiProvider);
  return await api.getUser(userId);
}

// Derived: User initials
@riverpod
AsyncValue<String> userInitials(UserInitialsRef ref, String userId) {
  final userAsync = ref.watch(userProvider(userId));

  // Transform only the data
  return userAsync.whenData((user) {
    final names = user.name.split(' ');
    if (names.length >= 2) {
      return '${names[0][0]}${names[1][0]}'.toUpperCase();
    }
    return user.name[0].toUpperCase();
  });
}

// Derived: User display name
@riverpod
AsyncValue<String> userDisplayName(UserDisplayNameRef ref, String userId) {
  final userAsync = ref.watch(userProvider(userId));

  return userAsync.whenData((user) {
    return '${user.name} (${user.email})';
  });
}

// UI
class UserProfileScreen extends ConsumerWidget {
  final String userId;

  const UserProfileScreen({super.key, required this.userId});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider(userId));
    final initialsAsync = ref.watch(userInitialsProvider(userId));
    final displayNameAsync = ref.watch(userDisplayNameProvider(userId));

    return Scaffold(
      appBar: AppBar(
        title: const Text('Profile'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Avatar with initials
            initialsAsync.when(
              data: (initials) => CircleAvatar(
                radius: 50,
                child: Text(
                  initials,
                  style: const TextStyle(fontSize: 32),
                ),
              ),
              loading: () => const CircularProgressIndicator(),
              error: (_, __) => const Icon(Icons.error),
            ),

            const SizedBox(height: 16),

            // Display name
            displayNameAsync.when(
              data: (displayName) => Text(
                displayName,
                style: const TextStyle(
                  fontSize: 20,
                  fontWeight: FontWeight.bold,
                ),
              ),
              loading: () => const Text('Loading...'),
              error: (error, _) => Text('Error: $error'),
            ),

            const SizedBox(height: 32),

            // Full user data
            userAsync.when(
              data: (user) => Card(
                margin: const EdgeInsets.all(16),
                child: Padding(
                  padding: const EdgeInsets.all(16),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Text('ID: ${user.id}'),
                      Text('Name: ${user.name}'),
                      Text('Email: ${user.email}'),
                    ],
                  ),
                ),
              ),
              loading: () => const CircularProgressIndicator(),
              error: (error, stack) => Column(
                children: [
                  Text('Failed to load user: $error'),
                  const SizedBox(height: 8),
                  ElevatedButton(
                    onPressed: () => ref.invalidate(userProvider(userId)),
                    child: const Text('Retry'),
                  ),
                ],
              ),
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

## 🔗 الخطوة الجاية

في الملف الجاي هنتعلم **Pattern Matching** في Dart 3:
- كل أنواع الـ patterns
- Switch expressions
- If-case statements
- Destructuring
- أمثلة عملية مع AsyncValue

جاهز؟ يلا نكمل! 🚀

---

## 📚 المصادر

- [AsyncValue API Documentation](https://pub.dev/documentation/riverpod/latest/riverpod/AsyncValue-class.html)
- [AsyncValue Best Practices](https://codewithandrea.com/tips/async-value-guard-try-catch/)
- [Handling AsyncValue UI States](https://riverpod.dev/docs/essentials/side_effects#showing-loading-and-error-states)

</div>
