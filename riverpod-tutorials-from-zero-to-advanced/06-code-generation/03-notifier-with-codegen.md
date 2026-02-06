<div dir="rtl">

# Notifier & AsyncNotifier مع Code Generation

**المستوى**: 🟡 متوسط

## 📌 المفاهيم الأساسية

في الملف ده هنتعلم:
- إزاي نكتب Notifier و AsyncNotifier مع `@riverpod`
- الفرق بينهم ومتى نستخدم كل واحد
- إزاي نعمل methods للـ state management
- التعامل مع الـ state updates
- أمثلة عملية كاملة

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تكتب Notifier classes بالـ code generation
- تختار بين Notifier و AsyncNotifier صح
- تعمل complex state management
- تكتب methods واضحة وآمنة

---

## 📖 Notifier vs AsyncNotifier

### Notifier (Synchronous State)
- الـ `build()` method بيرجع قيمة **مباشرة** (مش Future)
- للـ state اللي **مش محتاج async initialization**
- مثال: Counter, Form state, UI state

### AsyncNotifier (Asynchronous State)
- الـ `build()` method بيرجع **Future**
- للـ state اللي **محتاج async initialization** (API call, database, file)
- مثال: User data from API, Todos list, Settings

---

## 1️⃣ Notifier - Synchronous State

### المثال الأبسط: Counter

</div>

```dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'counter.g.dart';

@riverpod
class Counter extends _$Counter {
  // Initial state (synchronous)
  @override
  int build() {
    return 0;  // Counter starts at 0
  }

  // Methods to modify state
  void increment() {
    state++;  // Modify state directly
  }

  void decrement() {
    state--;
  }

  void reset() {
    state = 0;
  }

  void setTo(int value) {
    state = value;
  }
}

// Generated: counterProvider
// Type: AutoDisposeNotifierProvider<Counter, int>
```

<div dir="rtl">

### الاستخدام في الـ UI:

</div>

```dart
class CounterPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Watch the state
    final count = ref.watch(counterProvider);

    return Scaffold(
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Count: $count', style: TextStyle(fontSize: 48)),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                ElevatedButton(
                  onPressed: () {
                    // Call methods on the notifier
                    ref.read(counterProvider.notifier).decrement();
                  },
                  child: Icon(Icons.remove),
                ),
                ElevatedButton(
                  onPressed: () {
                    ref.read(counterProvider.notifier).reset();
                  },
                  child: Icon(Icons.refresh),
                ),
                ElevatedButton(
                  onPressed: () {
                    ref.read(counterProvider.notifier).increment();
                  },
                  child: Icon(Icons.add),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

<div dir="rtl">

### مثال أعقد: Shopping Cart

</div>

```dart
@riverpod
class ShoppingCart extends _$ShoppingCart {
  @override
  List<CartItem> build() {
    // Start with empty cart
    return [];
  }

  // Add item to cart
  void addItem(Product product) {
    // Check if item already exists
    final existingIndex = state.indexWhere((item) => item.productId == product.id);

    if (existingIndex != -1) {
      // Item exists, increase quantity
      state = [
        for (int i = 0; i < state.length; i++)
          if (i == existingIndex)
            state[i].copyWith(quantity: state[i].quantity + 1)
          else
            state[i],
      ];
    } else {
      // New item, add to cart
      state = [
        ...state,
        CartItem(
          productId: product.id,
          name: product.name,
          price: product.price,
          quantity: 1,
        ),
      ];
    }
  }

  // Remove item from cart
  void removeItem(String productId) {
    state = state.where((item) => item.productId != productId).toList();
  }

  // Update quantity
  void updateQuantity(String productId, int newQuantity) {
    if (newQuantity <= 0) {
      removeItem(productId);
      return;
    }

    state = [
      for (final item in state)
        if (item.productId == productId)
          item.copyWith(quantity: newQuantity)
        else
          item,
    ];
  }

  // Clear cart
  void clear() {
    state = [];
  }

  // Computed properties (getters)
  double get totalPrice {
    return state.fold(0.0, (sum, item) => sum + (item.price * item.quantity));
  }

  int get itemCount {
    return state.fold(0, (sum, item) => sum + item.quantity);
  }
}

// Models
class CartItem {
  final String productId;
  final String name;
  final double price;
  final int quantity;

  CartItem({
    required this.productId,
    required this.name,
    required this.price,
    required this.quantity,
  });

