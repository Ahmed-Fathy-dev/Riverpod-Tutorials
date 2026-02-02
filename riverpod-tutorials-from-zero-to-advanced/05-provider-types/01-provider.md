<div dir="rtl">

# Provider - القيم الثابتة والمحسوبة

**المستوى**: 🟢 مبتدئ

## 📌 المفاهيم الأساسية

في الملف ده هنتكلم عن:
- إيه هو Provider وليه مهم
- متى نستخدم Provider
- أمثلة عملية شاملة
- Best practices
- الفرق بينه وبين StateProvider

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تفهم إيه هو Provider
- تستخدم Provider للقيم الثابتة
- تعمل Dependency Injection
- تحسب Computed values
- تعرف متى تستخدم Provider

---

## 🔍 إيه هو Provider؟

**Provider** هو أبسط نوع من Providers في Riverpod. بيستخدم للقيم اللي:
- **مش بتتغير** (read-only)
- أو **محسوبة** من providers تانية (computed)

</div>

```dart
// Simple Provider
final apiUrlProvider = Provider<String>((ref) {
  return 'https://api.example.com';
});

// Computed Provider
final userNameProvider = Provider<String>((ref) {
  final user = ref.watch(userProvider);
  return user.name;
});
```

<div dir="rtl">

**الفكرة الأساسية:**
- القيمة **مش بتتغير مباشرة** (immutable)
- لو محتاج تغيير، استخدم StateProvider أو Notifier
- ممكن يعتمد على providers تانية
- بيعمل rebuild لما الـ dependencies بتاعته تتغير

---

## 🎨 الاستخدامات الأساسية

### 1. Configuration Values

</div>

```dart
// API configuration
final apiBaseUrlProvider = Provider<String>((ref) {
  return 'https://api.myapp.com';
});

final apiVersionProvider = Provider<String>((ref) {
  return 'v1';
});

final apiTimeoutProvider = Provider<Duration>((ref) {
  return Duration(seconds: 30);
});

// App configuration
final appConfigProvider = Provider<AppConfig>((ref) {
  return AppConfig(
    appName: 'My Awesome App',
    version: '1.0.0',
    debugMode: false,
  );
});
```

<div dir="rtl">

### 2. Services & Dependencies (Dependency Injection)

</div>

```dart
// HTTP Client
final httpClientProvider = Provider<http.Client>((ref) {
  return http.Client();
});

// API Service
final apiServiceProvider = Provider<ApiService>((ref) {
  final client = ref.watch(httpClientProvider);
  final baseUrl = ref.watch(apiBaseUrlProvider);

  return ApiService(
    client: client,
    baseUrl: baseUrl,
  );
});

// Database
final databaseProvider = Provider<Database>((ref) {
  final db = Database();

  // Cleanup when provider is disposed
  ref.onDispose(() {
    db.close();
  });

  return db;
});

// Secure Storage
final secureStorageProvider = Provider<SecureStorage>((ref) {
  return SecureStorage();
});
```

<div dir="rtl">

**الميزة:** Dependency Injection تلقائي!

</div>

```dart
// Repository depends on ApiService
final userRepositoryProvider = Provider<UserRepository>((ref) {
  final api = ref.watch(apiServiceProvider);
  final storage = ref.watch(secureStorageProvider);

  return UserRepository(
    apiService: api,
    storage: storage,
  );
});
```

<div dir="rtl">

### 3. Computed Values (القيم المحسوبة)

</div>

```dart
// Cart items provider (from somewhere else)
final cartItemsProvider = StateProvider<List<CartItem>>((ref) => []);

// ✅ Computed: Total items count
final cartItemsCountProvider = Provider<int>((ref) {
  final items = ref.watch(cartItemsProvider);
  return items.length;
});

// ✅ Computed: Total price
final cartTotalPriceProvider = Provider<double>((ref) {
  final items = ref.watch(cartItemsProvider);
  return items.fold(0.0, (sum, item) => sum + (item.price * item.quantity));
});

// ✅ Computed: Has discount
final hasDiscountProvider = Provider<bool>((ref) {
  final total = ref.watch(cartTotalPriceProvider);
  return total > 100; // Discount if total > $100
});

// ✅ Computed: Final price (with discount)
final finalPriceProvider = Provider<double>((ref) {
  final total = ref.watch(cartTotalPriceProvider);
  final hasDiscount = ref.watch(hasDiscountProvider);

  if (hasDiscount) {
    return total * 0.9; // 10% discount
  }
  return total;
});
```

