<div dir="rtl">

# Automatic Retry - إعادة المحاولة التلقائية

**المستوى**: 🟡 متوسط
**جديد في Riverpod 3.0!** 🆕

## 📌 المفاهيم الأساسية

في الملف ده هنتعلم:
- ميزة Automatic Retry الجديدة في Riverpod 3.0
- كيف Providers تعيد المحاولة تلقائياً عند الفشل
- Exponential backoff algorithm
- كيف تتحكم في retry behavior
- متى تستخدم auto-retry ومتى تستخدم manual retry

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تفهم كيف automatic retry تشتغل
- تستخدم default retry behavior
- تخصص retry logic حسب احتياجك
- تختار متى تعطّل retry
- تعرف الفرق بين auto-retry و manual retry

---

## 🔄 إيه هي Automatic Retry؟

في **Riverpod 3.0**، لما provider يفشل (مثلاً بسبب network error)، Riverpod **مش بيرمي error فوراً**. بدلاً من كده، بيعيد المحاولة تلقائياً مع **exponential backoff**.

### قبل Riverpod 3.0 (Manual Retry):

</div>

```dart
// ❌ Riverpod 2.x - Manual retry (complicated)
@riverpod
class Products extends _$Products {
  @override
  Future<List<Product>> build() async {
    // لو فشل، هيرمي error فوراً
    return await api.getProducts(); // ⚠️ No automatic retry
  }

  // كنت مضطر تعمل retry logic بنفسك
  Future<void> retryFetch() async {
    state = const AsyncValue.loading();

    try {
      final products = await api.getProducts();
      state = AsyncValue.data(products);
    } catch (e, s) {
      state = AsyncValue.error(e, s);
    }
  }
}
```

<div dir="rtl">

### بعد Riverpod 3.0 (Automatic Retry):

</div>

```dart
// ✅ Riverpod 3.0 - Automatic retry (built-in!)
@riverpod
Future<List<Product>> products(Ref ref) async {
  // Riverpod يعيد المحاولة تلقائياً عند الفشل! 🎉
  return await api.getProducts();
  // Try 1: Fails → wait 200ms → Try again
  // Try 2: Fails → wait 400ms → Try again
  // Try 3: Fails → wait 800ms → Try again
  // ... continues with exponential backoff (max 6.4s)
}
```

<div dir="rtl">

**الفكرة:**
- ✅ **Automatic** - مفيش كود إضافي
- ✅ **Smart** - exponential backoff (ينتظر أكثر مع كل محاولة)
- ✅ **Resilient** - يتعامل مع transient failures (network glitches)

---

## ⏱️ Exponential Backoff - كيف يشتغل؟

Riverpod يستخدم **exponential backoff** algorithm - ينتظر وقت أطول قبل كل محاولة جديدة.

### Default Retry Timeline:

</div>

```
Attempt 1: Immediate call      → Fails
Wait:      200ms               ⏳
Attempt 2: Retry               → Fails
Wait:      400ms (200ms × 2)   ⏳
Attempt 3: Retry               → Fails
Wait:      800ms (400ms × 2)   ⏳
Attempt 4: Retry               → Fails
Wait:      1.6s (800ms × 2)    ⏳
Attempt 5: Retry               → Fails
Wait:      3.2s (1.6s × 2)     ⏳
Attempt 6: Retry               → Fails
Wait:      6.4s (3.2s × 2)     ⏳ (max delay)
Attempt 7+: Continues with 6.4s delay...
```

<div dir="rtl">

**ليه Exponential Backoff؟**

1. **يقلل Server Load** - مش بيقصف الـ server بـ requests متتالية
2. **يعطي فرصة للـ network** - يتعافى من temporary issues
3. **Efficient** - مش بيستهلك resources زيادة
4. **Industry Standard** - نفس اللي بيستخدمه AWS, Google, etc.

---

## ⚙️ كيفية تخصيص Retry Behavior

في Riverpod 3.0، فيه طريقتين لتخصيص retry behavior:

### 1. Global Configuration (لكل الـ Providers)

يمكنك تخصيص retry للـ application كله عن طريق `ProviderScope` أو `ProviderContainer`:

</div>

