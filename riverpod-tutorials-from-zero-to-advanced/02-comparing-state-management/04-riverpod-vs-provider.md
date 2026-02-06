<div dir="rtl">

# مقارنة مباشرة: Riverpod vs Provider

**المستوى**: 🟡 متوسط

## 📌 المفاهيم الأساسية

في الملف ده هنعمل:
- مقارنة side-by-side بين Riverpod و Provider
- نفس الأمثلة بالحلين
- المشاكل اللي Riverpod بيحلها
- دليل القرار: امتى تستخدم أيهم

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تشوف الفرق الواضح بين Riverpod و Provider
- تفهم ليه Riverpod أفضل في معظم الحالات
- تقرر بثقة امتى تنتقل من Provider لـ Riverpod
- تعرف خطوات Migration الأساسية

---

## 🎭 المقدمة: ليه المقارنة دي مهمة؟

**حقيقة مهمة:**
Riverpod و Provider اتعملوا بواسطة نفس الشخص - Remi Rousselet.

Riverpod مش "منافس" لـ Provider - ده **التطور الطبيعي** ليه. Remi عمل Riverpod عشان يحل كل المشاكل اللي اكتشفها في Provider بعد سنين من الاستخدام.

---

## 📊 المقارنة الشاملة

| الجانب | Provider | Riverpod | الفائز |
|--------|----------|----------|--------|
| **Type Safety** | ⭐⭐ Runtime | ⭐⭐⭐⭐⭐ Compile-time | 🏆 Riverpod |
| **BuildContext** | إلزامي | اختياري | 🏆 Riverpod |
| **Dependency Injection** | ⭐⭐ محدود | ⭐⭐⭐⭐⭐ كامل | 🏆 Riverpod |
| **Auto Disposal** | ❌ يدوي | ✅ تلقائي | 🏆 Riverpod |
| **Testing** | ⭐⭐ Widget tests | ⭐⭐⭐⭐⭐ Unit tests | 🏆 Riverpod |
| **Scoping** | ⭐⭐ معقد | ⭐⭐⭐⭐⭐ سهل | 🏆 Riverpod |
| **Learning Curve** | ⭐⭐⭐⭐ سهل | ⭐⭐⭐ متوسط | 🏆 Provider |
| **Community** | ⭐⭐⭐⭐⭐ ضخمة | ⭐⭐⭐⭐ كبيرة | 🏆 Provider |
| **Migration** | من setState | من Provider | متعادل |

---

## 💻 مقارنة الكود: مثال 1 - Counter بسيط

### باستخدام Provider

</div>

```dart
// ==========================================
// Provider Version
// ==========================================

// 1. Define ChangeNotifier
class CounterNotifier extends ChangeNotifier {
  int _count = 0;

  int get count => _count;

  void increment() {
    _count++;
    notifyListeners(); // Manual notification
  }

  void decrement() {
    _count--;
    notifyListeners();
  }
}

// 2. Provide it
void main() {
  runApp(
    ChangeNotifierProvider(
      create: (_) => CounterNotifier(),
      child: MyApp(),
    ),
  );
}

// 3. Watch it (rebuilds on change)
class CounterDisplay extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final counter = context.watch<CounterNotifier>();
    // ❌ Runtime error if provider not found!

    return Text('Count: ${counter.count}');
  }
}

// 4. Read it (no rebuild)
class IncrementButton extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () {
        context.read<CounterNotifier>().increment();
        // ❌ Requires BuildContext
      },
      child: Text('Increment'),
    );
  }
}
```

<div dir="rtl">

### باستخدام Riverpod

</div>

