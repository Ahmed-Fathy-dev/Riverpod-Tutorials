<div dir="rtl">

# الـ Ref Object بالتفصيل

**المستوى**: 🟡 متوسط

## 📌 المفاهيم الأساسية

في الملف ده هنتكلم عن:
- الـ Ref object وإيه دوره
- كل الـ methods المتاحة
- الفرق بين WidgetRef و Ref
- إزاي تستخدم كل method صح
- أمثلة عملية لكل حالة

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تفهم الـ Ref object بعمق
- تختار الـ method الصح لكل موقف
- تستخدم الـ dependencies بشكل صحيح
- تتجنب الأخطاء الشائعة

---

## 🔍 إيه هو الـ Ref Object؟

الـ **Ref** (اختصار Reference) هو الـ object اللي بيسمحلك تتفاعل مع الـ providers.

</div>

```dart
// Ref is your window to the provider ecosystem
class CounterNotifier extends Notifier<int> {
  @override
  int build() {
    // 'ref' is available here
    ref.onDispose(() {
      print('Counter disposed');
    });

    return 0;
  }
}

final counterProvider = NotifierProvider<CounterNotifier, int>(
  () => CounterNotifier(),
);

class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // 'ref' is also available here
    final count = ref.watch(counterProvider);
    return Text('$count');
  }
}
```

<div dir="rtl">

### الـ Ref بيسمحلك تعمل إيه؟

1. **قراءة providers** (watch, read, listen)
2. **التحكم في الـ lifecycle** (onDispose, invalidate)
3. **Refresh providers** (refresh)
4. **Check existence** (exists)
5. **التعامل مع الـ dependencies** تلقائياً

---

## 🎨 أنواع الـ Ref

في Riverpod، في أكثر من نوع من الـ Ref، كل واحد ليه استخدامه:

### 1. Ref (الأساسي)

</div>

```dart
// Inside providers
final userProvider = FutureProvider<User>((ref) async {
  // 'ref' is of type 'Ref'
  final token = ref.watch(authTokenProvider);
  return api.getUser(token);
});
```

<div dir="rtl">

**متى تستخدمه:**
- جوه الـ providers
- في الـ build method بتاع Notifier
- لما تعمل dependencies بين providers

### 2. WidgetRef

</div>

```dart
// Inside widgets
class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // 'ref' is of type 'WidgetRef'
    final data = ref.watch(dataProvider);
    return Text('$data');
  }
}
```

<div dir="rtl">

**متى تستخدمه:**
- في الـ widgets (ConsumerWidget, ConsumerStatefulWidget)
- لما تعرض UI based on provider state
- لما تقرأ providers في event handlers

### 3. ProviderRef

</div>

```dart
// For providers with manual disposal control
final myProvider = Provider<MyService>((ref) {
  // 'ref' is of type 'ProviderRef'
  ref.onDispose(() {
    print('Cleaning up');
  });

  return MyService();
});
```

<div dir="rtl">

**متى تستخدمه:**
- في الـ classic providers (مش code generation)
- لما تحتاج lifecycle control دقيق

---

## 🛠️ Methods الأساسية

خليني أفصلهالك method بـ method:

### 1. ref.watch() - المراقبة المستمرة

</div>

```dart
// The most common method
class ShoppingCartNotifier extends Notifier<List<Product>> {
  @override
  List<Product> build() {
    // Watch userId - rebuilds when it changes
    final userId = ref.watch(currentUserIdProvider);

    // Load cart for this user
    return _loadCartForUser(userId);
  }

  List<Product> _loadCartForUser(String userId) {
    // Implementation
    return [];
  }
}

final shoppingCartProvider = NotifierProvider<ShoppingCartNotifier, List<Product>>(
  () => ShoppingCartNotifier(),
);

// In widgets
class CartWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Watch cart - rebuilds when cart changes
    final cart = ref.watch(shoppingCartProvider);

    return ListView.builder(
      itemCount: cart.length,
      itemBuilder: (context, index) {
        return ProductTile(cart[index]);
      },
    );
  }
}
```

<div dir="rtl">

