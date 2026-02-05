<div dir="rtl">

# AutoDispose Modifier - إدارة الذاكرة التلقائية

**المستوى**: 🟡 متوسط

## 📌 المفاهيم الأساسية

في الملف ده هنتكلم عن:
- إيه هو AutoDispose
- إزاي بيشتغل
- Memory management
- متى تستخدم AutoDispose vs KeepAlive
- Best practices

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تفهم AutoDispose بعمق
- تختار بين AutoDispose و KeepAlive
- تدير الـ memory بكفاءة
- تتجنب memory leaks
- تحسن الـ performance

---

## 🔍 إيه هو AutoDispose؟

ال **AutoDispose** هو mechanism بيتأكد إن الـ provider بيتدمر تلقائياً لما ما يبقاش محتاج.

</div>

```dart
// AutoDispose by default in Riverpod 3
class CounterNotifier extends Notifier<int> {
  @override
  int build() {
    print('Counter created');

    ref.onDispose(() {
      print('Counter disposed');
    });

    return 0;
  }

  void increment() => state++;
}

final counterProvider = NotifierProvider.autoDispose<CounterNotifier, int>(
  () => CounterNotifier(),
);

// Widget lifecycle
class CounterPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);
    // Prints: "Counter created"

    return Text('$count');
  }
  // When CounterPage is removed from widget tree:
  // Prints: "Counter disposed"
}
```

<div dir="rtl">

**الفوائد:**
- ✅ Automatic memory cleanup
- ✅ Prevents memory leaks
- ✅ No manual disposal needed
- ✅ Default في Riverpod 3

---

## 🎨 كيف يعمل AutoDispose؟

### دورة الحياة

</div>

```dart
class DataProviderNotifier extends AsyncNotifier<Data> {
  Timer? _timer;

  @override
  Future<Data> build() async {
    print('1. Provider created');

    // Setup
    _timer = Timer.periodic(Duration(seconds: 1), (timer) {
      print('Polling...');
    });

    // Called when last listener is removed
    ref.onCancel(() {
      print('2. Last listener removed');
      // Provider is now "paused"
    });

    // Called when a listener is added after onCancel
    ref.onResume(() {
      print('3. Listener added again');
    });

    // Called when provider is disposed permanently
    ref.onDispose(() {
      print('4. Provider disposed');
      _timer?.cancel();
    });

    return await fetchData();
  }

  Future<Data> fetchData() async {
    return Data();
  }
}

final dataProvider = AsyncNotifierProvider.autoDispose<DataProviderNotifier, Data>(
  () => DataProviderNotifier(),
);
```

<div dir="rtl">

**Timeline:**

</div>

```
Widget appears → Provider created (1)
    ↓
Widget present → Provider active
    ↓
Widget removed → onCancel (2)
    ↓
[Brief pause]
    ↓
Widget added again? → onResume (3) → Active again
    ↓
Not used for too long → onDispose (4) → Destroyed
```

<div dir="rtl">

---

## ⚙️ AutoDispose vs KeepAlive

### AutoDispose (Default)

**متى يُستخدم:**
- UI state
- Temporary data
- Frequently changing data
- Data tied to specific screens

</div>

```dart
// Perfect for AutoDispose
class FormStateNotifier extends Notifier<FormData> {
  @override
  FormData build() {
    // No keepAlive - disposed when form closes
    return FormData.empty();
  }

  void updateField(String field, String value) {
    state = state.copyWith(field: value);
  }
}

final formStateProvider = NotifierProvider.autoDispose<FormStateNotifier, FormData>(
  () => FormStateNotifier(),
);

// Perfect for AutoDispose
class SearchResultsNotifier extends FamilyAsyncNotifier<List<Product>, String> {
  @override
  Future<List<Product>> build(String query) async {
    // Disposed when search screen closes
    return await api.search(query);
  }
}

final searchResultsProvider = AsyncNotifierProvider.autoDispose.family<SearchResultsNotifier, List<Product>, String>(
  () => SearchResultsNotifier(),
);
```

<div dir="rtl">

### KeepAlive

**متى يُستخدم:**
- Global app state
- Authentication state
- Configuration
- Expensive-to-recreate data