```dart
// ==========================================
// Riverpod Version
// ==========================================

// 1. Define provider (one line!)
final counterProvider = StateProvider<int>((ref) => 0);

// 2. No provider setup needed in main
void main() {
  runApp(
    ProviderScope( // Just wrap with ProviderScope once
      child: MyApp(),
    ),
  );
}

// 3. Watch it (rebuilds on change)
class CounterDisplay extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);
    // ✅ Compile-time error if provider doesn't exist!

    return Text('Count: $count');
  }
}

// 4. Update it (no rebuild)
class IncrementButton extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return ElevatedButton(
      onPressed: () {
        ref.read(counterProvider.notifier).state++;
        // ✅ No BuildContext needed for providers!
      },
      child: Text('Increment'),
    );
  }
}
```

<div dir="rtl">

**الفرق:**
- Provider: 20+ lines, manual `notifyListeners()`, runtime errors
- Riverpod: 10 lines, automatic updates, compile-time safety

---

## 💻 مقارنة الكود: مثال 2 - Async Data

### باستخدام Provider

</div>

```dart
// ==========================================
// Provider Version
// ==========================================

class UserNotifier extends ChangeNotifier {
  User? _user;
  bool _isLoading = false;
  String? _error;

  User? get user => _user;
  bool get isLoading => _isLoading;
  String? get error => _error;

  Future<void> loadUser() async {
    _isLoading = true;
    _error = null;
    notifyListeners();

    try {
      _user = await api.getUser();
      _isLoading = false;
      notifyListeners();
    } catch (e) {
      _error = e.toString();
      _isLoading = false;
      notifyListeners();
    }
  }
}

// Provide
ChangeNotifierProvider(
  create: (_) => UserNotifier()..loadUser(),
  child: MyApp(),
);

// Use
class UserProfile extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final userNotifier = context.watch<UserNotifier>();

    if (userNotifier.isLoading) {
      return CircularProgressIndicator();
    }

    if (userNotifier.error != null) {
      return Text('Error: ${userNotifier.error}');
    }

    if (userNotifier.user != null) {
      return Text('Hello ${userNotifier.user!.name}');
    }

    return Text('No user');
  }
}
```

<div dir="rtl">

### باستخدام Riverpod

</div>

```dart
// ==========================================
// Riverpod Version
// ==========================================

// One provider - handles loading, error, and data automatically!
final userProvider = FutureProvider<User>((ref) async {
  return await api.getUser();
});

// Use with .when (cleaner!)
class UserProfile extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider);

    return userAsync.when(
      data: (user) => Text('Hello ${user.name}'),
      loading: () => CircularProgressIndicator(),
      error: (error, stack) => Text('Error: $error'),
    );
  }
}

// Or manually
class UserProfile2 extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider);

    if (userAsync.isLoading) {
      return CircularProgressIndicator();
    }

    if (userAsync.hasError) {
      return Text('Error: ${userAsync.error}');
    }

    final user = userAsync.value!;
    return Text('Hello ${user.name}');
  }
}
```

<div dir="rtl">

**الفرق:**
- Provider: إدارة يدوية للـ loading, error, data states + 3 calls لـ notifyListeners
- Riverpod: AsyncValue يدير كل ده تلقائياً + API نضيف باستخدام .when

---

## 🔍 المشكلة 1: Runtime Errors

### مع Provider

</div>

```dart
// ❌ This compiles fine but crashes at runtime!

class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // Typo in provider type
    final user = context.watch<Userr>(); // Should be "User"

    // Compiles ✅
    // Runs ❌ - ProviderNotFoundException at runtime!

    return Text(user.name);
  }
}

// ❌ Provider not found - crashes!
class AnotherWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final cart = context.watch<CartNotifier>();
    // If CartNotifier wasn't provided, runtime crash!

    return Text('${cart.itemCount}');
  }
}
```

<div dir="rtl">

### مع Riverpod

</div>

```dart
// ✅ Compile-time errors catch everything!

class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Typo in provider name
    final user = ref.watch(userrProvider); // Should be "userProvider"

    // Doesn't compile ❌
    // Error: Undefined name 'userrProvider'

    return Text(user.name);
  }
}

// ✅ Provider must exist at compile time
final cartProvider = NotifierProvider<CartNotifier, CartState>(
  () => CartNotifier(),
);

class AnotherWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final cart = ref.watch(cartProvider);
    // If cartProvider doesn't exist, won't compile!

    return Text('${cart.itemCount}');
  }
}
```

