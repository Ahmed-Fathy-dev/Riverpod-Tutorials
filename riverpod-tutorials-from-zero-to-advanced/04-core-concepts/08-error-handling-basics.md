<div dir="rtl">

# Error Handling Basics - أساسيات معالجة الأخطاء

**المستوى**: 🟡 متوسط

## 📌 المفاهيم الأساسية

في الملف ده هنتكلم عن:
- AsyncValue وحالات الـ error
- معالجة الأخطاء في async providers
- Retry strategies
- Error recovery
- عرض الأخطاء للمستخدم
- Best practices

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تتعامل مع الأخطاء بكفاءة
- تستخدم AsyncValue صح
- تعمل retry logic
- تعافي من الأخطاء
- تعرض أخطاء user-friendly

---

## 🔍 AsyncValue والأخطاء

**AsyncValue** عنده 3 حالات: data, loading, error.

</div>

```dart
// FutureProvider automatically captures errors in AsyncValue
final userProvider = FutureProvider<User>((ref) async {
  try {
    return await api.getUser();
  } catch (error, stackTrace) {
    // Error is automatically captured by AsyncValue
    rethrow;
  }
});

// In widget
class UserProfile extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider);

    // Handle all three states
    return userAsync.when(
      data: (user) => Text('Welcome, ${user.name}!'),
      loading: () => CircularProgressIndicator(),
      error: (error, stackTrace) => Text('Error: $error'),
    );
  }
}
```

<div dir="rtl">

---

## 🎨 طرق معالجة الأخطاء

### الطريقة 1: when() - معالجة كل الحالات

</div>

```dart
class DataWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final dataAsync = ref.watch(dataProvider);

    return dataAsync.when(
      // Success
      data: (data) => SuccessView(data),

      // Loading
      loading: () => LoadingView(),

      // Error
      error: (error, stackTrace) {
        if (error is NetworkException) {
          return NetworkErrorView(
            onRetry: () => ref.invalidate(dataProvider),
          );
        }

        if (error is AuthException) {
          return AuthErrorView();
        }

        return GenericErrorView(
          message: error.toString(),
          onRetry: () => ref.invalidate(dataProvider),
        );
      },
    );
  }
}
```

<div dir="rtl">

### الطريقة 2: maybeWhen() - معالجة حالات معينة

</div>

```dart
class SmartWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final dataAsync = ref.watch(dataProvider);

    return dataAsync.maybeWhen(
      // Only handle error, fallback to default for others
      error: (error, stackTrace) => ErrorView(error),

      // Default for data and loading
      orElse: () => DataView(dataAsync),
    );
  }
}
```

<div dir="rtl">

### الطريقة 3: map() - للـ Pattern Matching

</div>

```dart
class AdvancedWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final dataAsync = ref.watch(dataProvider);

    return dataAsync.map(
      data: (asyncData) {
        return SuccessView(asyncData.value);
      },
      loading: (asyncLoading) {
        // Can check if we have previous data
        if (asyncLoading.hasValue) {
          return RefreshingView(asyncLoading.value!);
        }
        return LoadingView();
      },
      error: (asyncError) {
        // Can check if we have previous data
        if (asyncError.hasValue) {
          return ErrorWithDataView(
            data: asyncError.value!,
            error: asyncError.error,
          );
        }
        return ErrorView(asyncError.error);
      },
    );
  }
}
```

<div dir="rtl">

### الطريقة 4: Direct Properties

</div>

```dart
class DirectCheckWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final dataAsync = ref.watch(dataProvider);

    // Direct checks
    if (dataAsync.isLoading && !dataAsync.hasValue) {
      return CircularProgressIndicator();
    }

    if (dataAsync.hasError) {
      return ErrorView(dataAsync.error!);
    }

    if (dataAsync.hasValue) {
      return DataView(dataAsync.value!);
    }

    return Container(); // Fallback
  }
}
```

<div dir="rtl">

---

## 🛡️ Error Handling في Providers

### Try-Catch في Async Providers

</div>

```dart
// FutureProvider with detailed error handling
final userProvider = FutureProvider<User>((ref) async {
  try {
    final response = await api.getUser();
    return response;
  } on NetworkException catch (e) {
    // Handle network errors specifically
    print('Network error: $e');
    throw NetworkError('Cannot connect to server');
  } on AuthException catch (e) {
    // Handle auth errors
    print('Auth error: $e');
    ref.read(authProvider.notifier).logout();
    throw AuthError('Please login again');
  } catch (e, stackTrace) {
    // Handle all other errors
    print('Unknown error: $e');
    print('Stack trace: $stackTrace');
    throw UnknownError('Something went wrong');
  }
});
```

<div dir="rtl">

### Error Handling في Notifiers

</div>

