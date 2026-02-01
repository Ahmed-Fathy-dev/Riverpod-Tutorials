<div dir="rtl">

# Family Modifier - Parameterized Providers

**المستوى**: 🟡 متوسط

## 📌 المفاهيم الأساسية

في الملف ده هنتكلم عن:
- إيه هو Family modifier
- إزاي تعمل providers بـ parameters
- Cache management
- متى تستخدم family
- Best practices

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تعمل parameterized providers
- تدير الـ cache بكفاءة
- تستخدم family في الحالات المختلفة
- تتجنب الأخطاء الشائعة
- تحسن الـ performance

---

## 🔍 إيه هو Family؟

**Family** بيسمحلك تعمل provider يستقبل parameters.

</div>

```dart
// Without family: One provider for all users
final userProvider = FutureProvider<UserData>((ref) async {
  // How do we know WHICH user?
  return await api.getUser(); // ❌ Missing user ID
});

// With family: Different provider instance for each user
final userProvider = FutureProvider.family<UserData, String>((ref, userId) async {
  // userId is the parameter!
  return await api.getUser(userId);
});

// Usage
class UserProfile extends ConsumerWidget {
  final String userId;

  UserProfile(this.userId);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Different provider instance for each userId
    final user = ref.watch(userProvider(userId));

    return user.when(
      data: (userData) => Text(userData.name),
      loading: () => CircularProgressIndicator(),
      error: (e, s) => Text('Error'),
    );
  }
}
```

<div dir="rtl">

---

## 🎨 الاستخدام الأساسي

### Simple Parameter

</div>

```dart
// Single parameter with family
final productProvider = FutureProvider.family<Product, String>((ref, productId) async {
  return await api.getProduct(productId);
});

// Usage
class ProductPage extends ConsumerWidget {
  final String productId;

  ProductPage(this.productId);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final product = ref.watch(productProvider(productId));

    return product.when(
      data: (p) => Text(p.name),
      loading: () => CircularProgressIndicator(),
      error: (e, s) => Text('Error'),
    );
  }
}
```

<div dir="rtl">

### Multiple Parameters

</div>

```dart
// Multiple parameters using a record with family
typedef UserPostsParams = ({String userId, int page});

final userPostsProvider = FutureProvider.family<List<Post>, UserPostsParams>(
  (ref, params) async {
    return await api.getUserPosts(
      userId: params.userId,
      page: params.page,
    );
  },
);

// Usage
class UserPostsList extends ConsumerWidget {
  final String userId;
  final int page;

  UserPostsList({required this.userId, required this.page});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final posts = ref.watch(
      userPostsProvider((userId: userId, page: page)),
    );

    return posts.when(
      data: (posts) => PostsList(posts),
      loading: () => CircularProgressIndicator(),
      error: (e, s) => ErrorWidget(e),
    );
  }
}
```

<div dir="rtl">

### مع NotifierProvider

</div>

```dart
// Stateful provider with parameter using family
class TodoListNotifier extends FamilyAsyncNotifier<List<Todo>, String> {
  @override
  Future<List<Todo>> build(String listId) async {
    // Parameter is available via arg property
    return await api.getTodos(listId);
  }

  Future<void> addTodo(String title) async {
    final newTodo = Todo(
      id: DateTime.now().toString(),
      title: title,
      listId: arg, // Access parameter via 'arg'
    );

    state = AsyncData([...state.value!, newTodo]);
    await api.saveTodo(newTodo);
  }

  Future<void> removeTodo(String todoId) async {
    state = AsyncData(
      state.value!.where((todo) => todo.id != todoId).toList(),
    );
    await api.deleteTodo(todoId);
  }
}

final todoListProvider = AsyncNotifierProvider.family<TodoListNotifier, List<Todo>, String>(
  () => TodoListNotifier(),
);

// Usage
class TodoListWidget extends ConsumerWidget {
  final String listId;

  TodoListWidget(this.listId);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final todosAsync = ref.watch(todoListProvider(listId));

    return todosAsync.when(
      data: (todos) => Column(
        children: [
          ...todos.map((todo) => TodoTile(todo)),
          ElevatedButton(
            onPressed: () {
              ref.read(todoListProvider(listId).notifier)
                .addTodo('New todo');
            },
            child: Text('Add Todo'),
          ),
        ],
      ),
      loading: () => CircularProgressIndicator(),
      error: (e, s) => Text('Error: $e'),
    );
  }
}
```

