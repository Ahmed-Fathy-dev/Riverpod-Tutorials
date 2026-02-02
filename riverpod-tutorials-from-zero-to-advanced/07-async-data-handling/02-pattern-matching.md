<div dir="rtl">

# Pattern Matching - Dart 3 + AsyncValue

**المستوى**: 🟡 متوسط

## 📌 المفاهيم الأساسية

في الملف ده هنتعلم:
- Pattern Matching في Dart 3
- Switch Expressions الجديدة
- كل أنواع الـ Patterns
- Migration من `.when()` لـ Pattern Matching
- Best Practices

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تستخدم Dart 3 Pattern Matching بكفاءة
- تكتب كود أوضح وأقصر
- تستغل Exhaustiveness Checking
- تهاجر من `.when()` لـ `switch`

---

## 🆕 ما هو Pattern Matching؟

**Pattern Matching** هو feature جديد في Dart 3 بيخليك تطابق values ضد "patterns" معينة.

### قبل Dart 3:

</div>

```dart
// Old way - verbose if-else chains
final asyncValue = ref.watch(userProvider);

if (asyncValue is AsyncData<User>) {
  final user = (asyncValue as AsyncData<User>).value;
  return Text('Hello ${user.name}');
} else if (asyncValue is AsyncError<User>) {
  final error = (asyncValue as AsyncError<User>).error;
  return Text('Error: $error');
} else {
  return const CircularProgressIndicator();
}
```

<div dir="rtl">

### بعد Dart 3:

</div>

```dart
// New way - clean switch expression
final asyncValue = ref.watch(userProvider);

return switch (asyncValue) {
  AsyncData(:final value) => Text('Hello ${value.name}'),
  AsyncError(:final error) => Text('Error: $error'),
  _ => const CircularProgressIndicator(),
};
```

<div dir="rtl">

**المميزات:**
- ✅ **Concise** - كود أقصر بكتير
- ✅ **Type-safe** - الـ compiler بيتأكد من الـ types
- ✅ **Exhaustive** - لو نسيت حالة، الـ compiler يديك warning
- ✅ **Readable** - واضح ومباشر

---

## 🔄 Switch Expressions vs Switch Statements

### Switch Statement (قديم)

</div>

```dart
// Switch statement - doesn't return value
final asyncValue = ref.watch(userProvider);
Widget widget;

switch (asyncValue) {
  case AsyncData(:final value):
    widget = Text('Hello ${value.name}');
    break;
  case AsyncError(:final error):
    widget = Text('Error: $error');
    break;
  default:
    widget = const CircularProgressIndicator();
}

return widget;
```

<div dir="rtl">

### Switch Expression (جديد)

</div>

```dart
// Switch expression - returns value directly
final asyncValue = ref.watch(userProvider);

return switch (asyncValue) {
  AsyncData(:final value) => Text('Hello ${value.name}'),
  AsyncError(:final error) => Text('Error: $error'),
  _ => const CircularProgressIndicator(),
};
```

<div dir="rtl">

**الفرق:**

| Feature | Switch Statement | Switch Expression |
|---------|------------------|-------------------|
| **Syntax** | `switch (x) { case ... }` | `switch (x) { case ... => ... }` |
| **Returns value** | ❌ لأ | ✅ أيوة |
| **break needed** | ✅ أيوة | ❌ لأ |
| **Use case** | Side effects | Direct return |

---

## 🎨 أنواع الـ Patterns

### 1. Constant Pattern - مطابقة قيم ثابتة

</div>

```dart
final status = 'loading';

final message = switch (status) {
  'loading' => 'جاري التحميل...',
  'success' => 'تم بنجاح!',
  'error' => 'حدث خطأ',
  _ => 'حالة غير معروفة',
};

print(message); // 'جاري التحميل...'

// With enums
enum LoadingState { idle, loading, success, error }

final state = LoadingState.loading;

final icon = switch (state) {
  LoadingState.idle => Icons.pause,
  LoadingState.loading => Icons.hourglass_empty,
  LoadingState.success => Icons.check,
  LoadingState.error => Icons.error,
};
```

<div dir="rtl">

---

### 2. Variable Pattern - استخراج القيم

</div>