```dart
// Define custom retry function
Duration? myRetry(int retryCount, Object error) {
  if (retryCount >= 5) return null; // Stop after 5 attempts
  if (error is ProviderException) return null; // Don't retry ProviderExceptions
  return Duration(milliseconds: 200 * (1 << retryCount)); // Exponential backoff
}

// في Flutter apps
void main() {
  runApp(
    ProviderScope(
      retry: myRetry, // ✅ Apply to all providers
      child: MyApp(),
    ),
  );
}

// في pure Dart
final container = ProviderContainer(
  retry: myRetry, // ✅ Apply to all providers
);
```

<div dir="rtl">

### 2. Per-Provider Configuration (لـ Provider محدد)

يمكنك تخصيص retry لـ provider واحد باستخدام `@Riverpod(retry: ...)`:

</div>

```dart
// Custom retry function for specific provider
Duration? customRetry(int retryCount, Object error) {
  if (error is UnauthorizedException) return null;
  return Duration(seconds: retryCount + 1);
}

@Riverpod(retry: customRetry) // ✅ Apply to this provider only
Future<Data> myData(MyDataRef ref) async {
  return await api.getData();
}
```

<div dir="rtl">

**💡 Note:**
- إذا حددت `retry` على الـ provider، هيتجاهل الـ global configuration
- إذا مش محدد على الـ provider، هيستخدم الـ global configuration
- Default retry يتجاهل `ProviderException` تلقائياً

---

## 🎨 أمثلة عملية

### مثال 1: Basic Retry (Default Behavior)

</div>

```dart
// ✅ Default retry - works out of the box
@riverpod
Future<List<User>> users(Ref ref) async {
  // Riverpod automatically retries on failure
  return await api.getUsers();
}

// Usage in UI
class UsersListWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final usersAsync = ref.watch(usersProvider);

    return usersAsync.when(
      data: (users) => ListView.builder(
        itemCount: users.length,
        itemBuilder: (context, index) => UserTile(users[index]),
      ),

      // هيعرض loading حتى لو فيه retries جارية
      loading: () => const CircularProgressIndicator(),

      // هيعرض error بعد ما كل الـ retries تفشل
      error: (error, stack) => ErrorWidget(error),
    );
  }
}
```

<div dir="rtl">

### مثال 2: Custom Retry Logic

يمكنك تخصيص retry behavior باستخدام `@Riverpod(retry: ...)`:

</div>

```dart
// Define custom retry function
Duration? customRetry(int retryCount, Object error) {
  // error: الـ exception اللي حصل
  // retryCount: رقم المحاولة (0, 1, 2, ...)

  // Don't retry on authentication errors
  if (error is UnauthorizedException) {
    return null; // null = don't retry
  }

  // Don't retry on 404
  if (error is NotFoundException) {
    return null;
  }

  // Custom delay: wait 1 second × (attempt + 1)
  return Duration(seconds: retryCount + 1);
  // Attempt 0: wait 1s
  // Attempt 1: wait 2s
  // Attempt 2: wait 3s
}

@Riverpod(retry: customRetry)
Future<Data> dataWithCustomRetry(DataWithCustomRetryRef ref) async {
  return await api.getData();
}
```

<div dir="rtl">

### مثال 3: Retry بناءً على Error Type

</div>

```dart
// Custom retry based on error type
Duration? errorBasedRetry(int retryCount, Object error) {
  // Network errors → retry with exponential backoff
  if (error is NetworkException) {
    final baseDelay = 200; // milliseconds
    final delay = baseDelay * pow(2, retryCount).toInt();
    return Duration(milliseconds: delay);
  }

  // Server errors (500) → retry with longer delay
  if (error is ServerException) {
    return Duration(seconds: 5);
  }

  // Client errors (400) → don't retry
  if (error is ClientException) {
    return null;
  }

  // Unknown errors → use default retry
  return null; // Uses Riverpod's default
}

@Riverpod(retry: errorBasedRetry)
Future<Product> productDetails(ProductDetailsRef ref, String productId) async {
  return await api.getProduct(productId);
}
```

<div dir="rtl">

### مثال 4: Retry مع Max Attempts

</div>

