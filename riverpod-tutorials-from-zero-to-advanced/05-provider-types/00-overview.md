<div dir="rtl">

# نظرة شاملة على أنواع Providers

**المستوى**: 🟡 متوسط

## 📌 المفاهيم الأساسية

في الملف ده هنتكلم عن:
- كل أنواع Providers المتاحة في Riverpod 3
- متى تستخدم كل نوع
- الفرق بين الأنواع المختلفة
- قواعد الاختيار الصحيح

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تعرف كل أنواع Providers المتاحة
- تختار النوع المناسب لكل use case
- تفهم الفرق بين كل نوع
- تعرف متى تستخدم إيه

---

## 🎨 أنواع Providers في Riverpod 3

في Riverpod 3، عندنا **7 أنواع أساسية** من Providers (Classic Syntax):

</div>

```
┌─────────────────────────────────────────────────────┐
│                  Riverpod Providers                  │
├─────────────────────────────────────────────────────┤
│                                                      │
│  1. Provider           → Read-only values            │
│  2. StateProvider      → Simple mutable state        │
│  3. FutureProvider     → One-time async data         │
│  4. StreamProvider     → Continuous async data       │
│  5. NotifierProvider   → Complex sync state          │
│  6. AsyncNotifierProvider → Complex async state      │
│  7. StreamNotifierProvider → Complex stream state    │
│                                                      │
└─────────────────────────────────────────────────────┘
```

<div dir="rtl">

---

## 📊 جدول المقارنة السريع

| النوع | متى تستخدمه | مثال |
|------|------------|------|
| **Provider** | قيم ثابتة أو محسوبة | Config, API client, Computed values |
| **StateProvider** | State بسيط (primitives) | Counter, isDarkMode, selectedTab |
| **FutureProvider** | Async مرة واحدة | Fetch user data, Load settings |
| **StreamProvider** | Async مستمر | Chat messages, Location updates |
| **NotifierProvider** | State معقد (sync) | Shopping cart, Form state |
| **AsyncNotifierProvider** | State معقد (async) | Todo list with API, User profile |
| **StreamNotifierProvider** | State معقد (stream) | Real-time notifications, Live feed |

---

## 1️⃣ Provider - القيم الثابتة

**الاستخدام:** للقيم اللي **مش بتتغير** أو القيم المحسوبة.

</div>

```dart
// Configuration
final apiUrlProvider = Provider<String>((ref) {
  return 'https://api.example.com';
});

// Service
final apiClientProvider = Provider<ApiClient>((ref) {
  final url = ref.watch(apiUrlProvider);
  return ApiClient(baseUrl: url);
});

// Computed value
final totalPriceProvider = Provider<double>((ref) {
  final items = ref.watch(cartItemsProvider);
  return items.fold(0.0, (sum, item) => sum + item.price);
});
```

<div dir="rtl">

**متى تستخدمه:**
- ✅ Configuration values
- ✅ Services (API clients, Database)
- ✅ Computed values
- ✅ Dependency injection

**متى ما تستخدموش:**
- ❌ لو محتاج تغير القيمة (use StateProvider)
- ❌ لو async (use FutureProvider)

---

## 2️⃣ StateProvider - State البسيط

**الاستخدام:** لـ state بسيط (primitives) اللي بيتغير.

</div>

```dart
// Counter
final counterProvider = StateProvider<int>((ref) => 0);

// Dark mode
final isDarkModeProvider = StateProvider<bool>((ref) => false);

// Selected tab
final selectedTabProvider = StateProvider<int>((ref) => 0);

// Usage
class CounterButton extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);

    return ElevatedButton(
      onPressed: () {
        ref.read(counterProvider.notifier).state++;
      },
      child: Text('Count: $count'),
    );
  }
}
```

<div dir="rtl">

**متى تستخدمه:**
- ✅ Primitives بسيطة (int, bool, String)
- ✅ UI state (selected index, filters)
- ✅ Settings toggles

**متى ما تستخدموش:**
- ❌ لو State معقد (use NotifierProvider)
- ❌ لو محتاج business logic (use NotifierProvider)

---

## 3️⃣ FutureProvider - Async المرة الواحدة

**الاستخدام:** لـ async operations اللي بتحصل **مرة واحدة**.

</div>

```dart
// Fetch user data
final userProvider = FutureProvider<User>((ref) async {
  final api = ref.watch(apiClientProvider);
  return await api.getUser();
});

// Load configuration
final configProvider = FutureProvider<AppConfig>((ref) async {
  final prefs = await SharedPreferences.getInstance();
  return AppConfig.fromJson(prefs.getString('config') ?? '{}');
});

// Usage
class UserProfile extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider);

    return userAsync.when(
      data: (user) => Text('Hello, ${user.name}'),
      loading: () => CircularProgressIndicator(),
      error: (error, stack) => Text('Error: $error'),
    );
  }
}
```