```dart
// Simple variable pattern
final value = 42;

switch (value) {
  case int x when x > 0:
    print('Positive: $x');
  case int x when x < 0:
    print('Negative: $x');
  case 0:
    print('Zero');
}

// With AsyncValue
final userAsync = ref.watch(userProvider);

switch (userAsync) {
  // Extract value into variable
  case AsyncData(value: var user):
    print('User: ${user.name}');
  case AsyncError(error: var err):
    print('Error: $err');
  case AsyncLoading():
    print('Loading...');
}
```

<div dir="rtl">

---

### 3. Destructuring Pattern - تفكيك القيم

الأهم والأقوى! بيخليك تستخرج properties مباشرة.

</div>

```dart
// Destructuring with named fields
final userAsync = ref.watch(userProvider);

switch (userAsync) {
  // Extract 'value' directly using :final
  case AsyncData(:final value):
    print('User name: ${value.name}');

  // Extract 'error' directly
  case AsyncError(:final error):
    print('Error: $error');

  // Loading - no fields to extract
  case AsyncLoading():
    print('Loading...');
}

// Multiple fields
class Point {
  final int x;
  final int y;
  Point(this.x, this.y);
}

final point = Point(10, 20);

switch (point) {
  case Point(:final x, :final y):
    print('x: $x, y: $y'); // x: 10, y: 20
}
```

<div dir="rtl">

**الـ Syntax:**
- `:final value` - استخرج property اسمها `value`
- `:var value` - نفس الشيء لكن mutable
- `:final int value` - مع تحديد الـ type

---

### 4. Null-Check Pattern - التحقق من null

</div>

```dart
// Check if value is not null
final userAsync = ref.watch(userProvider);

switch (userAsync) {
  // value? means "value is not null"
  case AsyncValue(:final value?):
    print('Has value: ${value.name}');

  // error? means "error is not null"
  case AsyncValue(:final error?):
    print('Has error: $error');

  // Everything else (loading, or null data)
  case _:
    print('Loading or null');
}

// Practical example
@riverpod
class Activity extends _$Activity {
  @override
  Future<String?> build() async {
    return await api.getActivity(); // Can return null
  }
}

// In UI
Widget build(BuildContext context, WidgetRef ref) {
  final activityAsync = ref.watch(activityProvider);

  return switch (activityAsync) {
    // Only match if value is not null
    AsyncValue(:final value?) => Text('Activity: $value'),

    // Explicitly handle null data
    AsyncValue(hasValue: true, :final valueOrNull) when valueOrNull == null =>
      const Text('No activity available'),

    AsyncValue(:final error?) => Text('Error: $error'),
    _ => const CircularProgressIndicator(),
  };
}
```

<div dir="rtl">

---

### 5. Property Pattern - مطابقة بناءً على Properties

</div>

```dart
// Check properties with guards
final userAsync = ref.watch(userProvider);

switch (userAsync) {
  // Only if hasValue is true
  case AsyncValue(hasValue: true, :final value):
    print('Has data: ${value.name}');

  // Only if hasError is true
  case AsyncValue(hasError: true, :final error):
    print('Has error: $error');

  // Only if isLoading is true
  case AsyncValue(isLoading: true):
    print('Is loading');
}

// Multiple property checks
switch (userAsync) {
  // Has value AND is not reloading
  case AsyncValue(hasValue: true, isReloading: false, :final value):
    return UserCard(value);

  // Has value BUT is reloading (show with indicator)
  case AsyncValue(hasValue: true, isReloading: true, :final value):
    return Column(
      children: [
        const LinearProgressIndicator(),
        UserCard(value),
      ],
    );

  case AsyncValue(:final error?):
    return ErrorMessage(error);

  case _:
    return const LoadingScreen();
}
```

<div dir="rtl">

---

### 6. Guard Clauses - شروط إضافية مع `when`

</div>

```dart
// Add conditions with 'when'
final number = 42;

switch (number) {
  case int x when x > 100:
    print('Large: $x');
  case int x when x > 10:
    print('Medium: $x');
  case int x when x > 0:
    print('Small: $x');
  default:
    print('Zero or negative');
}

// With AsyncValue
final userAsync = ref.watch(userProvider);

switch (userAsync) {
  // Only adult users
  case AsyncData(:final value) when value.age >= 18:
    return Text('Adult user: ${value.name}');

  // Only young users
  case AsyncData(:final value) when value.age < 18:
    return Text('Young user: ${value.name}');

  case AsyncError(:final error):
    return Text('Error: $error');

  case _:
    return const CircularProgressIndicator();
}
```

