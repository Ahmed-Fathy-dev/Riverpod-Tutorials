<div dir="rtl">

# Dependency Injection في Riverpod

**المستوى**: 🟡 متوسط

## 📌 المفاهيم الأساسية

في الملف ده هنتكلم عن:
- إيه هو Dependency Injection (DI)
- إزاي Riverpod بيعمل DI تلقائياً
- Dependencies بين الـ providers
- Dependency Graph
- Overriding للـ testing
- Best practices

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تفهم DI وليه مهم
- تعمل dependencies بين providers
- تستخدم ref.watch/read في providers
- تختبر الكود بـ overrides
- تتجنب circular dependencies

---

## 🔍 إيه هو Dependency Injection؟

**Dependency Injection** هو design pattern بيسمحلك:
- تفصل الـ code عن dependencies بتاعته
- تستبدل dependencies بسهولة (للتيست مثلاً)
- تكتب code قابل لإعادة الاستخدام

</div>

```dart
// Without DI - Tightly coupled
class UserService {
  final apiClient = ApiClient(); // Hard-coded dependency

  Future<User> getUser() async {
    return await apiClient.fetchUser();
  }
}

// With DI - Loosely coupled
class UserService {
  final ApiClient apiClient; // Injected dependency

  UserService(this.apiClient);

  Future<User> getUser() async {
    return await apiClient.fetchUser();
  }
}

// Now you can inject different implementations
final service1 = UserService(RealApiClient());
final service2 = UserService(MockApiClient()); // For testing
```

<div dir="rtl">

---

## 🎨 DI في Riverpod

في Riverpod، الـ DI **تلقائي** و **type-safe** و **compile-time verified**.

### مثال بسيط

</div>

```dart
// 1. Define dependencies
@riverpod
ApiClient apiClient(ApiClientRef ref) {
  return ApiClient(baseUrl: 'https://api.example.com');
}

// 2. Inject dependency automatically
@riverpod
class UserRepository extends _$UserRepository {
  @override
  Future<User> build() async {
    // Dependency injected via ref.watch
    final api = ref.watch(apiClientProvider);

    return await api.getUser();
  }
}

// 3. Use in widget
class UserProfile extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // All dependencies resolved automatically
    final userAsync = ref.watch(userRepositoryProvider);

    return userAsync.when(
      data: (user) => Text(user.name),
      loading: () => CircularProgressIndicator(),
      error: (error, stack) => Text('Error: $error'),
    );
  }
}
```

<div dir="rtl">

**المزايا:**
- ✅ No manual wiring
- ✅ Type-safe
- ✅ Compile-time verification
- ✅ Easy testing with overrides
- ✅ Automatic disposal of dependencies

---

## 🔗 أنواع الـ Dependencies

### 1. Direct Dependencies (ref.watch)

</div>

```dart
// ApiClient is a direct dependency
@riverpod
class UserRepository extends _$UserRepository {
  @override
  Future<User> build() async {
    // Direct dependency - rebuilds when apiClient changes
    final api = ref.watch(apiClientProvider);

    return await api.getUser();
  }
}
```

<div dir="rtl">

### 2. Transitive Dependencies (سلسلة)

</div>

```dart
// Three-level dependency chain
@riverpod
HttpClient httpClient(HttpClientRef ref) {
  return HttpClient();
}

@riverpod
ApiClient apiClient(ApiClientRef ref) {
  // Depends on httpClient
  final http = ref.watch(httpClientProvider);
  return ApiClient(http);
}

@riverpod
class UserRepository extends _$UserRepository {
  @override
  Future<User> build() async {
    // Depends on apiClient (which depends on httpClient)
    final api = ref.watch(apiClientProvider);
    return await api.getUser();
  }
}

// Dependency graph:
// UserRepository → ApiClient → HttpClient
```

<div dir="rtl">

### 3. Multiple Dependencies

</div>

```dart
@riverpod
class Dashboard extends _$Dashboard {
  @override
  Future<DashboardData> build() async {
    // Multiple dependencies
    final user = await ref.watch(userProvider.future);
    final posts = await ref.watch(postsProvider.future);
    final settings = ref.watch(settingsProvider);

    return DashboardData(
      user: user,
      posts: posts,
      settings: settings,
    );
  }
}
```

<div dir="rtl">

### 4. Conditional Dependencies

</div>

```dart
@riverpod
class ConditionalData extends _$ConditionalData {
  @override
  Future<Data> build() async {
    final isLoggedIn = ref.watch(isLoggedInProvider);

    if (isLoggedIn) {
      // Only create this dependency if logged in
      final user = await ref.watch(userProvider.future);
      return await fetchUserData(user.id);
    } else {
      return Data.guest();
    }
  }

  Future<Data> fetchUserData(String userId) async {
    // Implementation
    return Data.empty();
  }
}
```

