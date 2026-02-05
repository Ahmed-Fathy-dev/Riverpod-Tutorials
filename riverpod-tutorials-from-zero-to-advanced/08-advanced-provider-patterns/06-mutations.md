<div dir="rtl">

# Mutations - Side Effects الحديثة 🔥⚡

**المستوى**: 🔴 متقدم

## ⚠️ تحذير: Experimental Feature

> **ملاحظة مهمة:** Mutations feature لا تزال **experimental** في Riverpod 3.0!
>
> - ✅ API قد يتغير في الإصدارات القادمة
> - ✅ استخدمها بحذر في production code
> - ✅ Import من `package:riverpod/experimental.dart`
> - ✅ متابعة التحديثات من التوثيق الرسمي

**لكن**: الفكرة قوية جداً وتستحق التعلم! 🚀

---

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تفهم مشكلة Boolean loading flags
- تستخدم Mutations للـ side effects
- تتعامل مع Loading/Success/Error states بذكاء
- تبني UI تفاعلي مع Mutations
- تعرف متى تستخدم Mutations vs AsyncNotifier

---

## 💡 المشكلة: Boolean Loading Flags Hell

### ❌ الطريقة القديمة (مشاكل كتيرة):

</div>

```dart
// ❌ OLD WAY - Boolean flags everywhere!
class TodoList extends Notifier<List<Todo>> {
  bool isLoading = false;        // ← Flag 1
  bool isDeleting = false;       // ← Flag 2
  bool isUpdating = false;       // ← Flag 3
  String? error;                 // ← Error state

  @override
  List<Todo> build() => [];

  Future<void> addTodo(String title) async {
    isLoading = true;
    state = [...state];  // Trigger rebuild to show loading

    try {
      final newTodo = await api.addTodo(title);
      state = [...state, newTodo];
      error = null;
    } catch (e) {
      error = e.toString();
    } finally {
      isLoading = false;
      state = [...state];  // Trigger rebuild again
    }
  }

  Future<void> deleteTodo(String id) async {
    isDeleting = true;
    state = [...state];

    try {
      await api.deleteTodo(id);
      state = state.where((t) => t.id != id).toList();
      error = null;
    } catch (e) {
      error = e.toString();
    } finally {
      isDeleting = false;
      state = [...state];
    }
  }
}

// في الـ UI - NIGHTMARE! 😱
class TodoListWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final todos = ref.watch(todoListProvider);
    final controller = ref.read(todoListProvider.notifier);

    // How do we know which operation is loading? 🤔
    // How do we show the right error? 🤔
    // How do we disable the right button? 🤔

    if (controller.isLoading) {
      return CircularProgressIndicator();  // But which operation??
    }

    if (controller.error != null) {
      return Text('Error: ${controller.error}');  // From which operation??
    }

    return ListView(
      children: todos.map((todo) {
        return ListTile(
          title: Text(todo.title),
          trailing: IconButton(
            onPressed: controller.isDeleting
              ? null  // But which todo is being deleted??
              : () => controller.deleteTodo(todo.id),
            icon: Icon(Icons.delete),
          ),
        );
      }).toList(),
    );
  }
}
```

<div dir="rtl">

### المشاكل:

1. **Boolean flags كثيرة** - كل operation محتاج flag خاص
2. **مفيش type safety** - Error قد يكون من أي operation
3. **Trigger rebuilds يدوياً** - لازم تعمل `state = [...state]`
4. **صعب تتبع الـ state** - أي operation شغالة دلوقتي؟
5. **UI logic معقد** - كيف تعرف تعرض الـ loading/error الصح؟

---

## ✅ الحل: Mutations!

**Mutations** هي طريقة جديدة في Riverpod 3.0 للتعامل مع **side effects** (operations زي button clicks, form submissions, API calls).

### المبدأ الأساسي:

> **Mutation = Operation لها states واضحة**
>
> - 🔵 **Idle**: مفيش operation شغالة
> - 🟡 **Pending**: Operation شغالة (loading)
> - 🟢 **Success**: Operation نجحت
> - 🔴 **Error**: Operation فشلت

---

## 📦 التجهيز: Import من Experimental

</div>

