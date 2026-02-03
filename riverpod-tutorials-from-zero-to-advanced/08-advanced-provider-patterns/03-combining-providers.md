<div dir="rtl">

# Combining Providers - Derived State

**المستوى**: 🔴 متقدم

## 📌 الخلاصة

**Derived/Computed State** = provider يحسب قيمته بناءً على providers تانية.

```dart
// Base providers
@riverpod
class Cart extends _$Cart {
  @override
  List<CartItem> build() => [];
}

@riverpod
double taxRate(TaxRateRef ref) => 0.15;

// Derived: Subtotal from cart
@riverpod
double cartSubtotal(CartSubtotalRef ref) {
  final items = ref.watch(cartProvider);
  return items.fold(0.0, (sum, item) => sum + item.price);
}

// Derived: Tax from subtotal and tax rate
@riverpod
double cartTax(CartTaxRef ref) {
  final subtotal = ref.watch(cartSubtotalProvider);
  final rate = ref.watch(taxRateProvider);
  return subtotal * rate;
}

// Derived: Total from subtotal and tax
@riverpod
double cartTotal(CartTotalRef ref) {
  final subtotal = ref.watch(cartSubtotalProvider);
  final tax = ref.watch(cartTaxProvider);
  return subtotal + tax;
}
```

**Automatic reactivity**: لما cart يتغير → subtotal يتحدث → tax يتحدث → total يتحدث! 🎉

---

## 🎯 Common Patterns

### Pattern 1: Filtering

```dart
@riverpod
class Todos extends _$Todos {
  @override
  Future<List<Todo>> build() async => await api.getTodos();
}

@riverpod
class TodoFilter extends _$TodoFilter {
  @override
  String build() => 'all'; // all, active, completed

  void update(String filter) => state = filter;
}

// Derived: Filtered todos
@riverpod
List<Todo> filteredTodos(FilteredTodosRef ref) {
  final todosAsync = ref.watch(todosProvider);
  final filter = ref.watch(todoFilterProvider);

  final todos = todosAsync.value ?? [];

  return switch (filter) {
    'active' => todos.where((t) => !t.completed).toList(),
    'completed' => todos.where((t) => t.completed).toList(),
    _ => todos,
  };
}
```

### Pattern 2: Search

```dart
@riverpod
class SearchQuery extends _$SearchQuery {
  @override
  String build() => '';

  void update(String query) => state = query;
}

@riverpod
Future<List<Product>> allProducts(AllProductsRef ref) async {
  return await api.getProducts();
}

// Derived: Search results
@riverpod
List<Product> searchResults(SearchResultsRef ref) {
  final productsAsync = ref.watch(allProductsProvider);
  final query = ref.watch(searchQueryProvider).toLowerCase();

  final products = productsAsync.value ?? [];

  if (query.isEmpty) return products;

  return products
      .where((p) => p.name.toLowerCase().contains(query))
      .toList();
}
```

### Pattern 3: Aggregation

```dart
// Multiple data sources
@riverpod
Future<User> currentUser(CurrentUserRef ref) async {
  return await api.getCurrentUser();
}

@riverpod
Future<List<Order>> userOrders(UserOrdersRef ref) async {
  final user = await ref.watch(currentUserProvider.future);
  return await api.getUserOrders(user.id);
}

@riverpod
Future<UserStats> userStats(UserStatsRef ref) async {
  final user = await ref.watch(currentUserProvider.future);
  return await api.getUserStats(user.id);
}

// Aggregated dashboard
@riverpod
Future<Dashboard> dashboard(DashboardRef ref) async {
  final user = await ref.watch(currentUserProvider.future);
  final orders = await ref.watch(userOrdersProvider.future);
  final stats = await ref.watch(userStatsProvider.future);

  return Dashboard(
    user: user,
    orders: orders,
    stats: stats,
  );
}
```

---

## 📋 Best Practices

### 1. Keep Derived Providers Simple

```dart
// ✅ GOOD - One calculation
@riverpod
double total(TotalRef ref) {
  final subtotal = ref.watch(subtotalProvider);
  final tax = ref.watch(taxProvider);
  return subtotal + tax;
}

// ❌ BAD - Too much logic
@riverpod
ComplexResult complex(ComplexRef ref) {
  // 100 lines of complex calculations...
}
```

### 2. Use Meaningful Names

```dart
// ✅ GOOD
@riverpod
List<Todo> activeTodos(ActiveTodosRef ref) { ... }

@riverpod
List<Todo> completedTodos(CompletedTodosRef ref) { ... }

// ❌ BAD
@riverpod
List<Todo> todos1(Todos1Ref ref) { ... }

@riverpod
List<Todo> todos2(Todos2Ref ref) { ... }
```

### 3. Cache Expensive Computations

```dart
// ✅ Provider caches result automatically
@riverpod
List<Product> sortedProducts(SortedProductsRef ref) {
  final products = ref.watch(productsProvider).value ?? [];

  // Expensive sort - but only runs when products change!
  return products..sort((a, b) => a.price.compareTo(b.price));
}
```

---

## 📚 المصادر

- [Providers | Riverpod](https://riverpod.dev/docs/concepts2/providers)
- [Refs | Riverpod](https://riverpod.dev/docs/concepts2/refs)

</div>
