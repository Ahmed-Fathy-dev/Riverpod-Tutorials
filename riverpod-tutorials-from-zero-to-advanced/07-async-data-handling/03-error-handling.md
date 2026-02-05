<div dir="rtl">

# Error Handling - الدليل الاحترافي

**المستوى**: 🟡 متوسط

## 📌 المفاهيم الأساسية

في الملف ده هنتعلم:
- `AsyncValue.guard()` - الـ error handling المثالي
- متى تستخدم guard vs try-catch
- Error recovery strategies
- Retry mechanisms
- Error UI patterns
- `ref.listen()` للـ error handling

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تعمل error handling محترف
- تستخدم AsyncValue.guard بشكل صحيح
- تعمل retry logic
- تختار Error UI pattern المناسب
- تفرق بين global و local error handling

---

## 🛡️ AsyncValue.guard() - البطل الخارق

`AsyncValue.guard()` هي **static method** بتحول Future لـ AsyncValue وبتمسك الـ errors تلقائياً!

### المشكلة: try-catch Manual

</div>

```dart
// ❌ BAD - Manual try-catch (verbose and error-prone)
@riverpod
class Todos extends _$Todos {
  @override
  Future<List<Todo>> build() async {
    return await api.getTodos();
  }

  Future<void> addTodo(String title) async {
    // Set loading
    state = const AsyncValue.loading();

    try {
      // Do the work
      await api.addTodo(title);
      final todos = await api.getTodos();

      // Set data
      state = AsyncValue.data(todos);
    } catch (error, stackTrace) {
      // Set error - easy to forget stackTrace!
      state = AsyncValue.error(error, stackTrace);
    }
  }
}
```

<div dir="rtl">

**مشاكل try-catch:**
- ❌ Verbose - كود كتير
- ❌ Repetitive - هتكرر نفس الكود في كل method
- ❌ Error-prone - ممكن تنسى الـ stackTrace
- ❌ Boilerplate - شغل زيادة

### الحل: AsyncValue.guard()

</div>

```dart
// ✅ GOOD - AsyncValue.guard (clean and safe)
@riverpod
class Todos extends _$Todos {
  @override
  Future<List<Todo>> build() async {
    return await api.getTodos();
  }

  Future<void> addTodo(String title) async {
    // Set loading
    state = const AsyncValue.loading();

    // guard() handles everything!
    state = await AsyncValue.guard(() async {
      await api.addTodo(title);
      return await api.getTodos();
    });

    // If success: state = AsyncData(todos)
    // If error: state = AsyncError(error, stackTrace)
    // No try-catch needed! 🎉
  }
}
```

<div dir="rtl">

**مميزات guard():**
- ✅ **Concise** - كود أقل بكتير
- ✅ **Safe** - بتمسك الـ errors تلقائياً
- ✅ **Correct stackTrace** - بتسجل الـ stack trace صح
- ✅ **No boilerplate** - مفيش كود زيادة

---

## 📖 كيف تشتغل guard()?

### التعريف

</div>

```dart
// Signature
static Future<AsyncValue<T>> guard<T>(Future<T> Function() future)

// Returns:
// - AsyncData<T>(value) if successful
// - AsyncError<T>(error, stackTrace) if failed
```

<div dir="rtl">

### مثال تفصيلي

</div>

```dart
@riverpod
class Counter extends _$Counter {
  @override
  Future<int> build() async {
    return 0;
  }

  Future<void> increment() async {
    print('1. Setting loading');
    state = const AsyncValue.loading();

    print('2. Calling guard');
    state = await AsyncValue.guard(() async {
      print('3. Inside guard - starting async work');

      // Simulate API call
      await Future.delayed(const Duration(seconds: 1));

      // Simulate random failure (50% chance)
      if (DateTime.now().second % 2 == 0) {
        print('4. Throwing error');
        throw Exception('Random failure!');
      }

      final currentValue = state.valueOrNull ?? 0;
      print('5. Returning success value');
      return currentValue + 1;
    });

    // If success:
    // state = AsyncData(1)
    // print('6. State is now AsyncData')

    // If error:
    // state = AsyncError('Random failure!', stackTrace)
    // print('6. State is now AsyncError')

    print('7. Done');
  }
}
```

<div dir="rtl">

---

## 🎯 متى تستخدم guard vs try-catch؟

### استخدم guard() عندما:

</div>

