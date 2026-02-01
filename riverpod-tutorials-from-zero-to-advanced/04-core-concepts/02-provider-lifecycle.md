<div dir="rtl">

# Provider Lifecycle - دورة حياة الـ Provider

**المستوى**: 🟡 متوسط

## 📌 المفاهيم الأساسية

في الملف ده هنتكلم عن:
- دورة حياة الـ provider من البداية للنهاية
- متى يتم إنشاء وتدمير الـ providers
- الفرق بين AutoDispose و KeepAlive
- إزاي تتحكم في الـ lifecycle
- Memory management best practices

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تفهم امتى بيتم create الـ provider
- تعرف امتى بيتدمر الـ provider
- تستخدم AutoDispose و KeepAlive صح
- تمنع memory leaks
- تحسن الـ performance

---

## 🔄 دورة الحياة الكاملة

الـ Provider بيعدي بـ 4 مراحل رئيسية:

</div>

```dart
class UserProfileNotifier extends AsyncNotifier<User> {
  @override
  Future<User> build() async {
    print('1. Created - Provider created for first time');

    ref.onCancel(() {
      print('2. Paused - All listeners removed (AutoDispose)');
    });

    ref.onResume(() {
      print('3. Resumed - New listener added');
    });

    ref.onDispose(() {
      print('4. Disposed - Provider destroyed permanently');
    });

    return await _fetchUser();
  }

  Future<User> _fetchUser() async {
    // Fetch user from API
    return User(id: '1', name: 'Ahmed');
  }
}

final userProfileProvider = AsyncNotifierProvider.autoDispose<UserProfileNotifier, User>(
  () => UserProfileNotifier(),
);
```

<div dir="rtl">

### المراحل بالتفصيل:

#### 1️⃣ Creation (الإنشاء)

**متى بيحصل:**
- أول مرة حد يعمل `ref.watch()` أو `ref.read()`
- Lazy creation - مش بيتعمل إلا لما حد يطلبه

</div>

```dart
// Provider not created yet
void main() {
  runApp(
    ProviderScope(
      child: MyApp(), // Providers still not created
    ),
  );
}

// NOW it gets created
class HomePage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Created HERE - first access
    final user = ref.watch(userProfileProvider);

    return Text('${user.name}');
  }
}
```

<div dir="rtl">

#### 2️⃣ Active (نشط)

**متى بيحصل:**
- لما في listeners بيستخدموه
- بيستقبل updates ويبعت notifications

</div>

```dart
// Active phase
class UserWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Provider is active - has listeners
    final user = ref.watch(userProfileProvider);

    return Column(
      children: [
        Text(user.name),
        ElevatedButton(
          onPressed: () {
            // Update while active
            ref.read(userProfileProvider.notifier).updateName('Sara');
          },
          child: Text('Update'),
        ),
      ],
    );
  }
}
```

<div dir="rtl">

#### 3️⃣ Paused (متوقف - AutoDispose فقط)

**متى بيحصل:**
- لما كل الـ listeners يتشالوا
- بس لو الـ provider عنده AutoDispose

</div>

```dart
class AutoDisposeCounterNotifier extends Notifier<int> {
  @override
  int build() {
    ref.onCancel(() {
      print('Paused - no more listeners');
      // You can choose to keep or dispose
    });

    return 0;
  }
}

final autoDisposeCounterProvider = NotifierProvider.autoDispose<AutoDisposeCounterNotifier, int>(
  () => AutoDisposeCounterNotifier(),
);

// Widget removed = provider paused
class CounterPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(autoDisposeCounterProvider);

    return Text('$count');
  }
  // When this widget is removed, provider is paused
}
```

<div dir="rtl">

#### 4️⃣ Disposed (متدمر)

**متى بيحصل:**
- بعد فترة من الـ pause (AutoDispose)
- لما تعمل manual `ref.invalidate()`
- لما الـ ProviderScope يتدمر

</div>

```dart
class DisposableResourceNotifier extends Notifier<int> {
  Timer? _timer;

  @override
  int build() {
    _timer = Timer.periodic(Duration(seconds: 1), (timer) {
      state++;
    });

    ref.onDispose(() {
      print('Disposed - cleaning up');
      _timer?.cancel();
      // Clean up resources here
    });

    return 0;
  }
}

final disposableResourceProvider = NotifierProvider.autoDispose<DisposableResourceNotifier, int>(
  () => DisposableResourceNotifier(),
);
```

