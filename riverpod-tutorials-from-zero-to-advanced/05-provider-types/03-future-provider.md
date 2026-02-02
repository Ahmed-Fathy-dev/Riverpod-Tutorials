<div dir="rtl">

# FutureProvider - البيانات الـ Async لمرة واحدة

**المستوى**: 🟡 متوسط

## 📌 المفاهيم الأساسية

في الملف ده هنتكلم عن:
- إيه هو FutureProvider ومتى نستخدمه
- AsyncValue للتعامل مع loading/error/data
- أمثلة عملية شاملة
- متى نستخدم AsyncNotifierProvider بدلاً منه
- Best practices

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تستخدم FutureProvider للـ async data
- تتعامل مع AsyncValue
- تعرف متى تستخدم FutureProvider
- تفهم الفرق بينه وبين AsyncNotifierProvider

---

## 🔍 إيه هو FutureProvider؟

**FutureProvider** هو provider للبيانات اللي بتيجي من **async operation** لمرة واحدة.

</div>

```dart
// Simple FutureProvider
final userProvider = FutureProvider<User>((ref) async {
  // Async operation
  final response = await http.get('https://api.example.com/user');
  return User.fromJson(response.data);
});

// Usage in widget
final userAsync = ref.watch(userProvider);

userAsync.when(
  data: (user) => Text('Hello, ${user.name}'),
  loading: () => CircularProgressIndicator(),
  error: (error, stack) => Text('Error: $error'),
);
```

<div dir="rtl">

**متى تستخدمه:**
- ✅ One-time data fetch (مرة واحدة)
- ✅ Initial data loading
- ✅ Load from storage/database
- ✅ Configuration loading

**متى ما تستخدموش:**
- ❌ محتاج refresh → use AsyncNotifierProvider
- ❌ Continuous updates → use StreamProvider
- ❌ محتاج methods → use AsyncNotifierProvider

---

## 📦 AsyncValue - التعامل مع الحالات المختلفة

عندنا 3 حالات ممكنة:

</div>

```dart
// AsyncValue<T> has 3 states:
// 1. AsyncLoading - جاري التحميل
// 2. AsyncData<T> - البيانات جاهزة
// 3. AsyncError - حدث خطأ

final userAsync = ref.watch(userProvider);

// Method 1: when - Handle all states
userAsync.when(
  data: (user) => Text('Hello, ${user.name}'),
  loading: () => CircularProgressIndicator(),
  error: (error, stack) => Text('Error: $error'),
);

// Method 2: maybeWhen - Handle some states
userAsync.maybeWhen(
  data: (user) => Text('Hello, ${user.name}'),
  orElse: () => Text('Loading or error...'),
);

// Method 3: Pattern matching
switch (userAsync) {
  case AsyncData(:final value):
    return Text('Hello, ${value.name}');
  case AsyncError(:final error):
    return Text('Error: $error');
  case AsyncLoading():
    return CircularProgressIndicator();
}

// Method 4: Direct properties
if (userAsync.isLoading) {
  return CircularProgressIndicator();
}
if (userAsync.hasError) {
  return Text('Error: ${userAsync.error}');
}
if (userAsync.hasValue) {
  return Text('Hello, ${userAsync.value!.name}');
}
```

<div dir="rtl">

---

## 🎨 الاستخدامات الأساسية

### 1. Fetch User Data

</div>

```dart
// User API
class UserApi {
  Future<User> getCurrentUser() async {
    final response = await http.get('https://api.example.com/user/me');
    return User.fromJson(jsonDecode(response.body));
  }
}

// API provider
final userApiProvider = Provider<UserApi>((ref) => UserApi());

// User provider
final currentUserProvider = FutureProvider<User>((ref) async {
  final api = ref.watch(userApiProvider);
  return await api.getCurrentUser();
});

// Usage
class UserProfile extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(currentUserProvider);

    return userAsync.when(
      data: (user) => Column(
        children: [
          CircleAvatar(backgroundImage: NetworkImage(user.avatarUrl)),
          SizedBox(height: 8),
          Text(user.name, style: TextStyle(fontSize: 24)),
          Text(user.email, style: TextStyle(color: Colors.grey)),
        ],
      ),
      loading: () => Center(child: CircularProgressIndicator()),
      error: (error, stack) => Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(Icons.error_outline, size: 48, color: Colors.red),
            SizedBox(height: 16),
            Text('Failed to load user'),
            Text('$error', style: TextStyle(fontSize: 12)),
          ],
        ),
      ),
    );
  }
}
```

<div dir="rtl">

### 2. Load Configuration

</div>