<div dir="rtl">

---

### 7. Type Pattern - التحقق من الـ Type

</div>

```dart
// Check types
Object value = 'Hello';

switch (value) {
  case String s:
    print('String: $s');
  case int i:
    print('Int: $i');
  case double d:
    print('Double: $d');
  default:
    print('Unknown type');
}

// With AsyncValue - specific subclasses
final userAsync = ref.watch(userProvider);

switch (userAsync) {
  case AsyncData<User> data:
    print('Data: ${data.value}');
  case AsyncError<User> error:
    print('Error: ${error.error}');
  case AsyncLoading<User> loading:
    print('Loading with progress: ${loading.progress}');
}
```

<div dir="rtl">

---

### 8. List Pattern - مطابقة Lists

</div>

```dart
// Match list structure
final numbers = [1, 2, 3];

switch (numbers) {
  case []:
    print('Empty list');
  case [int x]:
    print('One element: $x');
  case [int x, int y]:
    print('Two elements: $x, $y');
  case [int first, ...List<int> rest]:
    print('First: $first, Rest: $rest');
}

// Real example with Riverpod
@riverpod
Future<List<User>> users(UsersRef ref) async {
  return await api.getUsers();
}

// UI
Widget build(BuildContext context, WidgetRef ref) {
  final usersAsync = ref.watch(usersProvider);

  return switch (usersAsync) {
    AsyncData(value: []) => const Text('No users'),
    AsyncData(value: [var user]) => Text('One user: ${user.name}'),
    AsyncData(value: [var first, var second]) => Text('Two users: ${first.name}, ${second.name}'),
    AsyncData(:final value) => Text('${value.length} users'),
    AsyncError(:final error) => Text('Error: $error'),
    _ => const CircularProgressIndicator(),
  };
}
```

<div dir="rtl">

---

### 9. Logical Patterns - OR / AND

</div>

```dart
// OR pattern (|)
final status = 404;

final message = switch (status) {
  200 | 201 | 204 => 'Success',
  400 | 404 => 'Client error',
  500 | 502 | 503 => 'Server error',
  _ => 'Unknown status',
};

// With AsyncValue
final userAsync = ref.watch(userProvider);

return switch (userAsync) {
  // Match EITHER loading OR reloading
  AsyncValue(isLoading: true) | AsyncValue(isReloading: true) =>
    const CircularProgressIndicator(),

  AsyncValue(:final value?) =>
    UserCard(value),

  AsyncValue(:final error?) =>
    ErrorMessage(error),

  _ => const SizedBox(),
};

// AND pattern (requires both conditions)
switch (userAsync) {
  // Has error AND is retrying
  case AsyncValue(hasError: true) when userAsync.isReloading:
    return const RetryingIndicator();

  case AsyncValue(hasError: true):
    return const ErrorMessage();

  case AsyncValue(:final value?):
    return UserCard(value);

  case _:
    return const LoadingScreen();
}
```

<div dir="rtl">

---

## 🔄 Migration: من `.when()` لـ Switch Expressions