```dart
// ✅ Import mutations من experimental
import 'package:riverpod/experimental.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'todo_list.g.dart';
```

<div dir="rtl">

---

## 🎨 Mutation States

كل mutation عندها 4 states ممكنة:

### 1. MutationIdle (لا يوجد operation)

</div>

```dart
MutationIdle()  // No operation running
```

<div dir="rtl">

### 2. MutationPending (Loading)

</div>

```dart
MutationPending()  // Operation in progress
```

<div dir="rtl">

### 3. MutationSuccess<T> (نجحت)

</div>

```dart
MutationSuccess(data: result)  // Operation succeeded with result
```

<div dir="rtl">

### 4. MutationError<E> (فشلت)

</div>

```dart
MutationError(error: exception)  // Operation failed with error
```

<div dir="rtl">

---

## 🔨 الاستخدام الأساسي

### مثال 1: Add Todo (أبسط مثال)

</div>

```dart
// ✅ مع Mutations - نظيف وواضح!
@riverpod
class TodoList extends _$TodoList {
  @override
  List<Todo> build() => [];

  // ✅ استخدم @mutation annotation
  @mutation
  Future<void> addTodo(String title) async {
    // ✅ No boolean flags needed!
    // ✅ State management automatic!

    final newTodo = await api.addTodo(title);

    state = [...state, newTodo];

    // Mutation will automatically track:
    // - Pending state (while running)
    // - Success state (when done)
    // - Error state (if exception thrown)
  }
}

// في الـ UI - SIMPLE! 🎉
class AddTodoWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final controller = ref.watch(todoListProvider.notifier);

    // ✅ Watch the mutation state!
    final addMutation = ref.watch(controller.addTodoMutation);

    return Column(
      children: [
        TextField(
          onSubmitted: (title) {
            controller.addTodo(title);
          },
          // ✅ Disable when pending
          enabled: !addMutation.isPending,
        ),

        // ✅ Show loading indicator
        if (addMutation.isPending)
          CircularProgressIndicator(),

        // ✅ Show success message
        if (addMutation case MutationSuccess())
          Text('Todo added!', style: TextStyle(color: Colors.green)),

        // ✅ Show error
        if (addMutation case MutationError(:final error))
          Text('Error: $error', style: TextStyle(color: Colors.red)),
      ],
    );
  }
}
```

<div dir="rtl">

**الفرق:**
- ✅ No boolean flags
- ✅ Type-safe states
- ✅ Automatic state tracking
- ✅ Clean UI code

---

## 💪 مثال متقدم: Login Form

</div>

```dart
// ✅ مثال: Login form مع Mutations
@riverpod
class Auth extends _$Auth {
  @override
  User? build() => null;  // Initial state: not logged in

  @mutation
  Future<User> login(String email, String password) async {
    // Validation
    if (email.isEmpty || password.isEmpty) {
      throw Exception('Email and password required');
    }

    // API call (throws on failure)
    final user = await api.login(email, password);

    // Update state
    state = user;

    return user;
  }

  @mutation
  Future<void> logout() async {
    await api.logout();
    state = null;
  }
}

// في الـ UI
class LoginScreen extends ConsumerStatefulWidget {
  @override
  ConsumerState<LoginScreen> createState() => _LoginScreenState();
}

class _LoginScreenState extends ConsumerState<LoginScreen> {
  final emailController = TextEditingController();
  final passwordController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    final auth = ref.watch(authProvider.notifier);
    final loginMutation = ref.watch(auth.loginMutation);

    return Scaffold(
      appBar: AppBar(title: Text('Login')),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            // Email field
            TextField(
              controller: emailController,
              decoration: InputDecoration(labelText: 'Email'),
              enabled: !loginMutation.isPending,
            ),

            SizedBox(height: 16),

            // Password field
            TextField(
              controller: passwordController,
              decoration: InputDecoration(labelText: 'Password'),
              obscureText: true,
              enabled: !loginMutation.isPending,
            ),

            SizedBox(height: 24),

            // Login button
            ElevatedButton(
              onPressed: loginMutation.isPending
                  ? null
                  : () {
                      auth.login(
                        emailController.text,
                        passwordController.text,
                      );
                    },
              child: loginMutation.isPending
                  ? CircularProgressIndicator(color: Colors.white)
                  : Text('Login'),
            ),

            SizedBox(height: 16),

            // Success/Error display
            switch (loginMutation) {
              MutationSuccess(:final data) => Text(
                'Welcome, ${data.name}!',
                style: TextStyle(color: Colors.green),
              ),
              MutationError(:final error) => Text(
                'Login failed: $error',
                style: TextStyle(color: Colors.red),
              ),
              _ => SizedBox.shrink(),
            },
          ],
        ),
      ),
    );
  }

  @override
  void dispose() {
    emailController.dispose();
    passwordController.dispose();
    super.dispose();
  }
}
```