**خصائص ref.watch:**
- ✅ Reactive - بيعمل rebuild لما الـ value يتغير
- ✅ Automatic cleanup - بيلغي الـ subscription لما الـ widget تتدمر
- ✅ Efficient - بيعمل rebuild بس للـ widgets اللي محتاجة
- ⚠️ **ممنوع** تستخدمه في event handlers أو async callbacks
- ⚠️ **لازم** يتنفذ في الـ build method بشكل مباشر

**أمثلة صح وغلط:**

</div>

```dart
// ✅ CORRECT: في build method
class GoodWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);
    return Text('$count');
  }
}

// ❌ WRONG: في event handler
class BadWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return ElevatedButton(
      onPressed: () {
        // DON'T DO THIS!
        final count = ref.watch(counterProvider); // ❌
        print(count);
      },
      child: Text('Click'),
    );
  }
}

// ✅ CORRECT: استخدم read في event handlers
class GoodWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return ElevatedButton(
      onPressed: () {
        final count = ref.read(counterProvider); // ✅
        print(count);
      },
      child: Text('Click'),
    );
  }
}
```

<div dir="rtl">

### 2. ref.read() - القراءة لمرة واحدة

</div>

```dart
// For one-time reads without listening to changes
class TodoForm extends ConsumerWidget {
  final TextEditingController _controller = TextEditingController();

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return TextField(
      controller: _controller,
      onSubmitted: (value) {
        // ✅ Use read to call a method
        ref.read(todosProvider.notifier).addTodo(value);
        _controller.clear();
      },
    );
  }
}
```

<div dir="rtl">

**خصائص ref.read:**
- ✅ للقراءة لمرة واحدة
- ✅ مثالي للـ event handlers
- ✅ مش بيعمل rebuild
- ✅ للوصول للـ notifiers
- ⚠️ **متستخدموش** في build method عشان تعرض data
- ⚠️ **استخدمه بس** للـ methods calls

**أمثلة عملية:**

</div>

```dart
// Example 1: Calling methods
class CounterButtons extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return Row(
      children: [
        IconButton(
          icon: Icon(Icons.remove),
          onPressed: () {
            // ✅ Read to call method
            ref.read(counterProvider.notifier).decrement();
          },
        ),
        IconButton(
          icon: Icon(Icons.add),
          onPressed: () {
            // ✅ Read to call method
            ref.read(counterProvider.notifier).increment();
          },
        ),
      ],
    );
  }
}

// Example 2: One-time check
class LoginButton extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return ElevatedButton(
      onPressed: () async {
        // ✅ Read current state once
        final isOnline = ref.read(connectivityProvider);

        if (!isOnline) {
          // Show error
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(content: Text('No internet connection')),
          );
          return;
        }

        // Proceed with login
        await ref.read(authProvider.notifier).login();
      },
      child: Text('Login'),
    );
  }
}
```

<div dir="rtl">

### 3. ref.listen() - للـ Side Effects

</div>

```dart
// For side effects like navigation, dialogs, snackbars
class HomePage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Listen to auth state for side effects
    ref.listen<AsyncValue<User?>>(
      authProvider,
      (previous, next) {
        next.when(
          data: (user) {
            if (user == null) {
              // Navigate to login
              Navigator.pushReplacementNamed(context, '/login');
            }
          },
          loading: () {},
          error: (error, stack) {
            // Show error dialog
            showDialog(
              context: context,
              builder: (context) => AlertDialog(
                title: Text('Error'),
                content: Text('$error'),
              ),
            );
          },
        );
      },
    );

    return Scaffold(
      appBar: AppBar(title: Text('Home')),
      body: Center(child: Text('Welcome!')),
    );
  }
}
```

<div dir="rtl">

**خصائص ref.listen:**
- ✅ للـ side effects (navigation, dialogs, snackbars)
- ✅ بيديك الـ previous و next values
- ✅ مش بيعمل rebuild للـ widget
- ✅ بيتنفذ بس لما الـ value يتغير
- ⚠️ **متستخدموش** لو هتعرض data في UI (استخدم watch)

**أمثلة متقدمة:**

</div>