<div dir="rtl">

---

## ⚙️ AutoDispose vs KeepAlive

في Riverpod 3، كل الـ providers بـ AutoDispose by default.

### AutoDispose (Default)

**السلوك:**
- بيتدمر لما ما يبقاش في listeners
- مناسب لمعظم الحالات
- بيوفر memory

</div>

```dart
// AutoDispose by default in Riverpod 3
class CounterNotifier extends Notifier<int> {
  @override
  int build() {
    // Will be disposed when no listeners
    return 0;
  }
}

final counterProvider = NotifierProvider.autoDispose<CounterNotifier, int>(
  () => CounterNotifier(),
);

// Usage
class CounterWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);
    // When widget is removed, provider is disposed
    return Text('$count');
  }
}
```

<div dir="rtl">

### KeepAlive (يفضل حي)

**السلوك:**
- بيفضل موجود حتى لو ما فيش listeners
- مناسب للـ cache
- بياخد memory أكتر

</div>

```dart
// Keep alive manually
class CachedUserNotifier extends AsyncNotifier<User> {
  @override
  Future<User> build() async {
    // Keep this data cached
    final link = ref.keepAlive();

    // Optional: dispose after timeout
    Timer(Duration(minutes: 5), () {
      link.close();
    });

    return await api.getUser();
  }
}

final cachedUserProvider = AsyncNotifierProvider.autoDispose<CachedUserNotifier, User>(
  () => CachedUserNotifier(),
);
```

<div dir="rtl">

### مقارنة شاملة

| Feature | AutoDispose | KeepAlive |
|---------|-------------|-----------|
| **Default في Riverpod 3** | ✅ نعم | ❌ لا |
| **Memory** | 🟢 Efficient | 🟡 Uses more |
| **Performance** | 🟡 Re-creates | 🟢 Cached |
| **Use Case** | UI state, temp data | Auth, config, cache |
| **Dispose** | تلقائي | يدوي |

---

## 🎮 التحكم في الـ Lifecycle

### 1. ref.keepAlive() - منع الـ Dispose

</div>

```dart
class AuthTokenNotifier extends AsyncNotifier<String> {
  @override
  Future<String> build() async {
    // Keep alive permanently
    ref.keepAlive();

    return await _fetchToken();
  }

  Future<String> _fetchToken() async {
    // Fetch from secure storage
    return 'token_123';
  }
}

final authTokenProvider = AsyncNotifierProvider.autoDispose<AuthTokenNotifier, String>(
  () => AuthTokenNotifier(),
);
```

<div dir="rtl">

### 2. Conditional KeepAlive

</div>

```dart
class SmartCacheNotifier extends AsyncNotifier<Data> {
  @override
  Future<Data> build() async {
    final data = await _fetchData();

    // Keep alive only if data is important
    if (data.isImportant) {
      ref.keepAlive();
    }

    return data;
  }

  Future<Data> _fetchData() async {
    // Fetch implementation
    return Data(isImportant: true);
  }
}

final smartCacheProvider = AsyncNotifierProvider.autoDispose<SmartCacheNotifier, Data>(
  () => SmartCacheNotifier(),
);
```

<div dir="rtl">

### 3. Timed KeepAlive

</div>

```dart
class TimedCacheNotifier extends AsyncNotifier<String> {
  @override
  Future<String> build() async {
    // Keep alive for 5 minutes
    final link = ref.keepAlive();

    Timer(Duration(minutes: 5), () {
      link.close(); // Now can be disposed
    });

    return await _fetchData();
  }

  Future<String> _fetchData() async {
    await Future.delayed(Duration(seconds: 1));
    return 'Cached data';
  }
}

final timedCacheProvider = AsyncNotifierProvider.autoDispose<TimedCacheNotifier, String>(
  () => TimedCacheNotifier(),
);
```

<div dir="rtl">

### 4. Manual Invalidation

</div>