<div dir="rtl">

---

## 🎯 Advanced Patterns

### Pattern 1: Delete with Optimistic Updates

</div>

```dart
// ✅ مثال: Delete مع optimistic update
@riverpod
class TodoList extends _$TodoList {
  @override
  List<Todo> build() => [];

  @mutation
  Future<void> deleteTodo(String todoId) async {
    // 1. Optimistic update - Remove immediately
    final originalState = state;
    state = state.where((todo) => todo.id != todoId).toList();

    try {
      // 2. API call
      await api.deleteTodo(todoId);

      // Success! No need to update state (already updated)

    } catch (error) {
      // 3. Rollback on error
      state = originalState;

      // Re-throw to mutation can catch it
      rethrow;
    }
  }
}

// في الـ UI
class TodoItem extends ConsumerWidget {
  final Todo todo;

  TodoItem(this.todo);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final controller = ref.watch(todoListProvider.notifier);
    final deleteMutation = ref.watch(controller.deleteTodoMutation);

    return ListTile(
      title: Text(todo.title),
      trailing: IconButton(
        onPressed: () => controller.deleteTodo(todo.id),
        icon: deleteMutation.isPending
            ? SizedBox(
                width: 20,
                height: 20,
                child: CircularProgressIndicator(strokeWidth: 2),
              )
            : Icon(Icons.delete),
      ),
    );
  }
}
```

<div dir="rtl">

### Pattern 2: Chaining Mutations

</div>

```dart
// ✅ مثال: Chaining operations
@riverpod
class Registration extends _$Registration {
  @override
  User? build() => null;

  @mutation
  Future<User> register(String email, String password) async {
    // Step 1: Create account
    final user = await api.createAccount(email, password);

    // Step 2: Send verification email
    await api.sendVerificationEmail(user.email);

    // Step 3: Update state
    state = user;

    return user;
  }
}

// في الـ UI
class RegistrationScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final registration = ref.watch(registrationProvider.notifier);
    final registerMutation = ref.watch(registration.registerMutation);

    return Column(
      children: [
        // Form fields...

        ElevatedButton(
          onPressed: registerMutation.isPending
              ? null
              : () => registration.register(email, password),
          child: Text('Register'),
        ),

        // ✅ Show progress
        if (registerMutation.isPending)
          Column(
            children: [
              CircularProgressIndicator(),
              Text('Creating account...'),
            ],
          ),

        // ✅ Show success with navigation
        if (registerMutation case MutationSuccess()) ...[
          Text('Account created! Check your email.'),
          ElevatedButton(
            onPressed: () => Navigator.pushReplacementNamed(context, '/login'),
            child: Text('Go to Login'),
          ),
        ],
      ],
    );
  }
}
```

<div dir="rtl">

### Pattern 3: Multiple Mutations في نفس الـ Notifier

</div>