```dart
// Custom retry with max attempts
Duration? limitedRetry(int retryCount, Object error) {
  const maxAttempts = 3;

  // Stop after 3 attempts (retryCount: 0, 1, 2)
  if (retryCount >= maxAttempts) {
    return null; // Don't retry anymore
  }

  // Exponential backoff
  return Duration(milliseconds: 200 * pow(2, retryCount).toInt());
}

@Riverpod(retry: limitedRetry)
Future<Config> appConfig(AppConfigRef ref) async {
  return await api.getConfig();
}
```

<div dir="rtl">

### مثال 5: Retry مع User Notification

</div>

```dart
// Custom retry with user notification
Duration? retryWithNotification(int retryCount, Object error) {
  // Show notification on retry
  if (retryCount > 0) {
    // يمكن تعمل notification للـ user
    print('Retry attempt ${retryCount + 1} after network error');
    // في الكود الحقيقي، استخدم notification service
  }

  // Retry with exponential backoff
  return Duration(milliseconds: 200 * pow(2, retryCount).toInt());
}

@Riverpod(retry: retryWithNotification)
class DataFetcher extends _$DataFetcher {
  @override
  Future<Data> build() async {
    return await api.getData();
  }
}
```

<div dir="rtl">

---

## ⚠️ متى تعطّل Automatic Retry؟

في بعض الحالات، **مش عايز** auto-retry - عايز تعرض error فوراً.

### مثال: User Input Errors

</div>

```dart
// Custom retry for login - only retry network errors
Duration? loginRetry(int retryCount, Object error) {
  // Don't retry login failures (wrong credentials)
  if (error is AuthenticationException) {
    return null; // Show error immediately
  }

  // Retry network errors only
  if (error is NetworkException) {
    return Duration(seconds: 2);
  }

  return null; // Don't retry by default
}

@Riverpod(retry: loginRetry)
Future<User> login(LoginRef ref, String email, String password) async {
  return await authService.login(email, password);
}
```

<div dir="rtl">

### مثال: Critical Operations

</div>

```dart
// Disable retry for critical operations
Duration? noRetry(int retryCount, Object error) => null;

@Riverpod(retry: noRetry)
Future<void> deleteAccount(DeleteAccountRef ref) async {
  await api.deleteUserAccount();
  // إذا فشل، عرض error - مش عايزين نحاول تاني تلقائياً
}
```

<div dir="rtl">

---

## 🔄 Automatic Retry vs Manual Retry

### متى تستخدم Automatic Retry؟

**✅ استخدم Automatic Retry عندما:**
- Network requests (GET operations)
- Reading data من API
- Transient failures (temporary network issues)
- Background data fetching
- Non-critical operations

</div>

```dart
// ✅ Perfect for auto-retry
@riverpod
Future<List<Product>> products(Ref ref) async {
  return await api.getProducts();
  // Auto-retry handles temporary network glitches
}
```

<div dir="rtl">

### متى تستخدم Manual Retry؟

**✅ استخدم Manual Retry عندما:**
- User-initiated actions (button clicks)
- Write operations (POST, PUT, DELETE)
- عايز تعرض retry button للـ user
- Critical operations (payments, etc.)

</div>

```dart
// ✅ Manual retry with button
// Disable auto-retry at provider level
Duration? noRetryForCheckout(int retryCount, Object error) => null;

@Riverpod(retry: noRetryForCheckout)
class Checkout extends _$Checkout {
  @override
  Future<Order?> build() async => null;

  // No auto-retry - user clicks "retry" button
  Future<void> processPayment(PaymentInfo info) async {
    state = const AsyncValue.loading();

    state = await AsyncValue.guard(() async {
      return await paymentService.process(info);
    });
  }
}

// UI with manual retry button
class CheckoutWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final orderAsync = ref.watch(checkoutProvider);

    return orderAsync.when(
      data: (order) => SuccessScreen(order),
      loading: () => LoadingScreen(),
      error: (error, stack) => Column(
        children: [
          Text('Payment failed: $error'),
          ElevatedButton(
            onPressed: () {
              // Manual retry على زر
              ref.read(checkoutProvider.notifier).processPayment(info);
            },
            child: Text('Retry Payment'),
          ),
        ],
      ),
    );
  }
}
```

<div dir="rtl">

---

## 🎯 Retry مع Loading Indicators

Riverpod بيحافظ على **previous data** خلال الـ retries، فتقدر تعرض loading indicator مع الـ data القديمة:

</div>

