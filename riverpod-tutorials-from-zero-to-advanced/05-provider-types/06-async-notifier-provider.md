<div dir="rtl">

# AsyncNotifierProvider - State المعقد (Asynchronous)

**المستوى**: 🟡 متوسط

## 📌 المفاهيم الأساسية

في الملف ده هنتكلم عن:
- إيه هو AsyncNotifierProvider ومتى نستخدمه
- الفرق بينه وبين FutureProvider و NotifierProvider
- CRUD operations مع API
- Optimistic updates
- Error handling
- Best practices

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تستخدم AsyncNotifierProvider للـ async state
- تعمل CRUD operations
- تطبق optimistic updates
- تتعامل مع errors بكفاءة

---

## 🔍 إيه هو AsyncNotifierProvider؟

**AsyncNotifierProvider** هو provider للـ **complex asynchronous state** مع business logic.

</div>

```dart
// AsyncNotifier class
class TodosNotifier extends AsyncNotifier<List<Todo>> {
  @override
  Future<List<Todo>> build() async {
    // Initial async load
    return await api.getTodos();
  }

  Future<void> addTodo(String title) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      await api.addTodo(title);
      return await api.getTodos();
    });
  }

  Future<void> deleteTodo(String id) async {
    // Optimistic update
    final previousState = state;

    state = AsyncData(
      state.value!.where((todo) => todo.id != id).toList(),
    );

    try {
      await api.deleteTodo(id);
    } catch (e) {
      state = previousState; // Revert on error
    }
  }
}

// Provider
final todosProvider = AsyncNotifierProvider<TodosNotifier, List<Todo>>(
  () => TodosNotifier(),
);
```

<div dir="rtl">

**متى تستخدمه:**
- ✅ Complex async state
- ✅ CRUD operations (Create, Read, Update, Delete)
- ✅ محتاج refresh/reload
- ✅ Optimistic updates
- ✅ Error handling

**متى ما تستخدموش:**
- ❌ One-time fetch → FutureProvider
- ❌ Synchronous state → NotifierProvider
- ❌ Continuous stream → StreamNotifierProvider

---

## 🎨 الاستخدامات الأساسية

### 1. Todo List مع API

</div>