Riverpod بيوصي بالانتقال من `.when()` لـ Pattern Matching. شوف [GitHub Issue #2715](https://github.com/rrousselGit/riverpod/issues/2715).

### المثال الأساسي

</div>

```dart
// ❌ OLD WAY - .when()
final userAsync = ref.watch(userProvider);

return userAsync.when(
  data: (user) => Text('Hello ${user.name}'),
  error: (error, stack) => Text('Error: $error'),
  loading: () => const CircularProgressIndicator(),
);

// ✅ NEW WAY - Switch expression
return switch (userAsync) {
  AsyncData(:final value) => Text('Hello ${value.name}'),
  AsyncError(:final error) => Text('Error: $error'),
  _ => const CircularProgressIndicator(),
};
```

<div dir="rtl">

---

### مع skipLoadingOnRefresh: true

الـ default behavior في `.when()` هو `skipLoadingOnRefresh: true`.

</div>

```dart
// ❌ OLD WAY
return userAsync.when(
  data: (user) => UserCard(user),
  error: (error, _) => ErrorMessage(error),
  loading: () => const LoadingScreen(),
  skipLoadingOnRefresh: true, // Default
);

// ✅ NEW WAY - Check error first
return switch (userAsync) {
  // Check error first (preserves refresh state with old data)
  AsyncValue(:final error?) => ErrorMessage(error),

  // Then check for data
  AsyncValue(hasValue: true, :final value) => UserCard(value),

  // Finally loading
  _ => const LoadingScreen(),
};

// Or simpler with null-check pattern
return switch (userAsync) {
  AsyncValue(:final error?) => ErrorMessage(error),
  AsyncValue(:final value?) => UserCard(value), // Only non-null values
  _ => const LoadingScreen(),
};
```

<div dir="rtl">

**ملاحظة:** ترتيب الـ cases مهم! لو حطيت error الأول، ده بيحافظ على الـ refresh behavior.

---

### مع skipLoadingOnReload: false

</div>

```dart
// ❌ OLD WAY
return userAsync.when(
  data: (user) => UserCard(user),
  error: (error, _) => ErrorMessage(error),
  loading: () => const LoadingScreen(),
  skipLoadingOnReload: false, // Show loading on reload
);

// ✅ NEW WAY - Add isReloading: false check
return switch (userAsync) {
  AsyncValue(:final error?) => ErrorMessage(error),

  // Only show data when NOT reloading
  AsyncValue(hasValue: true, isReloading: false, :final value) =>
    UserCard(value),

  _ => const LoadingScreen(),
};
```

<div dir="rtl">

---

### مع skipError: true

</div>

```dart
// ❌ OLD WAY
return userAsync.when(
  data: (user) => UserCard(user),
  error: (error, _) => UserCard(fallbackUser), // Show fallback on error
  loading: () => const LoadingScreen(),
  skipError: true,
);

// ✅ NEW WAY - Prioritize data
return switch (userAsync) {
  // Check data FIRST
  AsyncValue(hasValue: true, :final value) => UserCard(value),

  // Then error
  AsyncValue(:final error?) => UserCard(fallbackUser),

  // Finally loading
  _ => const LoadingScreen(),
};
```

<div dir="rtl">

---

### جدول Migration الكامل

| .when() flags | Switch pattern |
|--------------|----------------|
| **Default** | `AsyncValue(:final error?)` أولاً، بعدين `(:final value?)` |
| **skipLoadingOnReload: false** | أضف `isReloading: false` للـ data case |
| **skipError: true** | حط data case الأول |
| **skipLoadingOnRefresh: false** | استخدم `isRefreshing: false` |

---

## ✨ Exhaustiveness Checking

واحدة من أقوى المميزات: الـ compiler بيتأكد إنك غطيت كل الحالات!

### Sealed Classes

</div>

```dart
// AsyncValue is a sealed class
sealed class AsyncValue<T> {
  // Has 3 subtypes only
}

class AsyncData<T> extends AsyncValue<T> { ... }
class AsyncError<T> extends AsyncValue<T> { ... }
class AsyncLoading<T> extends AsyncValue<T> { ... }

// Switch must be exhaustive
final userAsync = ref.watch(userProvider);

// ❌ COMPILE ERROR - Missing AsyncError case!
return switch (userAsync) {
  AsyncData(:final value) => Text(value.name),
  AsyncLoading() => const CircularProgressIndicator(),
  // Missing AsyncError! Compiler error!
};

// ✅ CORRECT - All cases covered
return switch (userAsync) {
  AsyncData(:final value) => Text(value.name),
  AsyncError(:final error) => Text('Error: $error'),
  AsyncLoading() => const CircularProgressIndicator(),
};

// ✅ CORRECT - Using catch-all
return switch (userAsync) {
  AsyncData(:final value) => Text(value.name),
  _ => const CircularProgressIndicator(), // Covers error + loading
};
```

<div dir="rtl">

**المميزات:**
- ✅ لو نسيت حالة، الـ compiler يرفض compile
- ✅ لو السيلد كلاس اتغيرت، الكود القديم يديك error
- ✅ Refactoring-safe!

---

### مثال: Custom Sealed Class

</div>

```dart
// Define your own sealed class
sealed class Result<T> {}

class Success<T> extends Result<T> {
  final T data;
  Success(this.data);
}

class Failure<T> extends Result<T> {
  final String error;
  Failure(this.error);
}

class Loading<T> extends Result<T> {}

// Provider
@riverpod
Future<Result<User>> userResult(UserResultRef ref) async {
  try {
    final user = await api.getUser();
    return Success(user);
  } catch (e) {
    return Failure(e.toString());
  }
}

// UI - Exhaustive switch
Widget build(BuildContext context, WidgetRef ref) {
  final result = ref.watch(userResultProvider);

  return result.when(
    data: (value) => switch (value) {
      Success(:final data) => Text('User: ${data.name}'),
      Failure(:final error) => Text('Error: $error'),
      Loading() => const CircularProgressIndicator(),
      // No need for default - compiler knows these are all cases!
    },
    loading: () => const CircularProgressIndicator(),
    error: (error, _) => Text('Error: $error'),
  );
}
```

<div dir="rtl">

---

## 🎯 if-case Statements

لما محتاج تتحقق من pattern واحد بس.

</div>

```dart
// Basic if-case
final userAsync = ref.watch(userProvider);

if (userAsync case AsyncData(:final value)) {
  print('User name: ${value.name}');
}

// With else
if (userAsync case AsyncData(:final value)) {
  print('Has data: ${value.name}');
} else {
  print('No data yet');
}

// Multiple conditions
if (userAsync case AsyncData(:final value) when value.age >= 18) {
  print('Adult user: ${value.name}');
}

// Real example - Early return pattern
Widget build(BuildContext context, WidgetRef ref) {
  final userAsync = ref.watch(userProvider);

  // Handle error case first
  if (userAsync case AsyncValue(:final error?)) {
    return ErrorScreen(error);
  }

  // Handle loading
  if (userAsync case AsyncValue(isLoading: true)) {
    return const LoadingScreen();
  }

  // Extract data and build UI
  if (userAsync case AsyncData(:final value)) {
    return UserProfile(value);
  }

  // Fallback (shouldn't happen with exhaustive handling)
  return const SizedBox();
}
```

<div dir="rtl">

**متى تستخدم if-case:**
- ✅ لما محتاج تتحقق من pattern واحد
- ✅ Early returns
- ✅ Nested conditions
- ❌ لو عندك 3 cases أو أكتر، استخدم `switch` أفضل

---

## 🚀 أمثلة عملية متقدمة

### مثال 1: Pull-to-Refresh مع Refresh Indicator

</div>

```dart
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

class TodosScreen extends ConsumerWidget {
  const TodosScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final todosAsync = ref.watch(todosProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('Todos')),
      body: Column(
        children: [
          // Show linear indicator when refreshing with old data
          if (todosAsync case AsyncValue(isRefreshing: true, hasValue: true))
            const LinearProgressIndicator(),

          Expanded(
            child: RefreshIndicator(
              onRefresh: () => ref.read(todosProvider.notifier).refresh(),
              child: switch (todosAsync) {
                // Show todos
                AsyncValue(:final value?) when value.isNotEmpty =>
                  ListView.builder(
                    itemCount: value.length,
                    itemBuilder: (context, index) => TodoTile(value[index]),
                  ),

                // Empty list
                AsyncValue(:final value?) when value.isEmpty =>
                  const Center(child: Text('No todos')),

                // Error
                AsyncValue(:final error?) =>
                  Center(child: ErrorMessage(error)),

                // Loading (first time)
                _ => const Center(child: CircularProgressIndicator()),
              },
            ),
          ),
        ],
      ),
    );
  }
}
```

<div dir="rtl">

---

### مثال 2: Optimistic Updates مع Pattern Matching

</div>

```dart
@riverpod
class Counter extends _$Counter {
  @override
  Future<int> build() async {
    return 0;
  }

  Future<void> increment() async {
    // Get current value
    final currentValue = state.valueOrNull ?? 0;

    // Optimistically update
    state = AsyncValue.data(currentValue + 1);

    // Try to save on server
    try {
      await api.incrementCounter();
    } catch (error, stackTrace) {
      // Revert on error
      state = AsyncValue.data(currentValue);
      // Show error
      state = AsyncValue.error(error, stackTrace);
    }
  }
}

class CounterScreen extends ConsumerWidget {
  const CounterScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final counterAsync = ref.watch(counterProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('Counter')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Show counter with different states
            switch (counterAsync) {
              // Show value with saving indicator
              AsyncValue(hasValue: true, isReloading: true, :final value) =>
                Column(
                  children: [
                    Text('$value', style: const TextStyle(fontSize: 48)),
                    const SizedBox(height: 8),
                    const Text('Saving...', style: TextStyle(fontSize: 12)),
                  ],
                ),

              // Show value normally
              AsyncValue(:final value?) =>
                Text('$value', style: const TextStyle(fontSize: 48)),

              // Show error with last value if available
              AsyncValue(hasError: true, :final value) when value != null =>
                Column(
                  children: [
                    Text('$value', style: const TextStyle(fontSize: 48)),
                    const Text('Failed to save', style: TextStyle(color: Colors.red)),
                  ],
                ),

              // Loading first time
              _ => const CircularProgressIndicator(),
            },

            const SizedBox(height: 32),

            // Increment button
            ElevatedButton(
              onPressed: () => ref.read(counterProvider.notifier).increment(),
              child: const Text('Increment'),
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

### مثال 3: Pagination مع Pattern Matching

</div>

```dart
class PaginatedData<T> {
  final List<T> items;
  final int page;
  final bool hasMore;

  PaginatedData({
    required this.items,
    required this.page,
    required this.hasMore,
  });

  PaginatedData<T> copyWith({
    List<T>? items,
    int? page,
    bool? hasMore,
  }) {
    return PaginatedData(
      items: items ?? this.items,
      page: page ?? this.page,
      hasMore: hasMore ?? this.hasMore,
    );
  }
}

@riverpod
class PaginatedUsers extends _$PaginatedUsers {
  @override
  Future<PaginatedData<User>> build() async {
    final users = await api.getUsers(page: 1);
    return PaginatedData(
      items: users,
      page: 1,
      hasMore: users.length >= 20,
    );
  }

  Future<void> loadMore() async {
    // Get current state
    final currentData = state.valueOrNull;
    if (currentData == null || !currentData.hasMore) return;

    // Set loading for next page
    state = AsyncValue.data(currentData); // Keep current data

    // Load next page
    final result = await AsyncValue.guard(() async {
      final newUsers = await api.getUsers(page: currentData.page + 1);
      return currentData.copyWith(
        items: [...currentData.items, ...newUsers],
        page: currentData.page + 1,
        hasMore: newUsers.length >= 20,
      );
    });

    state = result;
  }
}

class UsersListScreen extends ConsumerWidget {
  const UsersListScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final usersAsync = ref.watch(paginatedUsersProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('Users')),
      body: switch (usersAsync) {
        // Error state
        AsyncValue(hasError: true, hasValue: false, :final error) =>
          Center(child: ErrorMessage(error)),

        // Error but with previous data
        AsyncValue(hasError: true, :final value, :final error) when value != null =>
          Column(
            children: [
              Expanded(child: _UsersList(value.items)),
              ErrorBanner(error),
            ],
          ),

        // Data state
        AsyncValue(:final value?) => NotificationListener<ScrollNotification>(
            onNotification: (notification) {
              if (notification.metrics.pixels >= notification.metrics.maxScrollExtent * 0.8) {
                if (value.hasMore) {
                  ref.read(paginatedUsersProvider.notifier).loadMore();
                }
              }
              return false;
            },
            child: CustomScrollView(
              slivers: [
                SliverList(
                  delegate: SliverChildBuilderDelegate(
                    (context, index) => UserTile(value.items[index]),
                    childCount: value.items.length,
                  ),
                ),
                // Loading indicator at bottom when loading more
                if (value.hasMore && usersAsync.isReloading)
                  const SliverToBoxAdapter(
                    child: Padding(
                      padding: EdgeInsets.all(16),
                      child: Center(child: CircularProgressIndicator()),
                    ),
                  ),
              ],
            ),
          ),

        // Initial loading
        _ => const Center(child: CircularProgressIndicator()),
      },
    );
  }
}