<div dir="rtl">

**الميزة:** بيتحدث تلقائياً لما الـ dependencies تتغير!

### 4. Filters & Transformations

</div>

```dart
// All todos
final allTodosProvider = StateProvider<List<Todo>>((ref) => []);

// Filter type
final todoFilterProvider = StateProvider<TodoFilter>((ref) => TodoFilter.all);

// ✅ Filtered todos (computed)
final filteredTodosProvider = Provider<List<Todo>>((ref) {
  final todos = ref.watch(allTodosProvider);
  final filter = ref.watch(todoFilterProvider);

  switch (filter) {
    case TodoFilter.all:
      return todos;
    case TodoFilter.active:
      return todos.where((todo) => !todo.isCompleted).toList();
    case TodoFilter.completed:
      return todos.where((todo) => todo.isCompleted).toList();
  }
});

// ✅ Completed count (computed)
final completedTodosCountProvider = Provider<int>((ref) {
  final todos = ref.watch(allTodosProvider);
  return todos.where((todo) => todo.isCompleted).length;
});

// ✅ Active count (computed)
final activeTodosCountProvider = Provider<int>((ref) {
  final todos = ref.watch(allTodosProvider);
  return todos.where((todo) => !todo.isCompleted).length;
});
```

<div dir="rtl">

---

## 💻 أمثلة عملية كاملة

### مثال 1: Multi-Environment Configuration

</div>

```dart
// Environment enum
enum Environment { dev, staging, production }

// Current environment
final currentEnvironmentProvider = Provider<Environment>((ref) {
  // In real app, get from build config or env variable
  const environment = String.fromEnvironment('ENV', defaultValue: 'dev');

  switch (environment) {
    case 'production':
      return Environment.production;
    case 'staging':
      return Environment.staging;
    default:
      return Environment.dev;
  }
});

// API URL based on environment
final apiUrlProvider = Provider<String>((ref) {
  final env = ref.watch(currentEnvironmentProvider);

  switch (env) {
    case Environment.production:
      return 'https://api.myapp.com';
    case Environment.staging:
      return 'https://staging-api.myapp.com';
    case Environment.dev:
      return 'http://localhost:3000';
  }
});

// Debug mode
final isDebugModeProvider = Provider<bool>((ref) {
  final env = ref.watch(currentEnvironmentProvider);
  return env == Environment.dev;
});

// API timeout based on environment
final apiTimeoutProvider = Provider<Duration>((ref) {
  final env = ref.watch(currentEnvironmentProvider);

  return env == Environment.dev
      ? Duration(seconds: 60) // Longer timeout for dev
      : Duration(seconds: 30);
});

// Usage in widget
class AppInfo extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final env = ref.watch(currentEnvironmentProvider);
    final apiUrl = ref.watch(apiUrlProvider);
    final isDebug = ref.watch(isDebugModeProvider);

    return Column(
      children: [
        Text('Environment: $env'),
        Text('API URL: $apiUrl'),
        if (isDebug) Text('🐛 Debug Mode'),
      ],
    );
  }
}
```

<div dir="rtl">

### مثال 2: Dependency Injection Chain

</div>

