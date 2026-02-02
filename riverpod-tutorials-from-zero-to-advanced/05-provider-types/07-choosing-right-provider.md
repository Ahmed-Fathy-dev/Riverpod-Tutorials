<div dir="rtl">

# دليل اختيار النوع المناسب من Providers

**المستوى**: 🟡 متوسط

## 📌 المفاهيم الأساسية

في الملف ده هنتكلم عن:
- إزاي تختار النوع المناسب من Providers
- Decision tree شامل
- أمثلة من الواقع
- Common scenarios
- Upgrade paths

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تختار النوع المناسب من Provider لأي use case
- تعرف متى تنتقل من نوع لنوع
- تتجنب الاختيارات الخاطئة
- تطبق best practices

---

## 🎯 Decision Tree

</div>

```
هل البيانات بتتغير؟
│
├─ لا (Read-only)
│  └─ استخدم Provider
│     ├─ Config
│     ├─ Services
│     └─ Computed values
│
└─ نعم (Mutable)
   │
   ├─ هل فيها async operations؟
   │  │
   │  ├─ لا (Synchronous)
   │  │  │
   │  │  ├─ هل State بسيط (primitive)?
   │  │  │  ├─ نعم → StateProvider
   │  │  │  │   ├─ int, bool, String
   │  │  │  │   ├─ Selected index
   │  │  │  │   └─ Toggle states
   │  │  │  │
   │  │  │  └─ لا (Complex) → NotifierProvider
   │  │  │      ├─ Objects
   │  │  │      ├─ Lists
   │  │  │      └─ محتاج methods
   │  │  │
   │  └─ نعم (Asynchronous)
   │     │
   │     ├─ هل بتحصل مرة واحدة؟
   │     │  │
   │     │  ├─ نعم
   │     │  │  │
   │     │  │  ├─ محتاج refresh/methods؟
   │     │  │  │  ├─ لا → FutureProvider
   │     │  │  │  │   ├─ Initial load
   │     │  │  │  │   └─ Config loading
   │     │  │  │  │
   │     │  │  │  └─ نعم → AsyncNotifierProvider
   │     │  │  │      ├─ CRUD operations
   │     │  │  │      ├─ محتاج refresh
   │     │  │  │      └─ Optimistic updates
   │     │  │  │
   │     │  └─ لا (Continuous)
   │     │     │
   │     │     ├─ محتاج methods؟
   │     │     │  ├─ لا → StreamProvider
   │     │     │  │   ├─ Firebase streams
   │     │     │  │   ├─ WebSocket
   │     │     │  │   └─ Location updates
   │     │     │  │
   │     │     │  └─ نعم → StreamNotifierProvider
   │     │     │      ├─ Stream + methods
   │     │     │      └─ Control over stream
```

<div dir="rtl">

---

## 📊 جدول المقارنة الشامل

| السيناريو | النوع المناسب | السبب |
|-----------|---------------|-------|
| API URL | Provider | Read-only config |
| API Client | Provider | Service (DI) |
| Dark mode toggle | StateProvider | Simple boolean |
| Selected tab | StateProvider | Simple int |
| Counter | StateProvider/NotifierProvider | Start simple, upgrade if needed |
| Shopping cart | NotifierProvider | Complex + methods |
| Form state | NotifierProvider | Complex + validation |
| Initial user fetch | FutureProvider | One-time async |
| User profile (with edit) | AsyncNotifierProvider | Async + methods |
| Todo list (with CRUD) | AsyncNotifierProvider | Async + CRUD |
| Chat messages | StreamProvider | Real-time data |
| Firebase Firestore | StreamProvider | Real-time stream |
| Location tracking | StreamProvider | Continuous updates |
| Notifications (with dismiss) | StreamNotifierProvider | Stream + methods |

---

## 🎨 أمثلة من الواقع

### مثال 1: E-commerce App

</div>

