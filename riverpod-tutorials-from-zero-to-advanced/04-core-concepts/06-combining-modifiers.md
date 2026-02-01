<div dir="rtl">

# Combining Modifiers - دمج المُعدلات

**المستوى**: 🟡 متوسط

## 📌 المفاهيم الأساسية

في الملف ده هنتكلم عن:
- دمج Family مع AutoDispose
- Cache management للـ parameterized providers
- Advanced patterns
- Performance optimization
- Best practices

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تدمج Family مع AutoDispose بكفاءة
- تدير الـ cache للـ parameterized providers
- تستخدم patterns متقدمة
- تحسن الـ performance
- تتجنب الأخطاء الشائعة

---

## 🔍 Family + AutoDispose

في Riverpod 3، **كل** الـ providers بـ AutoDispose by default، حتى لو family.

</div>

```dart
// Family with AutoDispose (default behavior)
@riverpod
Future<User> user(UserRef ref, String userId) async {
  print('Fetching user: $userId');

  ref.onDispose(() {
    print('Disposing user provider for: $userId');
  });

  return await api.getUser(userId);
}

// Usage
class UserProfile extends ConsumerWidget {
  final String userId;

  UserProfile(this.userId);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final user = ref.watch(userProvider(userId));
    // Provider created for this userId

    return user.when(
      data: (user) => Text(user.name),
      loading: () => CircularProgressIndicator(),
      error: (e, s) => Text('Error'),
    );
  }
  // When widget is removed:
  // - Provider for this userId is disposed
  // - Cache entry is removed
  // - Memory is freed
}
```

<div dir="rtl">

---

## 🎨 Cache Management Strategies

### Strategy 1: AutoDispose (Default)

**Use case:** Temporary data, frequently changing parameters

</div>

```dart
// Perfect for search results
@riverpod
Future<List<Product>> searchResults(
  SearchResultsRef ref,
  String query,
) async {
  print('Searching for: $query');

  ref.onDispose(() {
    print('Disposing search results for: $query');
  });

  // AutoDispose - disposed when not in use
  return await api.search(query);
}

// Usage
class SearchPage extends ConsumerStatefulWidget {
  @override
  ConsumerState<SearchPage> createState() => _SearchPageState();
}

class _SearchPageState extends ConsumerState<SearchPage> {
  String _query = '';

  @override
  Widget build(BuildContext context) {
    final resultsAsync = ref.watch(searchResultsProvider(_query));

    return Column(
      children: [
        TextField(
          onChanged: (value) => setState(() => _query = value),
        ),
        Expanded(
          child: resultsAsync.when(
            data: (results) => ListView.builder(
              itemCount: results.length,
              itemBuilder: (context, index) {
                return ProductTile(results[index]);
              },
            ),
            loading: () => CircularProgressIndicator(),
            error: (e, s) => Text('Error'),
          ),
        ),
      ],
    );
  }
}
// Each query creates a provider instance
// Old query providers are disposed automatically
// Memory efficient!
```

<div dir="rtl">

### Strategy 2: Timed Cache

**Use case:** API data that's expensive to fetch

</div>

```dart
// Cache for limited time
@riverpod
Future<UserDetails> userDetails(
  UserDetailsRef ref,
  String userId,
) async {
  print('Fetching details for user: $userId');

  // Keep alive for 5 minutes
  final link = ref.keepAlive();

  Timer(Duration(minutes: 5), () {
    print('Cache expired for user: $userId');
    link.close();
  });

  ref.onDispose(() {
    print('Disposing user details for: $userId');
  });

  return await api.getUserDetails(userId);
}

// Multiple widgets can share cached data
class UserProfile extends ConsumerWidget {
  final String userId;

  UserProfile(this.userId);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final details = ref.watch(userDetailsProvider(userId));
    // First access: fetches from API
    // Subsequent accesses (within 5 min): uses cache
    return Text(details.value?.bio ?? '');
  }
}
```

<div dir="rtl">

### Strategy 3: Conditional Cache

**Use case:** Cache only important data

</div>

```dart
@riverpod
Future<Article> article(
  ArticleRef ref,
  String articleId,
) async {
  final article = await api.getArticle(articleId);

  // Cache premium articles longer
  if (article.isPremium) {
    final link = ref.keepAlive();
    Timer(Duration(hours: 1), link.close);
  } else {
    // Regular articles: shorter cache
    final link = ref.keepAlive();
    Timer(Duration(minutes: 10), link.close);
  }

  return article;
}
```

<div dir="rtl">

### Strategy 4: Permanent Cache (Selective)

**Use case:** Reference data that rarely changes

</div>