class _UsersList extends StatelessWidget {
  final List<User> users;
  const _UsersList(this.users);

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: users.length,
      itemBuilder: (context, index) => UserTile(users[index]),
    );
  }
}
```

<div dir="rtl">

---

## 📋 Best Practices

### 1. استخدم Switch Expressions بدل .when()

</div>

```dart
// ✅ GOOD - Modern pattern matching
return switch (asyncValue) {
  AsyncData(:final value) => Text(value),
  AsyncError(:final error) => Text('Error: $error'),
  _ => const CircularProgressIndicator(),
};

// ❌ OLD - Still works but verbose
return asyncValue.when(
  data: (value) => Text(value),
  error: (error, _) => Text('Error: $error'),
  loading: () => const CircularProgressIndicator(),
);
```

<div dir="rtl">

### 2. استخدم Null-Check Pattern للاختصار

</div>

```dart
// ✅ GOOD - Concise
return switch (asyncValue) {
  AsyncValue(:final value?) => UserCard(value),
  AsyncValue(:final error?) => ErrorMessage(error),
  _ => const LoadingScreen(),
};

// ❌ VERBOSE
return switch (asyncValue) {
  AsyncValue(hasValue: true, :final value) => UserCard(value),
  AsyncValue(hasError: true, :final error) => ErrorMessage(error),
  AsyncValue(isLoading: true) => const LoadingScreen(),
};
```

<div dir="rtl">

### 3. رتب الـ Cases حسب الأولوية

</div>

```dart
// ✅ GOOD - Check error first (preserves refresh behavior)
return switch (asyncValue) {
  AsyncValue(:final error?) => ErrorMessage(error),
  AsyncValue(:final value?) => UserCard(value),
  _ => const LoadingScreen(),
};