  CartItem copyWith({
    String? productId,
    String? name,
    double? price,
    int? quantity,
  }) {
    return CartItem(
      productId: productId ?? this.productId,
      name: name ?? this.name,
      price: price ?? this.price,
      quantity: quantity ?? this.quantity,
    );
  }
}
```

<div dir="rtl">

### الاستخدام:

</div>

```dart
class CartPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final cart = ref.watch(shoppingCartProvider);
    final notifier = ref.read(shoppingCartProvider.notifier);

    return Scaffold(
      appBar: AppBar(
        title: Text('Cart (${notifier.itemCount} items)'),
      ),
      body: cart.isEmpty
          ? Center(child: Text('Cart is empty'))
          : ListView.builder(
              itemCount: cart.length,
              itemBuilder: (context, index) {
                final item = cart[index];
                return ListTile(
                  title: Text(item.name),
                  subtitle: Text('\$${item.price} x ${item.quantity}'),
                  trailing: IconButton(
                    icon: Icon(Icons.delete),
                    onPressed: () => notifier.removeItem(item.productId),
                  ),
                );
              },
            ),
      bottomNavigationBar: Container(
        padding: EdgeInsets.all(16),
        child: ElevatedButton(
          onPressed: cart.isEmpty ? null : () {
            // Proceed to checkout
          },
          child: Text('Checkout - \$${notifier.totalPrice.toStringAsFixed(2)}'),
        ),
      ),
    );
  }
}
```

<div dir="rtl">

---

## 2️⃣ AsyncNotifier - Asynchronous State

للـ state اللي بيحتاج async initialization (API calls, database):

### مثال بسيط: User Profile

</div>

```dart
@riverpod
class UserProfile extends _$UserProfile {
  // Async initialization
  @override
  Future<User> build() async {
    // Fetch user from API
    final response = await http.get(Uri.parse('https://api.example.com/user'));

    if (response.statusCode != 200) {
      throw Exception('Failed to load user');
    }

    return User.fromJson(jsonDecode(response.body));
  }

  // Async method to update user
  Future<void> updateName(String newName) async {
    // Set loading state
    state = const AsyncLoading();

    // Make API call
    state = await AsyncValue.guard(() async {
      await http.post(
        Uri.parse('https://api.example.com/user'),
        body: jsonEncode({'name': newName}),
      );

      // Refresh data from server
      final response = await http.get(Uri.parse('https://api.example.com/user'));
      return User.fromJson(jsonDecode(response.body));
    });
  }

  // Refresh method
  Future<void> refresh() async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() => build());
  }
}
```

<div dir="rtl">

### الاستخدام:

</div>

```dart
class ProfilePage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProfileProvider);

    return Scaffold(
      appBar: AppBar(title: Text('Profile')),
      body: userAsync.when(
        data: (user) => Column(
          children: [
            Text('Name: ${user.name}'),
            Text('Email: ${user.email}'),
            ElevatedButton(
              onPressed: () {
                ref.read(userProfileProvider.notifier).updateName('New Name');
              },
              child: Text('Update Name'),
            ),
          ],
        ),
        loading: () => Center(child: CircularProgressIndicator()),
        error: (error, stack) => Center(child: Text('Error: $error')),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          ref.read(userProfileProvider.notifier).refresh();
        },
        child: Icon(Icons.refresh),
      ),
    );
  }
}
```

<div dir="rtl">

### مثال متقدم: Todos List

</div>

```dart
@riverpod
class Todos extends _$Todos {
  @override
  Future<List<Todo>> build() async {
    // Initial load from API
    return await _fetchTodos();
  }

  Future<List<Todo>> _fetchTodos() async {
    final response = await http.get(Uri.parse('https://api.example.com/todos'));

    if (response.statusCode != 200) {
      throw Exception('Failed to load todos');
    }

    final List<dynamic> json = jsonDecode(response.body);
    return json.map((item) => Todo.fromJson(item)).toList();
  }

  // Add new todo
  Future<void> addTodo(String title) async {
    // Optimistic update: Add to UI immediately
    final previousState = state;

    state = AsyncValue.data([
      ...?state.value,
      Todo(
        id: DateTime.now().toString(),
        title: title,
        completed: false,
      ),
    ]);

    // Make API call
    state = await AsyncValue.guard(() async {
      final response = await http.post(
        Uri.parse('https://api.example.com/todos'),
        body: jsonEncode({'title': title}),
      );

      if (response.statusCode != 201) {
        // Revert on error
        throw Exception('Failed to add todo');
      }

      // Refresh from server to get the real ID
      return await _fetchTodos();
    });

    // If error occurred, previous state is already reverted by AsyncValue.guard
  }

