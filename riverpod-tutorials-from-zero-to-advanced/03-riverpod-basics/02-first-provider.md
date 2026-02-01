<div dir="rtl">

# أول Provider ليك

**المستوى**: 🟢 مبتدئ

## 📌 المفاهيم الأساسية

في الملف ده هنتكلم عن:
- إزاي تعمل أول provider
- الأنواع المختلفة من Providers (Classic Syntax)
- أمثلة عملية لكل نوع
- Best practices للـ providers

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تعمل provider لأي نوع بيانات
- تختار النوع المناسب من Providers
- تستخدم الـ Classic Syntax بثقة
- تتبع Best Practices

---

## 🎨 أنواع Providers في Classic Syntax

في Riverpod 3، عندنا 5 أنواع أساسية من Providers:

### نظرة سريعة

| النوع | الاستخدام | مثال |
|-------|-----------|------|
| **Provider** | قيم ثابتة أو مُحسوبة | Config, Services, Computed values |
| **StateProvider** | State بسيط (primitives) | Counter, isDarkMode, selectedIndex |
| **FutureProvider** | Async data (one-time) | Fetch user, Load config |
| **StreamProvider** | Async data (continuous) | Chat messages, Location updates |
| **NotifierProvider** | State معقد (objects) | TodoList, Cart, UserProfile |

**ملحوظة مهمة:** في الـ tutorial ده بنستخدم Classic Syntax. في Section 06 هنتعلم Code Generation اللي بيبسط الكود أكتر.

---

## 🚀 النوع الأول: Provider (Read-only)

حل `Provider` بيستخدم للقيم اللي **مش بتتغير** أو القيم المُحسوبة.

### مثال 1: قيمة ثابتة (Constant)

</div>

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';

// Simple constant value
final appNameProvider = Provider<String>((ref) {
  return 'My Awesome App';
});

// App version
final appVersionProvider = Provider<String>((ref) {
  return '1.0.0';
});
```

<div dir="rtl">

### مثال 2: Configuration Object

</div>

```dart
class AppConfig {
  final String apiUrl;
  final Duration timeout;
  final bool enableLogging;

  AppConfig({
    required this.apiUrl,
    required this.timeout,
    this.enableLogging = false,
  });
}

final configProvider = Provider<AppConfig>((ref) {
  return AppConfig(
    apiUrl: 'https://api.example.com',
    timeout: const Duration(seconds: 30),
    enableLogging: true,
  );
});
```

<div dir="rtl">

### مثال 3: Service (Dependency Injection)

</div>

```dart
class ApiService {
  final String baseUrl;

  ApiService(this.baseUrl);

  Future<User> getUser() async {
    // Implementation
  }
}

// Provide the service
final apiServiceProvider = Provider<ApiService>((ref) {
  final config = ref.watch(configProvider);
  return ApiService(config.apiUrl);
});
```

<div dir="rtl">

### مثال 4: Computed Value (قيمة مُحسوبة)

</div>

```dart
final counterProvider = StateProvider<int>((ref) => 0);

// Computed: double the counter value
final doubledCounterProvider = Provider<int>((ref) {
  final count = ref.watch(counterProvider);
  return count * 2;
});

// Computed: is counter even?
final isEvenProvider = Provider<bool>((ref) {
  final count = ref.watch(counterProvider);
  return count % 2 == 0;
});
```

<div dir="rtl">

### متى تستخدم Provider؟

**استخدم Provider لو:**
- ✅ القيمة ثابتة (constants)
- ✅ Services أو Dependencies
- ✅ Configuration
- ✅ قيمة مُحسوبة من providers تانية

**ما تستخدموش لو:**
- ❌ محتاج تعدل القيمة (استخدم StateProvider)
- ❌ Async operations (استخدم FutureProvider/StreamProvider)

---

## 🔄 النوع التاني: StateProvider (Simple Mutable State)

حل `StateProvider` بيستخدم للـ state البسيط اللي **بيتغير**.

### مثال 1: Counter

</div>

```dart
// Simple counter
final counterProvider = StateProvider<int>((ref) {
  return 0; // Initial value
});
```

<div dir="rtl">

**إزاي تستخدمه:**

</div>

```dart
class CounterPage extends ConsumerWidget {
  const CounterPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Read the value
    final count = ref.watch(counterProvider);