```dart
// ✅ Provider - API Client (service)
final apiClientProvider = Provider<ApiClient>((ref) {
  return ApiClient(baseUrl: 'https://api.shop.com');
});

// ✅ StateProvider - Selected category (simple)
final selectedCategoryProvider = StateProvider<String>((ref) => 'all');

// ✅ StateProvider - Sort option (simple enum)
final sortOptionProvider = StateProvider<SortOption>((ref) => SortOption.popular);

// ✅ AsyncNotifierProvider - Products (async + methods)
class ProductsNotifier extends AsyncNotifier<List<Product>> {
  @override
  Future<List<Product>> build() async {
    final category = ref.watch(selectedCategoryProvider);
    return await api.getProducts(category: category);
  }

  Future<void> refresh() async { /* ... */ }
}

final productsProvider = AsyncNotifierProvider<ProductsNotifier, List<Product>>(
  () => ProductsNotifier(),
);

// ✅ NotifierProvider - Shopping cart (sync + methods)
class CartNotifier extends Notifier<List<CartItem>> {
  @override
  List<CartItem> build() => [];

  void addItem(Product product) { /* ... */ }
  void removeItem(String id) { /* ... */ }
}

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

### مثال 2: Social Media App

</div>

```dart
// ✅ Provider - Auth service
final authServiceProvider = Provider<AuthService>((ref) => AuthService());

// ✅ StreamProvider - Auth state
final authStateProvider = StreamProvider<User?>((ref) {
  return FirebaseAuth.instance.authStateChanges();
});

// ✅ AsyncNotifierProvider - User profile
class UserProfileNotifier extends AsyncNotifier<UserProfile> {
  @override
  Future<UserProfile> build() async {
    return await api.getUserProfile();
  }

  Future<void> updateProfile(UserProfile profile) async { /* ... */ }
}

final userProfileProvider = AsyncNotifierProvider<UserProfileNotifier, UserProfile>(
  () => UserProfileNotifier(),
);

// ✅ AsyncNotifierProvider - Posts feed
class PostsNotifier extends AsyncNotifier<List<Post>> {
  @override
  Future<List<Post>> build() async {
    return await api.getPosts();
  }

  Future<void> likePost(String postId) async { /* ... */ }
  Future<void> refresh() async { /* ... */ }
}

final postsProvider = AsyncNotifierProvider<PostsNotifier, List<Post>>(
  () => PostsNotifier(),
);

// ✅ StreamProvider - Notifications
final notificationsProvider = StreamProvider<List<Notification>>((ref) {
  return notificationService.notificationsStream();
});

// ✅ StateProvider - Selected tab
final selectedTabProvider = StateProvider<int>((ref) => 0);
```

<div dir="rtl">

### مثال 3: Fitness Tracker App

</div>

```dart
// ✅ Provider - Database
final databaseProvider = Provider<Database>((ref) => Database());

// ✅ StreamProvider - Real-time steps
final stepsProvider = StreamProvider<int>((ref) {
  return pedometer.stepCountStream;
});

// ✅ StreamProvider - Heart rate
final heartRateProvider = StreamProvider<int>((ref) {
  return healthKit.heartRateStream();
});

// ✅ AsyncNotifierProvider - Workouts
class WorkoutsNotifier extends AsyncNotifier<List<Workout>> {
  @override
  Future<List<Workout>> build() async {
    return await db.getWorkouts();
  }

  Future<void> addWorkout(Workout workout) async { /* ... */ }
  Future<void> deleteWorkout(String id) async { /* ... */ }
}

final workoutsProvider = AsyncNotifierProvider<WorkoutsNotifier, List<Workout>>(
  () => WorkoutsNotifier(),
);

// ✅ StateProvider - Selected date
final selectedDateProvider = StateProvider<DateTime>((ref) => DateTime.now());