<div dir="rtl">

**متى تستخدمه:**
- ✅ API calls (one-time fetch)
- ✅ Loading data from storage
- ✅ Initialization tasks

**متى ما تستخدموش:**
- ❌ لو محتاج refresh (use AsyncNotifierProvider)
- ❌ لو continuous updates (use StreamProvider)

---

## 4️⃣ StreamProvider - Async المستمر

**الاستخدام:** لـ async data اللي بيتحدث **باستمرار**.

</div>

```dart
// Chat messages
final messagesProvider = StreamProvider<List<Message>>((ref) {
  return chatService.messagesStream();
});

// Location updates
final locationProvider = StreamProvider<Position>((ref) {
  return Geolocator.getPositionStream();
});

// Firebase auth state
final authStateProvider = StreamProvider<User?>((ref) {
  return FirebaseAuth.instance.authStateChanges();
});

// Usage
class ChatMessages extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final messagesAsync = ref.watch(messagesProvider);

    return messagesAsync.when(
      data: (messages) => ListView.builder(
        itemCount: messages.length,
        itemBuilder: (context, index) => MessageTile(messages[index]),
      ),
      loading: () => CircularProgressIndicator(),
      error: (error, stack) => Text('Error: $error'),
    );
  }
}
```

<div dir="rtl">

**متى تستخدمه:**
- ✅ Real-time data (chat, notifications)
- ✅ Continuous updates (location, sensors)
- ✅ Firebase/Websocket streams

**متى ما تستخدموش:**
- ❌ لو one-time fetch (use FutureProvider)
- ❌ لو محتاج methods (use StreamNotifierProvider)

---

## 5️⃣ NotifierProvider - State المعقد (Sync)

**الاستخدام:** لـ **synchronous** state معقد مع business logic.

</div>

```dart
// Shopping cart
class ShoppingCartNotifier extends Notifier<List<CartItem>> {
  @override
  List<CartItem> build() {
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

  double get total {
    return state.fold(0.0, (sum, item) => sum + item.price);
  }
}

final shoppingCartProvider = NotifierProvider<ShoppingCartNotifier, List<CartItem>>(
  () => ShoppingCartNotifier(),
);

// Usage
class CartButton extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final cart = ref.watch(shoppingCartProvider);

    return IconButton(
      icon: Badge(
        label: Text('${cart.length}'),
        child: Icon(Icons.shopping_cart),
      ),
      onPressed: () {
        // Access methods
        ref.read(shoppingCartProvider.notifier).clear();
      },
    );
  }
}
```

<div dir="rtl">

**متى تستخدمه:**
- ✅ State معقد (objects, lists)
- ✅ محتاج methods (add, remove, update)
- ✅ Business logic في الـ state
- ✅ Synchronous operations

**متى ما تستخدموش:**
- ❌ لو State بسيط (use StateProvider)
- ❌ لو async operations (use AsyncNotifierProvider)

---

## 6️⃣ AsyncNotifierProvider - State المعقد (Async)

**الاستخدام:** لـ **asynchronous** state معقد مع business logic.

</div>

```dart
// Todo list with API
class TodosNotifier extends AsyncNotifier<List<Todo>> {
  @override
  Future<List<Todo>> build() async {
    // Initial load
    return await _fetchTodos();
  }

  Future<void> addTodo(String title) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      await api.addTodo(title);
      return await _fetchTodos();
    });
  }

  Future<void> toggleTodo(String id) async {
    // Optimistic update
    final previousState = state;

    state = AsyncData(
      state.value!.map((todo) {
        if (todo.id == id) {
          return todo.copyWith(isCompleted: !todo.isCompleted);
        }
        return todo;
      }).toList(),
    );

    try {
      await api.toggleTodo(id);
    } catch (e) {
      // Revert on error
      state = previousState;
    }
  }

  Future<List<Todo>> _fetchTodos() async {
    return await api.getTodos();
  }
}

final todosProvider = AsyncNotifierProvider<TodosNotifier, List<Todo>>(
  () => TodosNotifier(),
);

// Usage
class TodosList extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final todosAsync = ref.watch(todosProvider);

    return todosAsync.when(
      data: (todos) => ListView.builder(
        itemCount: todos.length,
        itemBuilder: (context, index) => TodoTile(
          todo: todos[index],
          onToggle: () {
            ref.read(todosProvider.notifier).toggleTodo(todos[index].id);
          },
        ),
      ),
      loading: () => CircularProgressIndicator(),
      error: (error, stack) => Text('Error: $error'),
    );
  }
}
```