```dart
class LogoutButton extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return ElevatedButton(
      onPressed: () {
        // Invalidate all auth-related providers
        ref.invalidate(authTokenProvider);
        ref.invalidate(userProfileProvider);
        ref.invalidate(userPreferencesProvider);

        // Navigate to login
        Navigator.pushReplacementNamed(context, '/login');
      },
      child: Text('Logout'),
    );
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
class ShoppingCartNotifier extends Notifier<List<CartItem>> {
  @override
  List<CartItem> build() {
    print('Cart created');

    ref.onDispose(() {
      print('Cart disposed - user left');
    });

    // AutoDispose by default
    return [];
  }

  void addItem(CartItem item) {
    state = [...state, item];
  }

  void removeItem(String itemId) {
    state = state.where((item) => item.id != itemId).toList();
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
    // Cart disposed when page closes

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
// Auth should persist across navigation
class AuthStateNotifier extends AsyncNotifier<User?> {
  @override
  Future<User?> build() async {
    // Keep alive - we always need auth state
    ref.keepAlive();

    print('Auth state created');

    ref.onDispose(() {
      print('Auth state disposed - app closing');
    });

    // Load from secure storage
    return await _loadAuthState();
  }

  Future<User?> _loadAuthState() async {
    // Load from storage
    return null;
  }

  Future<void> login(String email, String password) async {
    // Login logic
    state = AsyncData(User(id: '1', name: 'Ahmed'));
  }

  Future<void> logout() async {
    // Logout logic
    state = AsyncData(null);
  }
}

final authStateProvider = AsyncNotifierProvider.autoDispose<AuthStateNotifier, User?>(
  () => AuthStateNotifier(),
);

// Auth persists even when widgets are removed
class AnyPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final authState = ref.watch(authStateProvider);
    // Auth state stays alive
    return Container();
  }
}
```

<div dir="rtl">

### مثال 3: API Cache (Timed)

</div>

```dart
// Cache API responses for 10 minutes
class ProductsCacheNotifier extends AsyncNotifier<List<Product>> {
  @override
  Future<List<Product>> build() async {
    print('Fetching products from API');

    // Keep alive for 10 minutes
    final link = ref.keepAlive();

    Timer(Duration(minutes: 10), () {
      print('Cache expired');
      link.close();
    });

    ref.onDispose(() {
      print('Products cache disposed');
    });

    return await api.getProducts();
  }
}

final productsCacheProvider = AsyncNotifierProvider.autoDispose<ProductsCacheNotifier, List<Product>>(
  () => ProductsCacheNotifier(),
);

// First access: fetches from API
class Page1 extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final products = ref.watch(productsCacheProvider);
    // API call happens here
    return ProductsList(products);
  }
}

// Second access (within 10 min): uses cache
class Page2 extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final products = ref.watch(productsCacheProvider);
    // No API call - uses cached data
    return ProductsList(products);
  }
}
```

<div dir="rtl">

### مثال 4: WebSocket Connection

</div>

```dart
// Manage WebSocket lifecycle
class ChatWebSocketNotifier extends StreamNotifier<ChatMessage> {
  IOWebSocketChannel? _channel;

  @override
  Stream<ChatMessage> build() {
    print('Opening WebSocket');

    _channel = IOWebSocketChannel.connect('wss://chat.example.com');

    ref.onCancel(() {
      print('Last listener removed - keeping connection');
      // Don't close yet, might reconnect soon
    });

    ref.onResume(() {
      print('Listener added again - connection still open');
    });

    ref.onDispose(() {
      print('Closing WebSocket');
      _channel?.sink.close();
    });

    return _channel!.stream.map((data) {
      return ChatMessage.fromJson(jsonDecode(data));
    });
  }

  void sendMessage(String message) {
    _channel?.sink.add(message);
  }
}

final chatWebSocketProvider = StreamNotifierProvider.autoDispose<ChatWebSocketNotifier, ChatMessage>(
  () => ChatWebSocketNotifier(),
);
```

<div dir="rtl">

---

## 🎯 Lifecycle Hooks الكاملة

### onCancel

**متى يُستدعى:**
- لما آخر listener يتشال (AutoDispose فقط)
- قبل onDispose

</div>

```dart
class StreamProviderNotifier extends StreamNotifier<int> {
  @override
  Stream<int> build() {
    ref.onCancel(() {
      print('No more listeners');
      // Pause stream, stop polling, etc.
    });

    return Stream.periodic(Duration(seconds: 1), (i) => i);
  }
}

final streamProvider = StreamNotifierProvider.autoDispose<StreamProviderNotifier, int>(
  () => StreamProviderNotifier(),
);
```

<div dir="rtl">