</div>

```dart
// Perfect for KeepAlive
class AuthStateNotifier extends AsyncNotifier<User?> {
  @override
  Future<User?> build() async {
    // Keep alive - needed throughout app
    ref.keepAlive();

    return await loadAuthState();
  }

  Future<User?> loadAuthState() async {
    return null;
  }

  Future<void> login(String email, String password) async {
    // Login logic
  }
}

final authStateProvider = AsyncNotifierProvider.autoDispose<AuthStateNotifier, User?>(
  () => AuthStateNotifier(),
);

// Perfect for KeepAlive
class AppConfigNotifier extends Notifier<Config> {
  @override
  Config build() {
    // Keep alive - configuration is always needed
    ref.keepAlive();

    return Config.load();
  }
}

final appConfigProvider = NotifierProvider.autoDispose<AppConfigNotifier, Config>(
  () => AppConfigNotifier(),
);
```

<div dir="rtl">

### جدول مقارنة

| Feature | AutoDispose | KeepAlive |
|---------|-------------|-----------|
| **Default في Riverpod 3** | ✅ نعم | ❌ لا |
| **Memory usage** | 🟢 Low | 🟡 Higher |
| **Performance** | 🟡 Recreates | 🟢 Cached |
| **Best for** | UI state | Global state |
| **Disposal** | Automatic | Manual |
| **When to use** | Temporary data | Persistent data |

---

## 🎮 التحكم في AutoDispose

### keepAlive() - تعطيل AutoDispose

</div>

```dart
class ApiDataNotifier extends AsyncNotifier<Data> {
  @override
  Future<Data> build() async {
    // Disable AutoDispose permanently
    ref.keepAlive();

    return await fetchData();
  }

  Future<Data> fetchData() async {
    return Data();
  }
}

final apiDataProvider = AsyncNotifierProvider.autoDispose<ApiDataNotifier, Data>(
  () => ApiDataNotifier(),
);
```

<div dir="rtl">

### Conditional KeepAlive

</div>

```dart
class SmartCacheNotifier extends AsyncNotifier<Data> {
  @override
  Future<Data> build() async {
    final data = await fetchData();

    // Keep alive only for important data
    if (data.isPriority) {
      ref.keepAlive();
    }

    return data;
  }

  Future<Data> fetchData() async {
    return Data();
  }
}

final smartCacheProvider = AsyncNotifierProvider.autoDispose<SmartCacheNotifier, Data>(
  () => SmartCacheNotifier(),
);
```

<div dir="rtl">

### Timed KeepAlive

</div>

```dart
class CachedProductsNotifier extends AsyncNotifier<List<Product>> {
  @override
  Future<List<Product>> build() async {
    // Keep alive for 5 minutes
    final link = ref.keepAlive();

    Timer(Duration(minutes: 5), () {
      link.close(); // Re-enable AutoDispose
    });

    return await api.getProducts();
  }
}

final cachedProductsProvider = AsyncNotifierProvider.autoDispose<CachedProductsNotifier, List<Product>>(
  () => CachedProductsNotifier(),
);
```

<div dir="rtl">

### Dynamic KeepAlive Control

</div>

```dart
class DynamicCacheNotifier extends AsyncNotifier<Data> {
  KeepAliveLink? _link;

  @override
  Future<Data> build() async {
    return await fetchData();
  }

  Future<Data> fetchData() async {
    return Data();
  }

  void enableCache() {
    _link = ref.keepAlive();
  }

  void disableCache() {
    _link?.close();
    _link = null;
  }
}

final dynamicCacheProvider = AsyncNotifierProvider.autoDispose<DynamicCacheNotifier, Data>(
  () => DynamicCacheNotifier(),
);
```

<div dir="rtl">

---

## 🧪 أمثلة عملية

### مثال 1: Shopping Cart (AutoDispose)

</div>