<div dir="rtl">

---

## 🎯 Dependency Graph

Riverpod بيبني dependency graph تلقائياً ويتتبع العلاقات بين الـ providers.

### مثال: E-commerce App

</div>

```dart
// Low-level dependencies
@riverpod
HttpClient httpClient(HttpClientRef ref) {
  return HttpClient();
}

@riverpod
AuthToken authToken(AuthTokenRef ref) {
  return AuthToken.load();
}

// Mid-level dependencies
@riverpod
ApiClient apiClient(ApiClientRef ref) {
  final http = ref.watch(httpClientProvider);
  final token = ref.watch(authTokenProvider);

  return ApiClient(http: http, token: token);
}

// High-level dependencies
@riverpod
class ProductCatalog extends _$ProductCatalog {
  @override
  Future<List<Product>> build() async {
    final api = ref.watch(apiClientProvider);
    return await api.getProducts();
  }
}

@riverpod
class ShoppingCart extends _$ShoppingCart {
  @override
  List<CartItem> build() {
    // Watches products to validate cart items
    ref.watch(productCatalogProvider);

    return [];
  }

  void addProduct(Product product) {
    state = [...state, CartItem.fromProduct(product)];
  }
}

@riverpod
class Checkout extends _$Checkout {
  @override
  CheckoutState build() {
    // Multiple dependencies
    final cart = ref.watch(shoppingCartProvider);
    final user = ref.watch(currentUserProvider);

    return CheckoutState(
      items: cart,
      userId: user?.id,
    );
  }

  Future<void> placeOrder() async {
    final api = ref.watch(apiClientProvider);
    await api.placeOrder(state.items);
  }
}
```

<div dir="rtl">

**الـ Dependency Graph:**

</div>

```
HttpClient ──┐
             ├─→ ApiClient ──┐
AuthToken ───┘               │
                             ├─→ ProductCatalog ──┐
                             │                     │
                             │                     ├─→ ShoppingCart ──┐
                             │                                         │
CurrentUser ─────────────────┼─────────────────────────────────────────┤
                             │                                         │
                             └─→ Checkout ←───────────────────────────┘
```

<div dir="rtl">

---

## 🔄 ref.watch vs ref.read vs ref.listen في Providers

### ref.watch - للـ Reactive Dependencies

</div>

```dart
@riverpod
class FilteredProducts extends _$FilteredProducts {
  @override
  List<Product> build() {
    final allProducts = ref.watch(productsProvider);
    final filter = ref.watch(filterProvider);

    // Rebuilds when products OR filter changes
    return allProducts.where((p) => p.category == filter).toList();
  }
}
```

<div dir="rtl">

**متى تستخدمه:**
- ✅ لما تحتاج rebuild عند تغيير الـ dependency
- ✅ للـ computed values
- ✅ في الـ build method

### ref.read - للـ One-time Access

</div>

```dart
@riverpod
class OrderProcessor extends _$OrderProcessor {
  @override
  ProcessorState build() {
    return ProcessorState.idle();
  }

  Future<void> processOrder(Order order) async {
    // One-time read - no rebuild needed
    final api = ref.read(apiClientProvider);
    final userId = ref.read(currentUserIdProvider);

    await api.submitOrder(order, userId);
  }
}
```

<div dir="rtl">

**متى تستخدمه:**
- ✅ في methods (مش في build)
- ✅ لما مش محتاج rebuild
- ✅ للـ one-time operations

### ref.listen - للـ Side Effects

</div>

```dart
@riverpod
class NotificationManager extends _$NotificationManager {
  @override
  int build() {
    // Listen to new messages
    ref.listen(
      newMessagesProvider,
      (previous, next) {
        if (next.isNotEmpty) {
          _showNotification(next.first);
        }
      },
    );

    return 0;
  }

  void _showNotification(Message message) {
    // Show notification
  }
}
```

<div dir="rtl">

**متى تستخدمه:**
- ✅ للـ side effects في providers
- ✅ لما محتاج previous و next values
- ✅ مش عايز rebuild

---

## 🧪 Testing مع Overrides

واحدة من أقوى مزايا DI في Riverpod: سهولة الـ testing.

### مثال: Repository Testing

</div>

```dart
// Production code
@riverpod
ApiClient apiClient(ApiClientRef ref) {
  return RealApiClient();
}

@riverpod
class UserRepository extends _$UserRepository {
  @override
  Future<User> build() async {
    final api = ref.watch(apiClientProvider);
    return await api.getUser();
  }
}

// Test code
void main() {
  test('UserRepository returns user from API', () async {
    // Create container with mock
    final container = ProviderContainer(
      overrides: [
        // Override with mock
        apiClientProvider.overrideWithValue(
          MockApiClient(),
        ),
      ],
    );

    // Test
    final user = await container.read(userRepositoryProvider.future);

    expect(user.name, 'Test User');
  });
}

// Mock implementation
class MockApiClient implements ApiClient {
  @override
  Future<User> getUser() async {
    return User(id: '1', name: 'Test User');
  }
}
```