```dart
// ✅ Use guard() in AsyncNotifier mutations
@riverpod
class Todos extends _$Todos {
  @override
  Future<List<Todo>> build() async {
    return await api.getTodos();
  }

  // ✅ Perfect for guard()
  Future<void> addTodo(String title) async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      await api.addTodo(title);
      return await api.getTodos();
    });
  }

  // ✅ Perfect for guard()
  Future<void> deleteTodo(String id) async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      await api.deleteTodo(id);
      return await api.getTodos();
    });
  }

  // ✅ Perfect for guard()
  Future<void> toggleTodo(String id) async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      await api.toggleTodo(id);
      return await api.getTodos();
    });
  }
}
```

<div dir="rtl">

### استخدم try-catch عندما:

</div>

```dart
// ✅ Use try-catch for specific error types
@riverpod
class Auth extends _$Auth {
  @override
  Future<User?> build() async {
    return await authService.getCurrentUser();
  }

  Future<void> login(String email, String password) async {
    state = const AsyncValue.loading();

    try {
      final user = await authService.login(email, password);
      state = AsyncValue.data(user);
    } on InvalidCredentialsException {
      // Handle specific error type
      state = AsyncValue.error(
        'Invalid email or password',
        StackTrace.current,
      );
    } on NetworkException {
      // Handle network errors differently
      state = AsyncValue.error(
        'No internet connection',
        StackTrace.current,
      );
    } catch (error, stackTrace) {
      // Generic error
      state = AsyncValue.error(error, stackTrace);
    }
  }
}

// ✅ Use try-catch for partial error handling
Future<void> updateProfile(User user) async {
  final previousState = state;

  try {
    state = const AsyncValue.loading();
    await api.updateUser(user);
    state = AsyncValue.data(user);

    // Show success message
    showSuccessSnackBar('Profile updated');
  } catch (error) {
    // Revert to previous state
    state = previousState;

    // Show error in SnackBar (don't update state to error)
    showErrorSnackBar('Failed to update profile');
  }
}
```

<div dir="rtl">

**الخلاصة:**

| Scenario | Use |
|----------|-----|
| **AsyncNotifier mutations** | ✅ `AsyncValue.guard()` |
| **Simple async operations** | ✅ `AsyncValue.guard()` |
| **Need specific error types** | ✅ `try-catch` |
| **Need partial error handling** | ✅ `try-catch` |
| **Want state to be error** | ✅ `AsyncValue.guard()` |
| **Want local error (SnackBar)** | ✅ `try-catch` |

---

## 🔄 Error Recovery Strategies

### 1. Optimistic Updates مع Rollback

</div>

```dart
@riverpod
class Todos extends _$Todos {
  @override
  Future<List<Todo>> build() async {
    return await api.getTodos();
  }

  Future<void> toggleTodo(String id) async {
    // Save previous state for rollback
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

    // Try to save on server
    try {
      await api.toggleTodo(id);
    } catch (error, stackTrace) {
      // Rollback on error
      state = previousState;

      // Show error
      state = AsyncValue.error(error, stackTrace);
    }
  }
}
```

<div dir="rtl">

---

### 2. Keep Previous Data on Error

</div>

```dart
@riverpod
class Users extends _$Users {
  @override
  Future<List<User>> build() async {
    return await api.getUsers();
  }

  Future<void> refresh() async {
    // Get current data
    final previousData = state.valueOrNull;

    // Set loading
    state = const AsyncValue.loading();

    // Try to refresh
    final result = await AsyncValue.guard(() => api.getUsers());

    // If error and we have previous data, keep it
    if (result.hasError && previousData != null) {
      // Show error but keep old data in UI
      state = AsyncValue.error(
        result.error!,
        result.stackTrace!,
      );

      // Alternative: Keep data and show error in SnackBar
      state = AsyncValue.data(previousData);
      showErrorSnackBar('Failed to refresh, showing cached data');
    } else {
      state = result;
    }
  }
}
```

<div dir="rtl">

---

### 3. Retry with Exponential Backoff

</div>

```dart
@riverpod
class DataController extends _$DataController {
  @override
  Future<Data> build() async {
    return await _fetchDataWithRetry();
  }

  Future<Data> _fetchDataWithRetry({int maxRetries = 3}) async {
    int retryCount = 0;
    Duration delay = const Duration(seconds: 1);

    while (true) {
      try {
        return await api.getData();
      } catch (error) {
        retryCount++;

        if (retryCount >= maxRetries) {
          // Max retries reached, throw error
          rethrow;
        }

        // Wait before retrying (exponential backoff)
        await Future.delayed(delay);
        delay *= 2; // Double the delay (1s, 2s, 4s)
      }
    }
  }

  Future<void> refresh() async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() => _fetchDataWithRetry());
  }
}
```

