<div dir="rtl">

# Performance Tips

**المستوى**: 🔴 متقدم

## ⚡ Optimization Techniques

### 1. Use .select() to Prevent Rebuilds

```dart
// ❌ BAD - Rebuilds on any user change
final user = ref.watch(userProvider);
return Text(user.name);

// ✅ GOOD - Rebuilds only when name changes
final name = ref.watch(userProvider.select((u) => u.name));
return Text(name);
```

### 2. Use AutoDispose

```dart
// ✅ GOOD - Default behavior
@riverpod
Future<Product> product(ProductRef ref, String id) async { ... }

// ⚠️ CAREFUL - Only when really needed
@Riverpod(keepAlive: true)
Future<Config> appConfig(AppConfigRef ref) async { ... }
```

### 3. Cache Expensive Computations

```dart
// ✅ GOOD - Riverpod caches automatically
@riverpod
List<Product> sortedProducts(SortedProductsRef ref) {
  final products = ref.watch(productsProvider).value ?? [];
  
  // Expensive sort - only runs when products change!
  return products..sort((a, b) => a.price.compareTo(b.price));
}
```

### 4. Use .future for Async Dependencies

```dart
// ✅ GOOD - Cleaner
@riverpod
Future<List<Order>> userOrders(UserOrdersRef ref) async {
  final user = await ref.watch(userProvider.future);
  return await api.getUserOrders(user.id);
}

// ❌ VERBOSE
@riverpod
Future<List<Order>> userOrders(UserOrdersRef ref) async {
  final userAsync = ref.watch(userProvider);
  return userAsync.when(
    data: (user) async => await api.getUserOrders(user.id),
    loading: () async => [],
    error: (_, __) async => [],
  );
}
```

### 5. Batch Updates

```dart
// ✅ GOOD - Single update
void updateMultiple() {
  state = state.copyWith(
    name: 'New Name',
    age: 25,
    email: 'new@email.com',
  );
}

// ❌ BAD - Multiple updates
void updateMultiple() {
  state = state.copyWith(name: 'New Name');
  state = state.copyWith(age: 25);
  state = state.copyWith(email: 'new@email.com');
}
```

</div>