```dart
// Example 1: Form validation errors
class RegisterForm extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Listen for validation errors
    ref.listen<String?>(
      formErrorProvider,
      (previous, next) {
        if (next != null) {
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(
              content: Text(next),
              backgroundColor: Colors.red,
            ),
          );
        }
      },
    );

    return Form(/* ... */);
  }
}

// Example 2: Success notification
class CheckoutPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Listen for successful order
    ref.listen<AsyncValue<Order>>(
      checkoutProvider,
      (previous, next) {
        if (next is AsyncData && previous is AsyncLoading) {
          // Order completed successfully
          Navigator.pushReplacementNamed(
            context,
            '/order-success',
            arguments: next.value,
          );
        }
      },
    );

    return CheckoutWidget();
  }
}

// Example 3: Listen to specific property
class ShoppingCartNotifier2 extends Notifier<CartState> {
  @override
  CartState build() {
    // Listen to currency changes
    ref.listen(
      currencyProvider,
      (previous, next) {
        // Recalculate prices when currency changes
        _recalculatePrices(next);
      },
    );

    return CartState.initial();
  }

  void _recalculatePrices(Currency currency) {
    // Implementation
  }
}

final shoppingCartProvider2 = NotifierProvider<ShoppingCartNotifier2, CartState>(
  () => ShoppingCartNotifier2(),
);
```

<div dir="rtl">

### 4. ref.listenSelf() - مراقبة نفسك

</div>

```dart
// Listen to your own state changes (in Notifiers)
class AutoSaveEditorNotifier extends Notifier<String> {
  @override
  String build() {
    // Listen to self and auto-save after changes
    ref.listenSelf((previous, next) {
      if (previous != null && previous != next) {
        _autoSave(next);
      }
    });

    return ''; // Initial text
  }

  void updateText(String text) {
    state = text;
    // This will trigger listenSelf callback
  }

  Future<void> _autoSave(String text) async {
    await Future.delayed(Duration(seconds: 2));
    print('Auto-saved: $text');
  }
}

final autoSaveEditorProvider = NotifierProvider<AutoSaveEditorNotifier, String>(
  () => AutoSaveEditorNotifier(),
);
```

<div dir="rtl">

### 5. ref.invalidate() - إعادة بناء Provider

</div>

```dart
// Force a provider to rebuild
class RefreshButton extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return IconButton(
      icon: Icon(Icons.refresh),
      onPressed: () {
        // Invalidate causes the provider to rebuild
        ref.invalidate(userProfileProvider);

        // Can invalidate multiple providers
        ref.invalidate(userPostsProvider);
        ref.invalidate(userFollowersProvider);
      },
    );
  }
}

// Example: Pull-to-refresh
class FeedPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final posts = ref.watch(feedProvider);

    return RefreshIndicator(
      onRefresh: () async {
        // Invalidate and wait for new data
        ref.invalidate(feedProvider);

        // Wait for the provider to rebuild
        await ref.read(feedProvider.future);
      },
      child: ListView(/* ... */),
    );
  }
}
```

<div dir="rtl">

### 6. ref.refresh() - Invalidate + Read الجديد

</div>

```dart
// Invalidate and immediately get the new value
class DataPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final data = ref.watch(dataProvider);

    return Column(
      children: [
        Text('Data: $data'),
        ElevatedButton(
          onPressed: () {
            // Refresh returns the new value
            final newData = ref.refresh(dataProvider);
            print('New data: $newData');

            // For async providers, returns Future/Stream
            final futureData = ref.refresh(asyncDataProvider);
            futureData.then((value) => print('Got: $value'));
          },
          child: Text('Refresh'),
        ),
      ],
    );
  }
}
```

<div dir="rtl">

**الفرق بين invalidate و refresh:**

</div>

```dart
// invalidate: Just marks for rebuild
ref.invalidate(userProvider);
// Provider rebuilds next time it's accessed

// refresh: Invalidate + immediate read
final user = ref.refresh(userProvider);
// Provider rebuilds NOW and returns new value
```

<div dir="rtl">

### 7. ref.exists() - التحقق من وجود Provider

</div>

```dart
// Check if a provider exists in the scope
class ConditionalFeatureNotifier extends Notifier<bool> {
  @override
  bool build() {
    // Check if premium features provider exists
    if (ref.exists(premiumFeaturesProvider)) {
      final premium = ref.watch(premiumFeaturesProvider);
      return premium.isEnabled;
    }

    return false; // Default for non-premium users
  }
}

final conditionalFeatureProvider = NotifierProvider<ConditionalFeatureNotifier, bool>(
  () => ConditionalFeatureNotifier(),
);
```