<div dir="rtl">

---

### 4. Fallback Values

</div>

```dart
@riverpod
class Config extends _$Config {
  @override
  Future<AppConfig> build() async {
    try {
      // Try to fetch from API
      return await api.getConfig();
    } catch (error) {
      // Fallback to default config
      return AppConfig.defaultConfig();
    }
  }
}

// Or with AsyncValue
@riverpod
Future<List<Product>> products(ProductsRef ref) async {
  try {
    return await api.getProducts();
  } catch (error) {
    // Fallback to cached products
    final cached = await cache.getProducts();
    if (cached.isNotEmpty) {
      return cached;
    }
    rethrow; // No cache, propagate error
  }
}
```

<div dir="rtl">

---

## 🔁 Retry Mechanisms

### 1. Manual Retry Button

</div>

```dart
// Provider
@riverpod
Future<User> user(UserRef ref) async {
  return await api.getUser();
}

// UI
class UserScreen extends ConsumerWidget {
  const UserScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider);

    return switch (userAsync) {
      AsyncData(:final value) => UserCard(value),

      AsyncError(:final error) => Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Text('Error: $error'),
              const SizedBox(height: 16),
              ElevatedButton(
                // Retry by invalidating provider
                onPressed: () => ref.invalidate(userProvider),
                child: const Text('Retry'),
              ),
            ],
          ),
        ),

      _ => const Center(child: CircularProgressIndicator()),
    };
  }
}
```

<div dir="rtl">

---

### 2. Automatic Retry Logic

</div>

```dart
// Retry helper
Future<T> retryOperation<T>(
  Future<T> Function() operation, {
  int maxAttempts = 3,
  Duration initialDelay = const Duration(seconds: 1),
  double delayFactor = 2.0,
}) async {
  int attempt = 0;
  Duration delay = initialDelay;

  while (true) {
    try {
      return await operation();
    } catch (error) {
      attempt++;

      if (attempt >= maxAttempts) {
        rethrow; // Max attempts reached
      }

      // Wait before retrying
      await Future.delayed(delay);
      delay *= delayFactor; // Exponential backoff
    }
  }
}

// Usage in provider
@riverpod
Future<User> user(UserRef ref) async {
  return await retryOperation(
    () => api.getUser(),
    maxAttempts: 3,
    initialDelay: const Duration(seconds: 1),
  );
}
```

<div dir="rtl">

---

### 3. Retry with User Feedback

</div>

```dart
@riverpod
class DataWithRetry extends _$DataWithRetry {
  @override
  Future<Data> build() async {
    return await _fetchWithRetry();
  }

  int _retryCount = 0;
  static const _maxRetries = 3;

  Future<Data> _fetchWithRetry() async {
    try {
      return await api.getData();
    } catch (error) {
      _retryCount++;

      if (_retryCount < _maxRetries) {
        // Show retry indicator
        state = AsyncValue.error(
          'Failed (attempt $_retryCount/$_maxRetries)',
          StackTrace.current,
        );

        // Wait before retry
        await Future.delayed(Duration(seconds: _retryCount));

        // Retry
        return await _fetchWithRetry();
      }

      // Max retries reached
      rethrow;
    }
  }

  Future<void> reset() async {
    _retryCount = 0;
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() => _fetchWithRetry());
  }
}
```

<div dir="rtl">

---

## 🛡️ Async Safety مع ref.mounted

**جديد في Riverpod 3.0!** 🆕

عندما تتعامل مع async operations، في خطر إن الـ provider/notifier يكون اتعمل dispose خلال الـ await. ده بيسبب **race conditions** وممكن يؤدي لـ exceptions أو memory leaks.

الحل: استخدم `ref.mounted` للتحقق قبل أي state update.

### المشكلة: Race Conditions في Error Handling

</div>

```dart
// ❌ خطر - بدون ref.mounted
@riverpod
class UserProfile extends _$UserProfile {
  @override
  Future<User> build() async {
    return await api.getUser();
  }

  Future<void> updateProfile(User updatedUser) async {
    state = const AsyncValue.loading();

    try {
      await api.updateUser(updatedUser);
      final freshUser = await api.getUser();

      // ⚠️ خطر! لو User طلع من الشاشة خلال الـ await،
      // الـ notifier قد يكون disposed والـ state update هيرمي exception!
      state = AsyncValue.data(freshUser);
    } catch (e, s) {
      // ⚠️ نفس الخطر في catch!
      state = AsyncValue.error(e, s);
    }
  }
}
```