```dart
// Layer 1: Core services
final httpClientProvider = Provider<http.Client>((ref) {
  return http.Client();
});

final loggerProvider = Provider<Logger>((ref) {
  return Logger();
});

// Layer 2: API Service
final apiServiceProvider = Provider<ApiService>((ref) {
  final client = ref.watch(httpClientProvider);
  final logger = ref.watch(loggerProvider);
  final baseUrl = ref.watch(apiUrlProvider);

  return ApiService(
    client: client,
    logger: logger,
    baseUrl: baseUrl,
  );
});

// Layer 3: Repositories
final userRepositoryProvider = Provider<UserRepository>((ref) {
  final api = ref.watch(apiServiceProvider);
  final logger = ref.watch(loggerProvider);

  return UserRepository(
    apiService: api,
    logger: logger,
  );
});

final productsRepositoryProvider = Provider<ProductsRepository>((ref) {
  final api = ref.watch(apiServiceProvider);

  return ProductsRepository(apiService: api);
});

// Layer 4: Use in other providers
final currentUserProvider = FutureProvider<User>((ref) async {
  final repo = ref.watch(userRepositoryProvider);
  return await repo.getCurrentUser();
});

// Usage in widget - automatic dependency injection!
class UserProfile extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(currentUserProvider);

    return userAsync.when(
      data: (user) => Text('Hello, ${user.name}'),
      loading: () => CircularProgressIndicator(),
      error: (error, stack) => Text('Error: $error'),
    );
  }
}
```

<div dir="rtl">

### مثال 3: Theme System

</div>

```dart
// Theme mode
final themeModeProvider = StateProvider<ThemeMode>((ref) {
  return ThemeMode.system;
});

// Primary color
final primaryColorProvider = StateProvider<Color>((ref) {
  return Colors.blue;
});

// ✅ Light theme (computed)
final lightThemeProvider = Provider<ThemeData>((ref) {
  final primaryColor = ref.watch(primaryColorProvider);

  return ThemeData(
    brightness: Brightness.light,
    primaryColor: primaryColor,
    colorScheme: ColorScheme.fromSeed(
      seedColor: primaryColor,
      brightness: Brightness.light,
    ),
    useMaterial3: true,
  );
});

// ✅ Dark theme (computed)
final darkThemeProvider = Provider<ThemeData>((ref) {
  final primaryColor = ref.watch(primaryColorProvider);

  return ThemeData(
    brightness: Brightness.dark,
    primaryColor: primaryColor,
    colorScheme: ColorScheme.fromSeed(
      seedColor: primaryColor,
      brightness: Brightness.dark,
    ),
    useMaterial3: true,
  );
});

// Usage in MaterialApp
class MyApp extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final themeMode = ref.watch(themeModeProvider);
    final lightTheme = ref.watch(lightThemeProvider);
    final darkTheme = ref.watch(darkThemeProvider);

    return MaterialApp(
      themeMode: themeMode,
      theme: lightTheme,
      darkTheme: darkTheme,
      home: HomeScreen(),
    );
  }
}
```

<div dir="rtl">

### مثال 4: Shopping Cart Calculations

</div>