<div dir="rtl">

### مثال متقدم: Multiple Overrides

</div>

```dart
void main() {
  test('Checkout calculates total correctly', () async {
    final container = ProviderContainer(
      overrides: [
        // Override multiple dependencies
        productsProvider.overrideWith((ref) async {
          return [
            Product(id: '1', name: 'Product 1', price: 100),
            Product(id: '2', name: 'Product 2', price: 200),
          ];
        }),
        currentUserProvider.overrideWith((ref) {
          return User(id: 'test-user', name: 'Test');
        }),
        apiClientProvider.overrideWithValue(
          MockApiClient(),
        ),
      ],
    );

    // Test with all mocked dependencies
    final checkout = container.read(checkoutProvider);

    expect(checkout.total, 300);
  });
}
```

<div dir="rtl">

---

## ⚠️ Circular Dependencies

Riverpod **بيمنع** circular dependencies في compile time.

### مثال: Circular Dependency (مش هيشتغل)

</div>

```dart
// ❌ This will NOT compile
@riverpod
class ProviderA extends _$ProviderA {
  @override
  int build() {
    // Depends on B
    return ref.watch(providerBProvider);
  }
}

@riverpod
class ProviderB extends _$ProviderB {
  @override
  int build() {
    // Depends on A - CIRCULAR!
    return ref.watch(providerAProvider);
  }
}

// Error: Circular dependency detected
```

<div dir="rtl">

### الحل: استخدام Intermediate Provider

</div>

```dart
// ✅ Correct approach

// Shared state
@riverpod
class SharedCounter extends _$SharedCounter {
  @override
  int build() {
    return 0;
  }

  void increment() => state++;
}

// Both depend on shared state
@riverpod
int doubleCounter(DoubleCounterRef ref) {
  final count = ref.watch(sharedCounterProvider);
  return count * 2;
}

@riverpod
int tripleCounter(TripleCounterRef ref) {
  final count = ref.watch(sharedCounterProvider);
  return count * 3;
}
```

<div dir="rtl">

---

## 🎯 أمثلة عملية شاملة

### مثال 1: Multi-Layer Architecture

</div>

```dart
// Layer 1: Core Services
@riverpod
SecureStorage secureStorage(SecureStorageRef ref) {
  return SecureStorage();
}

@riverpod
HttpClient httpClient(HttpClientRef ref) {
  return HttpClient();
}

// Layer 2: API Services
@riverpod
class AuthService extends _$AuthService {
  @override
  AuthState build() {
    final storage = ref.watch(secureStorageProvider);
    final http = ref.watch(httpClientProvider);

    return AuthState(storage: storage, http: http);
  }

  Future<void> login(String email, String password) async {
    // Use dependencies
  }
}

// Layer 3: Repositories
@riverpod
class UserRepository extends _$UserRepository {
  @override
  Future<User?> build() async {
    // Watch auth to get token
    final auth = ref.watch(authServiceProvider);
    final http = ref.watch(httpClientProvider);

    if (!auth.isAuthenticated) {
      return null;
    }

    return await http.get('/user');
  }
}

// Layer 4: Use Cases
@riverpod
class UpdateProfile extends _$UpdateProfile {
  @override
  UpdateState build() {
    return UpdateState.idle();
  }

  Future<void> execute(ProfileData data) async {
    final repo = ref.read(userRepositoryProvider.notifier);
    await repo.updateProfile(data);
  }
}

// Layer 5: Presentation
class ProfilePage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userRepositoryProvider);
    final updateUseCase = ref.read(updateProfileProvider.notifier);

    return userAsync.when(
      data: (user) => ProfileForm(
        user: user,
        onSave: (data) => updateUseCase.execute(data),
      ),
      loading: () => CircularProgressIndicator(),
      error: (e, s) => ErrorWidget(e),
    );
  }
}
```

<div dir="rtl">

### مثال 2: Feature Flags مع DI

</div>

```dart
// Configuration
@riverpod
FeatureFlags featureFlags(FeatureFlagsRef ref) {
  return FeatureFlags.load();
}

// Service that depends on feature flags
@riverpod
class AnalyticsService extends _$AnalyticsService {
  @override
  Analytics build() {
    final flags = ref.watch(featureFlagsProvider);

    // Only enable analytics if feature is on
    if (flags.isEnabled('analytics')) {
      return RealAnalytics();
    } else {
      return NoOpAnalytics();
    }
  }

  void trackEvent(String event) {
    state.track(event);
  }
}

// Widget adapts based on feature flags
class HomePage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final flags = ref.watch(featureFlagsProvider);

    return Scaffold(
      body: Column(
        children: [
          if (flags.isEnabled('new_ui'))
            NewUserInterface()
          else
            OldUserInterface(),
        ],
      ),
    );
  }
}
```