```dart
// App configuration
class AppConfig {
  final String apiUrl;
  final int timeout;
  final bool debugMode;

  AppConfig({
    required this.apiUrl,
    required this.timeout,
    required this.debugMode,
  });

  factory AppConfig.fromJson(Map<String, dynamic> json) {
    return AppConfig(
      apiUrl: json['apiUrl'],
      timeout: json['timeout'],
      debugMode: json['debugMode'],
    );
  }
}

// Config provider
final appConfigProvider = FutureProvider<AppConfig>((ref) async {
  // Load from local storage
  final prefs = await SharedPreferences.getInstance();
  final configJson = prefs.getString('app_config');

  if (configJson != null) {
    return AppConfig.fromJson(jsonDecode(configJson));
  }

  // Default config
  return AppConfig(
    apiUrl: 'https://api.example.com',
    timeout: 30,
    debugMode: false,
  );
});

// Usage in other providers
final apiClientProvider = Provider<ApiClient>((ref) {
  // Wait for config to load
  final config = ref.watch(appConfigProvider).value;

  if (config == null) {
    throw Exception('Config not loaded yet');
  }

  return ApiClient(
    baseUrl: config.apiUrl,
    timeout: Duration(seconds: config.timeout),
  );
});
```

<div dir="rtl">

### 3. Initial Data Fetch

</div>

```dart
// Products provider
final productsProvider = FutureProvider<List<Product>>((ref) async {
  final api = ref.watch(apiProvider);
  return await api.getProducts();
});

// Categories provider
final categoriesProvider = FutureProvider<List<Category>>((ref) async {
  final api = ref.watch(apiProvider);
  return await api.getCategories();
});

// Combined data
final initialDataProvider = FutureProvider<InitialData>((ref) async {
  // Fetch both in parallel
  final results = await Future.wait([
    ref.watch(productsProvider.future),
    ref.watch(categoriesProvider.future),
  ]);

  return InitialData(
    products: results[0] as List<Product>,
    categories: results[1] as List<Category>,
  );
});

// Home Screen
class HomeScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final initialDataAsync = ref.watch(initialDataProvider);

    return initialDataAsync.when(
      data: (data) => HomeContent(
        products: data.products,
        categories: data.categories,
      ),
      loading: () => SplashScreen(),
      error: (error, stack) => ErrorScreen(error: error),
    );
  }
}
```

<div dir="rtl">

---

## 💻 أمثلة عملية كاملة

### مثال 1: Weather App

</div>

```dart
// Weather model
class Weather {
  final String city;
  final double temperature;
  final String condition;
  final String icon;

  Weather({
    required this.city,
    required this.temperature,
    required this.condition,
    required this.icon,
  });
}

// Weather API
class WeatherApi {
  Future<Weather> getWeather(String city) async {
    await Future.delayed(Duration(seconds: 1)); // Simulate network

    // In real app, fetch from API
    return Weather(
      city: city,
      temperature: 25.0,
      condition: 'Sunny',
      icon: '☀️',
    );
  }
}

// API provider
final weatherApiProvider = Provider<WeatherApi>((ref) => WeatherApi());

// Selected city
final selectedCityProvider = StateProvider<String>((ref) => 'Cairo');

// Weather provider (depends on selected city)
final weatherProvider = FutureProvider<Weather>((ref) async {
  final api = ref.watch(weatherApiProvider);
  final city = ref.watch(selectedCityProvider);

  // This will re-run when city changes!
  return await api.getWeather(city);
});

// Weather Screen
class WeatherScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final weatherAsync = ref.watch(weatherProvider);
    final selectedCity = ref.watch(selectedCityProvider);

    return Scaffold(
      appBar: AppBar(title: Text('Weather')),
      body: Column(
        children: [
          // City selector
          Padding(
            padding: EdgeInsets.all(16),
            child: DropdownButton<String>(
              value: selectedCity,
              isExpanded: true,
              items: ['Cairo', 'Alexandria', 'Giza', 'Luxor']
                  .map((city) => DropdownMenuItem(
                        value: city,
                        child: Text(city),
                      ))
                  .toList(),
              onChanged: (city) {
                if (city != null) {
                  ref.read(selectedCityProvider.notifier).state = city;
                }
              },
            ),
          ),

          // Weather display
          Expanded(
            child: weatherAsync.when(
              data: (weather) => Center(
                child: Column(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    Text(weather.icon, style: TextStyle(fontSize: 72)),
                    SizedBox(height: 16),
                    Text(
                      '${weather.temperature}°C',
                      style: TextStyle(fontSize: 48),
                    ),
                    Text(
                      weather.condition,
                      style: TextStyle(fontSize: 24, color: Colors.grey),
                    ),
                  ],
                ),
              ),
              loading: () => Center(child: CircularProgressIndicator()),
              error: (error, stack) => Center(
                child: Column(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    Icon(Icons.error, size: 64, color: Colors.red),
                    SizedBox(height: 16),
                    Text('Failed to load weather'),
                  ],
                ),
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

<div dir="rtl">

### مثال 2: Language Localization

</div>

```dart
// Translations
class Translations {
  final Map<String, String> data;