```dart
// Todo model
class Todo {
  final String id;
  final String title;
  final bool isCompleted;

  Todo({
    required this.id,
    required this.title,
    required this.isCompleted,
  });

  Todo copyWith({
    String? id,
    String? title,
    bool? isCompleted,
  }) {
    return Todo(
      id: id ?? this.id,
      title: title ?? this.title,
      isCompleted: isCompleted ?? this.isCompleted,
    );
  }
}

// API
class TodosApi {
  Future<List<Todo>> getTodos() async {
    await Future.delayed(Duration(seconds: 1));
    return [
      Todo(id: '1', title: 'Learn Riverpod', isCompleted: false),
      Todo(id: '2', title: 'Build app', isCompleted: true),
    ];
  }

  Future<void> addTodo(String title) async {
    await Future.delayed(Duration(milliseconds: 500));
  }

  Future<void> toggleTodo(String id) async {
    await Future.delayed(Duration(milliseconds: 300));
  }

  Future<void> deleteTodo(String id) async {
    await Future.delayed(Duration(milliseconds: 300));
  }

  Future<void> updateTodo(String id, String title) async {
    await Future.delayed(Duration(milliseconds: 500));
  }
}

// Providers
final todosApiProvider = Provider<TodosApi>((ref) => TodosApi());

// Todos Notifier
class TodosNotifier extends AsyncNotifier<List<Todo>> {
  @override
  Future<List<Todo>> build() async {
    // Initial load
    final api = ref.watch(todosApiProvider);
    return await api.getTodos();
  }

  Future<void> addTodo(String title) async {
    state = const AsyncLoading();

    state = await AsyncValue.guard(() async {
      final api = ref.read(todosApiProvider);
      await api.addTodo(title);
      return await api.getTodos();
    });
  }

  Future<void> toggleTodo(String id) async {
    // Optimistic update
    final previousState = state;

    state = AsyncData(
      state.value!.map((todo) {
        if (todo.id == id) {
          return todo.copyWith(isCompleted: !todo.isCompleted);
        }
        return todo;
      }).toList(),
    );

    try {
      final api = ref.read(todosApiProvider);
      await api.toggleTodo(id);
    } catch (e, stack) {
      // Revert on error
      state = previousState;
      // Show error
      state = AsyncError(e, stack);
    }
  }

  Future<void> deleteTodo(String id) async {
    // Optimistic update
    final previousState = state;

    state = AsyncData(
      state.value!.where((todo) => todo.id != id).toList(),
    );

    try {
      final api = ref.read(todosApiProvider);
      await api.deleteTodo(id);
    } catch (e) {
      // Revert on error
      state = previousState;
    }
  }

  Future<void> updateTodo(String id, String title) async {
    // Optimistic update
    final previousState = state;

    state = AsyncData(
      state.value!.map((todo) {
        if (todo.id == id) {
          return todo.copyWith(title: title);
        }
        return todo;
      }).toList(),
    );

    try {
      final api = ref.read(todosApiProvider);
      await api.updateTodo(id, title);
    } catch (e) {
      state = previousState;
    }
  }

  Future<void> refresh() async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      final api = ref.read(todosApiProvider);
      return await api.getTodos();
    });
  }
}

final todosProvider = AsyncNotifierProvider<TodosNotifier, List<Todo>>(
  () => TodosNotifier(),
);

// Todos Screen
class TodosScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final todosAsync = ref.watch(todosProvider);

    return Scaffold(
      appBar: AppBar(
        title: Text('Todos'),
        actions: [
          IconButton(
            icon: Icon(Icons.refresh),
            onPressed: () {
              ref.read(todosProvider.notifier).refresh();
            },
          ),
        ],
      ),
      body: todosAsync.when(
        data: (todos) {
          if (todos.isEmpty) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Icon(Icons.inbox, size: 64, color: Colors.grey),
                  SizedBox(height: 16),
                  Text('No todos yet'),
                ],
              ),
            );
          }

          return ListView.builder(
            itemCount: todos.length,
            itemBuilder: (context, index) {
              final todo = todos[index];
              return ListTile(
                title: Text(
                  todo.title,
                  style: TextStyle(
                    decoration: todo.isCompleted
                        ? TextDecoration.lineThrough
                        : null,
                  ),
                ),
                leading: Checkbox(
                  value: todo.isCompleted,
                  onChanged: (_) {
                    ref.read(todosProvider.notifier).toggleTodo(todo.id);
                  },
                ),
                trailing: IconButton(
                  icon: Icon(Icons.delete),
                  onPressed: () {
                    ref.read(todosProvider.notifier).deleteTodo(todo.id);
                  },
                ),
                onTap: () {
                  // Edit todo
                  _showEditDialog(context, ref, todo);
                },
              );
            },
          );
        },
        loading: () => Center(child: CircularProgressIndicator()),
        error: (error, stack) => Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Icon(Icons.error, color: Colors.red, size: 48),
              SizedBox(height: 16),
              Text('Error: $error'),
              SizedBox(height: 16),
              ElevatedButton(
                onPressed: () {
                  ref.read(todosProvider.notifier).refresh();
                },
                child: Text('Retry'),
              ),
            ],
          ),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          _showAddDialog(context, ref);
        },
        child: Icon(Icons.add),
      ),
    );
  }

  void _showAddDialog(BuildContext context, WidgetRef ref) {
    final controller = TextEditingController();

    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: Text('Add Todo'),
        content: TextField(
          controller: controller,
          decoration: InputDecoration(labelText: 'Title'),
          autofocus: true,
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: Text('Cancel'),
          ),
          TextButton(
            onPressed: () {
              if (controller.text.isNotEmpty) {
                ref.read(todosProvider.notifier).addTodo(controller.text);
                Navigator.pop(context);
              }
            },
            child: Text('Add'),
          ),
        ],
      ),
    );
  }

  void _showEditDialog(BuildContext context, WidgetRef ref, Todo todo) {
    final controller = TextEditingController(text: todo.title);

    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: Text('Edit Todo'),
        content: TextField(
          controller: controller,
          decoration: InputDecoration(labelText: 'Title'),
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: Text('Cancel'),
          ),
          TextButton(
            onPressed: () {
              if (controller.text.isNotEmpty) {
                ref.read(todosProvider.notifier).updateTodo(
                      todo.id,
                      controller.text,
                    );
                Navigator.pop(context);
              }
            },
            child: Text('Save'),
          ),
        ],
      ),
    );
  }
}
```

<div dir="rtl">

### 2. User Profile Management

</div>