```dart
// Cart should be disposed when user leaves
class ShoppingCartNotifier extends Notifier<List<CartItem>> {
  @override
  List<CartItem> build() {
    print('Shopping cart created');

    ref.onDispose(() {
      print('Shopping cart disposed - clearing memory');
    });

    // AutoDispose by default - perfect for this use case
    return [];
  }

  void addItem(Product product) {
    state = [...state, CartItem.fromProduct(product)];
  }

  void removeItem(String itemId) {
    state = state.where((item) => item.id != itemId).toList();
  }

  void clear() {
    state = [];
  }
}

final shoppingCartProvider = NotifierProvider.autoDispose<ShoppingCartNotifier, List<CartItem>>(
  () => ShoppingCartNotifier(),
);

// Usage
class CartPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final cart = ref.watch(shoppingCartProvider);
    // Cart created when page opens
    // Cart disposed when user navigates away
    // Memory cleaned up automatically

    return ListView.builder(
      itemCount: cart.length,
      itemBuilder: (context, index) {
        return CartItemTile(cart[index]);
      },
    );
  }
}
```

<div dir="rtl">

### مثال 2: Auth State (KeepAlive)

</div>

```dart
// Auth must persist across the app
class AuthStateNotifier2 extends AsyncNotifier<User?> {
  @override
  Future<User?> build() async {
    print('Auth state initialized');

    // Keep alive - we ALWAYS need auth state
    ref.keepAlive();

    ref.onDispose(() {
      print('Auth state disposed - app is closing');
    });

    // Load from secure storage
    return await secureStorage.getUser();
  }

  Future<void> login(String email, String password) async {
    state = const AsyncLoading();

    try {
      final user = await api.login(email, password);
      await secureStorage.saveUser(user);
      state = AsyncData(user);
    } catch (e, s) {
      state = AsyncError(e, s);
    }
  }

  Future<void> logout() async {
    await secureStorage.clearUser();
    state = const AsyncData(null);
  }
}

final authStateProvider2 = AsyncNotifierProvider.autoDispose<AuthStateNotifier2, User?>(
  () => AuthStateNotifier2(),
);

// Auth persists even when no widgets are watching
class AnyPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final authState = ref.watch(authStateProvider);
    // Auth state is always available
    return Container();
  }
}
```

<div dir="rtl">

### مثال 3: API Cache (Timed)

</div>

```dart
// Cache API data for limited time
class ProductsCatalogNotifier extends AsyncNotifier<List<Product>> {
  @override
  Future<List<Product>> build() async {
    print('Fetching products from API');

    // Keep alive for 10 minutes
    final link = ref.keepAlive();

    Timer(Duration(minutes: 10), () {
      print('Cache expired - ready to dispose');
      link.close();
    });

    ref.onDispose(() {
      print('Products catalog disposed');
    });

    return await api.getProducts();
  }

  Future<void> refresh() async {
    state = const AsyncLoading();

    try {
      final products = await api.getProducts();
      state = AsyncData(products);
    } catch (e, s) {
      state = AsyncError(e, s);
    }
  }
}

final productsCatalogProvider = AsyncNotifierProvider.autoDispose<ProductsCatalogNotifier, List<Product>>(
  () => ProductsCatalogNotifier(),
);

// First access: Fetches from API
class Page1 extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final products = ref.watch(productsCatalogProvider);
    // API call happens here
    // Prints: "Fetching products from API"
    return ProductsList(products);
  }
}

// Second access (within 10 min): Uses cache
class Page2 extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final products = ref.watch(productsCatalogProvider);
    // No API call - uses cached data
    // No print
    return ProductsList(products);
  }
}

// After 10 minutes: Cache expired, will refetch on next access
```

<div dir="rtl">

### مثال 4: WebSocket (Smart Lifecycle)

</div>