  // Toggle todo completion
  Future<void> toggleTodo(String id) async {
    // Find the todo
    final currentTodos = state.value ?? [];
    final todoIndex = currentTodos.indexWhere((t) => t.id == id);

    if (todoIndex == -1) return;

    // Optimistic update
    final previousState = state;
    final updatedTodo = currentTodos[todoIndex].copyWith(
      completed: !currentTodos[todoIndex].completed,
    );

    state = AsyncValue.data([
      for (int i = 0; i < currentTodos.length; i++)
        if (i == todoIndex) updatedTodo else currentTodos[i],
    ]);

    // API call
    state = await AsyncValue.guard(() async {
      final response = await http.patch(
        Uri.parse('https://api.example.com/todos/$id'),
        body: jsonEncode({'completed': updatedTodo.completed}),
      );

      if (response.statusCode != 200) {
        throw Exception('Failed to update todo');
      }

      return await _fetchTodos();
    });
  }

  // Delete todo
  Future<void> deleteTodo(String id) async {
    // Optimistic update
    final previousState = state;

    state = AsyncValue.data(
      state.value?.where((todo) => todo.id != id).toList() ?? [],
    );

    // API call
    state = await AsyncValue.guard(() async {
      final response = await http.delete(
        Uri.parse('https://api.example.com/todos/$id'),
      );

      if (response.statusCode != 204) {
        throw Exception('Failed to delete todo');
      }

      return await _fetchTodos();
    });
  }

  // Refresh
  Future<void> refresh() async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() => _fetchTodos());
  }
}

class Todo {
  final String id;
  final String title;
  final bool completed;

  Todo({
    required this.id,
    required this.title,
    required this.completed,
  });

  Todo copyWith({String? id, String? title, bool? completed}) {
    return Todo(
      id: id ?? this.id,
      title: title ?? this.title,
      completed: completed ?? this.completed,
    );
  }

  factory Todo.fromJson(Map<String, dynamic> json) {
    return Todo(
      id: json['id'],
      title: json['title'],
      completed: json['completed'],
    );
  }
}
```

<div dir="rtl">

---

## 3️⃣ AsyncNotifier مع Progress Tracking

تقدر تتبع progress الـ async operations:

</div>

```dart
@riverpod
class FileDownloader extends _$FileDownloader {
  @override
  Future<String?> build() async {
    // No initial file
    return null;
  }

  Future<void> downloadFile(String url) async {
    // Show loading with 0% progress
    state = AsyncLoading(progress: 0.0);

    try {
      // Simulate download with progress updates
      for (int i = 0; i <= 100; i += 10) {
        await Future.delayed(Duration(milliseconds: 200));

        // Update progress
        state = AsyncLoading(progress: i / 100);
      }

      // Download complete
      state = AsyncValue.data(url);
    } catch (error, stack) {
      state = AsyncValue.error(error, stack);
    }
  }
}

// Usage
class DownloadPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final downloadState = ref.watch(fileDownloaderProvider);

    return Scaffold(
      body: Center(
        child: downloadState.when(
          data: (filePath) => filePath == null
              ? Text('No file downloaded')
              : Text('Downloaded: $filePath'),
          loading: (progress) => Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              CircularProgressIndicator(value: progress),
              SizedBox(height: 16),
              Text('${(progress * 100).toInt()}%'),
            ],
          ),
          error: (error, stack) => Text('Error: $error'),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          ref.read(fileDownloaderProvider.notifier).downloadFile('file.pdf');
        },
        child: Icon(Icons.download),
      ),
    );
  }
}
```

<div dir="rtl">

---

## 4️⃣ Notifier مع Parameters

تقدر تستخدم parameters مع Notifier:

</div>

```dart
// Notifier for specific todo item
@riverpod
class TodoItem extends _$TodoItem {
  late String _id; // Store the parameter for later use

  @override
  Future<Todo> build(String id) async {
    _id = id; // Save parameter value

    // Fetch specific todo by ID
    final response = await http.get(
      Uri.parse('https://api.example.com/todos/$id'),
    );

    if (response.statusCode != 200) {
      throw Exception('Failed to load todo');
    }

    return Todo.fromJson(jsonDecode(response.body));
  }

