<div dir="rtl">

# Advanced Provider Patterns - نظرة عامة

**المستوى**: 🔴 متقدم

## 🎯 الهدف من هذا القسم

في الـ Sections السابقة تعلمنا:
- ✅ أساسيات Riverpod 3
- ✅ كل أنواع الـ Providers
- ✅ Code Generation
- ✅ Async Data Handling

دلوقتي في **Section 08** هنتعلم الأنماط المتقدمة للعمل مع Providers:
- 🔗 **Provider Dependencies** - ربط providers ببعض
- 🎛️ **Provider Families** - parameters و dynamic providers
- 🧩 **Combining Providers** - derived/computed state
- ⚡ **Select Optimization** - تقليل الـ rebuilds
- 🔄 **Provider Scoping** - overriding providers
- 🏗️ **Advanced Patterns** - real-world architectures

---

## 📚 محتوى القسم

### 1. Provider Dependencies (01-provider-dependencies.md)
**المفاهيم:**
- `ref.watch()` داخل providers
- Provider chains
- Automatic updates
- Dependency graphs
- Error propagation

**مثال:**
```dart
@riverpod
Future<Weather> weather(WeatherRef ref) async {
  // Watch city provider
  final city = ref.watch(cityProvider);

  // Automatically refetches when city changes!
  return await api.getWeather(city);
}
```

---

### 2. Provider Families (02-provider-families.md)
**المفاهيم:**
- Parameters في providers
- Multiple provider instances
- الفرق بين Riverpod 2 و 3
- Memory management
- hashCode و equality

**مثال:**
```dart
// Riverpod 3 - No .family needed!
@riverpod
Future<Product> product(ProductRef ref, String id) async {
  return await api.getProduct(id);
}

// Usage
final productAsync = ref.watch(productProvider('product-123'));
```

---

### 3. Combining Providers (03-combining-providers.md)
**المفاهيم:**
- Derived/computed state
- Watching multiple providers
- Transforming data
- Complex calculations
- State aggregation

**مثال:**
```dart
@riverpod
double totalPrice(TotalPriceRef ref) {
  final cart = ref.watch(cartProvider);
  final discount = ref.watch(discountProvider);

  final subtotal = cart.fold(0.0, (sum, item) => sum + item.price);
  return subtotal * (1 - discount);
}
```

---

### 4. Select Optimization (04-select-optimization.md)
**المفاهيم:**
- `ref.watch().select()`
- Partial rebuilds
- Performance optimization
- When to use select
- Benchmarking

**مثال:**
```dart
// Rebuild only when name changes (not age, email, etc.)
class UserNameWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final name = ref.watch(userProvider.select((user) => user.name));

    return Text(name);
  }
}
```

---

### 5. Provider Scoping (05-provider-scoping.md)
**المفاهيم:**
- ProviderScope
- Overriding providers
- Testing with overrides
- Multi-tenancy
- Feature flags

**مثال:**
```dart
ProviderScope(
  overrides: [
    // Override for testing
    userProvider.overrideWith((ref) => User.mock()),
  ],
  child: MyApp(),
)
```

---

## 🎨 ما الذي ستتعلمه

بعد إنهاء هذا القسم، ستكون قادراً على:

### 1. بناء Provider Graphs معقدة
```dart
@riverpod
Future<Dashboard> dashboard(DashboardRef ref) async {
  final user = await ref.watch(userProvider.future);
  final stats = await ref.watch(statsProvider(user.id).future);
  final notifications = await ref.watch(notificationsProvider(user.id).future);

  return Dashboard(
    user: user,
    stats: stats,
    notifications: notifications,
  );
}
```

### 2. تحسين الـ Performance
```dart
// ❌ BAD - Rebuilds on ANY user change
final user = ref.watch(userProvider);
return Text(user.name);

// ✅ GOOD - Rebuilds only when name changes
final name = ref.watch(userProvider.select((u) => u.name));
return Text(name);
```

### 3. استخدام Parameters بذكاء
```dart
// One provider, multiple instances!
@riverpod
class TodoController extends _$TodoController {
  @override
  Future<Todo> build(String todoId) async {
    return await api.getTodo(todoId);
  }

  Future<void> toggleComplete() async { ... }
}

// Usage
ref.watch(todoControllerProvider('todo-1'));
ref.watch(todoControllerProvider('todo-2'));
ref.watch(todoControllerProvider('todo-3'));
```