```dart
// Cart items (mutable state)
final cartItemsProvider = StateProvider<List<CartItem>>((ref) => []);

// Promo code
final promoCodeProvider = StateProvider<String?>((ref) => null);

// ✅ Subtotal (computed)
final subtotalProvider = Provider<double>((ref) {
  final items = ref.watch(cartItemsProvider);
  return items.fold(0.0, (sum, item) => sum + (item.price * item.quantity));
});

// ✅ Discount amount (computed)
final discountAmountProvider = Provider<double>((ref) {
  final subtotal = ref.watch(subtotalProvider);
  final promoCode = ref.watch(promoCodeProvider);

  if (promoCode == null) return 0.0;

  // Check promo code
  switch (promoCode) {
    case 'SAVE10':
      return subtotal * 0.1; // 10% off
    case 'SAVE20':
      return subtotal * 0.2; // 20% off
    case 'SAVE50':
      return subtotal >= 100 ? 50 : 0; // $50 off if subtotal >= $100
    default:
      return 0.0;
  }
});

// ✅ Shipping cost (computed)
final shippingCostProvider = Provider<double>((ref) {
  final subtotal = ref.watch(subtotalProvider);

  // Free shipping if subtotal >= $50
  if (subtotal >= 50) return 0.0;

  return 5.0; // $5 shipping
});

// ✅ Tax (computed)
final taxProvider = Provider<double>((ref) {
  final subtotal = ref.watch(subtotalProvider);
  final discount = ref.watch(discountAmountProvider);

  final taxableAmount = subtotal - discount;
  return taxableAmount * 0.1; // 10% tax
});

// ✅ Total (computed)
final totalProvider = Provider<double>((ref) {
  final subtotal = ref.watch(subtotalProvider);
  final discount = ref.watch(discountAmountProvider);
  final shipping = ref.watch(shippingCostProvider);
  final tax = ref.watch(taxProvider);

  return subtotal - discount + shipping + tax;
});

// ✅ Can checkout? (computed)
final canCheckoutProvider = Provider<bool>((ref) {
  final items = ref.watch(cartItemsProvider);
  final total = ref.watch(totalProvider);

  return items.isNotEmpty && total > 0;
});

// Usage in widget
class CartSummary extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final subtotal = ref.watch(subtotalProvider);
    final discount = ref.watch(discountAmountProvider);
    final shipping = ref.watch(shippingCostProvider);
    final tax = ref.watch(taxProvider);
    final total = ref.watch(totalProvider);
    final canCheckout = ref.watch(canCheckoutProvider);

    return Card(
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text('Order Summary', style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
            SizedBox(height: 16),
            _SummaryRow('Subtotal', subtotal),
            if (discount > 0) _SummaryRow('Discount', -discount, color: Colors.green),
            _SummaryRow('Shipping', shipping),
            _SummaryRow('Tax', tax),
            Divider(),
            _SummaryRow('Total', total, isBold: true),
            SizedBox(height: 16),
            ElevatedButton(
              onPressed: canCheckout ? () {
                // Checkout logic
              } : null,
              child: Text('Checkout'),
            ),
          ],
        ),
      ),
    );
  }

  Widget _SummaryRow(String label, double amount, {bool isBold = false, Color? color}) {
    return Padding(
      padding: EdgeInsets.symmetric(vertical: 4),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          Text(
            label,
            style: TextStyle(
              fontWeight: isBold ? FontWeight.bold : FontWeight.normal,
            ),
          ),
          Text(
            '\$${amount.toStringAsFixed(2)}',
            style: TextStyle(
              fontWeight: isBold ? FontWeight.bold : FontWeight.normal,
              color: color,
            ),
          ),
        ],
      ),
    );
  }
}
```

<div dir="rtl">

---

## 🔄 Lifecycle & Cleanup

### ref.onDispose - تنضيف الموارد

</div>

```dart
// HTTP Client with cleanup
final httpClientProvider = Provider<http.Client>((ref) {
  final client = http.Client();

  // Cleanup when provider is disposed
  ref.onDispose(() {
    print('Closing HTTP client');
    client.close();
  });

  return client;
});

// Database with cleanup
final databaseProvider = Provider<Database>((ref) {
  final db = Database.open('my_app.db');

  ref.onDispose(() {
    print('Closing database');
    db.close();
  });

  return db;
});

// Stream subscription with cleanup
final locationStreamProvider = Provider<Stream<Position>>((ref) {
  final controller = StreamController<Position>();

  final subscription = Geolocator.getPositionStream().listen(
    (position) => controller.add(position),
  );

  ref.onDispose(() {
    print('Cancelling location subscription');
    subscription.cancel();
    controller.close();
  });

  return controller.stream;
});
```

<div dir="rtl">

---

## ⚠️ أخطاء شائعة

### ❌ خطأ 1: استخدام Provider لحاجة بتتغير

</div>

```dart
// ❌ WRONG - Can't change the value!
final counterProvider = Provider<int>((ref) => 0);

// How to increment? Can't!
// ref.read(counterProvider) = 5; // ERROR!

// ✅ CORRECT - Use StateProvider for mutable state
final counterProvider = StateProvider<int>((ref) => 0);

// Now you can change it
ref.read(counterProvider.notifier).state++;
```

<div dir="rtl">

### ❌ خطأ 2: إعادة إنشاء Objects في كل rebuild

</div>