```dart
// AsyncNotifier for mutable async state with error handling
class TodosNotifier extends AsyncNotifier<List<Todo>> {
  @override
  Future<List<Todo>> build() async {
    try {
      return await api.getTodos();
    } catch (e, stackTrace) {
      // Initial load error
      return Future.error(e, stackTrace);
    }
  }

  Future<void> addTodo(String title) async {
    // Optimistic update
    final previousState = state;

    // Add to UI immediately
    state = AsyncData([
      ...state.value ?? [],
      Todo(id: 'temp', title: title),
    ]);

    try {
      // Save to server
      final newTodo = await api.createTodo(title);

      // Update with real data
      state = AsyncData([
        ...(state.value ?? []).where((t) => t.id != 'temp'),
        newTodo,
      ]);
    } catch (e, stackTrace) {
      // Rollback on error
      state = previousState;

      // Show error to user
      state = AsyncError(e, stackTrace);
    }
  }

  Future<void> deleteTodo(String id) async {
    final previousState = state;

    // Optimistic delete
    state = AsyncData(
      state.value?.where((t) => t.id != id).toList() ?? [],
    );

    try {
      await api.deleteTodo(id);
    } catch (e, stackTrace) {
      // Rollback on error
      state = previousState;
      state = AsyncError(e, stackTrace);
    }
  }
}

final todosProvider = AsyncNotifierProvider<TodosNotifier, List<Todo>>(
  () => TodosNotifier(),
);
```

<div dir="rtl">

---

## 🔄 Retry Strategies

### Manual Retry

</div>

```dart
class DataPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final dataAsync = ref.watch(dataProvider);

    return dataAsync.when(
      data: (data) => DataView(data),
      loading: () => LoadingView(),
      error: (error, stackTrace) => ErrorView(
        error: error,
        onRetry: () {
          // Invalidate to retry
          ref.invalidate(dataProvider);
        },
      ),
    );
  }
}
```

<div dir="rtl">

### Automatic Retry

</div>

```dart
// FutureProvider with automatic retry logic
final dataWithRetryProvider = FutureProvider<Data>((ref) async {
  var retries = 0;
  const maxRetries = 3;

  while (true) {
    try {
      return await api.getData();
    } catch (e) {
      retries++;

      if (retries >= maxRetries) {
        // Max retries reached, give up
        throw Exception('Failed after $maxRetries retries: $e');
      }

      // Wait before retrying (exponential backoff)
      await Future.delayed(Duration(seconds: retries * 2));

      print('Retry $retries/$maxRetries');
    }
  }
});
```

<div dir="rtl">

### Smart Retry (بناءً على نوع الخطأ)

</div>

```dart
// FutureProvider with smart retry (based on error type)
final userWithSmartRetryProvider = FutureProvider<User>((ref) async {
  var retries = 0;
  const maxRetries = 3;

  while (true) {
    try {
      return await api.getUser();
    } on NetworkException catch (e) {
      // Retry network errors
      retries++;

      if (retries >= maxRetries) {
        throw Exception('Network error after $maxRetries retries');
      }

      await Future.delayed(Duration(seconds: retries));
      print('Retrying due to network error...');

    } on AuthException catch (e) {
      // Don't retry auth errors
      throw Exception('Authentication failed: $e');

    } catch (e) {
      // Don't retry unknown errors
      throw Exception('Unknown error: $e');
    }
  }
});
```

<div dir="rtl">

### Pull-to-Refresh

</div>

```dart
class RefreshablePage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final dataAsync = ref.watch(dataProvider);

    return RefreshIndicator(
      onRefresh: () async {
        // Invalidate and wait for new data
        ref.invalidate(dataProvider);

        // Wait for the provider to finish loading
        await ref.read(dataProvider.future);
      },
      child: dataAsync.when(
        data: (data) => ListView(
          children: data.map((item) => ItemTile(item)).toList(),
        ),
        loading: () => Center(child: CircularProgressIndicator()),
        error: (error, stack) => ErrorView(error),
      ),
    );
  }
}
```

<div dir="rtl">

---

## 🎯 Error Recovery Patterns

### Pattern 1: Fallback Data

</div>

```dart
// FutureProvider with fallback to cached data
final productsProvider = FutureProvider<List<Product>>((ref) async {
  try {
    // Try to fetch from API
    return await api.getProducts();
  } catch (e) {
    print('API failed, using cached data');

    // Fallback to cache
    final cached = await cache.getProducts();

    if (cached.isNotEmpty) {
      return cached;
    }

    // No cache available, rethrow error
    rethrow;
  }
});
```

<div dir="rtl">

### Pattern 2: Partial Success

</div>