```dart
@riverpod
Future<Category> category(
  CategoryRef ref,
  String categoryId,
) async {
  // Categories rarely change - keep permanently
  ref.keepAlive();

  return await api.getCategory(categoryId);
}

// Manual refresh when needed
class CategoryManager extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return ElevatedButton(
      onPressed: () {
        // Invalidate all category instances
        ref.invalidate(categoryProvider);
      },
      child: Text('Refresh Categories'),
    );
  }
}
```

<div dir="rtl">

---

## 🎯 Advanced Patterns

### Pattern 1: Paginated Data with Smart Caching

</div>

```dart
// Cache each page separately
@riverpod
Future<Page<Product>> productsPage(
  ProductsPageRef ref,
  ({String category, int page}) params,
) async {
  print('Fetching page ${params.page} of ${params.category}');

  // Keep pages cached for 10 minutes
  final link = ref.keepAlive();
  Timer(Duration(minutes: 10), link.close);

  ref.onDispose(() {
    print('Disposing page ${params.page}');
  });

  return await api.getProducts(
    category: params.category,
    page: params.page,
  );
}

// Aggregated view provider
@riverpod
class ProductsList extends _$ProductsList {
  @override
  Future<List<Product>> build(String category) async {
    // Start with first page
    final firstPage = await ref.watch(
      productsPageProvider((category: category, page: 1)).future,
    );

    return firstPage.items;
  }

  Future<void> loadNextPage() async {
    final currentItems = state.value ?? [];
    final nextPage = (currentItems.length ~/ 20) + 1;

    final pageData = await ref.watch(
      productsPageProvider((
        category: arg,
        page: nextPage,
      )).future,
    );

    state = AsyncData([...currentItems, ...pageData.items]);
  }
}
```

<div dir="rtl">

### Pattern 2: Filtering with Multiple Parameters

</div>

```dart
// Complex filter parameters
typedef Filters = ({
  String? category,
  double? minPrice,
  double? maxPrice,
  String? brand,
  bool? inStock,
});

@riverpod
Future<List<Product>> filteredProducts(
  FilteredProductsRef ref,
  Filters filters,
) async {
  // Cache filtered results for 2 minutes
  final link = ref.keepAlive();
  Timer(Duration(minutes: 2), link.close);

  return await api.getProducts(
    category: filters.category,
    minPrice: filters.minPrice,
    maxPrice: filters.maxPrice,
    brand: filters.brand,
    inStock: filters.inStock,
  );
}

// Filter state management
@riverpod
class ProductFilters extends _$ProductFilters {
  @override
  Filters build() {
    return (
      category: null,
      minPrice: null,
      maxPrice: null,
      brand: null,
      inStock: null,
    );
  }

  void updateCategory(String? category) {
    state = (
      category: category,
      minPrice: state.minPrice,
      maxPrice: state.maxPrice,
      brand: state.brand,
      inStock: state.inStock,
    );
  }

  void updatePriceRange(double? min, double? max) {
    state = (
      category: state.category,
      minPrice: min,
      maxPrice: max,
      brand: state.brand,
      inStock: state.inStock,
    );
  }

  void reset() {
    state = (
      category: null,
      minPrice: null,
      maxPrice: null,
      brand: null,
      inStock: null,
    );
  }
}

// Usage
class ProductsPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final filters = ref.watch(productFiltersProvider);
    final productsAsync = ref.watch(filteredProductsProvider(filters));

    return Column(
      children: [
        FilterPanel(), // Updates filters
        Expanded(
          child: productsAsync.when(
            data: (products) => ProductGrid(products),
            loading: () => CircularProgressIndicator(),
            error: (e, s) => ErrorWidget(e),
          ),
        ),
      ],
    );
  }
}
```

<div dir="rtl">

### Pattern 3: Nested Family Providers

</div>

```dart
// User provider
@riverpod
Future<User> user(UserRef ref, String userId) async {
  final link = ref.keepAlive();
  Timer(Duration(minutes: 30), link.close);

  return await api.getUser(userId);
}

// User's posts (depends on user)
@riverpod
Future<List<Post>> userPosts(
  UserPostsRef ref,
  ({String userId, int page}) params,
) async {
  // Watch the user
  final user = await ref.watch(userProvider(params.userId).future);

  // Cache posts for 5 minutes
  final link = ref.keepAlive();
  Timer(Duration(minutes: 5), link.close);

  return await api.getPosts(
    userId: user.id,
    page: params.page,
  );
}

// Post details (depends on user)
@riverpod
Future<PostDetails> postDetails(
  PostDetailsRef ref,
  ({String userId, String postId}) params,
) async {
  // Reuse cached user
  final user = await ref.watch(userProvider(params.userId).future);

  return await api.getPostDetails(
    userId: user.id,
    postId: params.postId,
  );
}

// Usage creates an efficient dependency graph
class UserPage extends ConsumerWidget {
  final String userId;

  UserPage(this.userId);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final user = ref.watch(userProvider(userId));
    final posts = ref.watch(
      userPostsProvider((userId: userId, page: 1)),
    );

    // All share the same cached user instance!
    return Column(
      children: [
        user.when(
          data: (u) => UserHeader(u),
          loading: () => CircularProgressIndicator(),
          error: (e, s) => ErrorWidget(e),
        ),
        posts.when(
          data: (p) => PostsList(p),
          loading: () => CircularProgressIndicator(),
          error: (e, s) => ErrorWidget(e),
        ),
      ],
    );
  }
}
```