```dart
@riverpod
Future<List<Product>> products(Ref ref) async {
  return await api.getProducts();
  // Auto-retry enabled by default
}

class ProductsList extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final productsAsync = ref.watch(productsProvider);

    return productsAsync.when(
      data: (products) => Column(
        children: [
          // Show retry indicator if needed
          if (productsAsync.isRefreshing)
            const LinearProgressIndicator(),

          Expanded(
            child: ListView.builder(
              itemCount: products.length,
              itemBuilder: (context, index) => ProductCard(products[index]),
            ),
          ),
        ],
      ),

      loading: () => const CircularProgressIndicator(),

      error: (error, stack) => ErrorRetryWidget(
        error: error,
        onRetry: () => ref.invalidate(productsProvider),
      ),
    );
  }
}
```

<div dir="rtl">

---

## 📊 Retry مع Analytics/Logging

تقدر تتبع retry attempts للـ analytics:

</div>

```dart
// Custom retry with logging
Duration? retryWithLogging(int retryCount, Object error) {
  // Log retry attempts
  analyticsService.logEvent('data_fetch_retry', {
    'attempt': retryCount + 1,
    'error': error.toString(),
    'timestamp': DateTime.now().toIso8601String(),
  });

  // Exponential backoff
  return Duration(milliseconds: 200 * pow(2, retryCount).toInt());
}

@Riverpod(retry: retryWithLogging)
Future<Data> dataWithLogging(DataWithLoggingRef ref) async {
  try {
    final data = await api.getData();

    // Log success
    analyticsService.logEvent('data_fetch_success', {
      'timestamp': DateTime.now().toIso8601String(),
    });

    return data;
  } catch (e) {
    // Log final failure
    analyticsService.logEvent('data_fetch_failed', {
      'error': e.toString(),
    });
    rethrow;
  }
}
```

<div dir="rtl">

---

## 💡 Best Practices

### 1. استخدم Default Retry للـ Read Operations

</div>

```dart
// ✅ Good - auto-retry enabled
@riverpod
Future<List<Post>> posts(Ref ref) async {
  return await api.getPosts();
}
```

<div dir="rtl">

### 2. عطّل Retry للـ Write Operations

</div>

```dart
// ✅ Good - auto-retry for reads only
// Note: retry only applies to build() method (reading posts)
// Mutation methods like createPost don't auto-retry
@riverpod
class PostManager extends _$PostManager {
  @override
  Future<List<Post>> build() async {
    return await api.getPosts(); // This will auto-retry on failure
  }

  Future<void> createPost(Post post) async {
    // Mutations don't auto-retry by default
    await api.createPost(post);
  }
}
```

<div dir="rtl">

### 3. خصص Retry Logic حسب Error Type

</div>

```dart
// ✅ Good - different retry for different errors
Duration? smartRetryLogic(int retryCount, Object error) {
  if (error is NetworkException) {
    // Network errors → retry with exponential backoff
    return Duration(milliseconds: 200 * pow(2, retryCount).toInt());
  }

  if (error is ServerException && error.statusCode == 503) {
    // Service unavailable → retry with longer delay
    return Duration(seconds: 10);
  }

  // Client errors (400) → don't retry
  return null;
}

@Riverpod(retry: smartRetryLogic)
Future<Data> smartRetry(SmartRetryRef ref) async {
  return await api.getData();
}
```

<div dir="rtl">

### 4. أضف Max Retry Limit

</div>

```dart
// ✅ Good - limit retry attempts
Duration? limitedRetryLogic(int retryCount, Object error) {
  const maxRetries = 5;

  if (retryCount >= maxRetries) {
    return null; // Stop retrying
  }

  return Duration(milliseconds: 200 * pow(2, retryCount).toInt());
}

@Riverpod(retry: limitedRetryLogic)
Future<Config> configWithLimit(ConfigWithLimitRef ref) async {
  return await api.getConfig();
}
```

<div dir="rtl">

### 5. اجمع بين Auto-Retry و Manual Retry

</div>