<div dir="rtl">

---

## 💾 Caching Behavior

Family providers بيعملوا **automatic caching** لكل parameter value مختلف.

</div>

```dart
final userProvider = FutureProvider.family<User, String>((ref, userId) async {
  print('Fetching user: $userId');
  return await api.getUser(userId);
});

// Widget 1
class ProfilePage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Fetches user '1' - prints: "Fetching user: 1"
    final user = ref.watch(userProvider('1'));
    return Text(user.value?.name ?? '');
  }
}

// Widget 2
class FollowersList extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Uses cached user '1' - no print (cached!)
    final user = ref.watch(userProvider('1'));
    return Text('${user.value?.followers} followers');
  }
}

// Widget 3
class AnotherProfile extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Fetches user '2' - prints: "Fetching user: 2"
    final user = ref.watch(userProvider('2'));
    return Text(user.value?.name ?? '');
  }
}
```

<div dir="rtl">

**الـ Cache:**

</div>

```
userProvider('1') → Instance 1 (cached)
userProvider('2') → Instance 2 (cached)
userProvider('3') → Instance 3 (cached)
// Each parameter creates a separate cached instance
```

<div dir="rtl">

---

## 🎮 Cache Management

### تلقائي (AutoDispose)

</div>

```dart
// Default: AutoDispose with family
final productProvider = FutureProvider.family.autoDispose<Product, String>(
  (ref, id) async {
    print('Creating provider for product: $id');

    ref.onDispose(() {
      print('Disposing provider for product: $id');
    });

    return await api.getProduct(id);
  },
);

// When widget is removed, the provider instance is disposed
class ProductCard extends ConsumerWidget {
  final String productId;

  ProductCard(this.productId);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final product = ref.watch(productProvider(productId));
    // Provider created for this productId
    return Text(product.value?.name ?? '');
  }
  // When ProductCard is removed:
  // - Provider for this productId is disposed
  // - Cache entry is removed
}
```

<div dir="rtl">

### Manual Cache Control

</div>

```dart
final articleProvider = FutureProvider.family.autoDispose<Article, String>(
  (ref, id) async {
    // Keep in cache for 5 minutes
    final link = ref.keepAlive();

    Timer(Duration(minutes: 5), () {
      link.close();
    });

    return await api.getArticle(id);
  },
);
```

<div dir="rtl">

### Manual Invalidation

</div>

```dart
class RefreshButton extends ConsumerWidget {
  final String productId;

  RefreshButton(this.productId);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return IconButton(
      icon: Icon(Icons.refresh),
      onPressed: () {
        // Invalidate specific instance
        ref.invalidate(productProvider(productId));

        // Or invalidate ALL instances
        ref.invalidate(productProvider);
      },
    );
  }
}
```

<div dir="rtl">

---

## 🧪 أمثلة عملية

### مثال 1: Paginated List

</div>

```dart
// Model
class PaginatedData<T> {
  final List<T> items;
  final int page;
  final bool hasMore;

  PaginatedData({
    required this.items,
    required this.page,
    required this.hasMore,
  });
}

// Provider with page parameter using family
final productsPageProvider = FutureProvider.family<PaginatedData<Product>, int>(
  (ref, page) async {
    final products = await api.getProducts(page: page, limit: 20);

    return PaginatedData(
      items: products,
      page: page,
      hasMore: products.length == 20,
    );
  },
);

// Widget
class ProductList extends ConsumerStatefulWidget {
  @override
  ConsumerState<ProductList> createState() => _ProductListState();
}

class _ProductListState extends ConsumerState<ProductList> {
  int _currentPage = 1;
  final List<Product> _allProducts = [];

  @override
  Widget build(BuildContext context) {
    final pageData = ref.watch(productsPageProvider(_currentPage));

    return pageData.when(
      data: (data) {
        // Add to cumulative list
        if (!_allProducts.contains(data.items.first)) {
          _allProducts.addAll(data.items);
        }

        return ListView.builder(
          itemCount: _allProducts.length + (data.hasMore ? 1 : 0),
          itemBuilder: (context, index) {
            if (index == _allProducts.length) {
              // Load more button
              return ElevatedButton(
                onPressed: () {
                  setState(() {
                    _currentPage++;
                  });
                },
                child: Text('Load More'),
              );
            }

            return ProductTile(_allProducts[index]);
          },
        );
      },
      loading: () => CircularProgressIndicator(),
      error: (e, s) => Text('Error: $e'),
    );
  }
}
```