<div dir="rtl">

### Pattern 4: Smart Prefetching

</div>

```dart
// Prefetch next page
@riverpod
class ProductsPrefetcher extends _$ProductsPrefetcher {
  @override
  void build() {
    // No state needed
  }

  Future<void> prefetchNextPage(String category, int currentPage) async {
    final nextPage = currentPage + 1;

    // Prefetch in background
    ref.read(
      productsPageProvider((
        category: category,
        page: nextPage,
      )).future,
    );
  }
}

// Usage in list
class ProductList extends ConsumerWidget {
  final String category;

  ProductList(this.category);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return NotificationListener<ScrollNotification>(
      onNotification: (notification) {
        // Prefetch when user scrolls to 80%
        if (notification.metrics.pixels >
            notification.metrics.maxScrollExtent * 0.8) {
          ref.read(productsPrefetcherProvider).prefetchNextPage(
            category,
            1, // current page
          );
        }
        return false;
      },
      child: ListView.builder(
        itemBuilder: (context, index) {
          return ProductTile(index);
        },
      ),
    );
  }
}
```

<div dir="rtl">

---

## 🎮 Cache Size Management

### Monitoring Cache Size

</div>

```dart
@riverpod
class CacheMonitor extends _$CacheMonitor {
  final Map<String, DateTime> _cacheEntries = {};

  @override
  int build() {
    return 0; // Count of active cache entries
  }

  void trackEntry(String key) {
    _cacheEntries[key] = DateTime.now();
    state = _cacheEntries.length;
  }

  void removeEntry(String key) {
    _cacheEntries.remove(key);
    state = _cacheEntries.length;
  }

  void clearOldEntries(Duration maxAge) {
    final now = DateTime.now();
    _cacheEntries.removeWhere((key, time) {
      return now.difference(time) > maxAge;
    });
    state = _cacheEntries.length;
  }
}

// Track in providers
@riverpod
Future<Data> trackedData(TrackedDataRef ref, String id) async {
  ref.read(cacheMonitorProvider.notifier).trackEntry('data_$id');

  ref.onDispose(() {
    ref.read(cacheMonitorProvider.notifier).removeEntry('data_$id');
  });

  final link = ref.keepAlive();
  Timer(Duration(minutes: 5), link.close);

  return await fetchData(id);
}
```

<div dir="rtl">

### LRU Cache Pattern

</div>

```dart
// Least Recently Used cache
@riverpod
class LruCache extends _$LruCache {
  final _cache = <String, (DateTime, dynamic)>{};
  final _maxSize = 100;

  @override
  int build() {
    return 0;
  }

  void set(String key, dynamic value) {
    // Remove oldest if at max size
    if (_cache.length >= _maxSize) {
      final oldest = _cache.entries.reduce((a, b) {
        return a.value.$1.isBefore(b.value.$1) ? a : b;
      });
      _cache.remove(oldest.key);
    }

    _cache[key] = (DateTime.now(), value);
    state = _cache.length;
  }

  dynamic get(String key) {
    final entry = _cache[key];
    if (entry == null) return null;

    // Update timestamp
    _cache[key] = (DateTime.now(), entry.$2);
    return entry.$2;
  }

  void remove(String key) {
    _cache.remove(key);
    state = _cache.length;
  }
}
```

<div dir="rtl">

---

## ⚠️ مشاكل شائعة

### مشكلة 1: Cache Explosion

</div>

```dart
// ❌ WRONG - Unbounded cache growth
@riverpod
Future<Data> search(SearchRef ref, String query) async {
  ref.keepAlive(); // Every search cached forever!
  return await api.search(query);
}

// User types "f", "fl", "flu", "flut", "flutt", "flutte", "flutter"
// = 7 cached instances that NEVER expire!

// ✅ CORRECT - Timed expiration
@riverpod
Future<Data> search(SearchRef ref, String query) async {
  final link = ref.keepAlive();
  Timer(Duration(minutes: 2), link.close); // Cleanup old searches
  return await api.search(query);
}
```

<div dir="rtl">