```dart
// FutureProvider with partial success handling
final dashboardProvider = FutureProvider<DashboardData>((ref) async {
  // Fetch multiple pieces of data
  final results = await Future.wait([
    api.getUser().catchError((e) => null),
    api.getPosts().catchError((e) => <Post>[]),
    api.getStats().catchError((e) => Stats.empty()),
  ]);

  return DashboardData(
    user: results[0] as User?,
    posts: results[1] as List<Post>,
    stats: results[2] as Stats,
  );
});

// Widget shows what succeeded
class DashboardPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final dataAsync = ref.watch(dashboardProvider);

    return dataAsync.when(
      data: (data) => Column(
        children: [
          if (data.user != null)
            UserHeader(data.user!)
          else
            Text('Could not load user'),

          if (data.posts.isNotEmpty)
            PostsList(data.posts)
          else
            Text('Could not load posts'),

          StatsWidget(data.stats),
        ],
      ),
      loading: () => CircularProgressIndicator(),
      error: (e, s) => ErrorView(e),
    );
  }
}
```

<div dir="rtl">

### Pattern 3: Error with Previous Data

</div>

```dart
// AsyncNotifier with polling and error handling
class LiveDataNotifier extends AsyncNotifier<Data> {
  @override
  Future<Data> build() async {
    // Start polling
    _startPolling();

    return await _fetchData();
  }

  void _startPolling() {
    Timer.periodic(Duration(seconds: 30), (timer) async {
      try {
        final newData = await _fetchData();
        state = AsyncData(newData);
      } catch (e, stackTrace) {
        // Keep previous data, but show error
        if (state.hasValue) {
          state = AsyncError(e, stackTrace, previousValue: state.value);
        } else {
          state = AsyncError(e, stackTrace);
        }
      }
    });
  }

  Future<Data> _fetchData() async {
    return await api.getData();
  }
}

final liveDataProvider = AsyncNotifierProvider<LiveDataNotifier, Data>(
  () => LiveDataNotifier(),
);

// Widget shows error but keeps displaying data
class LiveDataWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final dataAsync = ref.watch(liveDataProvider);

    return Column(
      children: [
        // Show error banner if error exists
        if (dataAsync.hasError)
          ErrorBanner(dataAsync.error!),

        // Still show data if available
        if (dataAsync.hasValue)
          DataView(dataAsync.value!)
        else if (dataAsync.isLoading)
          CircularProgressIndicator(),
      ],
    );
  }
}
```

<div dir="rtl">

---

## 🧪 أمثلة عملية

### مثال 1: Form Submission with Error Handling

</div>

```dart
// Notifier for form state management with error handling
class RegistrationFormNotifier extends Notifier<FormState> {
  @override
  FormState build() {
    return FormState.idle();
  }

  Future<void> submit({
    required String email,
    required String password,
  }) async {
    // Start loading
    state = FormState.loading();

    try {
      // Validate
      if (email.isEmpty || !email.contains('@')) {
        throw ValidationException('Invalid email');
      }

      if (password.length < 8) {
        throw ValidationException('Password too short');
      }

      // Submit
      final user = await api.register(email, password);

      // Success
      state = FormState.success(user);

    } on ValidationException catch (e) {
      state = FormState.validationError(e.message);

    } on NetworkException catch (e) {
      state = FormState.networkError();

    } on ServerException catch (e) {
      state = FormState.serverError(e.message);

    } catch (e) {
      state = FormState.unknownError();
    }
  }
}

final registrationFormProvider = NotifierProvider<RegistrationFormNotifier, FormState>(
  () => RegistrationFormNotifier(),
);

// State class
sealed class FormState {
  const FormState();

  factory FormState.idle() = IdleState;
  factory FormState.loading() = LoadingState;
  factory FormState.success(User user) = SuccessState;
  factory FormState.validationError(String message) = ValidationErrorState;
  factory FormState.networkError() = NetworkErrorState;
  factory FormState.serverError(String message) = ServerErrorState;
  factory FormState.unknownError() = UnknownErrorState;
}

class IdleState extends FormState {}
class LoadingState extends FormState {}
class SuccessState extends FormState {
  final User user;
  SuccessState(this.user);
}
class ValidationErrorState extends FormState {
  final String message;
  ValidationErrorState(this.message);
}
class NetworkErrorState extends FormState {}
class ServerErrorState extends FormState {
  final String message;
  ServerErrorState(this.message);
}
class UnknownErrorState extends FormState {}

// Widget
class RegistrationPage extends ConsumerWidget {
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final formState = ref.watch(registrationFormProvider);

    // Listen for success
    ref.listen<FormState>(
      registrationFormProvider,
      (previous, next) {
        if (next is SuccessState) {
          Navigator.pushReplacementNamed(context, '/home');
        }
      },
    );

    return Scaffold(
      body: Column(
        children: [
          TextField(
            controller: _emailController,
            decoration: InputDecoration(
              labelText: 'Email',
              errorText: formState is ValidationErrorState
                  ? formState.message
                  : null,
            ),
          ),
          TextField(
            controller: _passwordController,
            decoration: InputDecoration(labelText: 'Password'),
            obscureText: true,
          ),

          if (formState is NetworkErrorState)
            Text(
              'No internet connection',
              style: TextStyle(color: Colors.red),
            ),

          if (formState is ServerErrorState)
            Text(
              formState.message,
              style: TextStyle(color: Colors.red),
            ),

          ElevatedButton(
            onPressed: formState is LoadingState
                ? null
                : () {
                    ref.read(registrationFormProvider.notifier).submit(
                          email: _emailController.text,
                          password: _passwordController.text,
                        );
                  },
            child: formState is LoadingState
                ? CircularProgressIndicator()
                : Text('Register'),
          ),
        ],
      ),
    );
  }
}
```