```dart
// User model
class UserProfile {
  final String id;
  final String name;
  final String email;
  final String? bio;
  final String? avatarUrl;

  UserProfile({
    required this.id,
    required this.name,
    required this.email,
    this.bio,
    this.avatarUrl,
  });

  UserProfile copyWith({
    String? id,
    String? name,
    String? email,
    String? bio,
    String? avatarUrl,
  }) {
    return UserProfile(
      id: id ?? this.id,
      name: name ?? this.name,
      email: email ?? this.email,
      bio: bio ?? this.bio,
      avatarUrl: avatarUrl ?? this.avatarUrl,
    );
  }
}

// User Profile Notifier
class UserProfileNotifier extends AsyncNotifier<UserProfile> {
  @override
  Future<UserProfile> build() async {
    final api = ref.watch(apiProvider);
    return await api.getUserProfile();
  }

  Future<void> updateProfile({
    String? name,
    String? email,
    String? bio,
  }) async {
    // Optimistic update
    final previousState = state;

    if (state.hasValue) {
      state = AsyncData(
        state.value!.copyWith(
          name: name,
          email: email,
          bio: bio,
        ),
      );
    }

    try {
      final api = ref.read(apiProvider);
      await api.updateProfile(
        name: name,
        email: email,
        bio: bio,
      );

      // Reload to get server state
      final updatedProfile = await api.getUserProfile();
      state = AsyncData(updatedProfile);
    } catch (e, stack) {
      // Revert on error
      state = previousState;
      // Rethrow to show in UI
      rethrow;
    }
  }

  Future<void> uploadAvatar(File imageFile) async {
    state = const AsyncLoading();

    state = await AsyncValue.guard(() async {
      final api = ref.read(apiProvider);
      final avatarUrl = await api.uploadAvatar(imageFile);

      final updatedProfile = state.value!.copyWith(
        avatarUrl: avatarUrl,
      );

      return updatedProfile;
    });
  }

  Future<void> refresh() async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      final api = ref.read(apiProvider);
      return await api.getUserProfile();
    });
  }
}

final userProfileProvider = AsyncNotifierProvider<UserProfileNotifier, UserProfile>(
  () => UserProfileNotifier(),
);
```

<div dir="rtl">

### 3. Paginated List

</div>

```dart
// Paginated data
class PaginatedProducts {
  final List<Product> items;
  final int currentPage;
  final int totalPages;
  final bool hasMore;

  PaginatedProducts({
    required this.items,
    required this.currentPage,
    required this.totalPages,
  }) : hasMore = currentPage < totalPages;
}

// Products Notifier
class ProductsNotifier extends AsyncNotifier<PaginatedProducts> {
  @override
  Future<PaginatedProducts> build() async {
    return await _fetchPage(1);
  }

  Future<void> loadMore() async {
    if (!state.hasValue || !state.value!.hasMore) return;

    final currentData = state.value!;
    final nextPage = currentData.currentPage + 1;

    // Show loading for new items
    state = AsyncData(currentData);

    try {
      final newPage = await _fetchPage(nextPage);

      state = AsyncData(
        PaginatedProducts(
          items: [...currentData.items, ...newPage.items],
          currentPage: newPage.currentPage,
          totalPages: newPage.totalPages,
        ),
      );
    } catch (e, stack) {
      state = AsyncError(e, stack);
    }
  }

  Future<void> refresh() async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() => _fetchPage(1));
  }

  Future<PaginatedProducts> _fetchPage(int page) async {
    final api = ref.read(apiProvider);
    return await api.getProducts(page: page);
  }
}

final productsProvider = AsyncNotifierProvider<ProductsNotifier, PaginatedProducts>(
  () => ProductsNotifier(),
);

// Products List
class ProductsList extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final productsAsync = ref.watch(productsProvider);

    return productsAsync.when(
      data: (data) => RefreshIndicator(
        onRefresh: () => ref.read(productsProvider.notifier).refresh(),
        child: ListView.builder(
          itemCount: data.items.length + (data.hasMore ? 1 : 0),
          itemBuilder: (context, index) {
            if (index == data.items.length) {
              // Load more button
              return Padding(
                padding: EdgeInsets.all(16),
                child: ElevatedButton(
                  onPressed: () {
                    ref.read(productsProvider.notifier).loadMore();
                  },
                  child: Text('Load More'),
                ),
              );
            }

            final product = data.items[index];
            return ListTile(
              title: Text(product.name),
              subtitle: Text('\$${product.price}'),
            );
          },
        ),
      ),
      loading: () => Center(child: CircularProgressIndicator()),
      error: (error, stack) => Center(child: Text('Error: $error')),
    );
  }
}
```