<div dir="rtl">

---

## 🔍 المشكلة 2: BuildContext Dependency

### مع Provider

</div>

```dart
// ❌ Can't access providers outside widgets

class AuthService {
  // How do I access UserProvider here?
  // I don't have BuildContext!

  Future<void> login(String email, String password) async {
    final user = await api.login(email, password);

    // ❌ Can't update provider - no context!
    // Ugly workarounds needed:
    // 1. Pass context as parameter (bad!)
    // 2. Use global key (worse!)
    // 3. Create circular dependency (worst!)
  }
}

// Ugly workaround
class AuthService {
  final BuildContext context; // ❌ Services shouldn't need context!

  AuthService(this.context);

  Future<void> login(String email, String password) async {
    final user = await api.login(email, password);
    context.read<UserNotifier>().setUser(user); // Ugly!
  }
}
```

<div dir="rtl">

### مع Riverpod

</div>

```dart
// ✅ Access providers anywhere with Ref!

final authServiceProvider = Provider<AuthService>((ref) {
  return AuthService(ref); // Pass ref, not context!
});

class AuthService {
  final Ref ref; // ✅ Clean dependency!

  AuthService(this.ref);

  Future<void> login(String email, String password) async {
    final user = await api.login(email, password);

    // ✅ Easy access to providers!
    ref.read(userProvider.notifier).state = user;

    // ✅ Can read other providers too
    final cart = ref.read(cartProvider);
    cart.clear();
  }
}

// Use in widget
class LoginButton extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final authService = ref.read(authServiceProvider);

    return ElevatedButton(
      onPressed: () => authService.login(email, password),
      child: Text('Login'),
    );
  }
}
```

<div dir="rtl">

---

## 🔍 المشكلة 3: Dependency Injection

### مع Provider

</div>

```dart
// ❌ Hard to inject dependencies

final apiProvider = Provider<ApiService>((ref) {
  return ApiService();
});

final userRepositoryProvider = Provider<UserRepository>((ref) {
  // ❌ How do I get ApiService here?
  // Can't use ref.watch or ref.read in Provider!

  // Ugly workaround: use ProxyProvider
  return UserRepository(/* ??? */);
});

// Need ProxyProvider (complicated!)
ProxyProvider<ApiService, UserRepository>(
  update: (context, api, previous) {
    return UserRepository(api);
  },
);

// Gets messy with multiple dependencies!
ProxyProvider3<Database, ApiService, CacheService, UserRepository>(
  update: (context, db, api, cache, previous) {
    return UserRepository(db, api, cache);
  },
);
```

<div dir="rtl">

### مع Riverpod

</div>

```dart
// ✅ Natural dependency injection!

final apiProvider = Provider<ApiService>((ref) {
  return ApiService();
});

final userRepositoryProvider = Provider<UserRepository>((ref) {
  // ✅ Easy! Just ref.watch
  final api = ref.watch(apiProvider);
  return UserRepository(api);
});

final authServiceProvider = Provider<AuthService>((ref) {
  // ✅ Multiple dependencies? No problem!
  final api = ref.watch(apiProvider);
  final userRepo = ref.watch(userRepositoryProvider);
  final cache = ref.watch(cacheProvider);

  return AuthService(api, userRepo, cache);
});

// Dependencies are clear and explicit!
// No ProxyProvider mess!
```

<div dir="rtl">

---

## 🔍 المشكلة 4: Auto Disposal

### مع Provider

</div>