<div dir="rtl">

### مثال 2: Image Loading with Retry

</div>

```dart
// FutureProvider.family for image loading with retry
final imageProvider = FutureProvider.family<Uint8List, String>((ref, url) async {
  var retries = 0;
  const maxRetries = 3;

  while (true) {
    try {
      final response = await http.get(Uri.parse(url));

      if (response.statusCode == 200) {
        return response.bodyBytes;
      }

      throw Exception('HTTP ${response.statusCode}');

    } catch (e) {
      retries++;

      if (retries >= maxRetries) {
        throw Exception('Failed to load image after $maxRetries attempts');
      }

      // Exponential backoff
      await Future.delayed(Duration(seconds: math.pow(2, retries).toInt()));
    }
  }
});

// Widget
class NetworkImage extends ConsumerWidget {
  final String url;

  NetworkImage(this.url);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final imageAsync = ref.watch(imageProvider(url));

    return imageAsync.when(
      data: (bytes) => Image.memory(bytes),
      loading: () => Center(child: CircularProgressIndicator()),
      error: (error, stack) => Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Icon(Icons.error, color: Colors.red),
          Text('Failed to load image'),
          TextButton(
            onPressed: () => ref.invalidate(imageProvider(url)),
            child: Text('Retry'),
          ),
        ],
      ),
    );
  }
}
```

<div dir="rtl">

---

## 💡 Best Practices

### 1. دايماً Handle All States

</div>

```dart
// ✅ GOOD: Handles all states
dataAsync.when(
  data: (data) => DataView(data),
  loading: () => LoadingView(),
  error: (e, s) => ErrorView(e),
);

// ❌ BAD: Assumes data is always there
final data = dataAsync.value!; // Can crash!
```

<div dir="rtl">

### 2. استخدم Specific Exception Types

</div>

```dart
// ✅ GOOD: Specific exceptions
class NetworkException implements Exception {
  final String message;
  NetworkException(this.message);
}

class ValidationException implements Exception {
  final String message;
  ValidationException(this.message);
}

// Can handle differently
try {
  await api.call();
} on NetworkException catch (e) {
  // Show offline message
} on ValidationException catch (e) {
  // Show validation error
}
```

<div dir="rtl">

### 3. Provide Retry Options

</div>

```dart
// ✅ GOOD: Give user control
error: (error, stack) => ErrorView(
  error: error,
  onRetry: () => ref.invalidate(dataProvider),
);
```

<div dir="rtl">

### 4. Log Errors Properly

</div>

```dart
try {
  return await api.getData();
} catch (e, stackTrace) {
  // Log for debugging
  print('Error fetching data: $e');
  print('Stack trace: $stackTrace');

  // Could send to crash reporting
  // crashReporting.recordError(e, stackTrace);

  rethrow;
}
```

<div dir="rtl">

---

## 📝 ملخص

**Error Handling:**
- ✅ Use AsyncValue for async operations
- ✅ Handle all three states (data, loading, error)
- ✅ Provide user-friendly error messages
- ✅ Implement retry logic
- ✅ Use specific exception types

**AsyncValue Methods:**
- `when()` → Handle all states
- `maybeWhen()` → Handle specific states
- `map()` → Pattern matching
- Direct properties → `hasError`, `hasValue`, `isLoading`

**Best Practices:**
- Always handle errors
- Provide retry options
- Show previous data when possible
- Log errors for debugging
- Use optimistic updates with rollback

---

## 🔗 الخطوة الجاية

دلوقتي خلصنا Section 04 - Core Concepts! 🎉

الجاي:
- **Section 05: Provider Types** بالتفصيل
- كل نوع من الـ providers
- متى تستخدم كل واحد

---

## 📚 المصادر

- [Error Handling](https://riverpod.dev/docs/concepts/reading#error-handling)
- [AsyncValue](https://riverpod.dev/docs/concepts/async_value)

</div>