// ✅ Provider - Daily stats (computed)
final dailyStatsProvider = Provider<DailyStats>((ref) {
  final steps = ref.watch(stepsProvider).value ?? 0;
  final workouts = ref.watch(workoutsProvider).value ?? [];

  return DailyStats(
    steps: steps,
    workoutCount: workouts.length,
  );
});
```

<div dir="rtl">

---

## 🔄 Upgrade Paths

### متى تنتقل من نوع لنوع؟

#### 1. StateProvider → NotifierProvider

</div>

```dart
// بدأت بـ StateProvider
final counterProvider = StateProvider<int>((ref) => 0);

// ✅ Upgrade عندما:
// - احتجت methods (increment, decrement, reset)
// - State بقى أكتر من primitive
// - محتاج business logic

class CounterNotifier extends Notifier<int> {
  @override
  int build() => 0;

  void increment() => state++;
  void decrement() => state--;
  void reset() => state = 0;
  void incrementBy(int value) => state += value;
}
```

<div dir="rtl">

#### 2. FutureProvider → AsyncNotifierProvider

</div>

```dart
// بدأت بـ FutureProvider
final todosProvider = FutureProvider<List<Todo>>((ref) async {
  return await api.getTodos();
});

// ✅ Upgrade عندما:
// - احتجت refresh
// - احتجت add/delete/update methods
// - احتجت optimistic updates

class TodosNotifier extends AsyncNotifier<List<Todo>> {
  @override
  Future<List<Todo>> build() async {
    return await api.getTodos();
  }

  Future<void> refresh() async { /* ... */ }
  Future<void> addTodo(String title) async { /* ... */ }
}
```

<div dir="rtl">

#### 3. StreamProvider → StreamNotifierProvider

</div>

```dart
// بدأت بـ StreamProvider
final messagesProvider = StreamProvider<List<Message>>((ref) {
  return chatService.messagesStream();
});

// ✅ Upgrade عندما:
// - احتجت methods (send, delete, mark as read)
// - احتجت تتحكم في الـ stream

class MessagesNotifier extends StreamNotifier<List<Message>> {
  @override
  Stream<List<Message>> build() {
    return chatService.messagesStream();
  }

  void sendMessage(String text) { /* ... */ }
  void deleteMessage(String id) { /* ... */ }
}
```

<div dir="rtl">

---

## 🎯 Common Scenarios

### Scenario 1: Counter

</div>

```dart
// ⭐ Simple counter (0-10 range)
// → StateProvider
final counterProvider = StateProvider<int>((ref) => 0);

// ⭐ Counter with methods (increment, decrement, reset)
// → NotifierProvider
class CounterNotifier extends Notifier<int> {
  @override
  int build() => 0;
  void increment() => state++;
  void decrement() => state--;
}
```

<div dir="rtl">

### Scenario 2: Theme Mode

</div>

```dart
// ⭐ Just storing theme mode
// → StateProvider
final themeModeProvider = StateProvider<ThemeMode>((ref) => ThemeMode.system);

// ⭐ Theme mode + computed themes
// → StateProvider + Provider
final themeModeProvider = StateProvider<ThemeMode>((ref) => ThemeMode.system);

final lightThemeProvider = Provider<ThemeData>((ref) {
  // Computed based on settings
});
```

<div dir="rtl">

### Scenario 3: User Data

</div>

```dart
// ⭐ Just fetch user once
// → FutureProvider
final userProvider = FutureProvider<User>((ref) async {
  return await api.getUser();
});

// ⭐ Fetch + update profile
// → AsyncNotifierProvider
class UserNotifier extends AsyncNotifier<User> {
  @override
  Future<User> build() async => await api.getUser();

  Future<void> updateProfile(UserProfile profile) async { /* ... */ }
}

// ⭐ Real-time user updates (Firebase)
// → StreamProvider
final userProvider = StreamProvider<User>((ref) {
  return firestore.collection('users').doc(uid).snapshots();
});
```

<div dir="rtl">

### Scenario 4: Form State

</div>

```dart
// ⭐ Simple form (2-3 fields, no validation)
// → StateProvider for each field
final nameProvider = StateProvider<String>((ref) => '');
final emailProvider = StateProvider<String>((ref) => '');