### 4. كتابة Testable Code
```dart
void main() {
  testWidgets('displays user name', (tester) async {
    await tester.pumpWidget(
      ProviderScope(
        overrides: [
          userProvider.overrideWith((ref) => User(name: 'Test User')),
        ],
        child: MyApp(),
      ),
    );

    expect(find.text('Test User'), findsOneWidget);
  });
}
```

---

## 🔑 المفاهيم الأساسية

### 1. ref.watch vs ref.read vs ref.listen

| Method | Use Case | في الـ Providers | في الـ Widgets |
|--------|----------|------------------|----------------|
| **ref.watch** | Subscribe to changes | ✅ في build() | ✅ في build() |
| **ref.read** | One-time read | ✅ في methods | ❌ استخدم listen |
| **ref.listen** | Side effects | ✅ Callbacks | ✅ Callbacks |

### 2. Provider Lifecycle

```dart
@riverpod
Future<Data> data(DataRef ref) async {
  print('1. Provider created');

  ref.onDispose(() {
    print('4. Provider disposed');
  });

  final data = await api.getData();
  print('2. Data loaded');

  return data;
  // 3. Data returned to watchers
}
```

### 3. Parameter Equality

```dart
// ✅ GOOD - int has proper equality
@riverpod
Future<User> user(UserRef ref, int userId) async {
  return await api.getUser(userId);
}

// ⚠️ CAREFUL - Custom class needs ==
@riverpod
Future<List<Todo>> todos(TodosRef ref, TodoFilter filter) async {
  return await api.getTodos(filter);
}

class TodoFilter {
  final String category;
  final bool completed;

  TodoFilter(this.category, this.completed);

  // MUST implement == and hashCode!
  @override
  bool operator ==(Object other) =>
      other is TodoFilter &&
      other.category == category &&
      other.completed == completed;

  @override
  int get hashCode => Object.hash(category, completed);
}
```

---

## 🎯 Best Practices Preview

### 1. Always Use AutoDispose with Families
```dart
// Code generation defaults to autoDispose ✅
@riverpod
Future<Product> product(ProductRef ref, String id) async {
  return await api.getProduct(id);
}
// Automatically disposed when no listeners
```

### 2. Use Select for Specific Properties
```dart
// Only rebuild when specific field changes
final isAdmin = ref.watch(userProvider.select((u) => u.isAdmin));
```

### 3. Keep Provider Logic Simple
```dart
// ✅ GOOD - Each provider has one responsibility
@riverpod
Future<User> user(UserRef ref) async => await api.getUser();

@riverpod
bool isAdmin(IsAdminRef ref) {
  final user = ref.watch(userProvider).value;
  return user?.role == 'admin';
}

// ❌ BAD - Too much logic in one provider
@riverpod
Future<Map<String, dynamic>> userAndPermissions(UserAndPermissionsRef ref) async {
  final user = await api.getUser();
  final permissions = await api.getPermissions(user.id);
  final settings = await api.getSettings(user.id);
  return {'user': user, 'permissions': permissions, 'settings': settings};
}
```

### 4. Document Complex Dependencies
```dart
/// Provides the current user's dashboard data.
///
/// **Dependencies:**
/// - [userProvider] - Current authenticated user
/// - [statsProvider] - User statistics
/// - [notificationsProvider] - User notifications
///
/// **Updates when:**
/// - User changes (login/logout)
/// - Stats are refreshed
/// - New notification arrives
@riverpod
Future<Dashboard> dashboard(DashboardRef ref) async {
  // Implementation
}
```

---

## 📊 ما الفرق عن Riverpod 2؟

### في Riverpod 2:
```dart
// Had to use .family modifier
final productProvider = FutureProvider.family<Product, String>((ref, id) async {
  return await api.getProduct(id);
});

// Multiple modifiers were verbose
final productProvider = FutureProvider.autoDispose.family<Product, String>(
  (ref, id) async {
    return await api.getProduct(id);
  },
);
```