// ❌ BAD - Data first might hide errors during refresh
return switch (asyncValue) {
  AsyncValue(:final value?) => UserCard(value),
  AsyncValue(:final error?) => ErrorMessage(error),
  _ => const LoadingScreen(),
};
```

<div dir="rtl">

### 4. استخدم if-case للـ Early Returns

</div>

```dart
// ✅ GOOD - Clear early returns
Widget build(BuildContext context, WidgetRef ref) {
  final userAsync = ref.watch(userProvider);

  if (userAsync case AsyncValue(:final error?)) {
    return ErrorScreen(error);
  }

  if (userAsync case AsyncValue(isLoading: true)) {
    return const LoadingScreen();
  }

  final user = userAsync.requireValue;
  return UserProfile(user);
}

// ❌ VERBOSE - Deep nesting
Widget build(BuildContext context, WidgetRef ref) {
  final userAsync = ref.watch(userProvider);

  return switch (userAsync) {
    AsyncValue(:final error?) => ErrorScreen(error),
    AsyncValue(isLoading: true) => const LoadingScreen(),
    AsyncValue(:final value) => UserProfile(value),
  };
}
```

<div dir="rtl">

### 5. استغل Exhaustiveness Checking

</div>

```dart
// ✅ GOOD - Compiler ensures all cases covered
sealed class Status {}
class Idle extends Status {}
class Loading extends Status {}
class Success extends Status {}
class Error extends Status {}