### onResume

**متى يُستدعى:**
- لما listener جديد يتضاف بعد onCancel
- قبل ما يتدمر الـ provider

</div>

```dart
class PollingProviderNotifier extends Notifier<int> {
  Timer? _timer;

  @override
  int build() {
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

    return 0;
  }

  void _startPolling() {
    _timer = Timer.periodic(Duration(seconds: 5), (timer) {
      // Poll API
      state++;
    });
  }
}

final pollingProvider = NotifierProvider.autoDispose<PollingProviderNotifier, int>(
  () => PollingProviderNotifier(),
);
```

<div dir="rtl">

### onDispose

**متى يُستدعى:**
- لما الـ provider يتدمر نهائياً
- بعد onCancel (لو موجود)

</div>

```dart
class ResourceManagerNotifier extends AsyncNotifier<void> {
  late DatabaseConnection _db;
  late CacheManager _cache;

  @override
  Future<void> build() async {
    _db = await DatabaseConnection.open();
    _cache = CacheManager();

    ref.onDispose(() {
      print('Cleaning up all resources');
      _db.close();
      _cache.clear();
    });
  }
}

final resourceManagerProvider = AsyncNotifierProvider.autoDispose<ResourceManagerNotifier, void>(
  () => ResourceManagerNotifier(),
);
```

<div dir="rtl">

### onAddListener

**متى يُستدعى:**
- لما listener جديد يتضاف

</div>

```dart
class AnalyticsProviderNotifier extends Notifier<int> {
  @override
  int build() {
    ref.onAddListener(() {
      print('New listener added');
      // Track analytics event
    });

    ref.onRemoveListener(() {
      print('Listener removed');
    });

    return 0;
  }
}

final analyticsProvider = NotifierProvider.autoDispose<AnalyticsProviderNotifier, int>(
  () => AnalyticsProviderNotifier(),
);
```

<div dir="rtl">

---

## ⚠️ مشاكل شائعة

### مشكلة 1: Memory Leaks (نسيان التنظيف)

</div>

```dart
// ❌ WRONG - Memory leak!
class BadProviderNotifier extends StreamNotifier<int> {
  @override
  Stream<int> build() {
    final controller = StreamController<int>();

    Timer.periodic(Duration(seconds: 1), (timer) {
      controller.add(timer.tick);
    });

    // Timer and controller never cleaned up!
    return controller.stream;
  }
}

final badProvider = StreamNotifierProvider.autoDispose<BadProviderNotifier, int>(
  () => BadProviderNotifier(),
);

// ✅ CORRECT
class GoodProviderNotifier extends StreamNotifier<int> {
  Timer? _timer;
  StreamController<int>? _controller;

  @override
  Stream<int> build() {
    _controller = StreamController<int>();

    _timer = Timer.periodic(Duration(seconds: 1), (timer) {
      _controller!.add(timer.tick);
    });

    ref.onDispose(() {
      _timer?.cancel();
      _controller?.close();
    });

    return _controller!.stream;
  }
}

final goodProvider = StreamNotifierProvider.autoDispose<GoodProviderNotifier, int>(
  () => GoodProviderNotifier(),
);
```

<div dir="rtl">

### مشكلة 2: Over-using KeepAlive

</div>

```dart
// ❌ WRONG - Too many keepAlive
class TemporaryDataNotifierBad extends Notifier<String> {
  @override
  String build() {
    ref.keepAlive(); // Unnecessary!
    return 'temp data';
  }
}

final temporaryDataBadProvider = NotifierProvider.autoDispose<TemporaryDataNotifierBad, String>(
  () => TemporaryDataNotifierBad(),
);

// ✅ CORRECT - Use AutoDispose for temporary data
class TemporaryDataNotifierGood extends Notifier<String> {
  @override
  String build() {
    // No keepAlive - will be disposed when not needed
    return 'temp data';
  }
}

final temporaryDataGoodProvider = NotifierProvider.autoDispose<TemporaryDataNotifierGood, String>(
  () => TemporaryDataNotifierGood(),
);
```

<div dir="rtl">

### مشكلة 3: Forgetting to Close KeepAlive Link

</div>