```dart
// Manage WebSocket connection intelligently
class ChatConnectionNotifier extends FamilyStreamNotifier<ChatMessage, String> {
  IOWebSocketChannel? _channel;

  @override
  Stream<ChatMessage> build(String roomId) {
    print('Opening WebSocket for room: $roomId');

    _channel = IOWebSocketChannel.connect(
      'wss://chat.example.com/room/$roomId',
    );

    // Don't dispose immediately when last listener is removed
    ref.onCancel(() {
      print('Last listener removed - keeping connection for 30 sec');

      // Keep connection alive for 30 seconds
      // User might come back quickly
      final link = ref.keepAlive();

      Timer(Duration(seconds: 30), () {
        print('30 seconds passed - closing connection');
        link.close();
      });
    });

    ref.onResume(() {
      print('Listener added again - connection still open');
    });

    ref.onDispose(() {
      print('Closing WebSocket for room: $roomId');
      _channel?.sink.close();
    });

    return _channel!.stream.map((data) {
      return ChatMessage.fromJson(jsonDecode(data));
    });
  }

  void sendMessage(String message) {
    _channel?.sink.add(jsonEncode({
      'type': 'message',
      'content': message,
    }));
  }
}

final chatConnectionProvider = StreamNotifierProvider.autoDispose.family<ChatConnectionNotifier, ChatMessage, String>(
  () => ChatConnectionNotifier(),
);
```

<div dir="rtl">

### مثال 5: Polling Provider

</div>

```dart
// Polls API every 5 seconds, stops when not needed
class LiveDataNotifier extends AsyncNotifier<Data> {
  Timer? _timer;

  @override
  Future<Data> build() async {
    print('Starting polling');

    _startPolling();

    ref.onCancel(() {
      print('Pausing polling');
      _timer?.cancel();
    });

    ref.onResume(() {
      print('Resuming polling');
      _startPolling();
    });

    ref.onDispose(() {
      print('Stopping polling permanently');
      _timer?.cancel();
    });

    return await _fetchData();
  }

  void _startPolling() {
    _timer = Timer.periodic(Duration(seconds: 5), (timer) async {
      print('Polling...');

      try {
        final data = await _fetchData();
        state = AsyncData(data);
      } catch (e, s) {
        state = AsyncError(e, s);
      }
    });
  }

  Future<Data> _fetchData() async {
    return await api.getData();
  }
}

final liveDataProvider = AsyncNotifierProvider.autoDispose<LiveDataNotifier, Data>(
  () => LiveDataNotifier(),
);
```

<div dir="rtl">

---

## 🎯 Patterns المتقدمة

### Pattern 1: Smart Caching Strategy

</div>

```dart
class SmartCacheNotifier2 extends FamilyAsyncNotifier<ExpensiveData, String> {
  @override
  Future<ExpensiveData> build(String id) async {
    final data = await _fetchExpensiveData(id);

    // Keep alive based on data importance and freshness
    if (data.isImportant && data.isFresh) {
      final link = ref.keepAlive();

      // But still set an expiration
      Timer(Duration(hours: 1), () {
        link.close();
      });
    } else if (data.isFresh) {
      // Keep for shorter time
      final link = ref.keepAlive();
      Timer(Duration(minutes: 10), () {
        link.close();
      });
    }
    // Else: AutoDispose (stale or unimportant data)

    return data;
  }

  Future<ExpensiveData> _fetchExpensiveData(String id) async {
    await Future.delayed(Duration(seconds: 2)); // Expensive operation
    return ExpensiveData(id: id);
  }
}

final smartCacheProvider2 = AsyncNotifierProvider.autoDispose.family<SmartCacheNotifier2, ExpensiveData, String>(
  () => SmartCacheNotifier2(),
);
```

<div dir="rtl">

### Pattern 2: User Preference Based

</div>

```dart
class AdaptiveCacheNotifier extends AsyncNotifier<Data> {
  @override
  Future<Data> build() async {
    // Check user preference
    final settings = ref.watch(appSettingsProvider);

    final data = await fetchData();

    if (settings.aggressiveCaching) {
      // User wants aggressive caching
      ref.keepAlive();
    } else if (settings.moderateCaching) {
      // Moderate caching
      final link = ref.keepAlive();
      Timer(Duration(minutes: 5), link.close);
    }
    // Else: Default AutoDispose

    return data;
  }

  Future<Data> fetchData() async {
    await Future.delayed(Duration(seconds: 1));
    return Data();
  }
}

final adaptiveCacheProvider = AsyncNotifierProvider.autoDispose<AdaptiveCacheNotifier, Data>(
  () => AdaptiveCacheNotifier(),
);
```

<div dir="rtl">