```dart
// ✅ مثال: Multiple mutations
@riverpod
class Profile extends _$Profile {
  @override
  UserProfile? build() => null;

  @mutation
  Future<void> updateName(String newName) async {
    final updated = await api.updateName(newName);
    state = updated;
  }

  @mutation
  Future<void> updateAvatar(File imageFile) async {
    final imageUrl = await api.uploadImage(imageFile);
    final updated = await api.updateAvatar(imageUrl);
    state = updated;
  }

  @mutation
  Future<void> deleteAccount() async {
    await api.deleteAccount();
    state = null;
  }
}

// في الـ UI - كل mutation مستقل!
class ProfileScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final profile = ref.watch(profileProvider.notifier);

    // ✅ Watch each mutation separately
    final updateNameMutation = ref.watch(profile.updateNameMutation);
    final updateAvatarMutation = ref.watch(profile.updateAvatarMutation);
    final deleteAccountMutation = ref.watch(profile.deleteAccountMutation);

    return Column(
      children: [
        // Name update
        TextField(
          onSubmitted: profile.updateName,
          enabled: !updateNameMutation.isPending,
        ),
        if (updateNameMutation.isPending)
          Text('Updating name...'),

        SizedBox(height: 20),

        // Avatar update
        ElevatedButton(
          onPressed: updateAvatarMutation.isPending
              ? null
              : () async {
                  final file = await pickImage();
                  if (file != null) profile.updateAvatar(file);
                },
          child: updateAvatarMutation.isPending
              ? CircularProgressIndicator()
              : Text('Change Avatar'),
        ),

        SizedBox(height: 20),

        // Delete account
        ElevatedButton(
          onPressed: deleteAccountMutation.isPending
              ? null
              : () => showDeleteConfirmation(context, profile.deleteAccount),
          style: ElevatedButton.styleFrom(backgroundColor: Colors.red),
          child: Text('Delete Account'),
        ),
        if (deleteAccountMutation.isPending)
          Text('Deleting account...'),
      ],
    );
  }
}
```

<div dir="rtl">

---

## 🎭 Pattern Matching مع Mutations

### استخدام when()

</div>

```dart
// ✅ استخدم when() للتعامل مع كل الحالات
class SubmitButton extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final controller = ref.watch(formProvider.notifier);
    final submitMutation = ref.watch(controller.submitMutation);

    return submitMutation.when(
      idle: () => ElevatedButton(
        onPressed: controller.submit,
        child: Text('Submit'),
      ),
      pending: () => ElevatedButton(
        onPressed: null,
        child: Row(
          mainAxisSize: MainAxisSize.min,
          children: [
            SizedBox(
              width: 16,
              height: 16,
              child: CircularProgressIndicator(strokeWidth: 2),
            ),
            SizedBox(width: 8),
            Text('Submitting...'),
          ],
        ),
      ),
      success: (data) => Column(
        children: [
          Icon(Icons.check_circle, color: Colors.green, size: 48),
          Text('Success!'),
          ElevatedButton(
            onPressed: () => Navigator.pop(context),
            child: Text('Done'),
          ),
        ],
      ),
      error: (error) => Column(
        children: [
          Icon(Icons.error, color: Colors.red, size: 48),
          Text('Error: $error'),
          ElevatedButton(
            onPressed: controller.submit,
            child: Text('Retry'),
          ),
        ],
      ),
    );
  }
}
```

<div dir="rtl">

### استخدام Pattern Matching (Dart 3+)

</div>

```dart
// ✅ استخدم pattern matching
class MutationDisplay extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final controller = ref.watch(apiProvider.notifier);
    final mutation = ref.watch(controller.performActionMutation);

    return switch (mutation) {
      MutationIdle() => Text('Ready'),

      MutationPending() => Row(
        children: [
          CircularProgressIndicator(),
          SizedBox(width: 8),
          Text('Loading...'),
        ],
      ),

      MutationSuccess(:final data) => Column(
        children: [
          Icon(Icons.check, color: Colors.green),
          Text('Success: $data'),
        ],
      ),

      MutationError(:final error) => Column(
        children: [
          Icon(Icons.error, color: Colors.red),
          Text('Error: $error'),
          TextButton(
            onPressed: controller.performAction,
            child: Text('Retry'),
          ),
        ],
      ),
    };
  }
}
```

<div dir="rtl">

---

## 🧪 Testing Mutations

</div>