```dart
// ❌ WRONG - Link never closed
class BadCacheNotifier extends AsyncNotifier<Data> {
  @override
  Future<Data> build() async {
    final link = ref.keepAlive();
    // Link never closed - data stays forever!

    return await fetchData();
  }

  Future<Data> fetchData() async {
    return Data();
  }
}

final badCacheProvider = AsyncNotifierProvider.autoDispose<BadCacheNotifier, Data>(
  () => BadCacheNotifier(),
);

// ✅ CORRECT - Close link when done
class GoodCacheNotifier extends AsyncNotifier<Data> {
  @override
  Future<Data> build() async {
    final link = ref.keepAlive();

    // Close after timeout
    Timer(Duration(minutes: 5), () {
      link.close();
    });

    return await fetchData();
  }

  Future<Data> fetchData() async {
    return Data();
  }
}

final goodCacheProvider = AsyncNotifierProvider.autoDispose<GoodCacheNotifier, Data>(
  () => GoodCacheNotifier(),
);
```

<div dir="rtl">

---

## 💡 Best Practices

### 1. استخدم AutoDispose للـ UI State

</div>

```dart
// UI state should auto-dispose
class FormStateNotifier extends Notifier<FormData> {
  @override
  FormData build() {
    // No keepAlive - dispose when form closes
    return FormData.empty();
  }
}

final formStateProvider = NotifierProvider.autoDispose<FormStateNotifier, FormData>(
  () => FormStateNotifier(),
);
```

<div dir="rtl">

### 2. استخدم KeepAlive للـ Global State

</div>

```dart
// Global state should persist
class AppSettingsNotifier extends Notifier<Settings> {
  @override
  Settings build() {
    ref.keepAlive(); // Keep settings alive
    return Settings.fromStorage();
  }
}

final appSettingsProvider = NotifierProvider.autoDispose<AppSettingsNotifier, Settings>(
  () => AppSettingsNotifier(),
);
```

<div dir="rtl">

### 3. دايماً نضف الـ Resources

</div>

```dart
class NetworkStreamNotifier extends StreamNotifier<Data> {
  StreamSubscription? _subscription;

  @override
  Stream<Data> build() {
    final stream = networkService.listen();

    _subscription = stream.listen((data) {
      // Handle data
    });

    ref.onDispose(() {
      _subscription?.cancel();
    });

    return stream;
  }
}

final networkStreamProvider = StreamNotifierProvider.autoDispose<NetworkStreamNotifier, Data>(
  () => NetworkStreamNotifier(),
);
```

<div dir="rtl">

### 4. استخدم Timed Cache للـ API Data

</div>

```dart
class ApiDataNotifier extends AsyncNotifier<Data> {
  @override
  Future<Data> build() async {
    final link = ref.keepAlive();

    // Cache for 5 minutes
    Timer(Duration(minutes: 5), () {
      link.close();
    });

    return await api.fetchData();
  }
}

final apiDataProvider = AsyncNotifierProvider.autoDispose<ApiDataNotifier, Data>(
  () => ApiDataNotifier(),
);
```

<div dir="rtl">

---

## 📊 Decision Tree

</div>

```
هل الـ data ده هيتطلب كتير؟
├─ نعم
│  └─ هل محتاج يفضل موجود دايماً؟
│     ├─ نعم → keepAlive()
│     └─ لا → keepAlive() + Timer
└─ لا
   └─ AutoDispose (default) ✅
```

<div dir="rtl">

---

## 📝 ملخص

**Lifecycle Phases:**
1. Created → Provider بيتعمل
2. Active → في listeners بتستخدمه
3. Paused → Listeners اتشالت (AutoDispose)
4. Disposed → Provider اتدمر

**Control:**
- `ref.keepAlive()` → منع الـ dispose
- `ref.onDispose()` → تنظيف resources
- `ref.onCancel()` → pause handling
- `ref.onResume()` → resume handling

**Best Practice:**
- Default: AutoDispose ✅
- Global state: KeepAlive
- Temp data: AutoDispose
- دايماً نضف resources في onDispose

---

## 🔗 الخطوة الجاية

في الملف الجاي هنتكلم عن:
- **Dependency Injection** بالتفصيل
- إزاي providers تعتمد على بعض
- Best practices للـ DI في Riverpod

---

## 📚 المصادر

- [Provider Lifecycle](https://riverpod.dev/docs/concepts/provider_lifecycle)
- [AutoDispose](https://riverpod.dev/docs/concepts/modifiers/auto_dispose)

</div>