<div dir="rtl">

### مثال 2: Search Provider

</div>

```dart
// Search provider with query parameter using family
final searchProductsProvider = FutureProvider.family.autoDispose<List<Product>, String>(
  (ref, query) async {
    // Don't search for empty queries
    if (query.isEmpty) {
      return [];
    }

    // Debounce: wait for user to stop typing
    await Future.delayed(Duration(milliseconds: 300));

    // Check if still the same query
    if (ref.state is! AsyncLoading) {
      return ref.state.value ?? [];
    }

    return await api.searchProducts(query);
  },
);

// Search widget
class SearchPage extends ConsumerStatefulWidget {
  @override
  ConsumerState<SearchPage> createState() => _SearchPageState();
}

class _SearchPageState extends ConsumerState<SearchPage> {
  String _query = '';

  @override
  Widget build(BuildContext context) {
    final resultsAsync = ref.watch(searchProductsProvider(_query));

    return Column(
      children: [
        TextField(
          onChanged: (value) {
            setState(() {
              _query = value;
            });
          },
          decoration: InputDecoration(
            hintText: 'Search products...',
          ),
        ),
        Expanded(
          child: resultsAsync.when(
            data: (results) {
              if (_query.isEmpty) {
                return Center(child: Text('Start typing to search'));
              }

              if (results.isEmpty) {
                return Center(child: Text('No results'));
              }

              return ListView.builder(
                itemCount: results.length,
                itemBuilder: (context, index) {
                  return ProductTile(results[index]);
                },
              );
            },
            loading: () => Center(child: CircularProgressIndicator()),
            error: (e, s) => Center(child: Text('Error: $e')),
          ),
        ),
      ],
    );
  }
}
```

<div dir="rtl">

### مثال 3: Filter Combinations

</div>

```dart
// Filter parameters using a record
typedef ProductFilters = ({
  String? category,
  double? minPrice,
  double? maxPrice,
  String? sortBy,
});

final filteredProductsProvider = FutureProvider.family<List<Product>, ProductFilters>(
  (ref, filters) async {
    return await api.getProducts(
      category: filters.category,
      minPrice: filters.minPrice,
      maxPrice: filters.maxPrice,
      sortBy: filters.sortBy,
    );
  },
);

// Usage
class ProductsPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final filters = ref.watch(currentFiltersProvider);

    final productsAsync = ref.watch(
      filteredProductsProvider(filters),
    );

    return Column(
      children: [
        FilterControls(), // Updates currentFiltersProvider
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

// Filter state provider
class CurrentFiltersNotifier extends Notifier<ProductFilters> {
  @override
  ProductFilters build() {
    return (
      category: null,
      minPrice: null,
      maxPrice: null,
      sortBy: 'name',
    );
  }

  void updateCategory(String? category) {
    state = (
      category: category,
      minPrice: state.minPrice,
      maxPrice: state.maxPrice,
      sortBy: state.sortBy,
    );
  }

  void updatePriceRange(double? min, double? max) {
    state = (
      category: state.category,
      minPrice: min,
      maxPrice: max,
      sortBy: state.sortBy,
    );
  }

  void updateSort(String sortBy) {
    state = (
      category: state.category,
      minPrice: state.minPrice,
      maxPrice: state.maxPrice,
      sortBy: sortBy,
    );
  }
}

final currentFiltersProvider = NotifierProvider<CurrentFiltersNotifier, ProductFilters>(
  () => CurrentFiltersNotifier(),
);
```

<div dir="rtl">

