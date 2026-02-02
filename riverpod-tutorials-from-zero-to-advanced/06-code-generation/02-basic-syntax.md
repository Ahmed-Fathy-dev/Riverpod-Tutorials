<div dir="rtl">

# Basic Syntax - الـ Syntax الأساسي

**المستوى**: 🟢 مبتدئ

## 📌 المفاهيم الأساسية

في الملف ده هنتعلم:
- كل أنواع الـ providers مع `@riverpod` annotation
- إزاي Riverpod بيستنتج نوع الـ provider تلقائياً
- استخدام Parameters (بدون family!)
- الفرق بين AutoDispose و KeepAlive
- Dependencies بين الـ providers

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تكتب أي نوع provider بالـ code generation
- تستخدم parameters في الـ providers
- تتحكم في auto-dispose behavior
- تعمل dependencies بين providers بسهولة

---

## 📖 القاعدة الأساسية

**Riverpod بيستنتج نوع الـ Provider تلقائياً** بناءً على الـ return type!

</div>

```dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'examples.g.dart';

// Returns String → Provider<String>
@riverpod
String message(MessageRef ref) {
  return 'Hello World';
}

// Returns Future<T> → FutureProvider<T>
@riverpod
Future<User> user(UserRef ref) async {
  return await api.getUser();
}

// Returns Stream<T> → StreamProvider<T>
@riverpod
Stream<Message> messages(MessagesRef ref) {
  return chatService.messagesStream();
}

// Class with build() → NotifierProvider or AsyncNotifierProvider
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;
  // Methods...
}
```

<div dir="rtl">

---

## 1️⃣ Provider (Synchronous Values)

لما الـ function بترجع قيمة عادية (مش Future أو Stream):

</div>

```dart
// Example 1: Simple value
@riverpod
String greeting(GreetingRef ref) {
  return 'Hello Riverpod!';
}

// Example 2: Configuration object
@riverpod
AppConfig config(ConfigRef ref) {
  return AppConfig(
    apiUrl: 'https://api.example.com',
    timeout: Duration(seconds: 30),
  );
}

// Example 3: Computed value (depends on other provider)
@riverpod
String welcomeMessage(WelcomeMessageRef ref) {
  final greeting = ref.watch(greetingProvider);
  final user = ref.watch(userProvider);

  return '$greeting, ${user.name}!';
}

// Usage in UI
class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final message = ref.watch(welcomeMessageProvider);
    return Text(message);
  }
}
```

<div dir="rtl">

**ملاحظات:**
- الـ function name هو اللي بيحدد اسم الـ provider
- `greeting` → يولد `greetingProvider`
- `welcomeMessage` → يولد `welcomeMessageProvider`

---

## 2️⃣ FutureProvider (Async One-time Data)

لما الـ function بترجع `Future<T>`:

</div>

```dart
// Example 1: Fetch user from API
@riverpod
Future<User> user(UserRef ref) async {
  final response = await http.get(Uri.parse('https://api.example.com/user'));
  if (response.statusCode != 200) {
    throw Exception('Failed to load user');
  }
  return User.fromJson(jsonDecode(response.body));
}

// Example 2: Load configuration from file
@riverpod
Future<AppConfig> appConfig(AppConfigRef ref) async {
  final configString = await rootBundle.loadString('assets/config.json');
  return AppConfig.fromJson(jsonDecode(configString));
}

// Example 3: With dependencies
@riverpod
Future<List<Post>> userPosts(UserPostsRef ref) async {
  // Wait for user to load first
  final user = await ref.watch(userProvider.future);

  // Then fetch their posts
  return await api.getPosts(userId: user.id);
}

// Usage in UI
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
```

<div dir="rtl">

**مميزات:**
- بيرجع `AsyncValue<T>` تلقائياً
- عندك `.when()` للـ state handling
- الـ data بتتcache تلقائياً

---

## 3️⃣ StreamProvider (Real-time Data)