```dart
// ✅ Testing mutations
void main() {
  group('TodoList mutations', () {
    test('addTodo mutation succeeds', () async {
      final container = ProviderContainer(
        overrides: [
          // Mock API
          apiProvider.overrideWith((ref) => MockApi()),
        ],
      );

      final controller = container.read(todoListProvider.notifier);

      // Initial state
      expect(controller.addTodoMutation, isA<MutationIdle>());

      // Start mutation
      final future = controller.addTodo('Buy milk');

      // Should be pending
      await Future.delayed(Duration.zero);
      expect(controller.addTodoMutation, isA<MutationPending>());

      // Wait for completion
      await future;

      // Should be success
      expect(controller.addTodoMutation, isA<MutationSuccess>());

      // State should be updated
      expect(
        container.read(todoListProvider),
        contains(isA<Todo>()),
      );

      container.dispose();
    });

    test('addTodo mutation handles errors', () async {
      final container = ProviderContainer(
        overrides: [
          apiProvider.overrideWith((ref) => MockApiWithErrors()),
        ],
      );

      final controller = container.read(todoListProvider.notifier);

      // Start mutation (will fail)
      try {
        await controller.addTodo('Buy milk');
        fail('Should have thrown');
      } catch (e) {
        // Expected
      }

      // Should be error state
      expect(controller.addTodoMutation, isA<MutationError>());

      container.dispose();
    });
  });
}
```

<div dir="rtl">

---

## 🤔 متى تستخدم Mutations؟

### ✅ استخدم Mutations عندما:

1. **Button clicks** - أي action من الـ user
2. **Form submissions** - Login, register, update profile
3. **Delete operations** - حذف items
4. **Side effects** - Actions لها side effects واضحة
5. **Loading states** - محتاج تتبع loading/success/error

### ❌ لا تستخدم Mutations عندما:

1. **Data fetching** - استخدم `AsyncNotifier` أو `FutureProvider`
2. **Computed values** - استخدم `Provider` عادي
3. **Simple state** - استخدم `Notifier`
4. **Streams** - استخدم `StreamProvider`

---

## 📊 المقارنة: Mutations vs AsyncNotifier

| Feature | Mutations | AsyncNotifier |
|---------|-----------|---------------|
| **Use Case** | Side effects (clicks, submissions) | Data fetching |
| **State Type** | MutationState<T> | AsyncValue<T> |
| **Trigger** | User action (manual) | Provider watch (automatic) |
| **Multiple operations** | ✅ Each @mutation tracked | ❌ Single state |
| **Optimistic updates** | ✅ Easy | ⚠️ Manual |
| **Error recovery** | ✅ Built-in retry | ⚠️ Manual |
| **Loading per action** | ✅ Separate loading states | ❌ Single loading state |

---

## 🎓 Best Practices

### 1. واحد Mutation لكل Action

</div>

```dart
// ✅ GOOD - Separate mutations
@riverpod
class TodoList extends _$TodoList {
  @override
  List<Todo> build() => [];

  @mutation
  Future<void> addTodo(String title) async { ... }

  @mutation
  Future<void> deleteTodo(String id) async { ... }

  @mutation
  Future<void> updateTodo(String id, String newTitle) async { ... }
}

// ❌ BAD - Single mutation for everything
@riverpod
class TodoList extends _$TodoList {
  @override
  List<Todo> build() => [];

  @mutation
  Future<void> performAction(String action, Map<String, dynamic> params) async {
    // ❌ TOO GENERIC!
  }
}
```

<div dir="rtl">

### 2. استخدم Optimistic Updates للـ Better UX

</div>

```dart
// ✅ GOOD - Optimistic update
@mutation
Future<void> toggleComplete(String todoId) async {
  // Update immediately
  state = state.map((todo) {
    if (todo.id == todoId) {
      return todo.copyWith(completed: !todo.completed);
    }
    return todo;
  }).toList();

  try {
    await api.toggleTodo(todoId);
  } catch (error) {
    // Rollback on error
    state = state.map((todo) {
      if (todo.id == todoId) {
        return todo.copyWith(completed: !todo.completed);
      }
      return todo;
    }).toList();
    rethrow;
  }
}
```

<div dir="rtl">

### 3. اعرض الـ Success Message لفترة قصيرة

</div>

```dart
// ✅ GOOD - Auto-hide success message
class FormWidget extends ConsumerStatefulWidget {
  @override
  ConsumerState<FormWidget> createState() => _FormWidgetState();
}

class _FormWidgetState extends ConsumerState<FormWidget> {
  @override
  Widget build(BuildContext context) {
    final controller = ref.watch(formProvider.notifier);
    final submitMutation = ref.watch(controller.submitMutation);

    // Listen for success
    ref.listen(controller.submitMutation, (previous, next) {
      if (next is MutationSuccess) {
        // Show success message
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('Saved successfully!')),
        );

        // Auto-hide after 2 seconds
        Future.delayed(Duration(seconds: 2), () {
          if (mounted) {
            // Reset mutation state (optional)
          }
        });
      }
    });

    return Column(
      children: [
        // Form fields...
      ],
    );
  }
}
```