```dart
// ✅ Good - auto-retry + manual refresh button
@riverpod
Future<Feed> userFeed(Ref ref) async {
  // Auto-retry enabled for initial load
  return await api.getFeed();
}

class FeedWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final feedAsync = ref.watch(userFeedProvider);

    return RefreshIndicator(
      // Manual refresh on pull
      onRefresh: () async {
        ref.invalidate(userFeedProvider);
        await ref.read(userFeedProvider.future);
      },
      child: feedAsync.when(
        data: (feed) => FeedList(feed),
        loading: () => LoadingShimmer(),
        error: (error, stack) => ErrorView(error),
      ),
    );
  }
}
```

<div dir="rtl">

---

## ⚠️ أخطاء شائعة

### خطأ 1: عدم تعطيل Retry للـ Write Operations

</div>

```dart
// ❌ BAD - auto-retry enabled for POST (dangerous!)
@riverpod
Future<void> createOrder(Ref ref, Order order) async {
  await api.createOrder(order);
  // ⚠️ لو فشل، هيعيد المحاولة تلقائياً - قد ينشئ orders مكررة!
}

// ✅ GOOD - disable retry
Duration? noRetryForOrder(int retryCount, Object error) => null;

@Riverpod(retry: noRetryForOrder)
Future<void> createOrder(CreateOrderRef ref, Order order) async {
  await api.createOrder(order);
}
```

<div dir="rtl">

### خطأ 2: Retry Delay طويل جداً

</div>

```dart
// ❌ BAD - delay too long (poor UX)
Duration? badRetryTooLong(int retryCount, Object error) {
  return Duration(minutes: 1); // User waits 1 minute!
}

// ✅ GOOD - reasonable delays
Duration? goodRetryDelay(int retryCount, Object error) {
  return Duration(milliseconds: 200 * pow(2, retryCount).toInt());
  // Max ~6.4 seconds
}
```

<div dir="rtl">

### خطأ 3: Retry على Errors مش محتاجة

</div>

```dart
// ❌ BAD - retrying validation errors (useless)
Duration? badRetryAll(int retryCount, Object error) {
  // يعيد المحاولة حتى لو validation error!
  return Duration(seconds: 1);
}

// ✅ GOOD - retry only recoverable errors
Duration? goodRetrySelective(int retryCount, Object error) {
  if (error is ValidationException) {
    return null; // Don't retry validation errors
  }

  if (error is NetworkException) {
    return Duration(seconds: 2); // Retry network errors
  }

  return null;
};
```

<div dir="rtl">

---

## 📝 ملخص

| Feature | Riverpod 2.x | Riverpod 3.0 |
|---------|-------------|--------------|
| **Auto-Retry** | ❌ Manual only | ✅ Built-in |
| **Retry Logic** | Write yourself | Default exponential backoff |
| **Customization** | Full control | `@Riverpod(retry: ...)` |
| **Network Resilience** | Low | High |
| **Code Complexity** | High | Low |

### Key Points:

1. ✅ **Auto-retry enabled by default** في Riverpod 3.0
2. ✅ **Exponential backoff** - ينتظر أطول مع كل محاولة
3. ✅ **Customizable** - استخدم `@Riverpod(retry: customRetry)` للتحكم
4. ✅ **Smart** - يتعامل مع transient failures تلقائياً
5. ⚠️ **Disable for writes** - عطّل retry للـ POST/PUT/DELETE
6. ⚠️ **Error-specific** - retry بناءً على نوع الـ error

---

## 🔗 الخطوة الجاية

- [Error Handling](./03-error-handling.md) - تعلم error handling مع retry
- [Loading States](./04-loading-states.md) - عرض loading indicators خلال retry
- [Refresh Strategies](./05-refresh-strategies.md) - استراتيجيات refresh مختلفة

---

## 📚 المصادر

### Official Documentation:
- [Automatic Retry | Riverpod](https://riverpod.dev/docs/concepts2/retry)
- [What's new in Riverpod 3.0 | Riverpod](https://riverpod.dev/docs/whats_new)
- [Error Handling | Riverpod](https://riverpod.dev/docs/how_to/error_handling)

### Community Resources:
- [Riverpod 3.0 Release Notes](https://pub.dev/packages/riverpod/changelog)
- [Flutter Riverpod 3.0 New Features](https://medium.com/@lee645521797/flutter-riverpod-3-0-released-a-major-redesign-of-the-state-management-framework-f7e31f19b179)

---

**الحمد لله** - الآن عندك فهم كامل للـ Automatic Retry! 🎉

</div>