### مثال 4: Nested Dependencies

</div>

```dart
// User profile provider with family
final userProvider = FutureProvider.family<User, String>((ref, userId) async {
  return await api.getUser(userId);
});

// User's posts provider (depends on user)
final userPostsProvider = FutureProvider.family<List<Post>, String>((ref, userId) async {
  // Watch user to get additional info if needed
  final user = await ref.watch(userProvider(userId).future);

  return await api.getUserPosts(user.id);
});

// User's followers provider
final userFollowersProvider = FutureProvider.family<List<User>, String>((ref, userId) async {
  final followerIds = await api.getFollowerIds(userId);

  // Fetch each follower (reuses cached user providers!)
  final followers = await Future.wait(
    followerIds.map((id) => ref.watch(userProvider(id).future)),
  );

  return followers;
});

// Usage
class UserProfilePage extends ConsumerWidget {
  final String userId;

  UserProfilePage(this.userId);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider(userId));
    final postsAsync = ref.watch(userPostsProvider(userId));
    final followersAsync = ref.watch(userFollowersProvider(userId));

    return Column(
      children: [
        userAsync.when(
          data: (user) => UserHeader(user),
          loading: () => CircularProgressIndicator(),
          error: (e, s) => ErrorWidget(e),
        ),
        postsAsync.when(
          data: (posts) => PostsList(posts),
          loading: () => CircularProgressIndicator(),
          error: (e, s) => ErrorWidget(e),
        ),
        followersAsync.when(
          data: (followers) => FollowersList(followers),
          loading: () => CircularProgressIndicator(),
          error: (e, s) => ErrorWidget(e),
        ),
      ],
    );
  }
}
```

<div dir="rtl">

---

## ⚙️ Parameter Types

### Primitives (مباشر)

</div>

```dart
// String
final userProvider = FutureProvider.family<User, String>((ref, id) async {
  return await api.getUser(id);
});

// int
final pageProvider = FutureProvider.family<Page, int>((ref, pageNumber) async {
  return await api.getPage(pageNumber);
});

// bool
final productsProvider = Provider.family<List<Product>, bool>((ref, sortAscending) {
  final all = ref.watch(allProductsProvider);
  return sortAscending ? all : all.reversed.toList();
});
```

<div dir="rtl">

### Records (Multiple Parameters)

</div>

```dart
// Named record with family
typedef SearchParams = ({String query, int page});

final searchProvider = FutureProvider.family<SearchResults, SearchParams>(
  (ref, params) async {
    return await api.search(
      query: params.query,
      page: params.page,
    );
  },
);

// Usage
ref.watch(searchProvider((query: 'flutter', page: 1)));
```

<div dir="rtl">

### Custom Classes (مع Equality)

</div>

```dart
// Custom parameter class
class SearchParams {
  final String query;
  final int page;
  final String? category;

  SearchParams({
    required this.query,
    required this.page,
    this.category,
  });

  // IMPORTANT: Must override == and hashCode
  @override
  bool operator ==(Object other) {
    if (identical(this, other)) return true;

    return other is SearchParams &&
        other.query == query &&
        other.page == page &&
        other.category == category;
  }

  @override
  int get hashCode {
    return query.hashCode ^ page.hashCode ^ category.hashCode;
  }
}

// Provider with family
final advancedSearchProvider = FutureProvider.family<SearchResults, SearchParams>(
  (ref, params) async {
    return await api.search(
      query: params.query,
      page: params.page,
      category: params.category,
    );
  },
);

// Usage
final params = SearchParams(query: 'flutter', page: 1);
ref.watch(advancedSearchProvider(params));
```

<div dir="rtl">

---

## ⚠️ أخطاء شائعة

### خطأ 1: نسيان Equality في Custom Classes

</div>

```dart
// ❌ WRONG - No equality override
class Filters {
  final String category;
  final double price;

  Filters(this.category, this.price);
  // Missing == and hashCode
}

final filteredProvider = Provider.family<List<Product>, Filters>((ref, filters) {
  // Each call creates NEW provider instance!
  // Even with same values
  return [];
});

// ✅ CORRECT - With equality
class Filters {
  final String category;
  final double price;

  Filters(this.category, this.price);

  @override
  bool operator ==(Object other) {
    return identical(this, other) ||
        other is Filters &&
            other.category == category &&
            other.price == price;
  }

  @override
  int get hashCode => category.hashCode ^ price.hashCode;
}
```