Widget getIcon(Status status) {
  return switch (status) {
    Idle() => const Icon(Icons.pause),
    Loading() => const Icon(Icons.hourglass_empty),
    Success() => const Icon(Icons.check),
    Error() => const Icon(Icons.error),
    // If you add a new status, compiler will error here!
  };
}
```

<div dir="rtl">

### 6. Handle isRefreshing للـ Better UX

</div>

```dart
// ✅ GOOD - Show indicator when refreshing
return switch (asyncValue) {
  AsyncValue(isRefreshing: true, :final value?) =>
    Column(
      children: [
        const LinearProgressIndicator(),
        UserCard(value),
      ],
    ),

  AsyncValue(:final value?) => UserCard(value),
  AsyncValue(:final error?) => ErrorMessage(error),
  _ => const LoadingScreen(),
};
```

<div dir="rtl">

---

## 🔗 الخطوة الجاية

في الملف الجاي هنتعلم **Error Handling** بشكل احترافي:
- AsyncValue.guard
- Error recovery strategies
- Retry logic
- Error UI patterns
- Global error handling

جاهز؟ يلا نكمل! 🚀

---

## 📚 المصادر

- [Patterns | Dart](https://dart.dev/language/patterns)
- [Pattern Types | Dart](https://dart.dev/language/pattern-types)
- [Add a page showcasing migration from AsyncValue.map/when to Dart 3's switch-case | Riverpod](https://github.com/rrousselGit/riverpod/issues/2715)
- [Branches | Dart](https://dart.dev/language/branches)

</div>