<div dir="rtl">

### 8. ref.onDispose() - التنظيف

</div>

```dart
// Cleanup when provider is disposed
class WebSocketConnectionNotifier extends StreamNotifier<Message> {
  @override
  Stream<Message> build() {
    final socket = WebSocket.connect('ws://example.com');

    // Cleanup when disposed
    ref.onDispose(() {
      print('Closing WebSocket');
      socket.close();
    });

    return socket.stream;
  }
}

final webSocketConnectionProvider = StreamNotifierProvider<WebSocketConnectionNotifier, Message>(
  () => WebSocketConnectionNotifier(),
);

// Example: Timer cleanup
class PeriodicUpdaterNotifier extends Notifier<int> {
  Timer? _timer;

  @override
  int build() {
    _timer = Timer.periodic(Duration(seconds: 1), (timer) {
      state++;
    });

    ref.onDispose(() {
      _timer?.cancel();
    });

    return 0;
  }
}

final periodicUpdaterProvider = NotifierProvider<PeriodicUpdaterNotifier, int>(
  () => PeriodicUpdaterNotifier(),
);
```

<div dir="rtl">

### 9. ref.onCancel() و ref.onResume() - للـ AutoDispose

</div>

```dart
// Advanced lifecycle control
class CachedDataNotifier extends AsyncNotifier<Data> {
  @override
  Future<Data> build() async {
    // Called when last listener is removed
    ref.onCancel(() {
      print('No more listeners');
    });

    // Called when a new listener is added after onCancel
    ref.onResume(() {
      print('Listener added again');
    });

    // Called when provider is completely disposed
    ref.onDispose(() {
      print('Disposed completely');
    });

    return await fetchData();
  }

  Future<Data> fetchData() async {
    // Implementation
    return Data();
  }
}

final cachedDataProvider = AsyncNotifierProvider<CachedDataNotifier, Data>(
  () => CachedDataNotifier(),
);
```

<div dir="rtl">

### 10. ref.mounted - Async Safety Check

**جديد في Riverpod 3.0!** 🆕

الـ `ref.mounted` هي property بترجعلك `bool` بتقولك إذا الـ provider أو الـ notifier لسه موجود ولا اتعمل dispose.

**الفكرة:** زي `BuildContext.mounted` في Flutter، بتحميك من race conditions في الـ async operations.

</div>

```dart
@riverpod
Future<User> fetchUser(Ref ref, String userId) async {
  // Network request يأخذ وقت
  final user = await api.getUser(userId);

  // ⚠️ لو الـ provider اتعمله dispose خلال الـ await،
  // مينفعش نحدث الـ state!
  if (!ref.mounted) {
    // Provider was disposed, don't update
    return Future.error('Provider disposed');
  }

  // آمن الآن - الـ provider لسه موجود
  return user;
}
```

<div dir="rtl">

#### متى تستخدمه؟

**1. في Async Operations طويلة:**

</div>

```dart
class UserProfileNotifier extends AsyncNotifier<UserProfile> {
  @override
  Future<UserProfile> build() async {
    // Fetch من API - يمكن يأخذ ثواني
    final profile = await api.getUserProfile();

    // تحقق قبل تحديث الـ state
    if (!ref.mounted) {
      throw Exception('Notifier disposed during fetch');
    }

    return profile;
  }

  Future<void> updateProfile(UserProfile newProfile) async {
    state = const AsyncValue.loading();

    try {
      // Network request
      await api.updateProfile(newProfile);

      // ⚠️ لو User طلع من الشاشة خلال الـ await
      if (!ref.mounted) {
        return; // مش هنحدث الـ state
      }

      // آمن - نحدث الـ state
      state = AsyncValue.data(newProfile);
    } catch (e, s) {
      if (ref.mounted) {
        state = AsyncValue.error(e, s);
      }
    }
  }
}

final userProfileProvider = AsyncNotifierProvider<UserProfileNotifier, UserProfile>(
  () => UserProfileNotifier(),
);
```

<div dir="rtl">

**2. في Multi-Step Async Workflows:**

</div>