<div dir="rtl">

**النتيجة:**
- Exception: "Cannot update state of disposed notifier"
- Memory leaks
- Unpredictable behavior

### الحل: استخدم ref.mounted

</div>

```dart
// ✅ آمن - مع ref.mounted
@riverpod
class UserProfile extends _$UserProfile {
  @override
  Future<User> build() async {
    return await api.getUser();
  }

  Future<void> updateProfile(User updatedUser) async {
    state = const AsyncValue.loading();

    try {
      await api.updateUser(updatedUser);

      // تحقق قبل fetch
      if (!ref.mounted) return;

      final freshUser = await api.getUser();

      // تحقق قبل update
      if (!ref.mounted) return;

      state = AsyncValue.data(freshUser);
    } catch (e, s) {
      // تحقق حتى في catch
      if (ref.mounted) {
        state = AsyncValue.error(e, s);
      }
    }
  }
}
```

<div dir="rtl">

### مع AsyncValue.guard()

`ref.mounted` يشتغل بشكل ممتاز مع `AsyncValue.guard()`:

</div>

```dart
@riverpod
class Products extends _$Products {
  @override
  Future<List<Product>> build() async {
    final products = await api.getProducts();

    // تحقق بعد async operation
    if (!ref.mounted) {
      throw Exception('Provider disposed');
    }

    return products;
  }

  Future<void> deleteProduct(String id) async {
    state = const AsyncValue.loading();

    // استخدم guard مع mounted check
    state = await AsyncValue.guard(() async {
      await api.deleteProduct(id);

      // تحقق بعد delete
      if (!ref.mounted) {
        throw Exception('Provider disposed');
      }

      final updatedProducts = await api.getProducts();

      // تحقق بعد fetch
      if (!ref.mounted) {
        throw Exception('Provider disposed');
      }

      return updatedProducts;
    });
  }

  Future<void> addProduct(Product product) async {
    // Pattern مع guard و mounted
    state = await AsyncValue.guard(() async {
      await api.addProduct(product);

      if (!ref.mounted) throw Exception('Disposed');

      return await api.getProducts();
    });

    // guard تمسك الـ exception تلقائياً وتحوله لـ AsyncError
  }
}
```

<div dir="rtl">

### في Multi-Step Operations

</div>

```dart
@riverpod
class Checkout extends _$Checkout {
  @override
  Future<Order?> build() async => null;

  Future<void> processOrder(Cart cart) async {
    state = const AsyncValue.loading();

    try {
      // Step 1: Validate
      await validateCart(cart);
      if (!ref.mounted) return; // تحقق بعد كل step

      // Step 2: Calculate totals
      final totals = await calculateTotals(cart);
      if (!ref.mounted) return;

      // Step 3: Process payment
      final payment = await processPayment(totals);
      if (!ref.mounted) return;

      // Step 4: Create order
      final order = await createOrder(cart, payment);
      if (!ref.mounted) return;

      // Step 5: Update state
      state = AsyncValue.data(order);
    } catch (e, s) {
      // تحقق قبل error state
      if (ref.mounted) {
        state = AsyncValue.error(e, s);
      }
    }
  }
}
```

<div dir="rtl">

### مع Retry Logic

</div>

```dart
@riverpod
class DataWithRetry extends _$DataWithRetry {
  @override
  Future<Data> build() async {
    return await fetchDataWithRetry();
  }

  Future<Data> fetchDataWithRetry({int maxRetries = 3}) async {
    var attempts = 0;

    while (attempts < maxRetries) {
      try {
        final data = await api.getData();

        // تحقق قبل return
        if (!ref.mounted) {
          throw Exception('Provider disposed');
        }

        return data;
      } catch (e) {
        attempts++;

        // تحقق قبل retry
        if (!ref.mounted) {
          throw Exception('Provider disposed during retry');
        }

        if (attempts >= maxRetries) {
          rethrow;
        }

        // Wait before retry
        await Future.delayed(Duration(seconds: attempts * 2));

        // تحقق بعد delay
        if (!ref.mounted) {
          throw Exception('Provider disposed during delay');
        }
      }
    }

    throw Exception('Max retries reached');
  }
}
```

<div dir="rtl">

### Best Practices:

1. **دايماً تحقق بعد await:**
   ```dart
   final result = await someOperation();
   if (!ref.mounted) return; // 👈 Critical
   state = result;
   ```