### Pattern 3: Network Aware

</div>

```dart
class NetworkAwareDataNotifier extends AsyncNotifier<Data> {
  @override
  Future<Data> build() async {
    final connectivity = ref.watch(connectivityProvider);

    final data = await fetchData();

    // Keep data longer when offline
    if (!connectivity.isOnline) {
      print('Offline - keeping data cached');
      ref.keepAlive();
    } else {
      // Online - shorter cache
      final link = ref.keepAlive();
      Timer(Duration(minutes: 2), link.close);
    }

    // Listen to connectivity changes
    ref.listen(connectivityProvider, (previous, next) {
      if (!next.isOnline) {
        // Going offline - prevent disposal
        ref.keepAlive();
      }
    });

    return data;
  }

  Future<Data> fetchData() async {
    await Future.delayed(Duration(seconds: 1));
    return Data();
  }
}

final networkAwareDataProvider = AsyncNotifierProvider.autoDispose<NetworkAwareDataNotifier, Data>(
  () => NetworkAwareDataNotifier(),
);
```

<div dir="rtl">

---

## ⚠️ مشاكل شائعة

### مشكلة 1: Over-using KeepAlive

</div>

```dart
// ❌ WRONG - Keeping everything alive
class TemporaryFormStateNotifier extends Notifier<FormData> {
  @override
  FormData build() {
    ref.keepAlive(); // Unnecessary!
    return FormData.empty();
  }
}

final temporaryFormStateProviderWrong = NotifierProvider.autoDispose<TemporaryFormStateNotifier, FormData>(
  () => TemporaryFormStateNotifier(),
);

// ✅ CORRECT - Let AutoDispose work
class TemporaryFormStateNotifier2 extends Notifier<FormData> {
  @override
  FormData build() {
    // No keepAlive - disposed when form closes
    return FormData.empty();
  }
}

final temporaryFormStateProvider = NotifierProvider.autoDispose<TemporaryFormStateNotifier2, FormData>(
  () => TemporaryFormStateNotifier2(),
);
```

<div dir="rtl">

### مشكلة 2: Memory Leaks

</div>

```dart
// ❌ WRONG - Resources not cleaned
class LeakyProviderNotifier extends StreamNotifier<Data> {
  @override
  Stream<Data> build() {
    final controller = StreamController<Data>();

    Timer.periodic(Duration(seconds: 1), (timer) {
      controller.add(Data());
    });

    // Timer and controller never cleaned!
    return controller.stream;
  }
}

final leakyProvider = StreamNotifierProvider.autoDispose<LeakyProviderNotifier, Data>(
  () => LeakyProviderNotifier(),
);

// ✅ CORRECT - Proper cleanup
class CleanProviderNotifier extends StreamNotifier<Data> {
  Timer? _timer;
  StreamController<Data>? _controller;

  @override
  Stream<Data> build() {
    _controller = StreamController<Data>();

    _timer = Timer.periodic(Duration(seconds: 1), (timer) {
      _controller!.add(Data());
    });

    ref.onDispose(() {
      _timer?.cancel();
      _controller?.close();
    });

    return _controller!.stream;
  }
}

final cleanProvider = StreamNotifierProvider.autoDispose<CleanProviderNotifier, Data>(
  () => CleanProviderNotifier(),
);
```

<div dir="rtl">

### مشكلة 3: Forgetting to Close KeepAlive Link

</div>

```dart
// ❌ WRONG - Link never closed
class EternalCacheNotifier extends AsyncNotifier<Data> {
  @override
  Future<Data> build() async {
    final link = ref.keepAlive();
    // Link never closed - memory leak!

    return await fetchData();
  }

  Future<Data> fetchData() async {
    await Future.delayed(Duration(seconds: 1));
    return Data();
  }
}

final eternalCacheProvider = AsyncNotifierProvider.autoDispose<EternalCacheNotifier, Data>(
  () => EternalCacheNotifier(),
);

// ✅ CORRECT - Always close the link
class TimedCacheNotifier extends AsyncNotifier<Data> {
  @override
  Future<Data> build() async {
    final link = ref.keepAlive();

    // Always set a timeout
    Timer(Duration(minutes: 10), () {
      link.close();
    });

    return await fetchData();
  }

  Future<Data> fetchData() async {
    await Future.delayed(Duration(seconds: 1));
    return Data();
  }
}

final timedCacheProvider = AsyncNotifierProvider.autoDispose<TimedCacheNotifier, Data>(
  () => TimedCacheNotifier(),
);
```

