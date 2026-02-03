<div dir="rtl">

# Provider Dependencies - ربط Providers ببعض

**المستوى**: 🔴 متقدم

## 📌 المفاهيم الأساسية

في الملف ده هنتعلم:
- استخدام `ref.watch()` داخل providers
- بناء provider dependency chains
- Automatic updates and reactivity
- Error propagation
- Best practices

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تربط providers ببعضها
- تبني reactive data flows
- تتعامل مع complex dependencies
- تفهم متى الـ providers بتتحدث

---

## 🔗 ما هي Provider Dependencies؟

**Provider Dependencies** معناها إن provider يعتمد على provider تاني.

من [Riverpod Official Docs](https://riverpod.dev/docs/concepts2/providers):

> Ref.watch is Riverpod's defining feature that enables combining providers together seamlessly.

### مثال بسيط

</div>

```dart
// Provider 1: City name
@riverpod
class City extends _$City {
  @override
  String build() => 'Cairo';

  void update(String city) => state = city;
}

// Provider 2: Weather (depends on city)
@riverpod
Future<Weather> weather(WeatherRef ref) async {
  // Watch city provider
  final city = ref.watch(cityProvider);

  print('Fetching weather for $city');

  // Fetch weather for current city
  return await api.getWeather(city);
}

// Usage in UI
class WeatherWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final city = ref.watch(cityProvider);
    final weatherAsync = ref.watch(weatherProvider);

    return Column(
      children: [
        Text('City: $city'),

        weatherAsync.when(
          data: (weather) => Text('${weather.temperature}°C'),
          loading: () => const CircularProgressIndicator(),
          error: (error, _) => Text('Error: $error'),
        ),

        ElevatedButton(
          onPressed: () {
            // Change city
            ref.read(cityProvider.notifier).update('Alexandria');

            // weatherProvider automatically refetches! 🎉
          },
          child: const Text('Change to Alexandria'),
        ),
      ],
    );
  }
}
```

<div dir="rtl">

**ما يحصل:**
1. User clicks button
2. `cityProvider` updates to "Alexandria"
3. `weatherProvider` **automatically** sees city changed
4. `weatherProvider` re-executes and fetches new weather
5. UI updates with new weather

---

## 📊 Dependency Graphs

Providers يمكن تعتمد على providers تانية، واللي بدورها تعتمد على providers تالتة، وهكذا!

### مثال: Multi-Level Dependencies

</div>

```dart
// Level 1: User ID
@riverpod
class UserId extends _$UserId {
  @override
  String build() => 'user-123';

  void update(String id) => state = id;
}

// Level 2: User data (depends on userId)
@riverpod
Future<User> user(UserRef ref) async {
  final userId = ref.watch(userIdProvider);

  print('Fetching user: $userId');
  return await api.getUser(userId);
}

// Level 3: User posts (depends on user)
@riverpod
Future<List<Post>> userPosts(UserPostsRef ref) async {
  final user = await ref.watch(userProvider.future);

  print('Fetching posts for user: ${user.name}');
  return await api.getUserPosts(user.id);
}

// Level 4: Post count (depends on userPosts)
@riverpod
int postCount(PostCountRef ref) {
  final postsAsync = ref.watch(userPostsProvider);

  return postsAsync.when(
    data: (posts) => posts.length,
    loading: () => 0,
    error: (_, __) => 0,
  );
}

// Dependency Graph:
// userIdProvider
//    ↓
// userProvider
//    ↓
// userPostsProvider
//    ↓
// postCountProvider
```

<div dir="rtl">

**عند تغيير userId:**
```dart
ref.read(userIdProvider.notifier).update('user-456');

// Chain reaction:
// 1. userProvider refetches (new user ID)
// 2. userPostsProvider refetches (new user)
// 3. postCountProvider recalculates (new posts)
// All automatically! 🔥
```

---

## 🎨 ref.watch vs ref.read vs ref.listen

### في الـ Providers

</div>

```dart
@riverpod
Future<Data> myProvider(MyProviderRef ref) async {
  // ✅ ref.watch - Subscribe to changes
  // Provider rebuilds when dependency changes
  final userId = ref.watch(userIdProvider);

  // ✅ ref.read - One-time read
  // Use in methods (not build)
  void onAction() {
    final userId = ref.read(userIdProvider);
    // ...
  }

  // ✅ ref.listen - Side effects
  ref.listen(userIdProvider, (previous, next) {
    print('User ID changed from $previous to $next');
  });

  return await api.getData(userId);
}
```

<div dir="rtl">

**القواعد:**

| Method | في build() | في methods | Triggers rebuild |
|--------|-----------|-----------|------------------|
| **ref.watch** | ✅ أيوه | ❌ لأ | ✅ أيوه |
| **ref.read** | ❌ لأ | ✅ أيوه | ❌ لأ |
| **ref.listen** | ✅ أيوه | ✅ أيوه | ❌ لأ |

---

## 🔄 Watching Multiple Providers

ممكن تستخدم `ref.watch` أكتر من مرة في provider واحد!

</div>

```dart
@riverpod
Future<Dashboard> dashboard(DashboardRef ref) async {
  // Watch multiple providers
  final userId = ref.watch(userIdProvider);
  final settings = ref.watch(settingsProvider);
  final theme = ref.watch(themeProvider);

  print('Building dashboard for user: $userId');
  print('Settings: $settings');
  print('Theme: $theme');

  // Fetch dashboard data
  final user = await api.getUser(userId);
  final stats = await api.getStats(userId);
  final notifications = await api.getNotifications(userId);

  return Dashboard(
    user: user,
    stats: stats,
    notifications: notifications,
    settings: settings,
    theme: theme,
  );
}

// Dashboard rebuilds when ANY of these change:
// - userIdProvider
// - settingsProvider
// - themeProvider
```

<div dir="rtl">

---

## ⚡ Automatic Updates

من [Riverpod Docs](https://riverpod.dev/docs/concepts2/providers):

> If the watched provider ever changes, this will automatically call the dependent provider again and update the UI accordingly.

### مثال: Search with Filters

</div>

```dart
// Filter state
@riverpod
class SearchFilters extends _$SearchFilters {
  @override
  Filters build() => Filters(
        category: 'all',
        minPrice: 0,
        maxPrice: 1000,
      );

  void updateCategory(String category) {
    state = state.copyWith(category: category);
  }

  void updatePriceRange(double min, double max) {
    state = state.copyWith(minPrice: min, maxPrice: max);
  }
}

// Search query
@riverpod
class SearchQuery extends _$SearchQuery {
  @override
  String build() => '';

  void update(String query) => state = query;
}

// Search results (depends on query AND filters)
@riverpod
Future<List<Product>> searchResults(SearchResultsRef ref) async {
  final query = ref.watch(searchQueryProvider);
  final filters = ref.watch(searchFiltersProvider);

  if (query.isEmpty) return [];

  print('Searching for: $query');
  print('Filters: $filters');

  return await api.search(
    query: query,
    category: filters.category,
    minPrice: filters.minPrice,
    maxPrice: filters.maxPrice,
  );
}

// UI
class SearchScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final resultsAsync = ref.watch(searchResultsProvider);

    return Column(
      children: [
        // Search field
        TextField(
          onChanged: (value) {
            ref.read(searchQueryProvider.notifier).update(value);
            // searchResultsProvider refetches automatically!
          },
        ),

        // Filters
        FilterChips(
          onCategoryChanged: (category) {
            ref.read(searchFiltersProvider.notifier).updateCategory(category);
            // searchResultsProvider refetches automatically!
          },
        ),

        // Results
        Expanded(
          child: resultsAsync.when(
            data: (products) => ProductList(products),
            loading: () => const LoadingScreen(),
            error: (error, _) => ErrorMessage(error),
          ),
        ),
      ],
    );
  }
}
```

<div dir="rtl">

---

## 🎯 Error Propagation

لما dependent provider يفشل، الـ error بيوصل للـ UI.

</div>

```dart
// Base provider (might fail)
@riverpod
Future<User> user(UserRef ref, String userId) async {
  final result = await api.getUser(userId);

  if (result == null) {
    throw Exception('User not found');
  }

  return result;
}

// Dependent provider
@riverpod
Future<List<Post>> userPosts(UserPostsRef ref, String userId) async {
  // If userProvider fails, this provider never executes
  final user = await ref.watch(userProvider(userId).future);

  // This line never runs if user fetch failed
  return await api.getUserPosts(user.id);
}

// UI
class UserPostsWidget extends ConsumerWidget {
  final String userId;

  const UserPostsWidget(this.userId, {super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final postsAsync = ref.watch(userPostsProvider(userId));

    return postsAsync.when(
      data: (posts) => PostsList(posts),

      // Shows error from either userProvider OR userPostsProvider
      error: (error, stack) => Text('Error: $error'),

      loading: () => const CircularProgressIndicator(),
    );
  }
}
```

<div dir="rtl">

### معالجة الـ Errors بشكل منفصل

</div>

```dart
@riverpod
Future<Dashboard> dashboard(DashboardRef ref, String userId) async {
  try {
    // Try to get user
    final user = await ref.watch(userProvider(userId).future);

    try {
      // Try to get posts
      final posts = await api.getUserPosts(user.id);

      return Dashboard(user: user, posts: posts);
    } catch (postsError) {
      // Posts failed, but user succeeded - show partial data
      return Dashboard(user: user, posts: [], error: 'Failed to load posts');
    }
  } catch (userError) {
    // User failed - rethrow
    rethrow;
  }
}
```

<div dir="rtl">

---

## 📋 Best Practices

### 1. استخدم ref.watch في build() فقط

</div>

```dart
// ✅ GOOD
@riverpod
Future<Data> data(DataRef ref) async {
  final userId = ref.watch(userIdProvider);
  return await api.getData(userId);
}

// ❌ BAD - Don't watch in methods
@riverpod
class DataController extends _$DataController {
  @override
  Future<Data> build() async {
    return await api.getData();
  }

  Future<void> refresh() async {
    final userId = ref.watch(userIdProvider); // ❌ Error!
    // ...
  }
}
```

<div dir="rtl">

### 2. استخدم .future للـ Async Dependencies

</div>

```dart
// ✅ GOOD - Wait for async result
@riverpod
Future<List<Post>> userPosts(UserPostsRef ref) async {
  final user = await ref.watch(userProvider.future);
  return await api.getUserPosts(user.id);
}

// ❌ MESSY - Handle AsyncValue manually
@riverpod
Future<List<Post>> userPosts(UserPostsRef ref) async {
  final userAsync = ref.watch(userProvider);

  return userAsync.when(
    data: (user) async => await api.getUserPosts(user.id),
    loading: () async => [],
    error: (_, __) async => [],
  );
}
```

<div dir="rtl">

### 3. تجنب Circular Dependencies

</div>

```dart
// ❌ BAD - Circular dependency!
@riverpod
int providerA(ProviderARef ref) {
  final b = ref.watch(providerBProvider); // Watches B
  return b + 1;
}

@riverpod
int providerB(ProviderBRef ref) {
  final a = ref.watch(providerAProvider); // Watches A
  return a + 1;
}
// Runtime error! ⚠️

// ✅ GOOD - Extract shared state
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() => state++;
}

@riverpod
int doubleCounter(DoubleCounterRef ref) {
  final count = ref.watch(counterProvider);
  return count * 2;
}

@riverpod
int tripleCounter(TripleCounterRef ref) {
  final count = ref.watch(counterProvider);
  return count * 3;
}
```

<div dir="rtl">

### 4. وثق الـ Dependencies المعقدة

</div>

```dart
/// Provides dashboard data for the current user.
///
/// **Dependencies:**
/// - [userProvider] - Current user data
/// - [statsProvider] - User statistics
/// - [notificationsProvider] - Recent notifications
///
/// **Updates when:**
/// - User changes (login/logout)
/// - Stats refresh
/// - New notification arrives
///
/// **Errors:**
/// - Throws if user is not authenticated
/// - Returns partial data if stats/notifications fail
@riverpod
Future<Dashboard> dashboard(DashboardRef ref) async {
  // Implementation
}
```

<div dir="rtl">

### 5. استخدم ref.listen للـ Side Effects

</div>

```dart
@riverpod
class CartController extends _$CartController {
  @override
  List<CartItem> build() {
    // Listen to user changes
    ref.listen(userProvider, (previous, next) {
      // User changed - clear cart
      if (previous?.id != next.value?.id) {
        state = [];
      }
    });

    return [];
  }

  void addItem(Product product) {
    state = [...state, CartItem.fromProduct(product)];
  }
}
```

<div dir="rtl">

---

## 🎓 أمثلة عملية

### مثال 1: E-Commerce Cart System

</div>

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'cart.g.dart';

// Models
class Product {
  final String id;
  final String name;
  final double price;

  Product({required this.id, required this.name, required this.price});
}

class CartItem {
  final Product product;
  final int quantity;

  CartItem({required this.product, this.quantity = 1});

  double get total => product.price * quantity;

  CartItem copyWith({int? quantity}) {
    return CartItem(
      product: product,
      quantity: quantity ?? this.quantity,
    );
  }
}

// Cart state
@riverpod
class Cart extends _$Cart {
  @override
  List<CartItem> build() => [];

  void addItem(Product product) {
    final existing = state.where((item) => item.product.id == product.id);

    if (existing.isNotEmpty) {
      // Increment quantity
      state = [
        for (final item in state)
          if (item.product.id == product.id)
            item.copyWith(quantity: item.quantity + 1)
          else
            item,
      ];
    } else {
      // Add new item
      state = [...state, CartItem(product: product)];
    }
  }

  void removeItem(String productId) {
    state = state.where((item) => item.product.id != productId).toList();
  }

  void updateQuantity(String productId, int quantity) {
    if (quantity <= 0) {
      removeItem(productId);
      return;
    }

    state = [
      for (final item in state)
        if (item.product.id == productId)
          item.copyWith(quantity: quantity)
        else
          item,
    ];
  }

  void clear() {
    state = [];
  }
}

// Derived: Subtotal (depends on cart)
@riverpod
double cartSubtotal(CartSubtotalRef ref) {
  final items = ref.watch(cartProvider);
  return items.fold(0.0, (sum, item) => sum + item.total);
}

// Derived: Tax (depends on subtotal)
@riverpod
double cartTax(CartTaxRef ref) {
  final subtotal = ref.watch(cartSubtotalProvider);
  return subtotal * 0.15; // 15% tax
}

// Derived: Shipping (depends on subtotal)
@riverpod
double cartShipping(CartShippingRef ref) {
  final subtotal = ref.watch(cartSubtotalProvider);

  if (subtotal >= 100) {
    return 0.0; // Free shipping
  } else {
    return 10.0; // Flat rate
  }
}

// Derived: Total (depends on subtotal, tax, shipping)
@riverpod
double cartTotal(CartTotalRef ref) {
  final subtotal = ref.watch(cartSubtotalProvider);
  final tax = ref.watch(cartTaxProvider);
  final shipping = ref.watch(cartShippingProvider);

  return subtotal + tax + shipping;
}

// Derived: Item count (depends on cart)
@riverpod
int cartItemCount(CartItemCountRef ref) {
  final items = ref.watch(cartProvider);
  return items.fold(0, (sum, item) => sum + item.quantity);
}

// UI
class CartScreen extends ConsumerWidget {
  const CartScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final items = ref.watch(cartProvider);
    final subtotal = ref.watch(cartSubtotalProvider);
    final tax = ref.watch(cartTaxProvider);
    final shipping = ref.watch(cartShippingProvider);
    final total = ref.watch(cartTotalProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('Shopping Cart')),
      body: Column(
        children: [
          // Cart items
          Expanded(
            child: items.isEmpty
                ? const Center(child: Text('Cart is empty'))
                : ListView.builder(
                    itemCount: items.length,
                    itemBuilder: (context, index) {
                      final item = items[index];
                      return ListTile(
                        title: Text(item.product.name),
                        subtitle: Text(
                          '\$${item.product.price.toStringAsFixed(2)} x ${item.quantity}',
                        ),
                        trailing: Row(
                          mainAxisSize: MainAxisSize.min,
                          children: [
                            IconButton(
                              icon: const Icon(Icons.remove),
                              onPressed: () {
                                ref
                                    .read(cartProvider.notifier)
                                    .updateQuantity(
                                      item.product.id,
                                      item.quantity - 1,
                                    );
                              },
                            ),
                            Text('${item.quantity}'),
                            IconButton(
                              icon: const Icon(Icons.add),
                              onPressed: () {
                                ref
                                    .read(cartProvider.notifier)
                                    .updateQuantity(
                                      item.product.id,
                                      item.quantity + 1,
                                    );
                              },
                            ),
                            IconButton(
                              icon: const Icon(Icons.delete),
                              onPressed: () {
                                ref
                                    .read(cartProvider.notifier)
                                    .removeItem(item.product.id);
                              },
                            ),
                          ],
                        ),
                      );
                    },
                  ),
          ),

          // Summary
          Container(
            padding: const EdgeInsets.all(16),
            decoration: BoxDecoration(
              color: Colors.grey.shade200,
              border: Border(top: BorderSide(color: Colors.grey.shade400)),
            ),
            child: Column(
              children: [
                Row(
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                  children: [
                    const Text('Subtotal:'),
                    Text('\$${subtotal.toStringAsFixed(2)}'),
                  ],
                ),
                Row(
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                  children: [
                    const Text('Tax:'),
                    Text('\$${tax.toStringAsFixed(2)}'),
                  ],
                ),
                Row(
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                  children: [
                    const Text('Shipping:'),
                    Text(shipping == 0
                        ? 'FREE'
                        : '\$${shipping.toStringAsFixed(2)}'),
                  ],
                ),
                const Divider(),
                Row(
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                  children: [
                    const Text(
                      'Total:',
                      style: TextStyle(
                        fontWeight: FontWeight.bold,
                        fontSize: 18,
                      ),
                    ),
                    Text(
                      '\$${total.toStringAsFixed(2)}',
                      style: const TextStyle(
                        fontWeight: FontWeight.bold,
                        fontSize: 18,
                      ),
                    ),
                  ],
                ),
                const SizedBox(height: 16),
                SizedBox(
                  width: double.infinity,
                  child: ElevatedButton(
                    onPressed: items.isEmpty ? null : () {},
                    child: const Text('Checkout'),
                  ),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}

// Badge showing cart item count
class CartBadge extends ConsumerWidget {
  const CartBadge({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(cartItemCountProvider);

    return IconButton(
      icon: Badge(
        label: Text('$count'),
        isLabelVisible: count > 0,
        child: const Icon(Icons.shopping_cart),
      ),
      onPressed: () {
        Navigator.push(
          context,
          MaterialPageRoute(builder: (_) => const CartScreen()),
        );
      },
    );
  }
}
```

<div dir="rtl">

**Dependency Graph:**
```
cartProvider
    ↓
cartSubtotalProvider
    ↓              ↓
cartTaxProvider  cartShippingProvider
    ↓              ↓
    cartTotalProvider

cartProvider
    ↓
cartItemCountProvider
```

**Automatic Updates:**
- Add/remove item → subtotal updates → tax/shipping update → total updates
- Change quantity → subtotal updates → tax/shipping update → total updates
- All calculations happen automatically! 🎉

---

## 🔗 الخطوة الجاية

في الملف الجاي هنتعلم **Provider Families** - كيف تستخدم parameters مع providers!

جاهز؟ يلا نكمل! 🚀

---

## 📚 المصادر

- [Providers | Riverpod](https://riverpod.dev/docs/concepts2/providers)
- [Refs | Riverpod](https://riverpod.dev/docs/concepts2/refs)
- [Best practice for Provider depends on other providers | Discussion](https://github.com/rrousselGit/riverpod/discussions/1259)

</div>