```dart
class CheckoutNotifier extends AsyncNotifier<CheckoutState> {
  @override
  Future<CheckoutState> build() async {
    return CheckoutState.initial();
  }

  Future<void> processCheckout(Order order) async {
    state = const AsyncValue.loading();

    try {
      // Step 1: Validate order
      await validateOrder(order);
      if (!ref.mounted) return;

      // Step 2: Process payment
      final payment = await processPayment(order);
      if (!ref.mounted) return;

      // Step 3: Create order
      final confirmedOrder = await createOrder(payment);
      if (!ref.mounted) return;

      // Step 4: Send confirmation
      await sendConfirmation(confirmedOrder);
      if (!ref.mounted) return;

      state = AsyncValue.data(CheckoutState.success(confirmedOrder));
    } catch (e, s) {
      if (ref.mounted) {
        state = AsyncValue.error(e, s);
      }
    }
  }
}
```

<div dir="rtl">

**3. مع Navigation:**

</div>

```dart
class LoginNotifier extends AsyncNotifier<User?> {
  @override
  Future<User?> build() async => null;

  Future<void> login(String email, String password) async {
    state = const AsyncValue.loading();

    try {
      // Login API call
      final user = await authService.login(email, password);

      // تحقق قبل update
      if (!ref.mounted) {
        // User navigated away
        return;
      }

      state = AsyncValue.data(user);

      // Navigate to home
      // (في الكود الحقيقي، ده يكون في widget مع listen)
    } catch (e, s) {
      if (ref.mounted) {
        state = AsyncValue.error(e, s);
      }
    }
  }
}
```

<div dir="rtl">

#### ليه ده مهم؟

**بدون ref.mounted:**
```dart
// ❌ خطر - Race Condition
Future<void> loadData() async {
  final data = await api.getData();
  state = data; // ⚠️ قد يكون الـ provider اتعمل dispose!
  // Result: Exception أو memory leak
}
```

**مع ref.mounted:**
```dart
// ✅ آمن
Future<void> loadData() async {
  final data = await api.getData();

  if (!ref.mounted) return; // خرجنا بأمان

  state = data; // آمن - الـ provider موجود
}
```

#### Best Practices:

1. **دايماً تحقق بعد await**
   ```dart
   final result = await someAsyncOperation();
   if (!ref.mounted) return; // 👈 هنا
   ```

2. **في try-catch:**
   ```dart
   try {
     final data = await api.getData();
     if (!ref.mounted) return;
     state = AsyncValue.data(data);
   } catch (e, s) {
     if (ref.mounted) { // 👈 وهنا
       state = AsyncValue.error(e, s);
     }
   }
   ```

3. **في Multi-step operations:**
   ```dart
   await step1();
   if (!ref.mounted) return;

   await step2();
   if (!ref.mounted) return;

   await step3();
   ```

#### ⚠️ أخطاء شائعة:

**خطأ 1: نسيان التحقق:**
```dart
// ❌ خطأ
Future<void> loadUser() async {
  final user = await api.getUser();
  state = user; // Race condition!
}
```

**خطأ 2: التحقق قبل الـ await:**
```dart
// ❌ خطأ - التحقق في المكان الخطأ
Future<void> loadUser() async {
  if (!ref.mounted) return; // 👈 هنا مبدري!

  final user = await api.getUser();
  // الـ provider قد يكون disposed بعد الـ await
  state = user; // Race condition!
}
```

**خطأ 3: استخدامه في Synchronous code:**
```dart
// ❌ مش محتاج - مفيش async
void increment() {
  if (!ref.mounted) return; // غير ضروري
  state++;
}
```

</div>

```dart
// ✅ مثال كامل صحيح
class DataManagerNotifier extends AsyncNotifier<List<Data>> {
  @override
  Future<List<Data>> build() async {
    final data = await repository.fetchData();

    // تحقق بعد الـ async operation
    if (!ref.mounted) {
      throw Exception('Notifier disposed');
    }

    return data;
  }

  Future<void> refreshData() async {
    state = const AsyncValue.loading();

    try {
      final freshData = await repository.fetchData();

      // تحقق دايماً
      if (!ref.mounted) return;

      state = AsyncValue.data(freshData);
    } catch (e, s) {
      // حتى في الـ error
      if (ref.mounted) {
        state = AsyncValue.error(e, s);
      }
    }
  }

  Future<void> addData(Data newData) async {
    try {
      await repository.addData(newData);

      if (!ref.mounted) return;

      final currentData = state.value ?? [];
      state = AsyncValue.data([...currentData, newData]);
    } catch (e, s) {
      if (ref.mounted) {
        state = AsyncValue.error(e, s);
      }
    }
  }
}
```