<div dir="rtl">

**متى تستخدمه:**
- ✅ State معقد مع async operations
- ✅ CRUD operations (Create, Read, Update, Delete)
- ✅ محتاج refresh/reload
- ✅ Optimistic updates

**متى ما تستخدموش:**
- ❌ لو one-time fetch (use FutureProvider)
- ❌ لو synchronous (use NotifierProvider)

---

## 7️⃣ StreamNotifierProvider - State المعقد (Stream)

**الاستخدام:** لـ **stream-based** state معقد مع business logic.

</div>

```dart
// Real-time notifications
class NotificationsNotifier extends StreamNotifier<List<Notification>> {
  @override
  Stream<List<Notification>> build() {
    return notificationService.notificationsStream();
  }

  void markAsRead(String notificationId) {
    notificationService.markAsRead(notificationId);
    // Stream will automatically update
  }

  void clearAll() {
    notificationService.clearAll();
    // Stream will automatically update
  }
}

final notificationsProvider = StreamNotifierProvider<NotificationsNotifier, List<Notification>>(
  () => NotificationsNotifier(),
);
```

<div dir="rtl">

**متى تستخدمه:**
- ✅ Real-time data مع methods
- ✅ Stream + business logic
- ✅ محتاج تتحكم في الـ stream

**متى ما تستخدموش:**
- ❌ لو مش محتاج methods (use StreamProvider)
- ❌ لو one-time async (use AsyncNotifierProvider)

---

## 🎯 Decision Tree - إزاي تختار؟

</div>

```
هل الـ data بتتغير؟
├─ لا → Provider
│   ├─ Config
│   ├─ Services
│   └─ Computed values
│
└─ نعم → هل فيها async operations؟
    ├─ لا → هل State بسيط؟
    │   ├─ نعم → StateProvider (int, bool, String)
    │   └─ لا → NotifierProvider (objects, lists + methods)
    │
    └─ نعم → هل بتحصل مرة واحدة؟
        ├─ نعم → هل محتاج refresh؟
        │   ├─ لا → FutureProvider
        │   └─ نعم → AsyncNotifierProvider
        │
        └─ لا (continuous) → هل محتاج methods؟
            ├─ لا → StreamProvider
            └─ نعم → StreamNotifierProvider
```

<div dir="rtl">

---

## 📊 أمثلة من الواقع

### مثال 1: تطبيق تسوق

</div>

```dart
// ✅ Provider - API Client (لا يتغير)
final apiClientProvider = Provider<ApiClient>((ref) {
  return ApiClient(baseUrl: 'https://api.shop.com');
});

// ✅ StateProvider - Selected category (بسيط ومتغير)
final selectedCategoryProvider = StateProvider<String>((ref) => 'all');

// ✅ FutureProvider - Categories (one-time load)
final categoriesProvider = FutureProvider<List<Category>>((ref) async {
  final api = ref.watch(apiClientProvider);
  return await api.getCategories();
});

// ✅ AsyncNotifierProvider - Products (async مع methods)
final productsProvider = AsyncNotifierProvider<ProductsNotifier, List<Product>>(
  () => ProductsNotifier(),
);

// ✅ NotifierProvider - Shopping cart (sync مع methods)
final cartProvider = NotifierProvider<CartNotifier, List<CartItem>>(
  () => CartNotifier(),
);

// ✅ Provider - Total price (computed)
final totalPriceProvider = Provider<double>((ref) {
  final cart = ref.watch(cartProvider);
  return cart.fold(0.0, (sum, item) => sum + item.price);
});
```

<div dir="rtl">

### مثال 2: تطبيق شات

</div>

```dart
// ✅ Provider - Chat service
final chatServiceProvider = Provider<ChatService>((ref) {
  return ChatService();
});

// ✅ StreamProvider - Auth state
final authStateProvider = StreamProvider<User?>((ref) {
  return FirebaseAuth.instance.authStateChanges();
});

// ✅ StreamNotifierProvider - Messages (stream مع methods)
final messagesProvider = StreamNotifierProvider<MessagesNotifier, List<Message>>(
  () => MessagesNotifier(),
);

// ✅ AsyncNotifierProvider - Typing indicator (async state)
final typingProvider = AsyncNotifierProvider<TypingNotifier, Set<String>>(
  () => TypingNotifier(),
);
```

<div dir="rtl">

---

## ⚠️ أخطاء شائعة

### ❌ استخدام StateProvider لـ state معقد

</div>