```dart
// ❌ No auto disposal - memory leaks!

final messagesProvider = StreamProvider<List<Message>>((ref) {
  return chatService.messagesStream();
});

// When you navigate away from chat:
// - Stream still running!
// - Memory leak!
// - Resources not freed!

// Manual cleanup needed:
class ChatPage extends StatefulWidget {
  @override
  State<ChatPage> createState() => _ChatPageState();
}

class _ChatPageState extends State<ChatPage> {
  late StreamSubscription _subscription;

  @override
  void initState() {
    super.initState();
    _subscription = context.read<StreamController>().stream.listen(/*...*/);
  }

  @override
  void dispose() {
    _subscription.cancel(); // Manual cleanup!
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Container();
  }
}
```

<div dir="rtl">

### مع Riverpod

</div>

```dart
// ✅ Automatic disposal!

final messagesProvider = StreamProvider.autoDispose<List<Message>>((ref) {
  final stream = chatService.messagesStream();

  // Optional: cleanup callback
  ref.onDispose(() {
    print('Messages provider disposed - stream closed');
  });

  return stream;
});

// When you navigate away from chat:
// - Riverpod automatically disposes the provider
// - Stream is cancelled
// - No memory leaks!
// - No manual cleanup needed!

class ChatPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final messagesAsync = ref.watch(messagesProvider);

    // No dispose() method needed!
    // Riverpod handles everything!

    return messagesAsync.when(
      data: (messages) => MessagesList(messages),
      loading: () => CircularProgressIndicator(),
      error: (error, stack) => ErrorWidget(error),
    );
  }
}
```

<div dir="rtl">

---

## 🔍 المشكلة 5: Testing

### مع Provider

</div>

```dart
// ❌ Need widget tests (slow and complex)

test('login flow works', () async {
  // Can't test logic directly
  // Need to pump widgets!

  await tester.pumpWidget(
    ChangeNotifierProvider(
      create: (_) => LoginNotifier(mockRepository),
      child: MaterialApp(
        home: LoginPage(),
      ),
    ),
  );

  // Find widgets
  final emailField = find.byType(TextField).first;
  final passwordField = find.byType(TextField).last;
  final loginButton = find.byType(ElevatedButton);

  // Enter text
  await tester.enterText(emailField, 'test@example.com');
  await tester.enterText(passwordField, 'password');

  // Tap button
  await tester.tap(loginButton);
  await tester.pump();

  // Check result
  expect(find.text('Welcome'), findsOneWidget);

  // This is SLOW and BRITTLE!
});
```

<div dir="rtl">

### مع Riverpod

</div>

```dart
// ✅ Pure unit tests (fast and simple)

test('login flow works', () async {
  // Create container with mocks
  final container = ProviderContainer(
    overrides: [
      authRepositoryProvider.overrideWithValue(mockRepository),
    ],
  );

  // Get the notifier
  final notifier = container.read(loginProvider.notifier);

  // Test business logic directly!
  await notifier.login('test@example.com', 'password');

  // Check state
  final state = container.read(loginProvider);
  expect(state, isA<LoginSuccess>());

  // Clean up
  container.dispose();

  // FAST and CLEAN!
});

test('login handles errors', () async {
  when(() => mockRepository.login(any(), any()))
      .thenThrow(AuthException('Invalid credentials'));

  final container = ProviderContainer(
    overrides: [
      authRepositoryProvider.overrideWithValue(mockRepository),
    ],
  );

  final notifier = container.read(loginProvider.notifier);
  await notifier.login('test@example.com', 'wrong');

  final state = container.read(loginProvider);
  expect(state, isA<LoginFailure>());

  container.dispose();
});
```

<div dir="rtl">

---

## 🔍 المشكلة 6: Scoping

### مع Provider

</div>

```dart
// ❌ Scoped providers are complex

// Different todo list for each user
// With Provider: Need manual scoping

class UserTodosPage extends StatelessWidget {
  final String userId;

  @override
  Widget build(BuildContext context) {
    // Need to create a new provider for each user
    return ChangeNotifierProvider(
      key: ValueKey(userId), // Important!
      create: (_) => TodosNotifier(userId),
      child: TodosList(),
    );
  }
}

// Gets complicated fast!
```