<div dir="rtl">

**المصدر الرسمي:**
- [What's new in Riverpod 3.0](https://riverpod.dev/docs/whats_new) - Ref.mounted property

---

## 📊 جدول مقارنة شامل

| Method | متى تستخدمه | بيعمل rebuild؟ | Use Case |
|--------|-------------|----------------|----------|
| **watch** | في build method | ✅ نعم | عرض data في UI |
| **read** | في event handlers | ❌ لا | استدعاء methods |
| **listen** | للـ side effects | ❌ لا | Navigation, dialogs |
| **listenSelf** | جوه Notifier | ❌ لا | Auto-save, logging |
| **invalidate** | Pull-to-refresh | - | إعادة بناء provider |
| **refresh** | Forced reload | - | Reload + get value |
| **exists** | Conditional features | ❌ لا | Feature flags |
| **onDispose** | Cleanup | - | Close connections |
| **onCancel** | AutoDispose | - | Pause operations |
| **onResume** | AutoDispose | - | Resume operations |
| **mounted** 🆕 | بعد async ops | ❌ لا | Async safety check |

---

## 🎯 أمثلة عملية شاملة

### مثال 1: News Feed App

</div>

```dart
// Models
class Article {
  final String id;
  final String title;
  final String content;
  final DateTime publishedAt;

  Article({
    required this.id,
    required this.title,
    required this.content,
    required this.publishedAt,
  });
}

// Providers
class ArticlesListNotifier extends AsyncNotifier<List<Article>> {
  @override
  Future<List<Article>> build() async {
    // Watch user preferences
    final category = ref.watch(selectedCategoryProvider);

    // Cleanup when disposed
    ref.onDispose(() {
      print('Articles list disposed');
    });

    // Listen to refresh trigger
    ref.listen(
      refreshTriggerProvider,
      (previous, next) {
        if (next) {
          ref.invalidateSelf();
        }
      },
    );

    return await _fetchArticles(category);
  }

  Future<List<Article>> _fetchArticles(String category) async {
    // API call
    await Future.delayed(Duration(seconds: 1));
    return [];
  }
}

final articlesListProvider = AsyncNotifierProvider<ArticlesListNotifier, List<Article>>(
  () => ArticlesListNotifier(),
);

// Widget
class ArticlesFeed extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final articlesAsync = ref.watch(articlesListProvider);

    // Listen for errors
    ref.listen<AsyncValue<List<Article>>>(
      articlesListProvider,
      (previous, next) {
        next.whenOrNull(
          error: (error, stack) {
            ScaffoldMessenger.of(context).showSnackBar(
              SnackBar(content: Text('Error: $error')),
            );
          },
        );
      },
    );

    return RefreshIndicator(
      onRefresh: () async {
        // Refresh and wait
        ref.invalidate(articlesListProvider);
        await ref.read(articlesListProvider.future);
      },
      child: articlesAsync.when(
        data: (articles) => ListView.builder(
          itemCount: articles.length,
          itemBuilder: (context, index) {
            return ArticleTile(articles[index]);
          },
        ),
        loading: () => Center(child: CircularProgressIndicator()),
        error: (error, stack) => Center(
          child: Text('Error: $error'),
        ),
      ),
    );
  }
}
```

<div dir="rtl">

---

## ⚠️ أخطاء شائعة والحلول

### خطأ 1: استخدام watch في event handler

</div>

```dart
// ❌ WRONG
class BadWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return ElevatedButton(
      onPressed: () {
        final count = ref.watch(counterProvider); // Error!
        print(count);
      },
      child: Text('Click'),
    );
  }
}

// ✅ CORRECT
class GoodWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return ElevatedButton(
      onPressed: () {
        final count = ref.read(counterProvider);
        print(count);
      },
      child: Text('Click'),
    );
  }
}
```

<div dir="rtl">

### خطأ 2: استخدام read في build للعرض

</div>

```dart
// ❌ WRONG - Won't rebuild when value changes
class BadWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.read(counterProvider); // Won't update!
    return Text('$count');
  }
}

// ✅ CORRECT
class GoodWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);
    return Text('$count');
  }
}
```

<div dir="rtl">

### خطأ 3: نسيان onDispose

</div>

```dart
// ❌ WRONG - Memory leak!
class BadTimerNotifier extends Notifier<int> {
  @override
  int build() {
    Timer.periodic(Duration(seconds: 1), (timer) {
      state++;
    });
    // Timer never cancelled!

    return 0;
  }
}

final badTimerProvider = NotifierProvider<BadTimerNotifier, int>(
  () => BadTimerNotifier(),
);

// ✅ CORRECT
class GoodTimerNotifier extends Notifier<int> {
  Timer? _timer;

  @override
  int build() {
    _timer = Timer.periodic(Duration(seconds: 1), (timer) {
      state++;
    });

    ref.onDispose(() {
      _timer?.cancel();
    });

    return 0;
  }
}

final goodTimerProvider = NotifierProvider<GoodTimerNotifier, int>(
  () => GoodTimerNotifier(),
);
```

<div dir="rtl">

### خطأ 4: استخدام ref methods بعد dispose

**جديد في Riverpod 3.0!** 🆕

في Riverpod 3.0، كل الـ ref methods (ما عدا `mounted`) **ترمي exception** لو استخدمتها بعد ما الـ provider يتعمل dispose.

</div>

```dart
// ❌ WRONG - Using ref after dispose
@riverpod
class DataManager extends _$DataManager {
  @override
  Future<Data> build() async {
    return await api.getData();
  }

  Future<void> updateData() async {
    // Long async operation
    await Future.delayed(Duration(seconds: 5));

    // ⚠️ خطر! لو Provider اتعمل dispose خلال الـ 5 ثواني،
    // ref.read() هيرمي exception!
    final currentState = ref.read(dataManagerProvider);
    // Exception: "Cannot use ref methods after dispose"
  }
}
```

<div dir="rtl">

**المشكلة:**
- Provider قد يتعمل له dispose بسبب:
  - User طلع من الشاشة
  - Provider اتعمل له invalidate
  - AutoDispose أزال الـ provider
- لو استخدمت `ref.read()`, `ref.watch()`, `ref.invalidate()`, إلخ بعد dispose → Exception!

**الحل: استخدم ref.mounted للتحقق**

</div>

```dart
// ✅ CORRECT - Check ref.mounted first
@riverpod
class DataManager extends _$DataManager {
  @override
  Future<Data> build() async {
    return await api.getData();
  }

  Future<void> updateData() async {
    // Long async operation
    await Future.delayed(Duration(seconds: 5));

    // ✅ تحقق أول قبل أي ref method
    if (!ref.mounted) {
      return; // Provider disposed, exit safely
    }

    // آمن الآن - Provider لسه موجود
    final currentState = ref.read(dataManagerProvider);
    // Do something with currentState
  }

  Future<void> refreshData() async {
    final data = await api.getData();

    // تحقق قبل update
    if (!ref.mounted) return;

    // Update state safely
    state = AsyncValue.data(data);
  }
}
```

<div dir="rtl">

**مع Notifier.state:**

في Riverpod 3.0، `state` setter أيضاً يرمي exception بعد dispose:

</div>

```dart
// ❌ WRONG
class Counter extends Notifier<int> {
  @override
  int build() => 0;

  Future<void> incrementAsync() async {
    await Future.delayed(Duration(seconds: 2));

    // ⚠️ لو disposed خلال الـ delay، هيرمي exception!
    state++; // Exception!
  }
}

// ✅ CORRECT
class Counter extends Notifier<int> {
  @override
  int build() => 0;

  Future<void> incrementAsync() async {
    await Future.delayed(Duration(seconds: 2));

    // تحقق قبل update
    if (!ref.mounted) return;

    state++; // Safe
  }
}
```

<div dir="rtl">

**القاعدة الذهبية:**

> **بعد أي `await`، دايماً تحقق من `ref.mounted` قبل استخدام أي ref method أو state update!**

</div>

```dart
// ✅ The safe pattern
Future<void> someAsyncOperation() async {
  // Some async work
  await something();

  // 👇 Always check after await!
  if (!ref.mounted) return;

  // Now safe to use ref methods or update state
  ref.read(...);
  state = ...;
}
```

<div dir="rtl">

**الـ Exception اللي هتشوفها:**

```
StateError: Cannot update state, Notifier has already been disposed
```

أو

```
StateError: Cannot call ref methods after dispose
```

**متى تستخدم mounted:**
- ✅ **بعد أي await** في async functions
- ✅ **قبل state updates** في Notifier
- ✅ **قبل ref.read/watch/listen** بعد async operations
- ✅ **في multi-step async workflows**
- ❌ **مش محتاج** في synchronous code

**مثال كامل صحيح:**

</div>

```dart
@riverpod
class UserProfile extends _$UserProfile {
  @override
  Future<User> build(String userId) async {
    final user = await api.getUser(userId);

    // تحقق بعد async
    if (!ref.mounted) {
      throw Exception('Disposed during fetch');
    }

    return user;
  }

  Future<void> updateProfile(User updates) async {
    state = const AsyncValue.loading();

    try {
      // Step 1: Update on server
      await api.updateUser(updates);
      if (!ref.mounted) return; // تحقق

      // Step 2: Fetch fresh data
      final freshUser = await api.getUser(updates.id);
      if (!ref.mounted) return; // تحقق

      // Step 3: Update state
      state = AsyncValue.data(freshUser);
    } catch (e, s) {
      // تحقق حتى في catch
      if (ref.mounted) {
        state = AsyncValue.error(e, s);
      }
    }
  }

  Future<void> deleteAccount() async {
    try {
      await api.deleteAccount();

      if (!ref.mounted) return;

      // Navigate away (في الكود الحقيقي، ده يكون في widget)
      // navigator.pushReplacement(...)
    } catch (e) {
      if (ref.mounted) {
        // Show error
      }
    }
  }
}
```

<div dir="rtl">

**المصدر الرسمي:**
- [What's new in Riverpod 3.0 | Riverpod](https://riverpod.dev/docs/whats_new) - Breaking change: ref methods throw after dispose

---

## 💡 Best Practices

### 1. استخدم الـ Method الصح

</div>

```dart
// For displaying data: watch
final data = ref.watch(dataProvider);

// For calling methods: read
ref.read(dataProvider.notifier).update();

// For side effects: listen
ref.listen(authProvider, (prev, next) {
  // Navigate, show dialog, etc.
});
```

<div dir="rtl">

### 2. دايماً نضف ورا نفسك

</div>

```dart
class MyStateNotifier extends Notifier<MyState> {
  @override
  MyState build() {
    final subscription = someStream.listen((data) {
      // Handle data
    });

    ref.onDispose(() {
      subscription.cancel();
    });

    return MyState.initial();
  }
}

final myStateProvider = NotifierProvider<MyStateNotifier, MyState>(
  () => MyStateNotifier(),
);
```

<div dir="rtl">

### 3. استخدم listen بحذر

</div>

```dart
// ✅ Good: Specific side effects
ref.listen(authProvider, (prev, next) {
  if (next == null) {
    Navigator.pushNamed(context, '/login');
  }
});

// ❌ Bad: Using listen when watch is better
ref.listen(counterProvider, (prev, next) {
  setState(() {
    _displayValue = next; // Just use watch!
  });
});
```

<div dir="rtl">

---

## 📝 ملخص

الـ **Ref object** هو البوابة بتاعتك للتفاعل مع providers:

- **watch**: للعرض في UI (reactive)
- **read**: للاستدعاء methods (one-time)
- **listen**: للـ side effects (navigation, dialogs)
- **invalidate/refresh**: لإعادة تحميل data
- **onDispose**: للتنظيف

**القاعدة الذهبية:**
- في build method → watch
- في event handlers → read
- للـ side effects → listen

---

## 🔗 الخطوة الجاية

في الملف الجاي هنتكلم عن:
- **Provider Lifecycle** بالتفصيل
- متى يتم creation و disposal
- إزاي تتحكم في الـ lifecycle

---

## 📚 المصادر

- [Ref API Reference](https://riverpod.dev/docs/concepts/reading#ref)
- [Provider Lifecycle](https://riverpod.dev/docs/concepts/provider_lifecycle)

</div>