### مشكلة 2: Parameter Mutability

</div>

```dart
// ❌ WRONG - Mutable parameter
class MutableFilter {
  String category;
  double price;

  MutableFilter(this.category, this.price);
}

@riverpod
List<Product> filtered(FilteredRef ref, MutableFilter filter) {
  // Dangerous! filter can change
  return [];
}

// ✅ CORRECT - Immutable parameter
class ImmutableFilter {
  final String category;
  final double price;

  const ImmutableFilter(this.category, this.price);

  @override
  bool operator ==(Object other) =>
      identical(this, other) ||
      other is ImmutableFilter &&
          category == other.category &&
          price == other.price;

  @override
  int get hashCode => category.hashCode ^ price.hashCode;
}
```

<div dir="rtl">

### مشكلة 3: Forgetting Equality Override

</div>

```dart
// ❌ WRONG - No equality
class Params {
  final String id;
  final int version;

  Params(this.id, this.version);
  // Missing == and hashCode
}

@riverpod
Future<Data> data(DataRef ref, Params params) async {
  // Params(id: '1', version: 1) != Params(id: '1', version: 1)
  // Creates duplicate providers!
  return await fetchData(params);
}

// ✅ CORRECT - With equality
class Params {
  final String id;
  final int version;

  Params(this.id, this.version);

  @override
  bool operator ==(Object other) =>
      identical(this, other) ||
      other is Params &&
          id == other.id &&
          version == other.version;

  @override
  int get hashCode => id.hashCode ^ version.hashCode;
}
```

<div dir="rtl">

---

## 💡 Best Practices

### 1. Choose the Right Cache Strategy

</div>

```dart
// Search: Short-lived
@riverpod
Future<List<Product>> search(SearchRef ref, String query) async {
  final link = ref.keepAlive();
  Timer(Duration(minutes: 2), link.close);
  return await api.search(query);
}

// User data: Medium-lived
@riverpod
Future<User> user(UserRef ref, String id) async {
  final link = ref.keepAlive();
  Timer(Duration(minutes: 30), link.close);
  return await api.getUser(id);
}

// Reference data: Long-lived
@riverpod
Future<Category> category(CategoryRef ref, String id) async {
  ref.keepAlive(); // Permanent
  return await api.getCategory(id);
}
```

<div dir="rtl">

### 2. Use Records for Multiple Parameters

</div>

```dart
// ✅ Good: Clear and type-safe
@riverpod
Future<Data> data(
  DataRef ref,
  ({String id, int version, String? locale}) params,
) async {
  return await api.getData(
    id: params.id,
    version: params.version,
    locale: params.locale,
  );
}

// Usage is self-documenting
ref.watch(dataProvider((
  id: '123',
  version: 2,
  locale: 'en',
)));
```

<div dir="rtl">

### 3. Monitor Cache in Debug Mode

</div>

```dart
@riverpod
Future<Data> monitoredData(MonitoredDataRef ref, String id) async {
  if (kDebugMode) {
    print('Creating provider for: $id');
    ref.onDispose(() => print('Disposing provider for: $id'));
  }

  final link = ref.keepAlive();
  Timer(Duration(minutes: 5), link.close);

  return await fetchData(id);
}
```

<div dir="rtl">

### 4. Invalidate Related Providers

</div>

```dart
class RefreshButton extends ConsumerWidget {
  final String userId;

  RefreshButton(this.userId);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return IconButton(
      icon: Icon(Icons.refresh),
      onPressed: () {
        // Invalidate all related providers
        ref.invalidate(userProvider(userId));
        ref.invalidate(userPostsProvider((userId: userId, page: 1)));
        ref.invalidate(userFollowersProvider(userId));
      },
    );
  }
}
```

<div dir="rtl">

---

## 📝 ملخص

**Combining Family + AutoDispose:**
- ✅ Default: Each parameter creates auto-disposing instance
- ✅ Cache management: Use keepAlive + Timer
- ✅ Choose strategy based on data type
- ✅ Monitor cache size in production

**Cache Strategies:**
- Temporary (search): 1-5 minutes
- User data: 10-30 minutes
- Reference data: Permanent
- AutoDispose: UI state

**Best Practices:**
- Immutable parameters
- Override equality for custom types
- Use records for multiple params
- Set cache expiration
- Monitor cache size

---

## 🔗 الخطوة الجاية

في الملف الجاي هنتكلم عن:
- **Provider Observers** بالتفصيل
- Logging and debugging
- Analytics integration

---

## 📚 المصادر

- [Family Modifier](https://riverpod.dev/docs/concepts/modifiers/family)
- [AutoDispose](https://riverpod.dev/docs/concepts/modifiers/auto_dispose)

</div>