    return Scaffold(
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Count: $count'),
            const SizedBox(height: 20),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                // Decrement
                ElevatedButton(
                  onPressed: () {
                    ref.read(counterProvider.notifier).state--;
                  },
                  child: const Text('-'),
                ),
                const SizedBox(width: 20),
                // Increment
                ElevatedButton(
                  onPressed: () {
                    ref.read(counterProvider.notifier).state++;
                  },
                  child: const Text('+'),
                ),
                const SizedBox(width: 20),
                // Reset
                ElevatedButton(
                  onPressed: () {
                    ref.read(counterProvider.notifier).state = 0;
                  },
                  child: const Text('Reset'),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

<div dir="rtl">

### مثال 2: Boolean Toggle (Dark Mode)

</div>

```dart
// Dark mode toggle
final isDarkModeProvider = StateProvider<bool>((ref) {
  return false; // Light mode by default
});

// Usage
class ThemeSwitch extends ConsumerWidget {
  const ThemeSwitch({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final isDark = ref.watch(isDarkModeProvider);

    return Switch(
      value: isDark,
      onChanged: (value) {
        ref.read(isDarkModeProvider.notifier).state = value;
      },
    );
  }
}
```

<div dir="rtl">

### مثال 3: Selected Index (Bottom Navigation)

</div>

```dart
final selectedIndexProvider = StateProvider<int>((ref) {
  return 0; // First tab
});

class HomePage extends ConsumerWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final selectedIndex = ref.watch(selectedIndexProvider);

    return Scaffold(
      body: IndexedStack(
        index: selectedIndex,
        children: const [
          HomeTab(),
          SearchTab(),
          ProfileTab(),
        ],
      ),
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: selectedIndex,
        onTap: (index) {
          ref.read(selectedIndexProvider.notifier).state = index;
        },
        items: const [
          BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Home'),
          BottomNavigationBarItem(icon: Icon(Icons.search), label: 'Search'),
          BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Profile'),
        ],
      ),
    );
  }
}
```

<div dir="rtl">

### مثال 4: String Value (Selected Language)

</div>

```dart
final localeProvider = StateProvider<String>((ref) {
  return 'ar'; // Arabic by default
});

// Usage
ElevatedButton(
  onPressed: () {
    final currentLocale = ref.read(localeProvider);
    final newLocale = currentLocale == 'ar' ? 'en' : 'ar';
    ref.read(localeProvider.notifier).state = newLocale;
  },
  child: const Text('Toggle Language'),
);
```

<div dir="rtl">

### متى تستخدم StateProvider؟

**استخدم StateProvider لو:**
- ✅ State بسيط (int, bool, String, enum)
- ✅ مفيش business logic معقدة
- ✅ UI state (selected index, toggle, etc.)
- ✅ Temporary state

**ما تستخدموش لو:**
- ❌ State معقد (objects, lists) → استخدم NotifierProvider
- ❌ Async operations → استخدم FutureProvider/StreamProvider
- ❌ محتاج validation أو business logic → استخدم NotifierProvider

---

## ⏳ النوع التالت: FutureProvider (Async One-Time)

حل `FutureProvider` بيستخدم لـ async operations اللي **بتحصل مرة واحدة**.

### مثال 1: Fetch User Data

</div>

```dart
class User {
  final String id;
  final String name;
  final String email;

  User({required this.id, required this.name, required this.email});
}

class ApiService {
  Future<User> getUser() async {
    await Future.delayed(const Duration(seconds: 2)); // Simulate network
    return User(
      id: '1',
      name: 'Ahmed',
      email: 'ahmed@example.com',
    );
  }
}

// Provide ApiService
final apiServiceProvider = Provider<ApiService>((ref) {
  return ApiService();
});

// Fetch user
final userProvider = FutureProvider<User>((ref) async {
  final api = ref.watch(apiServiceProvider);
  return await api.getUser();
});
```

<div dir="rtl">

**إزاي تستخدمه:**

</div>

```dart
class UserProfile extends ConsumerWidget {
  const UserProfile({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider);

    return userAsync.when(
      // Data loaded successfully
      data: (user) {
        return Column(
          children: [
            Text('Name: ${user.name}'),
            Text('Email: ${user.email}'),
          ],
        );
      },
      // Loading
      loading: () {
        return const CircularProgressIndicator();
      },
      // Error occurred
      error: (error, stackTrace) {
        return Text('Error: $error');
      },
    );
  }
}
```

<div dir="rtl">

### مثال 2: Load Remote Config

</div>

```dart
class RemoteConfig {
  final String apiBaseUrl;
  final int maxRetries;
  final bool enableAnalytics;

  RemoteConfig({
    required this.apiBaseUrl,
    required this.maxRetries,
    required this.enableAnalytics,
  });

  factory RemoteConfig.fromJson(Map<String, dynamic> json) {
    return RemoteConfig(
      apiBaseUrl: json['apiBaseUrl'] as String,
      maxRetries: json['maxRetries'] as int,
      enableAnalytics: json['enableAnalytics'] as bool,
    );
  }
}

final remoteConfigProvider = FutureProvider<RemoteConfig>((ref) async {
  // Simulate loading from server
  await Future.delayed(const Duration(seconds: 1));

  return RemoteConfig(
    apiBaseUrl: 'https://api.example.com',
    maxRetries: 3,
    enableAnalytics: true,
  );
});
```

<div dir="rtl">

### مثال 3: Fetch Data with Dependencies

</div>

```dart
final authTokenProvider = StateProvider<String?>((ref) => null);

final authenticatedUserProvider = FutureProvider<User>((ref) async {
  final token = ref.watch(authTokenProvider);

  if (token == null) {
    throw Exception('Not authenticated');
  }

  final api = ref.watch(apiServiceProvider);
  return await api.getUserWithToken(token);
});
```

<div dir="rtl">

### متى تستخدم FutureProvider؟

**استخدم FutureProvider لو:**
- ✅ Async operation بيحصل مرة واحدة
- ✅ API call لجلب بيانات
- ✅ Loading state مطلوب
- ✅ البيانات read-only (مش هتتعدل)

**ما تستخدموش لو:**
- ❌ محتاج تعدل البيانات → استخدم AsyncNotifierProvider
- ❌ Continuous data stream → استخدم StreamProvider
- ❌ محتاج retry أو refresh logic → استخدم AsyncNotifierProvider

---

## 📡 النوع الرابع: StreamProvider (Continuous Data)

حل `StreamProvider` بيستخدم للبيانات اللي بتيجي بشكل **مستمر** (stream).

### مثال 1: Chat Messages

</div>

```dart
class Message {
  final String id;
  final String text;
  final DateTime timestamp;

  Message({
    required this.id,
    required this.text,
    required this.timestamp,
  });
}

class ChatService {
  Stream<List<Message>> messagesStream() async* {
    // Simulate real-time messages
    int count = 0;
    while (true) {
      await Future.delayed(const Duration(seconds: 2));
      count++;
      yield [
        Message(
          id: '$count',
          text: 'Message $count',
          timestamp: DateTime.now(),
        ),
      ];
    }
  }
}

final chatServiceProvider = Provider<ChatService>((ref) {
  return ChatService();
});

final messagesProvider = StreamProvider<List<Message>>((ref) {
  final chatService = ref.watch(chatServiceProvider);
  return chatService.messagesStream();
});
```

<div dir="rtl">

**إزاي تستخدمه:**

</div>

```dart
class ChatPage extends ConsumerWidget {
  const ChatPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final messagesAsync = ref.watch(messagesProvider);

    return messagesAsync.when(
      data: (messages) {
        return ListView.builder(
          itemCount: messages.length,
          itemBuilder: (context, index) {
            final message = messages[index];
            return ListTile(
              title: Text(message.text),
              subtitle: Text(message.timestamp.toString()),
            );
          },
        );
      },
      loading: () => const CircularProgressIndicator(),
      error: (error, stack) => Text('Error: $error'),
    );
  }
}
```

<div dir="rtl">

### مثال 2: Location Updates

</div>

```dart
class LocationService {
  Stream<Position> locationStream() async* {
    while (true) {
      await Future.delayed(const Duration(seconds: 5));
      yield Position(
        latitude: 30.0444 + (DateTime.now().second % 10) / 1000,
        longitude: 31.2357 + (DateTime.now().second % 10) / 1000,
      );
    }
  }
}

class Position {
  final double latitude;
  final double longitude;

  Position({required this.latitude, required this.longitude});
}

final locationProvider = StreamProvider<Position>((ref) {
  final locationService = LocationService();
  return locationService.locationStream();
});
```

<div dir="rtl">

### مثال 3: Firebase Firestore Stream

</div>

```dart
// Assuming you have Firebase setup
final todosStreamProvider = StreamProvider<List<Todo>>((ref) {
  // This would be your actual Firestore query
  // return FirebaseFirestore.instance
  //     .collection('todos')
  //     .snapshots()
  //     .map((snapshot) => snapshot.docs.map((doc) => Todo.fromFirestore(doc)).toList());

  // For this example, we'll simulate
  return Stream.periodic(const Duration(seconds: 3), (count) {
    return List.generate(count + 1, (i) => Todo(id: '$i', title: 'Todo $i'));
  });
});
```

<div dir="rtl">

### متى تستخدم StreamProvider؟

**استخدم StreamProvider لو:**
- ✅ بيانات بتيجي بشكل مستمر (continuous)
- ✅ Real-time updates (chat, notifications)
- ✅ Location tracking
- ✅ Firebase/WebSocket streams

**ما تستخدموش لو:**
- ❌ One-time operation → استخدم FutureProvider
- ❌ محتاج تعدل البيانات → استخدم AsyncNotifierProvider

---

## 🗂️ النوع الخامس: NotifierProvider (Complex Mutable State)

حل `NotifierProvider` بيستخدم للـ state المعقد اللي **محتاج methods وbusiness logic**.

### مثال 1: Todo List

</div>

```dart
class Todo {
  final String id;
  final String title;
  final bool completed;

  Todo({
    required this.id,
    required this.title,
    this.completed = false,
  });

  Todo copyWith({String? title, bool? completed}) {
    return Todo(
      id: id,
      title: title ?? this.title,
      completed: completed ?? this.completed,
    );
  }
}

// Notifier class (manages the state)
class TodosNotifier extends Notifier<List<Todo>> {
  @override
  List<Todo> build() {
    // Initial state
    return [];
  }

  void addTodo(String title) {
    final newTodo = Todo(
      id: DateTime.now().toString(),
      title: title,
    );

    // Update state (immutably)
    state = [...state, newTodo];
  }

  void removeTodo(String id) {
    state = state.where((todo) => todo.id != id).toList();
  }

  void toggleTodo(String id) {
    state = [
      for (final todo in state)
        if (todo.id == id)
          todo.copyWith(completed: !todo.completed)
        else
          todo,
    ];
  }

  void clearCompleted() {
    state = state.where((todo) => !todo.completed).toList();
  }
}

// Provider
final todosProvider = NotifierProvider<TodosNotifier, List<Todo>>(
  () => TodosNotifier(),
);
```

<div dir="rtl">

**إزاي تستخدمه:**

</div>

```dart
class TodoListPage extends ConsumerWidget {
  const TodoListPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final todos = ref.watch(todosProvider);

    return Scaffold(
      appBar: AppBar(
        title: const Text('Todos'),
        actions: [
          IconButton(
            icon: const Icon(Icons.delete_sweep),
            onPressed: () {
              ref.read(todosProvider.notifier).clearCompleted();
            },
          ),
        ],
      ),
      body: ListView.builder(
        itemCount: todos.length,
        itemBuilder: (context, index) {
          final todo = todos[index];
          return ListTile(
            title: Text(
              todo.title,
              style: TextStyle(
                decoration: todo.completed
                    ? TextDecoration.lineThrough
                    : null,
              ),
            ),
            leading: Checkbox(
              value: todo.completed,
              onChanged: (_) {
                ref.read(todosProvider.notifier).toggleTodo(todo.id);
              },
            ),
            trailing: IconButton(
              icon: const Icon(Icons.delete),
              onPressed: () {
                ref.read(todosProvider.notifier).removeTodo(todo.id);
              },
            ),
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          // Show dialog to add todo
          _showAddTodoDialog(context, ref);
        },
        child: const Icon(Icons.add),
      ),
    );
  }

  void _showAddTodoDialog(BuildContext context, WidgetRef ref) {
    showDialog(
      context: context,
      builder: (context) {
        String title = '';
        return AlertDialog(
          title: const Text('Add Todo'),
          content: TextField(
            onChanged: (value) => title = value,
            decoration: const InputDecoration(hintText: 'Todo title'),
          ),
          actions: [
            TextButton(
              onPressed: () => Navigator.pop(context),
              child: const Text('Cancel'),
            ),
            TextButton(
              onPressed: () {
                if (title.isNotEmpty) {
                  ref.read(todosProvider.notifier).addTodo(title);
                  Navigator.pop(context);
                }
              },
              child: const Text('Add'),
            ),
          ],
        );
      },
    );
  }
}
```

<div dir="rtl">

### مثال 2: Shopping Cart

</div>

```dart
class CartItem {
  final String productId;
  final String name;
  final double price;
  final int quantity;

  CartItem({
    required this.productId,
    required this.name,
    required this.price,
    this.quantity = 1,
  });

  CartItem copyWith({int? quantity}) {
    return CartItem(
      productId: productId,
      name: name,
      price: price,
      quantity: quantity ?? this.quantity,
    );
  }
}

class ShoppingCartNotifier extends Notifier<List<CartItem>> {
  @override
  List<CartItem> build() {
    return [];
  }

  void addItem(String productId, String name, double price) {
    // Check if item already exists
    final existingIndex = state.indexWhere(
      (item) => item.productId == productId,
    );

    if (existingIndex >= 0) {
      // Increment quantity
      state = [
        for (int i = 0; i < state.length; i++)
          if (i == existingIndex)
            state[i].copyWith(quantity: state[i].quantity + 1)
          else
            state[i],
      ];
    } else {
      // Add new item
      final newItem = CartItem(
        productId: productId,
        name: name,
        price: price,
      );
      state = [...state, newItem];
    }
  }

  void removeItem(String productId) {
    state = state.where((item) => item.productId != productId).toList();
  }

  void updateQuantity(String productId, int quantity) {
    if (quantity <= 0) {
      removeItem(productId);
      return;
    }

    state = [
      for (final item in state)
        if (item.productId == productId)
          item.copyWith(quantity: quantity)
        else
          item,
    ];
  }

  void clear() {
    state = [];
  }

  double get total {
    return state.fold(0, (sum, item) => sum + (item.price * item.quantity));
  }
}

final cartProvider = NotifierProvider<ShoppingCartNotifier, List<CartItem>>(
  () => ShoppingCartNotifier(),
);
```

<div dir="rtl">

### متى تستخدم NotifierProvider؟

**استخدم NotifierProvider لو:**
- ✅ State معقد (objects, lists, nested data)
- ✅ محتاج methods متعددة
- ✅ فيه business logic
- ✅ محتاج computed properties (getters)

**ما تستخدموش لو:**
- ❌ State بسيط → استخدم StateProvider
- ❌ Async state → استخدم AsyncNotifierProvider
- ❌ Read-only data → استخدم Provider

---

## 📊 مقارنة سريعة

| الميزة | Provider | StateProvider | FutureProvider | StreamProvider | NotifierProvider |
|-------|----------|---------------|----------------|----------------|------------------|
| **Mutable** | ❌ | ✅ | ❌ | ❌ | ✅ |
| **Async** | ❌ | ❌ | ✅ | ✅ | ❌ |
| **Methods** | ❌ | ❌ | ❌ | ❌ | ✅ |
| **Complexity** | 🟢 بسيط | 🟢 بسيط | 🟡 متوسط | 🟡 متوسط | 🟠 معقد |
| **Best for** | Constants, Services | Simple state | API calls | Real-time | Complex state |

---

## 💡 Best Practices

### 1. اختيار النوع المناسب

</div>

```dart
// ✅ GOOD: Use the right provider type
final nameProvider = Provider<String>((ref) => 'Ahmed'); // Constant
final counterProvider = StateProvider<int>((ref) => 0); // Simple mutable
final userProvider = FutureProvider<User>((ref) async => await api.getUser()); // Async

// ❌ BAD: Wrong provider type
final counterProvider = Provider<int>((ref) => 0); // Can't mutate!
final nameProvider = StateProvider<String>((ref) => 'Ahmed'); // Overkill
```

<div dir="rtl">

### 2. Naming Convention

</div>

```dart
// ✅ GOOD: Clear descriptive names with 'Provider' suffix
final userProfileProvider = FutureProvider<UserProfile>(...);
final todosListProvider = NotifierProvider<TodosNotifier, List<Todo>>(...);
final isDarkModeProvider = StateProvider<bool>(...);

// ❌ BAD: Unclear or missing suffix
final user = FutureProvider<UserProfile>(...); // Missing 'Provider'
final data = StateProvider<bool>(...); // Unclear
final p1 = Provider<String>(...); // Very bad
```

<div dir="rtl">

### 3. File Organization

</div>

```dart
// Good structure:
// lib/
//   providers/
//     auth_provider.dart
//     user_provider.dart
//     todos_provider.dart
//   models/
//     user.dart
//     todo.dart
//   screens/
//     home_screen.dart

// In auth_provider.dart:
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../models/user.dart';

final authTokenProvider = StateProvider<String?>((ref) => null);

final currentUserProvider = FutureProvider<User?>((ref) async {
  final token = ref.watch(authTokenProvider);
  if (token == null) return null;
  return await api.getUserWithToken(token);
});
```

<div dir="rtl">

### 4. Keep Providers Simple

</div>

```dart
// ✅ GOOD: Single responsibility
final userIdProvider = StateProvider<String?>((ref) => null);

final userProvider = FutureProvider<User>((ref) async {
  final userId = ref.watch(userIdProvider);
  if (userId == null) throw Exception('No user ID');
  return await api.getUser(userId);
});

// ❌ BAD: Doing too much
final everythingProvider = FutureProvider<Map<String, dynamic>>((ref) async {
  final user = await api.getUser();
  final posts = await api.getPosts();
  final comments = await api.getComments();
  // This is too much! Split into separate providers
  return {'user': user, 'posts': posts, 'comments': comments};
});
```

<div dir="rtl">

### 5. استخدم Dependencies

</div>

```dart
// ✅ GOOD: Providers depend on each other
final apiServiceProvider = Provider<ApiService>((ref) {
  return ApiService();
});

final authProvider = StateProvider<String?>((ref) => null);

final userRepositoryProvider = Provider<UserRepository>((ref) {
  final api = ref.watch(apiServiceProvider);
  final auth = ref.watch(authProvider);
  return UserRepository(api: api, authToken: auth);
});

// ❌ BAD: Everything is independent
final apiService = ApiService(); // Global! Not good
final userRepository = UserRepository(api: apiService); // Can't override for tests
```

<div dir="rtl">

---

## 📝 ملخص

**أنواع Providers الخمسة:**

النوع **Provider:**
- للقيم الثابتة والمُحسوبة
- مثال: `final nameProvider = Provider<String>((ref) => 'Ahmed');`

النوع **StateProvider:**
- للـ state البسيط اللي بيتغير
- مثال: `final counterProvider = StateProvider<int>((ref) => 0);`

النوع **FutureProvider:**
- للـ async operations اللي بتحصل مرة واحدة
- مثال: `final userProvider = FutureProvider<User>((ref) async => ...);`

النوع **StreamProvider:**
- للبيانات المستمرة (streams)
- مثال: `final messagesProvider = StreamProvider<List<Message>>((ref) => ...);`

النوع **NotifierProvider:**
- للـ state المعقد مع methods
- مثال: `final todosProvider = NotifierProvider<TodosNotifier, List<Todo>>(() => ...);`

**Best Practices:**
1. اختار النوع المناسب للـ state
2. استخدم naming convention واضح
3. نظم الـ providers في ملفات منفصلة
4. خلي كل provider عنده مسؤولية واحدة
5. استخدم dependencies بين الـ providers

---

## 🔗 الخطوة الجاية

دلوقتي بعد ما عرفت إزاي تعمل providers، الخطوة الجاية هي تتعلم إزاي **تقرأ** الـ providers في الـ UI:
- روح على `03-reading-providers.md`

**ملحوظة:** في الملف ده استخدمنا Classic Syntax. في Section 06 هنتعلم **Code Generation** اللي بيبسط الكود أكتر ويوفر type safety أحسن.

---

## 📚 المصادر

- [Riverpod Providers Guide](https://riverpod.dev/docs/providers/provider)
- [StateProvider Guide](https://riverpod.dev/docs/providers/state_provider)
- [FutureProvider Guide](https://riverpod.dev/docs/providers/future_provider)
- [StreamProvider Guide](https://riverpod.dev/docs/providers/stream_provider)
- [NotifierProvider Guide](https://riverpod.dev/docs/providers/notifier_provider)

</div>