<div dir="rtl">

### خطأ 2: استخدام Mutable Parameters

</div>

```dart
// ❌ WRONG - Mutable parameter
class MutableFilters {
  String category;
  double price;

  MutableFilters(this.category, this.price);
}

final filteredProvider2 = Provider.family<List<Product>, MutableFilters>((ref, filters) {
  // Dangerous! filters can change
  return [];
});

// ✅ CORRECT - Immutable parameters
class ImmutableFilters {
  final String category;
  final double price;

  const ImmutableFilters(this.category, this.price);
}
```

<div dir="rtl">

### خطأ 3: Over-caching

</div>

```dart
// ❌ WRONG - Keeping too many instances
final searchBadProvider = FutureProvider.family<SearchResults, String>((ref, query) async {
  // Keeps EVERY search query cached forever!
  ref.keepAlive();

  return await api.search(query);
});

// ✅ CORRECT - Use AutoDispose or timed cache
final searchGoodProvider = FutureProvider.family.autoDispose<SearchResults, String>((ref, query) async {
  // AutoDispose by default
  // Or use timed cache
  final link = ref.keepAlive();
  Timer(Duration(minutes: 5), link.close);

  return await api.search(query);
});
```

<div dir="rtl">

---

## 💡 Best Practices

### 1. استخدم Records للـ Multiple Parameters

</div>

```dart
// ✅ Good: Named record with family
typedef DataParams = ({String id, int version});

final dataProvider = FutureProvider.family<Data, DataParams>(
  (ref, params) async {
    return await api.getData(params.id, params.version);
  },
);

// Usage is clear
ref.watch(dataProvider((id: '123', version: 2)));
```

<div dir="rtl">

### 2. Immutable Parameters Always

</div>

```dart
// ✅ Good: Immutable
class SearchFilters {
  final String query;
  final String? category;

  const SearchFilters({
    required this.query,
    this.category,
  });

  // copyWith for updates
  SearchFilters copyWith({
    String? query,
    String? category,
  }) {
    return SearchFilters(
      query: query ?? this.query,
      category: category ?? this.category,
    );
  }

  @override
  bool operator ==(Object other) => /* ... */;

  @override
  int get hashCode => /* ... */;
}
```

<div dir="rtl">

### 3. Consider Cache Size

</div>

```dart
// For search/filters: Limited cache time
final searchProvider2 = FutureProvider.family.autoDispose<List<Product>, String>(
  (ref, query) async {
    final link = ref.keepAlive();

    // Cache for 2 minutes
    Timer(Duration(minutes: 2), link.close);

    return await api.search(query);
  },
);

// For user data: Longer cache time
final userProvider2 = FutureProvider.family.autoDispose<User, String>(
  (ref, userId) async {
    final link = ref.keepAlive();

    // Cache for 30 minutes
    Timer(Duration(minutes: 30), link.close);

    return await api.getUser(userId);
  },
);
```

<div dir="rtl">

---

## 📝 ملخص

**Family Modifier:**
- ✅ Creates parameterized providers
- ✅ Automatic caching per parameter
- ✅ Each parameter = separate instance
- ✅ AutoDispose by default

**Parameter Types:**
- Primitives: String, int, bool
- Records: For multiple params
- Custom classes: Must override == and hashCode

**Best Practices:**
- Use records for multiple parameters
- Always immutable parameters
- Override equality for custom classes
- Manage cache size with timed keepAlive
- Consider AutoDispose vs manual cache

---

## 🔗 الخطوة الجاية

في الملف الجاي هنتكلم عن:
- **AutoDispose Modifier** بالتفصيل
- Memory management
- When to use vs KeepAlive

---

## 📚 المصادر

- [Family Modifier](https://riverpod.dev/docs/concepts/modifiers/family)
- [Caching](https://riverpod.dev/docs/concepts/about_code_generation#caching-considerations)

</div>