2. **في try-catch blocks:**
   ```dart
   try {
     final data = await api.call();
     if (!ref.mounted) return;
     state = AsyncValue.data(data);
   } catch (e, s) {
     if (ref.mounted) { // 👈 في catch أيضاً
       state = AsyncValue.error(e, s);
     }
   }
   ```

3. **في كل step من multi-step operation:**
   ```dart
   await step1();
   if (!ref.mounted) return;
   await step2();
   if (!ref.mounted) return;
   await step3();
   ```

4. **مع guard():**
   ```dart
   state = await AsyncValue.guard(() async {
     final result = await api.call();
     if (!ref.mounted) throw Exception('Disposed');
     return result;
   });
   ```

### ⚠️ أخطاء شائعة:

**خطأ 1: نسيان التحقق**
```dart
// ❌ Race condition
await api.call();
state = result; // خطر!
```

**خطأ 2: التحقق في المكان الخطأ**
```dart
// ❌ التحقق قبل await (مفيش فايدة)
if (!ref.mounted) return;
await api.call();
state = result; // لسه خطر!
```

**خطأ 3: نسيان التحقق في catch**
```dart
// ❌ خطر في catch block
try {
  await api.call();
  if (!ref.mounted) return;
  state = data;
} catch (e, s) {
  state = error; // ❌ مفيش mounted check!
}
```

