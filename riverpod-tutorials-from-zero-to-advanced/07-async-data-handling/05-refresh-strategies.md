<div dir="rtl">

# Refresh & Caching Strategies

**المستوى**: 🟡 متوسط - 🔴 متقدم

## 📌 المفاهيم الأساسية

في الملف ده هنتعلم:
- `ref.refresh()` vs `ref.invalidate()`
- AutoDispose و Caching
- `keepAlive` Strategies
- Timer-based Caching
- Focus-based Refresh
- Lifecycle Methods

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تختار بين refresh و invalidate
- تعمل caching strategies مختلفة
- تستخدم keepAlive بشكل صحيح
- تعمل auto-refresh
- تتعامل مع provider lifecycle

---

## 🔄 ref.refresh() vs ref.invalidate()

### الفرق الأساسي

من [Riverpod Official FAQ](https://riverpod.dev/docs/root/faq):

> **ref.refresh** is syntax sugar for invalidate + read

</div>

```dart
// ref.refresh() = ref.invalidate() + ref.read()

// These are equivalent:
ref.refresh(userProvider);

// Is the same as:
ref.invalidate(userProvider);
ref.read(userProvider);
```

<div dir="rtl">

### متى تستخدم كل واحدة؟

</div>

```dart
// ✅ Use ref.invalidate() - محتاج تحدث بس
class MyScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return ElevatedButton(
      onPressed: () {
        // Just mark as invalid - will rebuild on next read
        ref.invalidate(userProvider);

        // Provider rebuilds automatically when UI reads it
      },
      child: const Text('Refresh'),
    );
  }
}

// ✅ Use ref.refresh() - محتاج القيمة الجديدة فوراً
class MyScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return ElevatedButton(
      onPressed: () async {
        // Get new value immediately
        final newUser = ref.refresh(userProvider);

        // Use the new value
        print('User refreshed: ${newUser.value?.name}');
      },
      child: const Text('Refresh'),
    );
  }
}

// ✅ Use ref.refresh().future - محتاج تستنى النتيجة
class MyScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return ElevatedButton(
      onPressed: () async {
        try {
          // Wait for new data
          final user = await ref.refresh(userProvider.future);
          print('User loaded: ${user.name}');
        } catch (error) {
          print('Failed to refresh: $error');
        }
      },
      child: const Text('Refresh'),
    );
  }
}
```

<div dir="rtl">

### الفرق في السلوك

</div>

```dart
// Example: Counter provider
@riverpod
class Counter extends _$Counter {
  @override
  int build() {
    print('Counter build()');
    return 0;
  }

  void increment() => state++;
}

// Scenario 1: Using invalidate()
void example1(WidgetRef ref) {
  print('Before invalidate');
  ref.invalidate(counterProvider); // Marks as invalid
  print('After invalidate'); // build() not called yet!

  // build() will be called on next read/watch
  final value = ref.read(counterProvider); // NOW build() is called
  print('Value: $value');
}
// Output:
// Before invalidate
// After invalidate
// Counter build()
// Value: 0

// Scenario 2: Using refresh()
void example2(WidgetRef ref) {
  print('Before refresh');
  ref.refresh(counterProvider); // Invalidates AND reads immediately
  print('After refresh');
}
// Output:
// Before refresh
// Counter build()
// After refresh
```

<div dir="rtl">

**الخلاصة:**

| Method | Behavior | Use Case |
|--------|----------|----------|
| **ref.invalidate()** | يحدد إن الـ provider بقى invalid، هيتبني على أول read | الأفضل - أخف على الـ performance |
| **ref.refresh()** | يحدد invalid **ويقرأ فوراً** | لما محتاج القيمة الجديدة حالاً |
| **ref.refresh().future** | ينتظر النتيجة الجديدة | لما محتاج async result |

---

## 🔐 AutoDispose vs KeepAlive

### Default Behavior

</div>

```dart
// With code generation: DEFAULT = AutoDispose ✅
@riverpod
Future<User> user(UserRef ref) async {
  return await api.getUser();
}
// Automatically disposes when no listeners

// Classic syntax: DEFAULT = NO AutoDispose ❌
final userProvider = FutureProvider<User>((ref) async {
  return await api.getUser();
});
// Lives forever (memory leak risk!)
```

<div dir="rtl">

### كيف يشتغل AutoDispose؟

</div>

```dart
@riverpod
Future<User> user(UserRef ref) async {
  print('Loading user...');
  return await api.getUser();
}

// في الـ Widget
class UserScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider);
    // Provider is active - has listener

    return switch (userAsync) {
      AsyncData(:final value) => Text(value.name),
      _ => const CircularProgressIndicator(),
    };
  }
}

// When UserScreen is removed from tree:
// 1. No more listeners
// 2. Provider is disposed
// 3. State is cleared
// 4. Memory is freed

// When UserScreen is rebuilt:
// 1. watch() called again
// 2. Provider rebuilds
// 3. 'Loading user...' printed again
```

<div dir="rtl">

### تعطيل AutoDispose

</div>

```dart
// Method 1: @Riverpod annotation (RECOMMENDED)
@Riverpod(keepAlive: true)
Future<AppConfig> appConfig(AppConfigRef ref) async {
  return await api.getConfig();
}
// Lives forever - never disposed

// Method 2: ref.keepAlive() in build
@riverpod
Future<User> user(UserRef ref) async {
  // Keep alive forever
  ref.keepAlive();

  return await api.getUser();
}
// Also lives forever

// Method 3: Conditional keepAlive
@riverpod
Future<User> user(UserRef ref) async {
  final user = await api.getUser();

  // Only keep alive if successful
  if (user != null) {
    ref.keepAlive();
  }

  return user;
}
```

<div dir="rtl">

**من [GitHub Discussion #3584](https://github.com/rrousselGit/riverpod/discussions/3584):**

> استخدم `@Riverpod(keepAlive: true)` بدل `ref.keepAlive()` عشان الـ linter يقدر يساعدك!

---

## ⏰ Timer-Based Caching

من [Riverpod Data Caching Guide](https://codewithandrea.com/articles/flutter-riverpod-data-caching-providers-lifecycle/):

### Cache لمدة معينة

</div>

```dart
@riverpod
Future<List<Product>> products(ProductsRef ref) async {
  // Keep alive for 5 minutes
  final link = ref.keepAlive();

  // After 5 minutes, re-enable auto dispose
  Timer(const Duration(minutes: 5), () {
    link.close();
  });

  return await api.getProducts();
}

// Usage
class ProductsScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final productsAsync = ref.watch(productsProvider);

    // First visit: Loads from API
    // Leave screen: State kept for 5 minutes
    // Return within 5 minutes: Uses cached data
    // Return after 5 minutes: Reloads from API

    return switch (productsAsync) {
      AsyncData(:final value) => ProductGrid(value),
      _ => const LoadingScreen(),
    };
  }
}
```

<div dir="rtl">

### Cache مع Conditions

</div>

```dart
@riverpod
Future<User> user(UserRef ref, String userId) async {
  final user = await api.getUser(userId);

  // Only cache successful results
  if (user != null) {
    final link = ref.keepAlive();

    // Cache for 2 minutes
    Timer(const Duration(minutes: 2), () {
      link.close();
    });
  }

  // Errors are not cached - will retry immediately
  return user;
}
```

<div dir="rtl">

### Custom Cache Duration

</div>

```dart
// Helper for timer-based caching
extension CacheExtension on Ref {
  void cacheFor(Duration duration) {
    final link = keepAlive();
    Timer(duration, link.close);
  }
}

// Usage
@riverpod
Future<Data> data(DataRef ref) async {
  // Cache for 10 minutes
  ref.cacheFor(const Duration(minutes: 10));

  return await api.getData();
}

// Different cache times for different providers
@riverpod
Future<AppConfig> appConfig(AppConfigRef ref) async {
  ref.cacheFor(const Duration(hours: 1)); // Long cache
  return await api.getConfig();
}

@riverpod
Future<LiveData> liveData(LiveDataRef ref) async {
  ref.cacheFor(const Duration(seconds: 30)); // Short cache
  return await api.getLiveData();
}
```

<div dir="rtl">

---

## 🔄 Auto-Refresh Strategies

### 1. Periodic Refresh

</div>

```dart
@riverpod
Future<Stock> stockPrice(StockPriceRef ref, String symbol) async {
  // Refresh every 30 seconds
  final timer = Timer.periodic(
    const Duration(seconds: 30),
    (_) => ref.invalidateSelf(),
  );

  // Cancel timer when provider is disposed
  ref.onDispose(timer.cancel);

  return await api.getStockPrice(symbol);
}

// Usage
class StockWidget extends ConsumerWidget {
  final String symbol;

  const StockWidget(this.symbol, {super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final stockAsync = ref.watch(stockPriceProvider(symbol));

    return switch (stockAsync) {
      AsyncData(:final value) => Text('\$${value.price}'),
      _ => const CircularProgressIndicator(),
    };
  }
}
// Auto-refreshes every 30 seconds while on screen!
```

<div dir="rtl">

### 2. Focus-Based Refresh

استخدم `ref.listen` مع `AppLifecycleState`.

</div>

```dart
// App lifecycle provider
@riverpod
AppLifecycleState appLifecycle(AppLifecycleRef ref) {
  final observer = _AppLifecycleObserver((state) {
    ref.state = state;
  });

  final binding = WidgetsBinding.instance;
  binding.addObserver(observer);

  ref.onDispose(() => binding.removeObserver(observer));

  return AppLifecycleState.resumed;
}

class _AppLifecycleObserver extends WidgetsBindingObserver {
  _AppLifecycleObserver(this.onStateChange);

  final void Function(AppLifecycleState) onStateChange;

  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    onStateChange(state);
  }
}

// Usage - Refresh when app resumes
class DataScreen extends ConsumerWidget {
  const DataScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final dataAsync = ref.watch(dataProvider);

    // Listen to app lifecycle
    ref.listen(appLifecycleProvider, (previous, next) {
      if (next == AppLifecycleState.resumed) {
        // App came to foreground - refresh data
        ref.invalidate(dataProvider);
      }
    });

    return switch (dataAsync) {
      AsyncData(:final value) => DataView(value),
      _ => const LoadingScreen(),
    };
  }
}
```

<div dir="rtl">

### 3. Connectivity-Based Refresh

</div>

```dart
// Connectivity provider (using connectivity_plus package)
@riverpod
Stream<bool> isOnline(IsOnlineRef ref) async* {
  final connectivity = Connectivity();

  await for (final result in connectivity.onConnectivityChanged) {
    yield result != ConnectivityResult.none;
  }
}

// Usage - Refresh when back online
class DataScreen extends ConsumerWidget {
  const DataScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final dataAsync = ref.watch(dataProvider);

    // Listen to connectivity
    ref.listen(isOnlineProvider, (previous, next) {
      // Was offline, now online
      if (previous?.value == false && next.value == true) {
        ref.invalidate(dataProvider);
      }
    });

    return switch (dataAsync) {
      AsyncData(:final value) => DataView(value),
      AsyncError() => const Text('Offline'),
      _ => const LoadingScreen(),
    };
  }
}
```

<div dir="rtl">

---

## 🔧 Lifecycle Methods

### ref.onDispose()

يتنادى لما الـ provider يتدمر.

</div>

```dart
@riverpod
Future<Data> data(DataRef ref) async {
  final subscription = someStream.listen((event) {
    // Handle event
  });

  // Clean up when disposed
  ref.onDispose(() {
    subscription.cancel();
    print('Provider disposed');
  });

  return await api.getData();
}

// Real example - WebSocket connection
@riverpod
class ChatMessages extends _$ChatMessages {
  WebSocket? _socket;

  @override
  Future<List<Message>> build() async {
    _socket = await WebSocket.connect('wss://chat.example.com');

    _socket!.listen((data) {
      // Update state with new messages
      final message = Message.fromJson(data);
      state = state.whenData((messages) => [...messages, message]);
    });

    // Close socket when disposed
    ref.onDispose(() {
      _socket?.close();
      print('WebSocket closed');
    });

    return [];
  }
}
```

<div dir="rtl">

### ref.onCancel()

يتنادى لما آخر listener يتشال (قبل ما يتدمر).

</div>

```dart
@riverpod
Future<Data> data(DataRef ref) async {
  print('build() called');

  ref.onCancel(() {
    print('No more listeners - will dispose soon');
  });

  ref.onDispose(() {
    print('Disposed');
  });

  return await api.getData();
}

// Timeline:
// 1. Widget watches provider → build() called
// 2. Widget removed → onCancel() called
// 3. After short delay → onDispose() called
```

<div dir="rtl">

### ref.onResume()

يتنادى لما listener يرجع تاني (بعد cancel).

</div>

```dart
@riverpod
Future<Data> data(DataRef ref) async {
  final link = ref.keepAlive();

  ref.onCancel(() {
    print('Last listener removed');
    // Start 5-second timer
    Timer(const Duration(seconds: 5), link.close);
  });

  ref.onResume(() {
    print('Listener added again - keep alive');
    // Cancel timer
    link.close();
  });

  return await api.getData();
}

// Scenario:
// 1. User on screen A → Provider active
// 2. User navigates to screen B → onCancel(), timer starts
// 3. User returns to screen A within 5 seconds → onResume(), timer cancelled
// 4. Provider still alive with cached data!
```

<div dir="rtl">

---

## 🎯 Caching Strategies Examples

### Strategy 1: Short-Lived Cache (30 seconds)

مناسب لـ real-time data.

</div>

```dart
@riverpod
Future<List<Message>> messages(MessagesRef ref) async {
  // Cache for 30 seconds only
  final link = ref.keepAlive();
  Timer(const Duration(seconds: 30), link.close);

  return await api.getMessages();
}
```

<div dir="rtl">

### Strategy 2: Long-Lived Cache (1 hour)

مناسب لـ rarely-changing data.

</div>

```dart
@riverpod
Future<AppConfig> appConfig(AppConfigRef ref) async {
  // Cache for 1 hour
  final link = ref.keepAlive();
  Timer(const Duration(hours: 1), link.close);

  return await api.getConfig();
}
```

<div dir="rtl">

### Strategy 3: Conditional Cache

Cache الـ successful results بس.

</div>

```dart
@riverpod
Future<User> user(UserRef ref, String userId) async {
  try {
    final user = await api.getUser(userId);

    // Success - cache for 5 minutes
    final link = ref.keepAlive();
    Timer(const Duration(minutes: 5), link.close);

    return user;
  } catch (error) {
    // Error - don't cache
    rethrow;
  }
}
```

<div dir="rtl">

### Strategy 4: Smart Cache with onResume

</div>

```dart
@riverpod
Future<List<Product>> products(ProductsRef ref) async {
  Timer? timer;

  // Keep alive initially
  final link = ref.keepAlive();

  ref.onCancel(() {
    // Start disposal timer after 2 minutes
    timer = Timer(const Duration(minutes: 2), link.close);
  });

  ref.onResume(() {
    // Cancel disposal if user comes back
    timer?.cancel();
  });

  ref.onDispose(() {
    timer?.cancel();
  });

  return await api.getProducts();
}
```

<div dir="rtl">

### Strategy 5: Cache-First with Background Refresh

</div>

```dart
@riverpod
class Products extends _$Products {
  @override
  Future<List<Product>> build() async {
    // Keep alive with 5 minute cache
    final link = ref.keepAlive();
    Timer(const Duration(minutes: 5), link.close);

    // Load initial data
    final products = await api.getProducts();

    // Background refresh after 30 seconds
    Timer(const Duration(seconds: 30), () {
      _backgroundRefresh();
    });

    return products;
  }

  Future<void> _backgroundRefresh() async {
    try {
      final products = await api.getProducts();
      state = AsyncValue.data(products);
    } catch (error) {
      // Silently fail - keep showing old data
      print('Background refresh failed: $error');
    }
  }
}
```

<div dir="rtl">

---

## 📋 Best Practices

### 1. استخدم invalidate بدل refresh

</div>

```dart
// ✅ GOOD - Lighter on performance
ElevatedButton(
  onPressed: () => ref.invalidate(dataProvider),
  child: const Text('Refresh'),
)

// ❌ AVOID - Unless you need the value immediately
ElevatedButton(
  onPressed: () => ref.refresh(dataProvider),
  child: const Text('Refresh'),
)
```

<div dir="rtl">

### 2. استخدم @Riverpod(keepAlive: true) بدل ref.keepAlive()

</div>

```dart
// ✅ GOOD - Better linter support
@Riverpod(keepAlive: true)
Future<AppConfig> appConfig(AppConfigRef ref) async {
  return await api.getConfig();
}

// ❌ AVOID
@riverpod
Future<AppConfig> appConfig(AppConfigRef ref) async {
  ref.keepAlive();
  return await api.getConfig();
}
```

<div dir="rtl">

### 3. Cache الـ Successful Results بس

</div>

```dart
// ✅ GOOD
@riverpod
Future<Data> data(DataRef ref) async {
  final data = await api.getData();

  // Only cache if successful
  if (data != null) {
    ref.cacheFor(const Duration(minutes: 5));
  }

  return data;
}
```

<div dir="rtl">

### 4. استخدم onDispose للـ Cleanup

</div>

```dart
// ✅ GOOD - Always clean up resources
@riverpod
Future<Data> data(DataRef ref) async {
  final timer = Timer.periodic(
    const Duration(seconds: 30),
    (_) => ref.invalidateSelf(),
  );

  ref.onDispose(timer.cancel);

  return await api.getData();
}
```

<div dir="rtl">

### 5. اختار Cache Duration المناسب

| Data Type | Cache Duration |
|-----------|----------------|
| **Real-time** (stock prices) | 10-30 seconds |
| **Frequently updated** (messages) | 1-2 minutes |
| **Normal data** (products) | 5-10 minutes |
| **Rarely changes** (config) | 1+ hour |
| **Static** (app settings) | Forever (keepAlive: true) |

### 6. استخدم ref.listen للـ Side Effects

</div>

```dart
// ✅ GOOD - Reactive refresh
ref.listen(appLifecycleProvider, (previous, next) {
  if (next == AppLifecycleState.resumed) {
    ref.invalidate(dataProvider);
  }
});
```

<div dir="rtl">

---

## 🎓 مثال عملي كامل: Smart Caching System

</div>

```dart
import 'dart:async';
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'smart_cache.g.dart';

// Cache extension
extension CacheExtension on Ref {
  void cacheFor(Duration duration) {
    final link = keepAlive();
    Timer(duration, link.close);
  }

  void smartCache({
    Duration cacheTime = const Duration(minutes: 5),
    Duration graceTime = const Duration(seconds: 30),
  }) {
    final link = keepAlive();
    Timer? disposeTimer;

    onCancel(() {
      // Start grace period timer
      disposeTimer = Timer(graceTime, () {
        // After grace period, start cache timer
        Timer(cacheTime, link.close);
      });
    });

    onResume(() {
      // Cancel grace period if resumed
      disposeTimer?.cancel();
    });

    onDispose(() {
      disposeTimer?.cancel();
    });
  }
}

// App lifecycle provider
@riverpod
class AppLifecycle extends _$AppLifecycle {
  @override
  AppLifecycleState build() {
    final observer = _LifecycleObserver((state) => this.state = state);

    final binding = WidgetsBinding.instance;
    binding.addObserver(observer);

    ref.onDispose(() => binding.removeObserver(observer));

    return AppLifecycleState.resumed;
  }
}

class _LifecycleObserver extends WidgetsBindingObserver {
  _LifecycleObserver(this.onStateChange);
  final void Function(AppLifecycleState) onStateChange;

  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    onStateChange(state);
  }
}

// API
class Product {
  final String id;
  final String name;
  final double price;

  Product({
    required this.id,
    required this.name,
    required this.price,
  });
}

class ProductsApi {
  Future<List<Product>> getProducts() async {
    print('[API] Fetching products...');
    await Future.delayed(const Duration(seconds: 2));
    return List.generate(
      10,
      (i) => Product(
        id: 'product_$i',
        name: 'Product $i',
        price: 10.0 + i * 5.0,
      ),
    );
  }
}

final productsApiProvider = Provider((ref) => ProductsApi());

// Products provider with smart caching
@riverpod
class Products extends _$Products {
  @override
  Future<List<Product>> build() async {
    print('[Provider] Building products...');

    // Smart cache:
    // - 30s grace period (keeps data if user navigates back quickly)
    // - 5min cache after grace period
    ref.smartCache(
      graceTime: const Duration(seconds: 30),
      cacheTime: const Duration(minutes: 5),
    );

    // Periodic refresh every 2 minutes (while active)
    final timer = Timer.periodic(
      const Duration(minutes: 2),
      (_) {
        print('[Provider] Auto-refreshing products...');
        ref.invalidateSelf();
      },
    );

    ref.onDispose(() {
      print('[Provider] Disposed');
      timer.cancel();
    });

    final api = ref.watch(productsApiProvider);
    return await api.getProducts();
  }

  Future<void> refresh() async {
    print('[Provider] Manual refresh');
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      final api = ref.read(productsApiProvider);
      return await api.getProducts();
    });
  }
}

// UI
class ProductsScreen extends ConsumerWidget {
  const ProductsScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final productsAsync = ref.watch(productsProvider);

    // Refresh when app resumes
    ref.listen(appLifecycleProvider, (previous, next) {
      if (previous == AppLifecycleState.paused &&
          next == AppLifecycleState.resumed) {
        print('[UI] App resumed - refreshing');
        ref.invalidate(productsProvider);
      }
    });

    return Scaffold(
      appBar: AppBar(
        title: const Text('Products (Smart Cache)'),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: () => ref.read(productsProvider.notifier).refresh(),
          ),
        ],
      ),
      body: Column(
        children: [
          if (productsAsync.isRefreshing)
            const LinearProgressIndicator(),

          Expanded(
            child: RefreshIndicator(
              onRefresh: () => ref.read(productsProvider.notifier).refresh(),
              child: switch (productsAsync) {
                AsyncData(:final value) => ListView.builder(
                    itemCount: value.length,
                    itemBuilder: (context, index) {
                      final product = value[index];
                      return ListTile(
                        title: Text(product.name),
                        subtitle: Text('\$${product.price.toStringAsFixed(2)}'),
                      );
                    },
                  ),

                AsyncError(:final error) => ListView(
                    children: [
                      const SizedBox(height: 100),
                      Center(child: Text('Error: $error')),
                      const SizedBox(height: 16),
                      Center(
                        child: ElevatedButton(
                          onPressed: () => ref.invalidate(productsProvider),
                          child: const Text('Retry'),
                        ),
                      ),
                    ],
                  ),

                _ => const Center(child: CircularProgressIndicator()),
              },
            ),
          ),
        ],
      ),
    );
  }
}

// Navigation example
class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Home')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (_) => const ProductsScreen(),
                  ),
                );
              },
              child: const Text('Go to Products'),
            ),
            const SizedBox(height: 16),
            const Text(
              'Products will:\n'
              '- Cache for 5 minutes\n'
              '- Keep data for 30s after navigation\n'
              '- Auto-refresh every 2 minutes\n'
              '- Refresh on app resume',
              textAlign: TextAlign.center,
            ),
          ],
        ),
      ),
    );
  }
}
```

<div dir="rtl">

**Features في المثال:**
- ✅ Smart caching مع grace period
- ✅ Auto-refresh كل دقيقتين
- ✅ Refresh لما التطبيق يرجع (app resume)
- ✅ Manual refresh
- ✅ Pull-to-refresh
- ✅ Proper cleanup

---

## 🎉 الخلاصة

### Refresh Methods

| Method | When | Use Case |
|--------|------|----------|
| **ref.invalidate()** | Mark invalid, rebuild later | ⭐ Recommended |
| **ref.refresh()** | Invalidate + read immediately | Need value now |
| **ref.refresh().future** | Wait for async result | Need to await |

### Caching Strategies

| Strategy | Implementation | Best For |
|----------|---------------|----------|
| **No cache** | Default (auto dispose) | Rarely used data |
| **Forever** | `@Riverpod(keepAlive: true)` | App config |
| **Time-based** | `ref.cacheFor(duration)` | Most data |
| **Smart cache** | `onCancel` + `onResume` | Complex scenarios |
| **Conditional** | Cache only on success | API calls |

### الـ Golden Rules

1. ✅ استخدم `ref.invalidate()` مش `ref.refresh()`
2. ✅ استخدم `@Riverpod(keepAlive: true)` مش `ref.keepAlive()`
3. ✅ Cache الـ successful results بس
4. ✅ استخدم `onDispose` للـ cleanup
5. ✅ اختار cache duration مناسب للـ data
6. ✅ استخدم `ref.listen` للـ side effects

---

## 🔗 الخطوة الجاية

مبروك! خلصت **Section 07: Async Data Handling**! 🎉

دلوقتي انت فاهم:
- ✅ AsyncValue بكل تفاصيله
- ✅ Pattern Matching مع Dart 3
- ✅ Error Handling محترف
- ✅ Loading States و UI Patterns
- ✅ Refresh و Caching Strategies

جاهز للـ Section الجاي؟ يلا نكمل! 🚀

---

## 📚 المصادر

- [Riverpod Data Caching and Providers Lifecycle | Code with Andrea](https://codewithandrea.com/articles/flutter-riverpod-data-caching-providers-lifecycle/)
- [Automatic disposal | Riverpod](https://riverpod.dev/docs/concepts2/auto_dispose)
- [using ref.keepAlive() vs @Riverpod(keepAlive: true) | Discussion](https://github.com/rrousselGit/riverpod/discussions/3584)
- [FAQ | Riverpod](https://riverpod.dev/docs/root/faq)

</div>