لما الـ function بترجع `Stream<T>`:

</div>

```dart
// Example 1: Firebase Firestore stream
@riverpod
Stream<List<Message>> messages(MessagesRef ref) {
  return FirebaseFirestore.instance
      .collection('messages')
      .orderBy('timestamp', descending: true)
      .limit(50)
      .snapshots()
      .map((snapshot) {
    return snapshot.docs.map((doc) => Message.fromFirestore(doc)).toList();
  });
}

// Example 2: WebSocket connection
@riverpod
Stream<String> notifications(NotificationsRef ref) {
  final channel = WebSocketChannel.connect(
    Uri.parse('ws://example.com/notifications'),
  );

  // Clean up when provider is disposed
  ref.onDispose(() {
    channel.sink.close();
  });

  return channel.stream.cast<String>();
}

// Example 3: Timer/periodic updates
@riverpod
Stream<DateTime> currentTime(CurrentTimeRef ref) {
  return Stream.periodic(
    Duration(seconds: 1),
    (_) => DateTime.now(),
  );
}

// Usage
class ChatScreen extends ConsumerWidget {
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

**ملاحظات:**
- الـ stream بيتclosed تلقائياً لما الـ provider يتdispose
- بيرجع `AsyncValue<T>` زي FutureProvider
- بيupdate تلقائياً مع كل قيمة جديدة من الـ stream

---

## 4️⃣ Parameters (بدون Family!)

واحدة من أكبر مميزات Code Generation: **Parameters مباشرة بدون family!**

</div>

```dart
// Example 1: Single parameter
@riverpod
Future<Todo> todo(TodoRef ref, String id) async {
  return await api.getTodo(id);
}

// Usage
final todo = ref.watch(todoProvider('123'));

// Example 2: Multiple parameters
@riverpod
Future<List<Product>> products(
  ProductsRef ref,
  String category,
  int page,
) async {
  return await api.getProducts(
    category: category,
    page: page,
  );
}

// Usage
final products = ref.watch(productsProvider('electronics', 1));

// Example 3: Named parameters
@riverpod
Future<SearchResults> search(
  SearchRef ref, {
  required String query,
  int limit = 10,
  String? category,
}) async {
  return await api.search(
    query: query,
    limit: limit,
    category: category,
  );
}

// Usage
final results1 = ref.watch(searchProvider(query: 'laptop'));
final results2 = ref.watch(searchProvider(query: 'phone', limit: 20));
final results3 = ref.watch(searchProvider(
  query: 'tablet',
  category: 'electronics',
));

// Example 4: Optional parameters with defaults
@riverpod
String formatName(
  FormatNameRef ref,
  String firstName,
  String lastName, [
  bool uppercase = false,
]) {
  final fullName = '$firstName $lastName';
  return uppercase ? fullName.toUpperCase() : fullName;
}

// Usage
final name1 = ref.watch(formatNameProvider('Ahmed', 'Ali'));
final name2 = ref.watch(formatNameProvider('Ahmed', 'Ali', true));
```

<div dir="rtl">

**القاعدة المهمة:**
- الـ parameter الأول **لازم** يكون `Ref` (XxxRef)
- باقي الـ parameters هي اللي بتتمرر للـ provider

### Caching مع Parameters

كل مجموعة parameters بتولد provider instance منفصل:

</div>

```dart
@riverpod
Future<Todo> todo(TodoRef ref, String id) async {
  print('Fetching todo $id');
  return await api.getTodo(id);
}