  Future<void> updateTitle(String newTitle) async {
    state = const AsyncLoading();

    state = await AsyncValue.guard(() async {
      await http.patch(
        Uri.parse('https://api.example.com/todos/$_id'),
        body: jsonEncode({'title': newTitle}),
      );

      return await build(_id);
    });
  }

  // Alternative: Pass id as parameter to the method
  // Future<void> updateTitle(String id, String newTitle) async {
  //   state = const AsyncLoading();
  //   state = await AsyncValue.guard(() async {
  //     await http.patch(
  //       Uri.parse('https://api.example.com/todos/$id'),
  //       body: jsonEncode({'title': newTitle}),
  //     );
  //     return await build(id);
  //   });
  // }
}

// Usage - different instances for different IDs
final todo1 = ref.watch(todoItemProvider('1'));
final todo2 = ref.watch(todoItemProvider('2'));

// Call methods on the notifier
ref.read(todoItemProvider('1').notifier).updateTitle('New Title');
```

<div dir="rtl">

---

## 5️⃣ Best Practices

### ✅ Do:

**1. استخدم AsyncValue.guard للـ error handling:**
</div>

```dart
// ✅ Good
state = await AsyncValue.guard(() async {
  return await apiCall();
});

// ❌ Bad
try {
  state = AsyncValue.data(await apiCall());
} catch (error, stack) {
  state = AsyncValue.error(error, stack);
}
```

<div dir="rtl">

**2. استخدم Optimistic Updates للـ better UX:**

</div>

```dart
// ✅ Good - Updates UI immediately
Future<void> addItem(Item item) async {
  state = AsyncValue.data([...?state.value, item]);  // Optimistic
  state = await AsyncValue.guard(() => _fetchFromApi());  // Then sync
}

// ❌ Bad - User waits for API
Future<void> addItem(Item item) async {
  state = await AsyncValue.guard(() => _addAndFetch(item));
}
```

<div dir="rtl">

**3. عمل methods واضحة ومفهومة:**

</div>

```dart
// ✅ Good - Clear method names
void addToCart(Product product)
void removeFromCart(String productId)
void updateQuantity(String productId, int quantity)

// ❌ Bad - Generic names
void update(dynamic data)
void handle(String id, int value)
```

<div dir="rtl">

### ❌ Don't:

**1. متعملش side effects في build():**

</div>

```dart
// ❌ Bad
@override
Future<Data> build() async {
  analytics.log('page_viewed');  // Side effect!
  return await fetchData();
}

// ✅ Good - Side effects in methods only
@override
Future<Data> build() async {
  return await fetchData();
}

void logPageView() {
  analytics.log('page_viewed');
}
```

<div dir="rtl">

**2. متعدلش state مباشرة في Async operations بدون AsyncValue:**

</div>

```dart
// ❌ Bad
Future<void> updateUser() async {
  final user = await api.getUser();
  state = user;  // Wrong! state is AsyncValue<User>, not User
}

// ✅ Good
Future<void> updateUser() async {
  state = await AsyncValue.guard(() async {
    return await api.getUser();
  });
}
```

<div dir="rtl">

---

## 📊 ملخص: Notifier vs AsyncNotifier

| Aspect | Notifier | AsyncNotifier |
|--------|----------|---------------|
| **build() returns** | `T` | `Future<T>` |
| **Initial state** | Synchronous | Asynchronous (API, DB) |
| **State type** | `T` | `AsyncValue<T>` |
| **UI consumption** | `ref.watch(provider)` | `ref.watch(provider).when()` |
| **Methods** | Sync or Async | Usually Async |
| **Use case** | UI state, counters, local data | API data, async operations |

---

## 🔗 الخطوة الجاية

في الملف الجاي هنعمل **مقارنة تفصيلية** بين Classic Syntax و Code Generation:
- نفس الأمثلة بالطريقتين
- الفروقات الأساسية
- إزاي تختار

جاهز؟ يلا نكمل! 🚀

---

## 📚 المصادر

- [Notifier and AsyncNotifier | Riverpod](https://codewithandrea.com/articles/flutter-riverpod-async-notifier/)
- [AsyncNotifier API Documentation](https://pub.dev/documentation/riverpod/latest/riverpod/AsyncNotifier-class.html)
- [Riverpod Official Docs](https://riverpod.dev/docs/concepts2/providers)

</div>