```dart
// ❌ WRONG - StateProvider for complex state
final todosProvider = StateProvider<List<Todo>>((ref) => []);

// Usage - BAD!
ref.read(todosProvider.notifier).state = [
  ...ref.read(todosProvider),
  newTodo,
]; // طويل ومش واضح

// ✅ CORRECT - NotifierProvider/AsyncNotifierProvider
class TodosNotifier extends AsyncNotifier<List<Todo>> {
  @override
  Future<List<Todo>> build() async => [];

  Future<void> addTodo(Todo todo) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      await api.addTodo(todo);
      return [...state.value!, todo];
    });
  }
}
```

<div dir="rtl">

### ❌ استخدام FutureProvider لما محتاج refresh

</div>

```dart
// ❌ WRONG - Can't refresh easily
final userProvider = FutureProvider<User>((ref) async {
  return await api.getUser();
});

// ✅ CORRECT - Use AsyncNotifierProvider
class UserNotifier extends AsyncNotifier<User> {
  @override
  Future<User> build() async {
    return await api.getUser();
  }

  Future<void> refresh() async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() => api.getUser());
  }

  Future<void> updateProfile(UserData data) async {
    // Easy to add methods!
  }
}
```

<div dir="rtl">

### ❌ استخدام Provider لحاجة بتتغير

</div>

```dart
// ❌ WRONG - Can't change!
final counterProvider = Provider<int>((ref) => 0);
// مفيش طريقة نغير القيمة!

// ✅ CORRECT - Use StateProvider
final counterProvider = StateProvider<int>((ref) => 0);
// يمكن التغيير: ref.read(counterProvider.notifier).state++;
```

<div dir="rtl">

---

## 💡 Best Practices

### 1. ابدأ بالأبسط دايماً

</div>

```dart
// Start simple
final counterProvider = StateProvider<int>((ref) => 0);

// Grow when needed
class CounterNotifier extends Notifier<int> {
  @override
  int build() => 0;

  void increment() => state++;
  void decrement() => state--;
  void reset() => state = 0;
}
```

<div dir="rtl">

### 2. استخدم Provider للحاجات اللي مش بتتغير

</div>

```dart
// ✅ Good - Services, config, computed
final apiProvider = Provider((ref) => ApiClient());
final totalProvider = Provider((ref) {
  final items = ref.watch(itemsProvider);
  return items.fold(0.0, (sum, item) => sum + item.price);
});
```

<div dir="rtl">

### 3. StateProvider للـ primitives البسيطة بس

</div>

```dart
// ✅ Good - Simple primitives
final isDarkModeProvider = StateProvider<bool>((ref) => false);
final selectedTabProvider = StateProvider<int>((ref) => 0);

// ❌ Bad - Complex objects
final userProvider = StateProvider<User>((ref) => User());
// Use NotifierProvider instead!
```

<div dir="rtl">

### 4. NotifierProvider لما محتاج methods

</div>

```dart
// ✅ Good - Methods + state
class CartNotifier extends Notifier<List<CartItem>> {
  @override
  List<CartItem> build() => [];

  void addItem(Product product) { /* ... */ }
  void removeItem(String id) { /* ... */ }
  void updateQuantity(String id, int qty) { /* ... */ }
  void clear() { /* ... */ }
}
```

<div dir="rtl">

---

## 📝 ملخص

| النوع | Use Case | مثال |
|------|---------|------|
| Provider | قيم ثابتة | Config, Services |
| StateProvider | Primitives بسيطة | Counter, isDarkMode |
| FutureProvider | Async مرة واحدة | Initial data fetch |
| StreamProvider | Async مستمر | Chat, Location |
| NotifierProvider | State معقد (sync) | Cart, Form |
| AsyncNotifierProvider | State معقد (async) | Todos, User profile |
| StreamNotifierProvider | Stream + methods | Notifications |

**القاعدة الذهبية:** ابدأ بالأبسط، واطور لما تحتاج!

---

## 🔗 الخطوة الجاية

في الملفات الجاية هنتعمق في كل نوع:
- **Provider** - القيم الثابتة بالتفصيل
- **StateProvider** - State البسيط
- **FutureProvider** - Async data
- **StreamProvider** - Real-time data
- **NotifierProvider** - State معقد sync
- **AsyncNotifierProvider** - State معقد async
- **دليل الاختيار** - Decision guide شامل

---

## 📚 المصادر

- [Provider Types - Riverpod Docs](https://riverpod.dev/docs/providers/provider)
- [Choosing a Provider](https://riverpod.dev/docs/concepts/providers)
- [Provider vs Notifier](https://riverpod.dev/docs/concepts/about_code_generation)

</div>