<div dir="rtl">

---

## 💡 Patterns الشائعة

### Pattern 1: Optimistic Updates

</div>

```dart
Future<void> toggleTodo(String id) async {
  // 1. Save previous state
  final previousState = state;

  // 2. Update UI immediately
  state = AsyncData(
    state.value!.map((todo) {
      if (todo.id == id) {
        return todo.copyWith(isCompleted: !todo.isCompleted);
      }
      return todo;
    }).toList(),
  );

  // 3. Try API call
  try {
    await api.toggleTodo(id);
  } catch (e) {
    // 4. Revert on error
    state = previousState;
  }
}
```

<div dir="rtl">

### Pattern 2: AsyncValue.guard

</div>

```dart
Future<void> addTodo(String title) async {
  state = const AsyncLoading();

  // Automatically handles error
  state = await AsyncValue.guard(() async {
    await api.addTodo(title);
    return await api.getTodos();
  });
}
```

<div dir="rtl">

### Pattern 3: Refresh Pattern

</div>

```dart
Future<void> refresh() async {
  state = const AsyncLoading();

  state = await AsyncValue.guard(() async {
    return await api.getTodos();
  });
}
```

<div dir="rtl">

---

## ⚠️ أخطاء شائعة

### ❌ خطأ 1: Forgetting to handle errors

</div>

```dart
// ❌ WRONG - No error handling
Future<void> addTodo(String title) async {
  await api.addTodo(title);
  state = AsyncData(await api.getTodos());
  // What if error happens?
}

// ✅ CORRECT - Use AsyncValue.guard
Future<void> addTodo(String title) async {
  state = const AsyncLoading();
  state = await AsyncValue.guard(() async {
    await api.addTodo(title);
    return await api.getTodos();
  });
}
```

<div dir="rtl">

### ❌ خطأ 2: Not using optimistic updates

</div>

```dart
// ❌ WRONG - Slow UX
Future<void> toggleTodo(String id) async {
  state = const AsyncLoading(); // Shows loading spinner!
  await api.toggleTodo(id);
  state = AsyncData(await api.getTodos());
}

// ✅ CORRECT - Instant feedback
Future<void> toggleTodo(String id) async {
  final previousState = state;

  // Update immediately
  state = AsyncData(/* updated list */);

  try {
    await api.toggleTodo(id);
  } catch (e) {
    state = previousState; // Revert only on error
  }
}
```

<div dir="rtl">

---

## 💡 Best Practices

### 1. استخدم AsyncValue.guard للـ Error Handling

</div>

```dart
// ✅ Good - Automatic error handling
state = await AsyncValue.guard(() async {
  return await api.getData();
});
```

<div dir="rtl">

### 2. Optimistic Updates للـ Better UX

</div>

```dart
// ✅ Good - Instant feedback
final previousState = state;
state = AsyncData(updatedData);

try {
  await api.update();
} catch (e) {
  state = previousState;
}
```

<div dir="rtl">

### 3. Refresh Method دايماً

</div>

```dart
// ✅ Good - Always provide refresh
Future<void> refresh() async {
  state = const AsyncLoading();
  state = await AsyncValue.guard(() => api.getData());
}
```

<div dir="rtl">

---

## 📝 ملخص

**AsyncNotifierProvider** يستخدم لـ:
- ✅ Complex async state
- ✅ CRUD operations
- ✅ محتاج refresh
- ✅ Optimistic updates

**مش يستخدم لـ:**
- ❌ One-time fetch → FutureProvider
- ❌ Sync state → NotifierProvider
- ❌ Streams → StreamNotifierProvider

**Best Practices:**
- استخدم AsyncValue.guard
- Optimistic updates
- دايماً refresh method

---

## 🔗 الخطوة الجاية

في الملف الجاي:
- **Decision Guide** - إزاي تختار النوع المناسب
- Decision tree شامل
- أمثلة من الواقع

---

## 📚 المصادر

- [AsyncNotifierProvider - Riverpod Docs](https://riverpod.dev/docs/providers/notifier_provider)
- [AsyncValue - Error Handling](https://riverpod.dev/docs/concepts/async_value)

</div>