<div dir="rtl">

### مع Riverpod

</div>

```dart
// ✅ Family makes scoping easy!

final todosProvider = NotifierProvider.family<TodosNotifier, TodosState, String>(
  () => TodosNotifier(),
);

// Note: في Riverpod 3.0، الـ userId parameter يتم تمريره في build() method:
// class TodosNotifier extends FamilyNotifier<TodosState, String> {
//   @override
//   TodosState build(String userId) { /* ... */ }
// }

// Use with parameter
class UserTodosPage extends ConsumerWidget {
  final String userId;

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Just pass the parameter!
    final todos = ref.watch(todosProvider(userId));

    return TodosList(todos);
  }
}

// Each userId gets its own provider instance
// Riverpod handles everything!
```

<div dir="rtl">

---

## 📊 الخلاصة النهائية

### نقاط قوة Provider

```
✅ Learning curve أسهل للمبتدئين
✅ Community ضخمة (تاريخياً)
✅ أمثلة كتيرة متاحة
✅ مُنصَح به رسمياً من Google (سابقاً)
```

### نقاط قوة Riverpod

```
✅ Compile-time safety كاملة
✅ مفيش BuildContext dependency
✅ Dependency injection طبيعية وسهلة
✅ Auto disposal تلقائي
✅ Testing أسهل بكتير (unit tests)
✅ Scoping بسيط باستخدام family
✅ Performance أفضل
✅ Code أقل (less boilerplate)
✅ من نفس مطور Provider (مبني على خبرته)
```

### القرار النهائي

</div>

```
┌─ بتبدأ مشروع جديد؟
│  └─ نعم → Riverpod ✅ (100%)
│
├─ عندك مشروع Provider وشغال كويس؟
│  └─ نعم → ممكن تكمل
│  └─ لكن لو هتعمل refactoring كبير
│      → هاجر لـ Riverpod ✅
│
├─ محتاج compile-time safety؟
│  └─ نعم → Riverpod ✅ (الوحيد اللي عنده ده)
│
├─ محتاج dependency injection؟
│  └─ نعم → Riverpod ✅
│
├─ محتاج testing سهل؟
│  └─ نعم → Riverpod ✅
│
└─ عايز أفضل developer experience؟
    └─ Riverpod ✅
```

<div dir="rtl">

### توصية نهائية

**للمشاريع الجديدة:** استخدم Riverpod دون تردد. هو التطور الطبيعي لـ Provider وأفضل منه في كل حاجة تقريباً (ماعدا Learning curve).

**للمشاريع القديمة بـ Provider:**
- لو شغال كويس ومفيش مشاكل: ممكن تكمل
- لو عندك مشاكل أو بتعمل refactoring: وقت مناسب للـ migration
- لو محتاج ميزة من ميزات Riverpod (compile-time safety, DI, etc.): لازم تهاجر

**Migration سهلة!** الـ API متشابه جداً لأن نفس الشخص عملهم.

---

## 🔗 الخطوة الجاية

دلوقتي بعد ما شفت المقارنة المباشرة، وقت:
- **مقارنة Riverpod vs BLoC** (الملف الجاي)
- **دليل Migration من Provider لـ Riverpod**
- **أمثلة عملية كاملة**

---

## 📚 المصادر

- [Why Riverpod?](https://riverpod.dev/docs/concepts/about)
- [Migrating from Provider](https://riverpod.dev/docs/from_provider/motivation)
- [Provider Package](https://pub.dev/packages/provider)
- [Riverpod Package](https://pub.dev/packages/flutter_riverpod)

---

## ✅ تأكد إنك فهمت

- [ ] شفت الفرق الواضح بين Provider و Riverpod؟
- [ ] فاهم المشاكل اللي Riverpod بيحلها؟
- [ ] عارف امتى تستخدم Provider وامتى Riverpod؟
- [ ] جاهز للـ migration؟

**جاهز تشوف Riverpod vs BLoC؟** ⚖️

</div>