### في Riverpod 3:
```dart
// No .family needed - just add parameters!
@riverpod
Future<Product> product(ProductRef ref, String id) async {
  return await api.getProduct(id);
}
// AutoDispose by default ✅
```

---

## 🚀 Real-World Scenarios

في هذا القسم سنبني:

### 1. E-Commerce Cart System
```dart
@riverpod
class Cart extends _$Cart {
  @override
  List<CartItem> build() => [];

  void addItem(Product product) { ... }
  void removeItem(String productId) { ... }
}

@riverpod
double cartSubtotal(CartSubtotalRef ref) {
  final items = ref.watch(cartProvider);
  return items.fold(0.0, (sum, item) => sum + item.total);
}

@riverpod
double cartTotal(CartTotalRef ref) {
  final subtotal = ref.watch(cartSubtotalProvider);
  final tax = ref.watch(taxProvider);
  final shipping = ref.watch(shippingProvider);

  return subtotal + tax + shipping;
}
```

### 2. Multi-User Chat Application
```dart
@riverpod
Stream<List<Message>> chatMessages(
  ChatMessagesRef ref,
  String chatId,
) async* {
  final socket = await ref.watch(socketProvider.future);

  yield* socket
      .channel('chat:$chatId')
      .stream<List<Message>>();
}

@riverpod
int unreadCount(UnreadCountRef ref, String chatId) {
  final messages = ref.watch(chatMessagesProvider(chatId)).value ?? [];
  final lastRead = ref.watch(lastReadProvider(chatId));

  return messages.where((m) => m.timestamp > lastRead).length;
}
```

### 3. Search with Debounce
```dart
@riverpod
class SearchQuery extends _$SearchQuery {
  @override
  String build() => '';

  void update(String query) {
    state = query;
  }
}

@riverpod
Future<List<Product>> searchResults(SearchResultsRef ref) async {
  final query = ref.watch(searchQueryProvider);

  if (query.isEmpty) return [];

  // Debounce
  await Future.delayed(const Duration(milliseconds: 500));

  // Check if query changed during delay
  if (query != ref.read(searchQueryProvider)) {
    return [];
  }

  return await api.search(query);
}
```

---

## 🎓 Prerequisites

قبل ما تبدأ القسم ده، تأكد إنك فاهم:
- ✅ Section 03-05: Riverpod Basics
- ✅ Section 06: Code Generation
- ✅ Section 07: Async Data Handling

---

## 📖 كيف تستخدم هذا القسم

### للمبتدئين في Advanced Patterns:
1. ابدأ بـ **Provider Dependencies** (01)
2. افهم **Provider Families** (02)
3. تعلم **Combining Providers** (03)
4. اقرأ **Select Optimization** (04) عند الحاجة
5. ارجع لـ **Provider Scoping** (05) عند Testing

### للـ Experienced Developers:
- اقرأ الملفات اللي محتاجها بس
- استخدم الأمثلة كـ reference
- راجع الـ Best Practices

### للـ Performance Optimization:
- ركز على **Select Optimization** (04)
- راجع **Combining Providers** (03) patterns
- استخدم DevTools للـ profiling

---

## 🎯 Goals

بنهاية هذا القسم، المفروض تقدر:
- ✅ تبني provider dependency graphs معقدة
- ✅ تستخدم parameters بكفاءة
- ✅ تحسن الـ performance باستخدام select
- ✅ تكتب testable و maintainable code
- ✅ تفهم متى تستخدم كل pattern
- ✅ تطبق real-world architectures

---

## 📚 المصادر

- [Providers | Riverpod](https://riverpod.dev/docs/concepts2/providers)
- [Family | Riverpod](https://riverpod.dev/docs/concepts2/family)
- [Refs | Riverpod](https://riverpod.dev/docs/concepts2/refs)
- [How to reduce provider/widget rebuilds | Riverpod](https://riverpod.dev/docs/how_to/select)
- [What's new in Riverpod 3.0 | Riverpod](https://riverpod.dev/docs/whats_new)

---

## 🚦 Let's Go!

جاهز لتعلم Advanced Provider Patterns؟

**الخطوة الأولى:** افتح `01-provider-dependencies.md` 📖

</div>