  Translations(this.data);

  String get(String key) => data[key] ?? key;
}

// Current locale
final currentLocaleProvider = StateProvider<String>((ref) => 'en');

// Translations provider
final translationsProvider = FutureProvider<Translations>((ref) async {
  final locale = ref.watch(currentLocaleProvider);

  // Load translations from assets
  final jsonString = await rootBundle.loadString('assets/i18n/$locale.json');
  final Map<String, dynamic> json = jsonDecode(jsonString);

  return Translations(json.map((key, value) => MapEntry(key, value.toString())));
});

// Helper to get translation
final tProvider = Provider<String Function(String)>((ref) {
  final translationsAsync = ref.watch(translationsProvider);

  return (String key) {
    return translationsAsync.maybeWhen(
      data: (translations) => translations.get(key),
      orElse: () => key,
    );
  };
});

// Usage
class HomeScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final t = ref.watch(tProvider);

    return Scaffold(
      appBar: AppBar(title: Text(t('home.title'))),
      body: Column(
        children: [
          Text(t('home.welcome')),
          Text(t('home.description')),
        ],
      ),
    );
  }
}
```

<div dir="rtl">

### مثال 3: Offline-First Data Loading

</div>

```dart
// Local database
class LocalDb {
  Future<List<Article>> getArticles() async {
    // Get from local database
    await Future.delayed(Duration(milliseconds: 100));
    return []; // Return cached articles
  }

  Future<void> saveArticles(List<Article> articles) async {
    // Save to local database
  }
}

// API
class ArticlesApi {
  Future<List<Article>> getArticles() async {
    final response = await http.get('https://api.example.com/articles');
    return (jsonDecode(response.body) as List)
        .map((json) => Article.fromJson(json))
        .toList();
  }
}

// Providers
final localDbProvider = Provider<LocalDb>((ref) => LocalDb());
final articlesApiProvider = Provider<ArticlesApi>((ref) => ArticlesApi());

// Articles provider (offline-first)
final articlesProvider = FutureProvider<List<Article>>((ref) async {
  final db = ref.watch(localDbProvider);
  final api = ref.watch(articlesApiProvider);

  // 1. Try to get from cache first
  final cachedArticles = await db.getArticles();

  if (cachedArticles.isNotEmpty) {
    // Return cached immediately
    return cachedArticles;
  }

  // 2. Fetch from API if cache is empty
  try {
    final articles = await api.getArticles();

    // 3. Save to cache
    await db.saveArticles(articles);

    return articles;
  } catch (e) {
    // If API fails, return empty (or throw)
    return [];
  }
});