**المصدر الرسمي:**
- [What's new in Riverpod 3.0 | Riverpod](https://riverpod.dev/docs/whats_new) - ref.mounted property

---

## 🎨 Error UI Patterns

### 1. Inline Error (في نفس المكان)

</div>

```dart
class UserProfile extends ConsumerWidget {
  const UserProfile({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider);

    return switch (userAsync) {
      AsyncData(:final value) => Column(
          children: [
            Text('Name: ${value.name}'),
            Text('Email: ${value.email}'),
          ],
        ),

      // Show error inline
      AsyncError(:final error) => Card(
          color: Colors.red.shade100,
          child: Padding(
            padding: const EdgeInsets.all(16),
            child: Column(
              children: [
                const Icon(Icons.error, color: Colors.red, size: 48),
                const SizedBox(height: 8),
                Text(
                  'Failed to load user',
                  style: const TextStyle(fontWeight: FontWeight.bold),
                ),
                Text('$error'),
                const SizedBox(height: 8),
                ElevatedButton(
                  onPressed: () => ref.invalidate(userProvider),
                  child: const Text('Retry'),
                ),
              ],
            ),
          ),
        ),

      _ => const CircularProgressIndicator(),
    };
  }
}
```

<div dir="rtl">

---

### 2. SnackBar Error (باستخدام ref.listen)

</div>

```dart
// Provider
@riverpod
class Todos extends _$Todos {
  @override
  Future<List<Todo>> build() async {
    return await api.getTodos();
  }

  Future<void> addTodo(String title) async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      await api.addTodo(title);
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

    // Listen for errors and show SnackBar
    ref.listen<AsyncValue<List<Todo>>>(
      todosProvider,
      (previous, next) {
        // Show error in SnackBar
        if (next.hasError) {
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(
              content: Text('Error: ${next.error}'),
              backgroundColor: Colors.red,
              action: SnackBarAction(
                label: 'Retry',
                onPressed: () => ref.invalidate(todosProvider),
              ),
            ),
          );
        }
      },
    );

    return Scaffold(
      appBar: AppBar(title: const Text('Todos')),
      body: switch (todosAsync) {
        AsyncData(:final value) => TodoList(value),
        _ => const Center(child: CircularProgressIndicator()),
        // No error case - handled by SnackBar!
      },
    );
  }
}
```

<div dir="rtl">

---

### 3. Error Dialog

</div>

```dart
class ProductsScreen extends ConsumerWidget {
  const ProductsScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final productsAsync = ref.watch(productsProvider);

    // Listen for errors and show dialog
    ref.listen<AsyncValue<List<Product>>>(
      productsProvider,
      (previous, next) {
        if (next.hasError) {
          showDialog(
            context: context,
            builder: (context) => AlertDialog(
              title: const Text('Error'),
              content: Text('Failed to load products:\n${next.error}'),
              actions: [
                TextButton(
                  onPressed: () => Navigator.pop(context),
                  child: const Text('Cancel'),
                ),
                TextButton(
                  onPressed: () {
                    Navigator.pop(context);
                    ref.invalidate(productsProvider);
                  },
                  child: const Text('Retry'),
                ),
              ],
            ),
          );
        }
      },
    );

    return Scaffold(
      appBar: AppBar(title: const Text('Products')),
      body: switch (productsAsync) {
        AsyncData(:final value) => ProductGrid(value),
        _ => const Center(child: CircularProgressIndicator()),
      },
    );
  }
}
```

<div dir="rtl">

---

### 4. Banner Error (فوق المحتوى)

</div>

```dart
class MessagesScreen extends ConsumerWidget {
  const MessagesScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final messagesAsync = ref.watch(messagesProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('Messages')),
      body: Column(
        children: [
          // Show error banner if error with previous data
          if (messagesAsync case AsyncValue(hasError: true, :final error))
            MaterialBanner(
              content: Text('Error: $error'),
              backgroundColor: Colors.red.shade100,
              leading: const Icon(Icons.error, color: Colors.red),
              actions: [
                TextButton(
                  onPressed: () => ref.invalidate(messagesProvider),
                  child: const Text('Retry'),
                ),
              ],
            ),

          // Main content
          Expanded(
            child: switch (messagesAsync) {
              AsyncData(:final value) => MessageList(value),
              AsyncValue(hasError: true, :final value) when value != null =>
                MessageList(value), // Show old data with error banner
              AsyncError() => const Center(child: Text('Failed to load')),
              _ => const Center(child: CircularProgressIndicator()),
            },
          ),
        ],
      ),
    );
  }
}
```

<div dir="rtl">

---

## 🌐 Global vs Local Error Handling

### Global Error Handling

استخدمه لـ:
- Logging errors
- Analytics
- Crash reporting (Sentry, Firebase Crashlytics)

</div>

```dart
// Global error observer
class AppErrorObserver extends ProviderObserver {
  @override
  void providerDidFail(
    ProviderBase<Object?> provider,
    Object error,
    StackTrace stackTrace,
    ProviderContainer container,
  ) {
    // Log to analytics
    Analytics.logError(error, stackTrace);

    // Send to crash reporting
    Crashlytics.recordError(error, stackTrace);

    // Log to console
    print('Provider ${provider.name} failed: $error');
  }
}

// In main.dart
void main() {
  runApp(
    ProviderScope(
      observers: [AppErrorObserver()],
      child: const MyApp(),
    ),
  );
}
```

<div dir="rtl">

### Local Error Handling

استخدمه لـ:
- UI feedback (SnackBar, Dialog)
- Error recovery
- Retry logic

</div>

```dart
class LocalErrorHandling extends ConsumerWidget {
  const LocalErrorHandling({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final dataAsync = ref.watch(dataProvider);

    // Local handling with ref.listen
    ref.listen<AsyncValue<Data>>(
      dataProvider,
      (previous, next) {
        // Show SnackBar on error
        if (next.hasError) {
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(content: Text('Error: ${next.error}')),
          );
        }

        // Show success message on data
        if (next.hasValue && previous?.hasValue == false) {
          ScaffoldMessenger.of(context).showSnackBar(
            const SnackBar(content: Text('Data loaded!')),
          );
        }
      },
    );

    return switch (dataAsync) {
      AsyncData(:final value) => DataWidget(value),
      _ => const CircularProgressIndicator(),
    };
  }
}
```

<div dir="rtl">

---

## 📋 Best Practices Summary

### 1. استخدم guard() في AsyncNotifier

</div>

```dart
// ✅ GOOD
Future<void> addTodo(String title) async {
  state = const AsyncValue.loading();
  state = await AsyncValue.guard(() async {
    await api.addTodo(title);
    return await api.getTodos();
  });
}

// ❌ BAD
Future<void> addTodo(String title) async {
  try {
    state = const AsyncValue.loading();
    await api.addTodo(title);
    final todos = await api.getTodos();
    state = AsyncValue.data(todos);
  } catch (error, stackTrace) {
    state = AsyncValue.error(error, stackTrace);
  }
}
```

<div dir="rtl">

### 2. استخدم ref.listen للـ Side Effects

</div>

```dart
// ✅ GOOD - Side effects in ref.listen
ref.listen<AsyncValue<void>>(
  actionProvider,
  (previous, next) {
    if (next.hasError) {
      showErrorSnackBar(context, next.error);
    }
  },
);

// ❌ BAD - Side effects in build
Widget build(BuildContext context, WidgetRef ref) {
  final actionAsync = ref.watch(actionProvider);
  if (actionAsync.hasError) {
    // ❌ Don't do this in build!
    showErrorSnackBar(context, actionAsync.error);
  }
  // ...
}
```

<div dir="rtl">

### 3. اختر Error UI pattern المناسب

| Pattern | Use Case |
|---------|----------|
| **Inline Error** | Data fetch errors (full screen) |
| **SnackBar** | Action errors (add, delete, update) |
| **Dialog** | Critical errors needing attention |
| **Banner** | Non-critical errors with data |

### 4. احتفظ بالـ Previous Data عند الإمكان

</div>

```dart
// ✅ GOOD - Keep showing old data
return switch (asyncValue) {
  AsyncValue(:final error?) => ErrorBanner(error),
  AsyncValue(:final value?) => DataWidget(value),
  _ => const LoadingScreen(),
};

// Shows old data with error banner!
```

<div dir="rtl">

### 5. استخدم Specific Error Types

</div>

```dart
// ✅ GOOD - Handle specific errors
try {
  await api.login(email, password);
} on InvalidCredentialsException {
  state = AsyncValue.error('Wrong email or password', StackTrace.current);
} on NetworkException {
  state = AsyncValue.error('No internet', StackTrace.current);
} catch (error, stackTrace) {
  state = AsyncValue.error(error, stackTrace);
}
```

<div dir="rtl">

### 6. أضف Retry Logic

</div>

```dart
// ✅ GOOD - Easy retry
AsyncError(:final error) => Column(
  children: [
    Text('Error: $error'),
    ElevatedButton(
      onPressed: () => ref.invalidate(provider),
      child: const Text('Retry'),
    ),
  ],
)
```

<div dir="rtl">

### 7. Log Errors للـ Debugging

</div>

```dart
// ✅ GOOD - Global error logging
class ErrorLogger extends ProviderObserver {
  @override
  void providerDidFail(
    ProviderBase provider,
    Object error,
    StackTrace stackTrace,
    ProviderContainer container,
  ) {
    // Log to console in development
    if (kDebugMode) {
      print('Error in ${provider.name}: $error\n$stackTrace');
    }

    // Send to crash reporting in production
    if (kReleaseMode) {
      Crashlytics.recordError(error, stackTrace);
    }
  }
}
```

<div dir="rtl">

---

## 🎓 أمثلة عملية كاملة

### مثال 1: Todo App مع Error Handling محترف

</div>

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'todos.g.dart';

// Models & API
class Todo {
  final String id;
  final String title;
  final bool completed;

  Todo({required this.id, required this.title, this.completed = false});

  Todo copyWith({bool? completed}) =>
      Todo(id: id, title: title, completed: completed ?? this.completed);
}

class TodosApi {
  Future<List<Todo>> getTodos() async {
    await Future.delayed(const Duration(seconds: 1));
    // Simulate random failure
    if (DateTime.now().second % 5 == 0) {
      throw Exception('Failed to fetch todos');
    }
    return [
      Todo(id: '1', title: 'Learn Riverpod'),
      Todo(id: '2', title: 'Build app'),
    ];
  }

  Future<void> addTodo(String title) async {
    await Future.delayed(const Duration(milliseconds: 500));
    if (title.isEmpty) {
      throw Exception('Title cannot be empty');
    }
  }

  Future<void> toggleTodo(String id) async {
    await Future.delayed(const Duration(milliseconds: 500));
  }

  Future<void> deleteTodo(String id) async {
    await Future.delayed(const Duration(milliseconds: 500));
  }
}

final todosApiProvider = Provider((ref) => TodosApi());

// Todos Provider
@riverpod
class Todos extends _$Todos {
  @override
  Future<List<Todo>> build() async {
    final api = ref.watch(todosApiProvider);
    return await api.getTodos();
  }

  Future<void> addTodo(String title) async {
    state = const AsyncValue.loading();

    state = await AsyncValue.guard(() async {
      final api = ref.read(todosApiProvider);
      await api.addTodo(title);
      return await api.getTodos();
    });
  }

  Future<void> toggleTodo(String id) async {
    // Optimistic update
    final previousState = state;

    state = state.whenData((todos) {
      return todos.map((todo) {
        if (todo.id == id) {
          return todo.copyWith(completed: !todo.completed);
        }
        return todo;
      }).toList();
    });

    // Try to save
    try {
      final api = ref.read(todosApiProvider);
      await api.toggleTodo(id);
    } catch (error, stackTrace) {
      // Rollback on error
      state = previousState;
      state = AsyncValue.error(error, stackTrace);
    }
  }

  Future<void> deleteTodo(String id) async {
    state = const AsyncValue.loading();

    state = await AsyncValue.guard(() async {
      final api = ref.read(todosApiProvider);
      await api.deleteTodo(id);
      return await api.getTodos();
    });
  }

  Future<void> refresh() async {
    // Keep previous data while refreshing
    final previousData = state.valueOrNull;

    state = const AsyncValue.loading();

    final result = await AsyncValue.guard(() async {
      final api = ref.read(todosApiProvider);
      return await api.getTodos();
    });

    // If error and we have previous data, show error but keep data
    if (result.hasError && previousData != null) {
      state = AsyncValue.data(previousData);
      // Error will be shown in SnackBar
    } else {
      state = result;
    }
  }
}

// UI
class TodosScreen extends ConsumerWidget {
  const TodosScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final todosAsync = ref.watch(todosProvider);

    // Listen for errors and show SnackBar
    ref.listen<AsyncValue<List<Todo>>>(
      todosProvider,
      (previous, next) {
        if (next.hasError) {
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(
              content: Text('Error: ${next.error}'),
              backgroundColor: Colors.red,
              action: SnackBarAction(
                label: 'Retry',
                textColor: Colors.white,
                onPressed: () => ref.invalidate(todosProvider),
              ),
            ),
          );
        }
      },
    );

    return Scaffold(
      appBar: AppBar(
        title: const Text('Todos with Error Handling'),
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

          Expanded(
            child: switch (todosAsync) {
              AsyncData(:final value) when value.isNotEmpty =>
                RefreshIndicator(
                  onRefresh: () => ref.read(todosProvider.notifier).refresh(),
                  child: ListView.builder(
                    itemCount: value.length,
                    itemBuilder: (context, index) {
                      final todo = value[index];
                      return ListTile(
                        leading: Checkbox(
                          value: todo.completed,
                          onChanged: (_) {
                            ref.read(todosProvider.notifier).toggleTodo(todo.id);
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
                            ref.read(todosProvider.notifier).deleteTodo(todo.id);
                          },
                        ),
                      );
                    },
                  ),
                ),

              AsyncData(:final value) when value.isEmpty =>
                const Center(child: Text('No todos yet')),

              AsyncError(:final error) => Center(
                  child: Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      const Icon(Icons.error, size: 64, color: Colors.red),
                      const SizedBox(height: 16),
                      Text('Error: $error'),
                      const SizedBox(height: 16),
                      ElevatedButton(
                        onPressed: () => ref.invalidate(todosProvider),
                        child: const Text('Retry'),
                      ),
                    ],
                  ),
                ),

              _ => const Center(child: CircularProgressIndicator()),
            },
          ),
        ],
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _showAddTodoDialog(context, ref),
        child: const Icon(Icons.add),
      ),
    );
  }

  Future<void> _showAddTodoDialog(BuildContext context, WidgetRef ref) async {
    final controller = TextEditingController();

    final result = await showDialog<bool>(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('Add Todo'),
        content: TextField(
          controller: controller,
          decoration: const InputDecoration(hintText: 'Todo title'),
          autofocus: true,
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context, false),
            child: const Text('Cancel'),
          ),
          TextButton(
            onPressed: () => Navigator.pop(context, true),
            child: const Text('Add'),
          ),
        ],
      ),
    );

    if (result == true && controller.text.isNotEmpty) {
      await ref.read(todosProvider.notifier).addTodo(controller.text);
    }
  }
}
```

<div dir="rtl">

---

## 🔗 الخطوة الجاية

في الملف الجاي هنتعلم **Loading States & UI Patterns**:
- Different loading indicators
- Skeleton loaders
- Pull-to-refresh
- Infinite scroll
- Shimmer effects

جاهز؟ يلا نكمل! 🚀

---

## 📚 المصادر

- [AsyncValue class | Riverpod](https://pub.dev/documentation/riverpod/latest/riverpod/AsyncValue-class.html)
- [Use AsyncValue.guard rather than try/catch](https://codewithandrea.com/tips/async-value-guard-try-catch/)
- [Best Practices: Handling loading/error mutation states | Riverpod Discussion](https://github.com/rrousselGit/riverpod/discussions/1231)
- [Why doesn't the new documentation use AsyncValue.guard? | Riverpod Discussion](https://github.com/rrousselGit/riverpod/discussions/3634)
- [Error Handling in Riverpod Guide](https://tillitsdone.com/blogs/error-handling-in-riverpod-guide/)

</div>
