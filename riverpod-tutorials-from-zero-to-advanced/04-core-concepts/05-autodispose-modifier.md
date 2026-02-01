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

**AutoDispose** هو mechanism بيتأكد إن الـ provider بيتدمر تلقائياً لما ما يبقاش محتاج.

</div>

```dart
// AutoDispose by default in Riverpod 3
@riverpod
class Counter extends _$Counter {
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
@riverpod
class DataProvider extends _$DataProvider {
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
}
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
@riverpod
class FormState extends _$FormState {
  @override
  FormData build() {
    // No keepAlive - disposed when form closes
    return FormData.empty();
  }

  void updateField(String field, String value) {
    state = state.copyWith(field: value);
  }
}

// Perfect for AutoDispose
@riverpod
class SearchResults extends _$SearchResults {
  @override
  Future<List<Product>> build(String query) async {
    // Disposed when search screen closes
    return await api.search(query);
  }
}
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
@riverpod
class AuthState extends _$AuthState {
  @override
  Future<User?> build() async {
    // Keep alive - needed throughout app
    ref.keepAlive();

    return await loadAuthState();
  }

  Future<void> login(String email, String password) async {
    // Login logic
  }
}

// Perfect for KeepAlive
@riverpod
class AppConfig extends _$AppConfig {
  @override
  Config build() {
    // Keep alive - configuration is always needed
    ref.keepAlive();

    return Config.load();
  }
}
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
@riverpod
class ApiData extends _$ApiData {
  @override
  Future<Data> build() async {
    // Disable AutoDispose permanently
    ref.keepAlive();

    return await fetchData();
  }
}
```

<div dir="rtl">

### Conditional KeepAlive

</div>

```dart
@riverpod
class SmartCache extends _$SmartCache {
  @override
  Future<Data> build() async {
    final data = await fetchData();

    // Keep alive only for important data
    if (data.isPriority) {
      ref.keepAlive();
    }

    return data;
  }
}
```

<div dir="rtl">

### Timed KeepAlive

</div>

```dart
@riverpod
class CachedProducts extends _$CachedProducts {
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
```

<div dir="rtl">

### Dynamic KeepAlive Control

</div>

```dart
@riverpod
class DynamicCache extends _$DynamicCache {
  KeepAliveLink? _link;

  @override
  Future<Data> build() async {
    return await fetchData();
  }

  void enableCache() {
    _link = ref.keepAlive();
  }

  void disableCache() {
    _link?.close();
    _link = null;
  }
}
```

<div dir="rtl">

---

## 🧪 أمثلة عملية

### مثال 1: Shopping Cart (AutoDispose)

</div>

```dart
// Cart should be disposed when user leaves
@riverpod
class ShoppingCart extends _$ShoppingCart {
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
@riverpod
class AuthState extends _$AuthState {
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
@riverpod
class ProductsCatalog extends _$ProductsCatalog {
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
@riverpod
class ChatConnection extends _$ChatConnection {
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
```

<div dir="rtl">

### مثال 5: Polling Provider

</div>

```dart
// Polls API every 5 seconds, stops when not needed
@riverpod
class LiveData extends _$LiveData {
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
```

<div dir="rtl">

---

## 🎯 Patterns المتقدمة

### Pattern 1: Smart Caching Strategy

</div>

```dart
@riverpod
class SmartCache extends _$SmartCache {
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
```

<div dir="rtl">

### Pattern 2: User Preference Based

</div>

```dart
@riverpod
class AdaptiveCache extends _$AdaptiveCache {
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
}
```

<div dir="rtl">

### Pattern 3: Network Aware

</div>

```dart
@riverpod
class NetworkAwareData extends _$NetworkAwareData {
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
}
```

<div dir="rtl">

---

## ⚠️ مشاكل شائعة

### مشكلة 1: Over-using KeepAlive

</div>

```dart
// ❌ WRONG - Keeping everything alive
@riverpod
class TemporaryFormState extends _$TemporaryFormState {
  @override
  FormData build() {
    ref.keepAlive(); // Unnecessary!
    return FormData.empty();
  }
}

// ✅ CORRECT - Let AutoDispose work
@riverpod
class TemporaryFormState extends _$TemporaryFormState {
  @override
  FormData build() {
    // No keepAlive - disposed when form closes
    return FormData.empty();
  }
}
```

<div dir="rtl">

### مشكلة 2: Memory Leaks

</div>

```dart
// ❌ WRONG - Resources not cleaned
@riverpod
class LeakyProvider extends _$LeakyProvider {
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

// ✅ CORRECT - Proper cleanup
@riverpod
class CleanProvider extends _$CleanProvider {
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
```

<div dir="rtl">

### مشكلة 3: Forgetting to Close KeepAlive Link

</div>

```dart
// ❌ WRONG - Link never closed
@riverpod
class EternalCache extends _$EternalCache {
  @override
  Future<Data> build() async {
    final link = ref.keepAlive();
    // Link never closed - memory leak!

    return await fetchData();
  }
}

// ✅ CORRECT - Always close the link
@riverpod
class TimedCache extends _$TimedCache {
  @override
  Future<Data> build() async {
    final link = ref.keepAlive();

    // Always set a timeout
    Timer(Duration(minutes: 10), () {
      link.close();
    });

    return await fetchData();
  }
}
```

<div dir="rtl">

---

## 💡 Best Practices

### 1. Default to AutoDispose

</div>

```dart
// ✅ Good: Let AutoDispose work for UI state
@riverpod
class PageState extends _$PageState {
  @override
  State build() {
    // No keepAlive - perfect!
    return State.initial();
  }
}
```

<div dir="rtl">

### 2. KeepAlive للـ Global State فقط

</div>

```dart
// ✅ Good: KeepAlive for truly global state
@riverpod
class AppTheme extends _$AppTheme {
  @override
  ThemeData build() {
    ref.keepAlive(); // Makes sense here
    return ThemeData.light();
  }
}
```

<div dir="rtl">

### 3. دايماً استخدم Timed KeepAlive للـ Cache

</div>

```dart
// ✅ Good: Time-limited cache
@riverpod
class CachedData extends _$CachedData {
  @override
  Future<Data> build() async {
    final link = ref.keepAlive();

    // Always set expiration
    Timer(Duration(minutes: 5), link.close);

    return await fetchData();
  }
}
```

<div dir="rtl">

### 4. نضف الـ Resources في onDispose

</div>

```dart
// ✅ Good: Always cleanup
@riverpod
class ResourceManager extends _$ResourceManager {
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
}
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