// Articles Screen
class ArticlesScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final articlesAsync = ref.watch(articlesProvider);

    return Scaffold(
      appBar: AppBar(title: Text('Articles')),
      body: articlesAsync.when(
        data: (articles) {
          if (articles.isEmpty) {
            return Center(child: Text('No articles'));
          }

          return ListView.builder(
            itemCount: articles.length,
            itemBuilder: (context, index) {
              final article = articles[index];
              return ListTile(
                title: Text(article.title),
                subtitle: Text(article.summary),
                onTap: () {
                  // Navigate to article details
                },
              );
            },
          );
        },
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

## 🔄 Dependencies و Re-computation

FutureProvider بيعيد التنفيذ لما الـ dependencies تتغير:

</div>

```dart
// Depends on user ID
final userIdProvider = StateProvider<String>((ref) => '1');

// User provider (re-runs when userId changes)
final userProvider = FutureProvider<User>((ref) async {
  final userId = ref.watch(userIdProvider);

  print('Fetching user: $userId'); // Will print when userId changes

  final api = ref.watch(apiProvider);
  return await api.getUser(userId);
});

// Change user ID
ref.read(userIdProvider.notifier).state = '2'; // Will trigger refetch!
```

<div dir="rtl">

---

## ⚠️ أخطاء شائعة

### ❌ خطأ 1: استخدام FutureProvider لما محتاج Refresh

</div>

```dart
// ❌ WRONG - Can't refresh easily
final todosProvider = FutureProvider<List<Todo>>((ref) async {
  return await api.getTodos();
});

// How to refresh? Hard!
// Need to use ref.invalidate or hack with dependencies

// ✅ CORRECT - Use AsyncNotifierProvider
class TodosNotifier extends AsyncNotifier<List<Todo>> {
  @override
  Future<List<Todo>> build() async {
    return await api.getTodos();
  }

  Future<void> refresh() async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() => api.getTodos());
  }
}

final todosProvider = AsyncNotifierProvider<TodosNotifier, List<Todo>>(
  () => TodosNotifier(),
);

// Easy to refresh!
ref.read(todosProvider.notifier).refresh();
```

<div dir="rtl">

### ❌ خطأ 2: Accessing .value مباشرة بدون check

</div>

```dart
// ❌ WRONG - May be null!
final user = ref.watch(userProvider).value!; // Crash if loading or error!

// ✅ CORRECT - Use .when or check first
final userAsync = ref.watch(userProvider);

// Option 1: when
userAsync.when(
  data: (user) => Text(user.name),
  loading: () => Text('Loading...'),
  error: (e, s) => Text('Error'),
);

// Option 2: Check manually
if (userAsync.hasValue) {
  final user = userAsync.value!;
  return Text(user.name);
}
```

<div dir="rtl">

### ❌ خطأ 3: Not handling errors

</div>

```dart
// ❌ WRONG - Only handles data
final userAsync = ref.watch(userProvider);
return Text(userAsync.value!.name); // Crash on error!

// ✅ CORRECT - Handle all states
return userAsync.when(
  data: (user) => Text(user.name),
  loading: () => CircularProgressIndicator(),
  error: (error, stack) => Text('Error: $error'),
);
```

<div dir="rtl">

---

## 💡 Best Practices

### 1. دايماً استخدم .when للتعامل مع كل الحالات

</div>

```dart
// ✅ Good - Handles all cases
userAsync.when(
  data: (user) => UserCard(user),
  loading: () => Shimmer(),
  error: (e, s) => ErrorCard(e),
);
```

<div dir="rtl">

### 2. استخدم .future للـ async/await في providers

</div>

```dart
// ✅ Good - Using .future
final combinedProvider = FutureProvider<CombinedData>((ref) async {
  final user = await ref.watch(userProvider.future);
  final posts = await ref.watch(postsProvider.future);

  return CombinedData(user: user, posts: posts);
});
```

<div dir="rtl">

### 3. استخدم FutureProvider للـ Initial Load فقط

</div>

```dart
// ✅ Good - One-time load
final configProvider = FutureProvider<AppConfig>((ref) async {
  return await loadConfig();
});

// ❌ Bad - Need refresh (use AsyncNotifierProvider)
final todosProvider = FutureProvider<List<Todo>>((ref) async {
  return await api.getTodos();
});
```

<div dir="rtl">

---

## 🆚 FutureProvider vs AsyncNotifierProvider

| Feature | FutureProvider | AsyncNotifierProvider |
|---------|----------------|----------------------|
| **Use Case** | One-time load | Load + methods |
| **Refresh** | صعب | سهل |
| **Methods** | لا | نعم |
| **Update** | Re-run provider | Methods |

</div>

```dart
// FutureProvider - One-time
final userProvider = FutureProvider<User>((ref) async {
  return await api.getUser();
});
// Hard to refresh

// AsyncNotifierProvider - With methods
class UserNotifier extends AsyncNotifier<User> {
  @override
  Future<User> build() async {
    return await api.getUser();
  }

  Future<void> refresh() async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() => api.getUser());
  }
}
// Easy to refresh!
```

<div dir="rtl">

---

## 📝 ملخص

**FutureProvider** يستخدم لـ:
- ✅ One-time async operations
- ✅ Initial data loading
- ✅ Configuration loading
- ✅ Load from storage

**مش يستخدم لـ:**
- ❌ محتاج refresh → AsyncNotifierProvider
- ❌ محتاج methods → AsyncNotifierProvider
- ❌ Continuous updates → StreamProvider

**Best Practices:**
- دايماً `.when` للتعامل مع الحالات
- استخدم `.future` للـ await في providers
- للـ initial load فقط

---

## 🔗 الخطوة الجاية

في الملف الجاي هنتكلم عن:
- **StreamProvider** - للبيانات المستمرة
- Real-time data
- Firebase integration

---

## 📚 المصادر

- [FutureProvider - Riverpod Docs](https://riverpod.dev/docs/providers/future_provider)
- [AsyncValue - Handling Async Data](https://riverpod.dev/docs/concepts/async_value)

</div>