<div dir="rtl">

---

## 💡 Best Practices

### 1. Default to AutoDispose

</div>

```dart
// ✅ Good: Let AutoDispose work for UI state
class PageStateNotifier extends Notifier<State> {
  @override
  State build() {
    // No keepAlive - perfect!
    return State.initial();
  }
}

final pageStateProvider = NotifierProvider.autoDispose<PageStateNotifier, State>(
  () => PageStateNotifier(),
);
```

<div dir="rtl">

### 2. KeepAlive للـ Global State فقط

</div>

```dart
// ✅ Good: KeepAlive for truly global state
class AppThemeNotifier extends Notifier<ThemeData> {
  @override
  ThemeData build() {
    ref.keepAlive(); // Makes sense here
    return ThemeData.light();
  }
}

final appThemeProvider = NotifierProvider.autoDispose<AppThemeNotifier, ThemeData>(
  () => AppThemeNotifier(),
);
```

<div dir="rtl">

### 3. دايماً استخدم Timed KeepAlive للـ Cache

</div>

```dart
// ✅ Good: Time-limited cache
class CachedDataNotifier extends AsyncNotifier<Data> {
  @override
  Future<Data> build() async {
    final link = ref.keepAlive();

    // Always set expiration
    Timer(Duration(minutes: 5), link.close);

    return await fetchData();
  }

  Future<Data> fetchData() async {
    await Future.delayed(Duration(seconds: 1));
    return Data();
  }
}

final cachedDataProvider = AsyncNotifierProvider.autoDispose<CachedDataNotifier, Data>(
  () => CachedDataNotifier(),
);
```

<div dir="rtl">

### 4. نضف الـ Resources في onDispose

</div>

```dart
// ✅ Good: Always cleanup
class ResourceManagerNotifier extends Notifier<Data> {
  late final subscription;

  @override
  Data build() {
    subscription = stream.listen((data) {
      // Handle data
    });

    ref.onDispose(() {
      subscription.cancel(); // Cleanup!
    });

    return Data.initial();
  }

  Stream<Data> get stream => Stream.periodic(Duration(seconds: 1), (_) => Data());
}

final resourceManagerProvider = NotifierProvider.autoDispose<ResourceManagerNotifier, Data>(
  () => ResourceManagerNotifier(),
);
```

<div dir="rtl">

---

## 📊 Decision Tree

</div>

```
هل الـ data ده global state؟
├─ نعم
│  └─ هل محتاج يفضل موجود طول الوقت؟
│     ├─ نعم → ref.keepAlive()
│     └─ لا → ref.keepAlive() + Timer
└─ لا
   └─ هل الـ data expensive لإعادة إنشائه؟
      ├─ نعم → Timed keepAlive
      └─ لا → AutoDispose (default) ✅
```

<div dir="rtl">

---

## 📝 ملخص

**AutoDispose:**
- ✅ Default في Riverpod 3
- ✅ Automatic memory management
- ✅ Perfect للـ UI state
- ✅ No manual cleanup needed

**KeepAlive:**
- Use sparingly
- Global state only
- Always set expiration (except for truly permanent data)
- Don't forget to close the link

**Best Practice:**
- Default: AutoDispose ✅
- Global/Auth: KeepAlive permanent
- Cache: Timed KeepAlive
- Always cleanup resources in onDispose

---

## 🔗 الخطوة الجاية

في الملف الجاي هنتكلم عن:
- **Combining Modifiers** (Family + AutoDispose)
- Advanced patterns
- Complex use cases

---

## 📚 المصادر

- [AutoDispose](https://riverpod.dev/docs/concepts/modifiers/auto_dispose)
- [Memory Management](https://riverpod.dev/docs/concepts/provider_lifecycle)

</div>