```dart
// ❌ WRONG - Creates new instance every time!
final apiServiceProvider = Provider<ApiService>((ref) {
  return ApiService(); // New instance every rebuild!
});

// ✅ CORRECT - Provider caches the value
final apiServiceProvider = Provider<ApiService>((ref) {
  // Created once, cached automatically
  return ApiService();
});

// Note: Provider automatically caches the value!
// You don't need to do anything special.
```

<div dir="rtl">

### ❌ خطأ 3: Side effects في Provider

</div>

```dart
// ❌ WRONG - Side effects in provider build
final loggerProvider = Provider<Logger>((ref) {
  final logger = Logger();

  // BAD! Side effect during build
  logger.log('Provider initialized');

  return logger;
});

// ✅ CORRECT - Side effects in callbacks
final loggerProvider = Provider<Logger>((ref) {
  final logger = Logger();

  // Good! Side effect in onDispose
  ref.onDispose(() {
    logger.log('Provider disposed');
  });

  return logger;
});
```

<div dir="rtl">

---

## 💡 Best Practices

### 1. استخدم Provider للـ Dependencies

</div>

```dart
// ✅ Good - Dependency injection
final apiProvider = Provider<ApiClient>((ref) => ApiClient());
final dbProvider = Provider<Database>((ref) => Database());
final storageProvider = Provider<Storage>((ref) => Storage());
```

<div dir="rtl">

### 2. استخدم Provider للـ Computed Values

</div>

```dart
// ✅ Good - Computed values
final totalProvider = Provider<double>((ref) {
  final items = ref.watch(itemsProvider);
  return items.fold(0.0, (sum, item) => sum + item.price);
});
```

<div dir="rtl">

### 3. استخدم ref.onDispose للـ Cleanup

</div>

```dart
// ✅ Good - Cleanup resources
final clientProvider = Provider<http.Client>((ref) {
  final client = http.Client();
  ref.onDispose(() => client.close());
  return client;
});
```

<div dir="rtl">

### 4. سمي الـ Providers بوضوح

</div>

```dart
// ✅ Good - Clear naming
final currentUserProvider = ...
final userRepositoryProvider = ...
final totalPriceProvider = ...

// ❌ Bad - Unclear naming
final user = ...
final repo = ...
final total = ...
```

<div dir="rtl">

---

## 🆚 Provider vs StateProvider

| Feature | Provider | StateProvider |
|---------|----------|---------------|
| **القيمة** | Read-only | Mutable |
| **التغيير** | مش ممكن مباشرة | ممكن بـ `.state =` |
| **Use Case** | Config, Services, Computed | Simple state (int, bool, String) |
| **مثال** | `apiClientProvider` | `counterProvider` |

</div>

```dart
// Provider - Can't change directly
final apiUrlProvider = Provider<String>((ref) {
  return 'https://api.example.com';
});
// No way to change it!

// StateProvider - Can change
final counterProvider = StateProvider<int>((ref) => 0);
// Can change: ref.read(counterProvider.notifier).state = 5;
```

<div dir="rtl">

---

## 📝 ملخص

**Provider** يستخدم لـ:
- ✅ Configuration values
- ✅ Services & Dependencies (DI)
- ✅ Computed values
- ✅ Filters & Transformations
- ✅ Read-only data

**مش يستخدم لـ:**
- ❌ Mutable state (use StateProvider)
- ❌ Async data (use FutureProvider/AsyncNotifierProvider)

**Best Practices:**
- استخدمه للـ dependencies
- استخدمه للـ computed values
- استخدم `ref.onDispose` للـ cleanup
- سمي الـ providers بوضوح

---

## 🔗 الخطوة الجاية

في الملف الجاي هنتكلم عن:
- **StateProvider** - للـ state البسيط اللي بيتغير
- متى نستخدم StateProvider بدل Provider
- أمثلة عملية

---

## 📚 المصادر

- [Provider - Riverpod Docs](https://riverpod.dev/docs/providers/provider)
- [Dependency Injection](https://riverpod.dev/docs/concepts/about_providers)

</div>