// ⭐ Complex form (many fields, validation, submission)
// → NotifierProvider
class FormNotifier extends Notifier<FormState> {
  @override
  FormState build() => FormState.initial();

  void updateName(String name) { /* validate + update */ }
  void updateEmail(String email) { /* validate + update */ }
  Future<void> submit() async { /* ... */ }
}
```

<div dir="rtl">

---

## ✅ Quick Reference

### استخدم Provider لو:
- ✅ قيم ثابتة (config, constants)
- ✅ Services (API client, database)
- ✅ Computed values (total, filtered list)

### استخدم StateProvider لو:
- ✅ Primitive بسيط (int, bool, String)
- ✅ UI state (selected tab, filter)
- ✅ Toggle states (dark mode, show/hide)

### استخدم FutureProvider لو:
- ✅ Async operation مرة واحدة
- ✅ Initial data fetch
- ✅ مش محتاج refresh أو methods

### استخدم StreamProvider لو:
- ✅ Real-time data (chat, notifications)
- ✅ Continuous updates (location, sensors)
- ✅ Firebase streams
- ✅ مش محتاج methods

### استخدم NotifierProvider لو:
- ✅ Complex sync state (objects, lists)
- ✅ محتاج methods (add, remove, update)
- ✅ Business logic

### استخدم AsyncNotifierProvider لو:
- ✅ Complex async state
- ✅ CRUD operations
- ✅ محتاج refresh
- ✅ Optimistic updates

### استخدم StreamNotifierProvider لو:
- ✅ Stream + methods
- ✅ محتاج تتحكم في الـ stream
- ✅ Real-time data مع actions

---

## 🚫 متى ما تستخدمش إيه

### ❌ لا تستخدم Provider لو:
- البيانات بتتغير → use StateProvider/NotifierProvider

### ❌ لا تستخدم StateProvider لو:
- State معقد (objects, lists) → use NotifierProvider
- محتاج methods → use NotifierProvider

### ❌ لا تستخدم FutureProvider لو:
- محتاج refresh → use AsyncNotifierProvider
- محتاج methods → use AsyncNotifierProvider
- Continuous updates → use StreamProvider

### ❌ لا تستخدم StreamProvider لو:
- One-time fetch → use FutureProvider
- محتاج methods → use StreamNotifierProvider

### ❌ لا تستخدم NotifierProvider لو:
- Async operations → use AsyncNotifierProvider
- Simple primitive → use StateProvider

### ❌ لا تستخدم AsyncNotifierProvider لو:
- One-time fetch بدون refresh → use FutureProvider
- Synchronous state → use NotifierProvider

---

## 📝 ملخص

**القاعدة الذهبية:** ابدأ بالأبسط، واطور لما تحتاج!

1. **مش بيتغير؟** → Provider
2. **Primitive بسيط؟** → StateProvider
3. **Async مرة واحدة؟** → FutureProvider
4. **Real-time stream؟** → StreamProvider
5. **محتاج methods؟** → Notifier/AsyncNotifier/StreamNotifier
6. **معقد؟** → Notifier providers

**Upgrade Path:**
- StateProvider → NotifierProvider (لما تحتاج methods)
- FutureProvider → AsyncNotifierProvider (لما تحتاج refresh)
- StreamProvider → StreamNotifierProvider (لما تحتاج methods)

---

## 🔗 الخطوة الجاية

دلوقتي فهمت كل أنواع Providers! الخطوة الجاية:
- **Section 06** - Code Generation
- إزاي نستخدم `@riverpod`
- الفرق بين Classic و Code Generation

---

## 📚 المصادر

- [Provider Types Overview](https://riverpod.dev/docs/concepts/providers)
- [Choosing a Provider](https://riverpod.dev/docs/concepts/about_providers)
- [Migration Guide](https://riverpod.dev/docs/migration/from_state_notifier)

</div>