<div dir="rtl">

### مثال 3: Environment-based Configuration

</div>

```dart
// Environment configuration
enum Environment { dev, staging, prod }

@riverpod
Environment environment(EnvironmentRef ref) {
  // Load from build config
  return Environment.dev;
}

@riverpod
String apiBaseUrl(ApiBaseUrlRef ref) {
  final env = ref.watch(environmentProvider);

  return switch (env) {
    Environment.dev => 'http://localhost:3000',
    Environment.staging => 'https://staging.api.com',
    Environment.prod => 'https://api.com',
  };
}

@riverpod
ApiClient apiClient(ApiClientRef ref) {
  final baseUrl = ref.watch(apiBaseUrlProvider);

  return ApiClient(baseUrl: baseUrl);
}

// Testing with different environments
void main() {
  test('API uses staging URL in staging environment', () {
    final container = ProviderContainer(
      overrides: [
        environmentProvider.overrideWithValue(Environment.staging),
      ],
    );

    final apiUrl = container.read(apiBaseUrlProvider);

    expect(apiUrl, 'https://staging.api.com');
  });
}
```

<div dir="rtl">

---

## 💡 Best Practices

### 1. Single Responsibility

</div>

```dart
// ✅ Good: Each provider has one job
@riverpod
HttpClient httpClient(HttpClientRef ref) {
  return HttpClient();
}

@riverpod
AuthToken authToken(AuthTokenRef ref) {
  return AuthToken.load();
}

@riverpod
ApiClient apiClient(ApiClientRef ref) {
  final http = ref.watch(httpClientProvider);
  final token = ref.watch(authTokenProvider);
  return ApiClient(http, token);
}

// ❌ Bad: Provider doing too much
@riverpod
Everything everything(EverythingRef ref) {
  final http = HttpClient();
  final token = AuthToken.load();
  final api = ApiClient(http, token);
  final user = api.getUser();
  // Too many responsibilities!
  return Everything(http, token, api, user);
}
```

<div dir="rtl">

### 2. Explicit Dependencies

</div>

```dart
// ✅ Good: Clear dependencies
@riverpod
class UserService extends _$UserService {
  @override
  Service build() {
    final api = ref.watch(apiProvider);
    final cache = ref.watch(cacheProvider);

    return Service(api: api, cache: cache);
  }
}

// ❌ Bad: Hidden dependencies
@riverpod
class UserService extends _$UserService {
  @override
  Service build() {
    // Hidden global access
    return Service(api: GlobalApi.instance);
  }
}
```

<div dir="rtl">

### 3. Layer Separation

</div>

```dart
// ✅ Good: Clear layers
// Data layer
@riverpod
UserRepository userRepo(UserRepoRef ref) {
  return UserRepository(ref.watch(apiProvider));
}

// Domain layer
@riverpod
class GetUserUseCase extends _$GetUserUseCase {
  @override
  Future<User> build(String id) async {
    final repo = ref.watch(userRepoProvider);
    return await repo.getUser(id);
  }
}

// Presentation layer
class UserPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final user = ref.watch(getUserUseCaseProvider('123'));
    return Text(user.name);
  }
}
```

<div dir="rtl">

### 4. Testability First

</div>

```dart
// ✅ Good: Easy to test
@riverpod
class PaymentService extends _$PaymentService {
  @override
  Service build() {
    // All dependencies injected
    final api = ref.watch(apiProvider);
    final validator = ref.watch(validatorProvider);

    return Service(api: api, validator: validator);
  }
}

// Test
test('payment validation works', () {
  final container = ProviderContainer(
    overrides: [
      validatorProvider.overrideWithValue(MockValidator()),
    ],
  );

  // Easy to test!
});
```

<div dir="rtl">

---

## 📝 ملخص

**Dependency Injection في Riverpod:**
- ✅ تلقائي و type-safe
- ✅ Compile-time verification
- ✅ No manual wiring
- ✅ Easy testing with overrides
- ✅ Automatic cleanup

**Key Methods:**
- `ref.watch` → Reactive dependencies
- `ref.read` → One-time access
- `ref.listen` → Side effects

**Best Practices:**
- Single responsibility per provider
- Explicit dependencies
- Clear layer separation
- Design for testability

---

## 🔗 الخطوة الجاية

في الملف الجاي هنتكلم عن:
- **Family Modifier** بالتفصيل
- إزاي تعمل parameterized providers
- Cache management
- Best practices

---

## 📚 المصادر

- [Dependency Injection](https://riverpod.dev/docs/concepts/provider_dependencies)
- [Testing with Overrides](https://riverpod.dev/docs/cookbooks/testing)

</div>