// In UI:
final todo1 = ref.watch(todoProvider('1'));  // Fetches todo 1
final todo2 = ref.watch(todoProvider('2'));  // Fetches todo 2
final todo1Again = ref.watch(todoProvider('1'));  // Uses cached result!
// Output:
// Fetching todo 1
// Fetching todo 2
// (no third fetch - cached!)
```

<div dir="rtl">

---

## 5️⃣ AutoDispose vs KeepAlive

**بشكل افتراضي، كل الـ providers بـ code generation هم AutoDispose!**

### AutoDispose (Default)

</div>

```dart
// Default: Auto-disposes when no longer used
@riverpod
Stream<List<Message>> messages(MessagesRef ref) {
  print('Stream started');

  ref.onDispose(() {
    print('Stream disposed');
  });

  return chatService.messagesStream();
}

// Lifecycle:
// 1. User opens chat screen
//    → Output: "Stream started"
// 2. User leaves chat screen
//    → Output: "Stream disposed"
// 3. User opens chat again
//    → Output: "Stream started" (new instance!)
```

<div dir="rtl">

**مميزات AutoDispose:**
- بيوفر memory تلقائياً
- مفيد للـ screens/features اللي مش مستخدمة دايماً
- الـ streams و timers بتتclosed تلقائياً

### KeepAlive (Disable AutoDispose)

لو عايز الـ provider يفضل حي طول الوقت:

</div>

```dart
// Keep alive: Never auto-disposes
@Riverpod(keepAlive: true)
Stream<User> currentUser(CurrentUserRef ref) {
  return authService.userStream();
}

// Alternative syntax (uppercase @Riverpod)
@Riverpod(keepAlive: true)
Future<AppConfig> appConfig(AppConfigRef ref) async {
  return await loadConfig();
}

// The data stays cached forever
// Even if no widget is watching it
```

<div dir="rtl">

**متى تستخدم KeepAlive:**
- Global state (user authentication, app settings)
- Expensive data اللي محتاج يفضل cached
- Singletons

### Dynamic KeepAlive (Advanced)

تقدر تتحكم في keep-alive بشكل dynamic:

</div>

```dart
@riverpod
Future<List<Todo>> todos(TodosRef ref) async {
  // Get keep-alive link
  final link = ref.keepAlive();

  // Start timer
  Timer? timer;
  ref.onCancel(() {
    // When last listener is removed, start 10-second timer
    timer = Timer(Duration(seconds: 10), () {
      // After 10 seconds of inactivity, dispose
      link.close();
    });
  });

  ref.onResume(() {
    // If listener is added again, cancel the timer
    timer?.cancel();
  });

  ref.onDispose(() {
    timer?.cancel();
  });

  return await api.getTodos();
}

// Behavior:
// - User watches provider → data loads
// - User stops watching → 10-second timer starts
// - If user watches again within 10s → cached data (instant!)
// - If 10s pass → provider disposes
```

<div dir="rtl">

---

## 6️⃣ Dependencies Between Providers

Providers بتقدر تعتمد على بعضها بسهولة:

</div>

```dart
// Provider 1: Current user
@riverpod
Future<User> currentUser(CurrentUserRef ref) async {
  return await authService.getCurrentUser();
}

// Provider 2: Depends on Provider 1
@riverpod
Future<List<Post>> userPosts(UserPostsRef ref) async {
  // Watch current user
  final user = await ref.watch(currentUserProvider.future);

  // Fetch posts for that user
  return await api.getUserPosts(user.id);
}

// Provider 3: Depends on Provider 2
@riverpod
int postsCount(PostsCountRef ref) {
  final postsAsync = ref.watch(userPostsProvider);

  return postsAsync.when(
    data: (posts) => posts.length,
    loading: () => 0,
    error: (_, __) => 0,
  );
}

// Chain of dependencies:
// currentUser → userPosts → postsCount
//
// If currentUser changes:
//  → userPosts automatically refetches
//  → postsCount automatically updates
```

<div dir="rtl">

### استخدام .future و .stream

</div>

```dart
// For FutureProvider, use .future to await
@riverpod
Future<Profile> userProfile(UserProfileRef ref) async {
  // Await the future directly
  final user = await ref.watch(currentUserProvider.future);
  return await api.getProfile(user.id);
}