<div dir="rtl">

### 4. Handle Errors بوضوح

</div>

```dart
// ✅ GOOD - Clear error handling
@mutation
Future<void> submitForm(FormData data) async {
  try {
    // Validation
    if (!data.isValid) {
      throw ValidationException('Invalid data');
    }

    // API call
    await api.submitForm(data);

  } on ValidationException catch (e) {
    // User error - show friendly message
    throw Exception('Please check your input: ${e.message}');

  } on NetworkException catch (e) {
    // Network error - show retry option
    throw Exception('Network error. Please try again.');

  } catch (e) {
    // Unknown error
    throw Exception('An unexpected error occurred.');
  }
}
```

<div dir="rtl">

---

## ⚠️ Common Pitfalls (أخطاء شائعة)

### ❌ خطأ 1: نسيت @mutation Annotation

</div>

```dart
// ❌ BAD - Forgot @mutation
@riverpod
class TodoList extends _$TodoList {
  @override
  List<Todo> build() => [];

  // ❌ This won't create a mutation!
  Future<void> addTodo(String title) async {
    await api.addTodo(title);
  }
}

// ✅ GOOD - Use @mutation
@riverpod
class TodoList extends _$TodoList {
  @override
  List<Todo> build() => [];

  @mutation  // ← Don't forget!
  Future<void> addTodo(String title) async {
    await api.addTodo(title);
  }
}
```

<div dir="rtl">

### ❌ خطأ 2: استخدام Mutations للـ Data Fetching

</div>

```dart
// ❌ BAD - Mutations for data fetching
@riverpod
class UserData extends _$UserData {
  @override
  User? build() => null;

  @mutation  // ❌ Wrong! Not a side effect
  Future<User> fetchUser() async {
    return await api.getUser();
  }
}

// ✅ GOOD - AsyncNotifier for data fetching
@riverpod
class UserData extends _$UserData {
  @override
  Future<User> build() async {
    // ✅ Fetches automatically
    return await api.getUser();
  }
}
```

<div dir="rtl">

---

## 🎯 الخلاصة

### Mutations في سطر واحد:
> **Mutations = طريقة نظيفة وآمنة للتعامل مع side effects في Riverpod 3.0**

### الفوائد:
- ✅ No boolean loading flags
- ✅ Type-safe state tracking
- ✅ Automatic loading/success/error states
- ✅ Clean UI code
- ✅ Easy testing
- ✅ Better UX with optimistic updates

### متى تستخدمها:
- ✅ Button clicks
- ✅ Form submissions
- ✅ Delete/Update operations
- ✅ Any user-triggered action

### متى لا تستخدمها:
- ❌ Data fetching (use AsyncNotifier)
- ❌ Computed values (use Provider)
- ❌ Simple state (use Notifier)

---

## 🔗 مصادر إضافية

### Official Documentation:
- [Mutations (Experimental) | Riverpod](https://riverpod.dev/docs/concepts2/mutations)
- [What's New in Riverpod 3.0 | Riverpod](https://riverpod.dev/docs/whats_new)

### Community Resources:
- [Mutations Example | GitHub](https://github.com/rrousselGit/riverpod/tree/master/examples/mutations)

---

## ✅ تأكد إنك فهمت

- [ ] فاهم المشكلة مع boolean loading flags؟
- [ ] عارف ال 4 mutation states؟
- [ ] تقدر تستخدم @mutation annotation؟
- [ ] تقدر تعرض mutation state في الـ UI؟
- [ ] فاهم الفرق بين Mutations و AsyncNotifier؟
- [ ] تقدر تعمل optimistic updates؟
- [ ] تقدر تكتب tests للـ mutations؟

---

**🚀 Mutations = مستقبل Side Effects في Riverpod!**

استخدمها بذكاء وهتوفر على نفسك كود كثير ومشاكل أكثر! 💪

</div>
