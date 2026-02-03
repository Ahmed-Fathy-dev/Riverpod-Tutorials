<div dir="rtl">

# Provider Families - Parameters في Providers

**المستوى**: 🔴 متقدم

## 📌 الخلاصة السريعة

في **Riverpod 3**، إضافة parameters للـ providers أصبحت **سهلة جداً**:

```dart
// Just add parameters! No .family needed
@riverpod
Future<Product> product(ProductRef ref, String productId) async {
  return await api.getProduct(productId);
}

// Usage
final productAsync = ref.watch(productProvider('product-123'));
```

من [Riverpod Family Docs](https://riverpod.dev/docs/concepts2/family):

> When using code-generation, providers can receive any number of parameters

---

## 🎯 ما هي Provider Families؟

**Provider Families** = نفس الـ provider، instances مختلفة، كل instance لها parameter مختلف.

### مثال توضيحي

```dart
// One provider definition
@riverpod
Future<Todo> todo(TodoRef ref, String todoId) async {
  print('Fetching todo: $todoId');
  return await api.getTodo(todoId);
}

// Multiple independent instances
ref.watch(todoProvider('todo-1')); // Instance 1
ref.watch(todoProvider('todo-2')); // Instance 2
ref.watch(todoProvider('todo-3')); // Instance 3

// Each has its own state!
```

---

## 🆚 Riverpod 2 vs Riverpod 3

### في Riverpod 2 (Classic):

```dart
// Had to use .family modifier
final productProvider = FutureProvider.family<Product, String>(
  (ref, productId) async {
    return await api.getProduct(productId);
  },
);

// Multiple modifiers = verbose!
final productProvider = FutureProvider.autoDispose.family<Product, String>(
  (ref, productId) async {
    return await api.getProduct(productId);
  },
);
```

### في Riverpod 3 (Code Generation):

```dart
// Just add parameters - that's it!
@riverpod
Future<Product> product(ProductRef ref, String productId) async {
  return await api.getProduct(productId);
}
// AutoDispose by default ✅
// Family automatically ✅
```

---

## 📖 Parameter Types

### 1. Single Parameter

```dart
@riverpod
Future<User> user(UserRef ref, String userId) async {
  return await api.getUser(userId);
}
```

### 2. Multiple Parameters

```dart
@riverpod
Future<List<Product>> products(
  ProductsRef ref,
  String category,
  double minPrice,
  double maxPrice,
) async {
  return await api.getProducts(
    category: category,
    minPrice: minPrice,
    maxPrice: maxPrice,
  );
}

// Usage
final products = ref.watch(productsProvider('electronics', 100, 500));
```

### 3. Named Parameters

```dart
@riverpod
Future<List<Todo>> todos(
  TodosRef ref, {
  required String userId,
  bool completed = false,
}) async {
  return await api.getTodos(
    userId: userId,
    completed: completed,
  );
}

// Usage
ref.watch(todosProvider(userId: 'user-123'));
ref.watch(todosProvider(userId: 'user-123', completed: true));
```

### 4. Objects as Parameters

```dart
class SearchFilters {
  final String query;
  final String category;
  final double minPrice;
  final double maxPrice;

  SearchFilters({
    required this.query,
    required this.category,
    required this.minPrice,
    required this.maxPrice,
  });

  // IMPORTANT: Must implement == and hashCode!
  @override
  bool operator ==(Object other) =>
      other is SearchFilters &&
      other.query == query &&
      other.category == category &&
      other.minPrice == minPrice &&
      other.maxPrice == maxPrice;

  @override
  int get hashCode => Object.hash(query, category, minPrice, maxPrice);
}

@riverpod
Future<List<Product>> searchProducts(
  SearchProductsRef ref,
  SearchFilters filters,
) async {
  return await api.search(filters);
}
```

---

## ⚠️ Parameter Requirements

من الـ [Official Docs](https://riverpod.dev/docs/concepts2/family):

> Parameters passed need to have a consistent ==/hashCode

```dart
// ✅ GOOD - Primitive types (already have == and hashCode)
@riverpod
Future<User> user(UserRef ref, int userId) async { ... }

@riverpod
Future<Data> data(DataRef ref, String key) async { ... }

// ✅ GOOD - Custom class WITH == and hashCode
class Filter {
  final String category;

  Filter(this.category);

  @override
  bool operator ==(Object other) =>
      other is Filter && other.category == category;

  @override
  int get hashCode => category.hashCode;
}

// ❌ BAD - Custom class WITHOUT == and hashCode
class BadFilter {
  final String category;
  BadFilter(this.category);
  // Missing == and hashCode!
}

@riverpod
Future<Data> data(DataRef ref, BadFilter filter) async { ... }
// Different instances with same values = different providers!
```

---

## 🧹 Memory Management

### AutoDispose (Default)

```dart
// By default, each family instance auto-disposes
@riverpod
Future<Product> product(ProductRef ref, String id) async {
  return await api.getProduct(id);
}

// When you stop watching:
// productProvider('product-123') disposes automatically ✅
```

### KeepAlive (Optional)

```dart
@Riverpod(keepAlive: true)
Future<Category> category(CategoryRef ref, String id) async {
  return await api.getCategory(id);
}

// Lives forever (even after no watchers)
// Use for: App config, rarely-changing data
```

---

## 🎯 Common Patterns

### Pattern 1: Todo Detail Screen

```dart
@riverpod
class TodoController extends _$TodoController {
  @override
  Future<Todo> build(String todoId) async {
    return await api.getTodo(todoId);
  }

  Future<void> toggle() async {
    final todo = state.requireValue;

    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      await api.toggleTodo(todoId);
      return todo.copyWith(completed: !todo.completed);
    });
  }
}

// UI
class TodoDetailScreen extends ConsumerWidget {
  final String todoId;

  const TodoDetailScreen(this.todoId, {super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final todoAsync = ref.watch(todoControllerProvider(todoId));

    return todoAsync.when(
      data: (todo) => ListTile(
        title: Text(todo.title),
        trailing: Checkbox(
          value: todo.completed,
          onChanged: (_) {
            ref.read(todoControllerProvider(todoId).notifier).toggle();
          },
        ),
      ),
      loading: () => const CircularProgressIndicator(),
      error: (error, _) => Text('Error: $error'),
    );
  }
}
```

### Pattern 2: Paginated Lists

```dart
@riverpod
Future<List<Product>> productsPage(
  ProductsPageRef ref,
  int page,
) async {
  return await api.getProducts(page: page);
}

// UI
class ProductsScreen extends ConsumerStatefulWidget {
  @override
  ConsumerState<ProductsScreen> createState() => _ProductsScreenState();
}

class _ProductsScreenState extends ConsumerState<ProductsScreen> {
  int _currentPage = 1;

  @override
  Widget build(BuildContext context) {
    final productsAsync = ref.watch(productsPageProvider(_currentPage));

    return productsAsync.when(
      data: (products) => Column(
        children: [
          Expanded(child: ProductList(products)),
          Row(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              IconButton(
                icon: const Icon(Icons.arrow_back),
                onPressed: _currentPage > 1
                    ? () => setState(() => _currentPage--)
                    : null,
              ),
              Text('Page $_currentPage'),
              IconButton(
                icon: const Icon(Icons.arrow_forward),
                onPressed: () => setState(() => _currentPage++),
              ),
            ],
          ),
        ],
      ),
      loading: () => const Center(child: CircularProgressIndicator()),
      error: (error, _) => Center(child: Text('Error: $error')),
    );
  }
}
```

### Pattern 3: User-Specific Data

```dart
@riverpod
Future<List<Order>> userOrders(
  UserOrdersRef ref,
  String userId,
) async {
  return await api.getUserOrders(userId);
}

@riverpod
Future<UserStats> userStats(
  UserStatsRef ref,
  String userId,
) async {
  return await api.getUserStats(userId);
}

@riverpod
Future<UserProfile> userProfile(
  UserProfileRef ref,
  String userId,
) async {
  return await api.getUserProfile(userId);
}

// Usage - all share same userId
class UserDashboard extends ConsumerWidget {
  final String userId;

  const UserDashboard(this.userId, {super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final ordersAsync = ref.watch(userOrdersProvider(userId));
    final statsAsync = ref.watch(userStatsProvider(userId));
    final profileAsync = ref.watch(userProfileProvider(userId));

    // ...
  }
}
```

---

## 📋 Best Practices

### 1. Use Primitive Types When Possible

```dart
// ✅ GOOD
@riverpod
Future<User> user(UserRef ref, String userId) async { ... }

// ⚠️ AVOID (unless necessary)
@riverpod
Future<User> user(UserRef ref, UserId userId) async { ... }
```

### 2. Implement == and hashCode for Custom Types

```dart
// Use freezed for automatic implementation
@freezed
class SearchParams with _$SearchParams {
  factory SearchParams({
    required String query,
    required String category,
  }) = _SearchParams;
}

@riverpod
Future<List<Product>> search(
  SearchRef ref,
  SearchParams params,
) async {
  return await api.search(params);
}
```

### 3. Enable AutoDispose (Default)

```dart
// ✅ GOOD - Default behavior
@riverpod
Future<Product> product(ProductRef ref, String id) async { ... }

// ⚠️ Only if really needed
@Riverpod(keepAlive: true)
Future<Category> category(CategoryRef ref, String id) async { ... }
```

---

## 📚 المصادر

- [Family | Riverpod](https://riverpod.dev/docs/concepts2/family)
- [What's new in Riverpod 3.0 | Riverpod](https://riverpod.dev/docs/whats_new)

</div>