// For StreamProvider, use .stream
@riverpod
Stream<List<Notification>> notifications(NotificationsRef ref) async* {
  // Watch the stream
  final userStream = ref.watch(currentUserProvider.stream);

  await for (final user in userStream) {
    yield await api.getNotifications(user.id);
  }
}
```

<div dir="rtl">

---

## 7️⃣ Ref Methods (الـ API المتاح)

الـ `ref` بيوفرلك methods مهمة:

</div>

```dart
@riverpod
Future<Data> example(ExampleRef ref) async {
  // 1. Watch other providers
  final user = ref.watch(userProvider);

  // 2. Read without listening
  final config = ref.read(configProvider);

  // 3. Listen to specific value changes
  ref.listen(
    userProvider,
    (previous, next) {
      print('User changed from $previous to $next');
    },
  );

  // 4. Invalidate other providers
  ref.invalidate(cacheProvider);

  // 5. Lifecycle callbacks
  ref.onDispose(() {
    print('Provider disposed');
  });

  ref.onCancel(() {
    print('Last listener removed');
  });

  ref.onResume(() {
    print('First listener added again');
  });

  // 6. Keep alive control
  final link = ref.keepAlive();
  // Later: link.close();

  return await fetchData();
}
```

<div dir="rtl">

---

## 8️⃣ Error Handling

</div>

```dart
// Errors are automatically caught and wrapped in AsyncValue
@riverpod
Future<User> user(UserRef ref) async {
  // If this throws, AsyncValue.error is returned automatically
  return await api.getUser();
}

// In UI, handle with .when()
final userAsync = ref.watch(userProvider);
userAsync.when(
  data: (user) => Text(user.name),
  loading: () => CircularProgressIndicator(),
  error: (error, stack) => Text('Error: $error'),
);

// Or check manually
if (userAsync.hasError) {
  final error = userAsync.error;
  // Handle error
}

// Custom error handling
@riverpod
Future<User> userWithRetry(UserWithRetryRef ref) async {
  try {
    return await api.getUser();
  } catch (e) {
    if (e is NetworkException) {
      // Retry after 3 seconds
      await Future.delayed(Duration(seconds: 3));
      return await api.getUser();
    }
    rethrow; // Let Riverpod handle other errors
  }
}
```

<div dir="rtl">

---

## 📊 ملخص سريع

| Return Type | Generated Provider | Auto Dispose |
|-------------|-------------------|--------------|
| `T` | `Provider<T>` | ✅ Yes |
| `Future<T>` | `FutureProvider<T>` | ✅ Yes |
| `Stream<T>` | `StreamProvider<T>` | ✅ Yes |
| Class with `build()` | `NotifierProvider` or `AsyncNotifierProvider` | ✅ Yes |

**لإيقاف AutoDispose:**
```dart
@Riverpod(keepAlive: true)
```

---

## 💡 Best Practices

### ✅ Do:
- استخدم parameters مباشرة (بدون family)
- خلي الـ providers صغيرة ومركزة
- استخدم auto-dispose للـ feature-specific data
- استخدم keepAlive للـ global state

### ❌ Don't:
- متحاولش تعمل side effects في الـ build method
- متستخدمش `ref.read()` في الـ build (استخدم `ref.watch()`)
- متنساش الـ `part` directive!

---

## 🔗 الخطوة الجاية

في الملف الجاي هنتعلم **Notifier و AsyncNotifier** بالتفصيل مع code generation:
- إزاي نكتب complex state logic
- Methods و state management
- مقارنة مع Classic syntax

جاهز؟ يلا نكمل! 🚀

---

## 📚 المصادر

- [About Code Generation | Riverpod](https://riverpod.dev/docs/concepts/about_code_generation)
- [Providers | Riverpod](https://riverpod.dev/docs/concepts2/providers)
- [How to use Riverpod Generator](https://codewithandrea.com/articles/flutter-riverpod-generator/)

</div>
