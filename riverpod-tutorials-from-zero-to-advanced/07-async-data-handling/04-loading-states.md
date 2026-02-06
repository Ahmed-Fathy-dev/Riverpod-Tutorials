<div dir="rtl">

# Loading States & UI Patterns

**المستوى**: 🟡 متوسط

## 📌 المفاهيم الأساسية

في الملف ده هنتعلم:
- أنواع Loading Indicators المختلفة
- Shimmer Effects
- Skeleton Loaders
- Pull-to-Refresh Patterns
- Infinite Scroll
- isRefreshing vs isReloading UI

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تختار Loading UI المناسب
- تعمل Shimmer Effects
- تستخدم Skeleton Loaders
- تنفذ Pull-to-Refresh
- تعمل Infinite Scroll
- تفرق بين Loading States

---

## 🔄 أنواع Loading States

### 1. Initial Loading - أول مرة تحميل

</div>

```dart
@riverpod
Future<List<Product>> products(ProductsRef ref) async {
  return await api.getProducts();
}

// UI
class ProductsScreen extends ConsumerWidget {
  const ProductsScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final productsAsync = ref.watch(productsProvider);

    return switch (productsAsync) {
      // First load - show centered spinner
      AsyncLoading() => const Center(
          child: CircularProgressIndicator(),
        ),

      AsyncData(:final value) => ProductGrid(value),
      AsyncError(:final error) => ErrorMessage(error),
    };
  }
}
```

<div dir="rtl">

---

### 2. Refreshing - تحديث البيانات

</div>

```dart
class ProductsScreen extends ConsumerWidget {
  const ProductsScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final productsAsync = ref.watch(productsProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('Products')),
      body: Column(
        children: [
          // Show indicator when refreshing with old data
          if (productsAsync.isRefreshing)
            const LinearProgressIndicator(),

          Expanded(
            child: switch (productsAsync) {
              AsyncData(:final value) => ProductGrid(value),
              AsyncError(:final error) => ErrorMessage(error),
              _ => const Center(child: CircularProgressIndicator()),
            },
          ),
        ],
      ),
    );
  }
}
```

<div dir="rtl">

---

### 3. Reloading - إعادة تحميل بسبب تغيير Parameters

</div>

```dart
@riverpod
Future<List<Product>> productsByCategory(
  ProductsByCategoryRef ref,
  String categoryId,
) async {
  return await api.getProductsByCategory(categoryId);
}

// UI
class ProductsScreen extends ConsumerWidget {
  const ProductsScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final selectedCategory = ref.watch(selectedCategoryProvider);
    final productsAsync = ref.watch(productsByCategoryProvider(selectedCategory));

    return Scaffold(
      appBar: AppBar(title: const Text('Products')),
      body: Column(
        children: [
          // Category selector
          CategorySelector(),

          // Show loading when category changes
          if (productsAsync.isReloading)
            const LinearProgressIndicator(),

          Expanded(
            child: switch (productsAsync) {
              AsyncData(:final value) => ProductGrid(value),
              AsyncError(:final error) => ErrorMessage(error),
              _ => const Center(child: CircularProgressIndicator()),
            },
          ),
        ],
      ),
    );
  }
}
```

<div dir="rtl">

---

### 4. Pagination Loading - تحميل صفحات إضافية

</div>

```dart
@riverpod
class PaginatedProducts extends _$PaginatedProducts {
  @override
  Future<List<Product>> build() async {
    return await api.getProducts(page: 1);
  }

  Future<void> loadMore() async {
    // Get current state
    final currentProducts = state.hasValue ? state.value! : <Product>[];

    // Keep showing current data
    state = AsyncValue.data(currentProducts);

    // Load more
    final newProducts = await api.getProducts(page: _currentPage + 1);

    // Add to current list
    state = AsyncValue.data([...currentProducts, ...newProducts]);
    _currentPage++;
  }

  int _currentPage = 1;
}

// UI
class ProductsScreen extends ConsumerWidget {
  const ProductsScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final productsAsync = ref.watch(paginatedProductsProvider);

    return switch (productsAsync) {
      AsyncData(:final value) => CustomScrollView(
          slivers: [
            SliverList(
              delegate: SliverChildBuilderDelegate(
                (context, index) => ProductTile(value[index]),
                childCount: value.length,
              ),
            ),

            // Loading indicator at bottom
            if (productsAsync.isReloading)
              const SliverToBoxAdapter(
                child: Center(
                  child: Padding(
                    padding: EdgeInsets.all(16),
                    child: CircularProgressIndicator(),
                  ),
                ),
              ),
          ],
        ),

      AsyncError(:final error) => ErrorMessage(error),
      _ => const Center(child: CircularProgressIndicator()),
    };
  }
}
```

<div dir="rtl">

---

## 🎨 Loading UI Patterns

### 1. Circular Progress Indicator

الأكتر شيوعاً - استخدمه للـ short waits.

</div>

```dart
// Basic usage
const CircularProgressIndicator()

// With color
CircularProgressIndicator(
  color: Colors.blue,
)

// With value (determinate)
CircularProgressIndicator(
  value: 0.6, // 60%
)

// Custom size
SizedBox(
  width: 24,
  height: 24,
  child: CircularProgressIndicator(strokeWidth: 2),
)

// In Riverpod
return switch (asyncValue) {
  AsyncLoading() => const Center(
      child: CircularProgressIndicator(),
    ),
  AsyncData(:final value) => DataWidget(value),
  AsyncError(:final error) => ErrorMessage(error),
};
```

<div dir="rtl">

---

### 2. Linear Progress Indicator

مناسب للـ refreshing و progress bars.

</div>

```dart
// Indeterminate (no value)
const LinearProgressIndicator()

// Determinate (with progress)
LinearProgressIndicator(
  value: 0.75, // 75%
)

// Custom color
LinearProgressIndicator(
  backgroundColor: Colors.grey.shade200,
  valueColor: const AlwaysStoppedAnimation<Color>(Colors.blue),
)

// At top of screen (common pattern)
class MyScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final dataAsync = ref.watch(dataProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('My Screen')),
      body: Column(
        children: [
          // Show linear indicator when refreshing
          if (dataAsync.isRefreshing)
            const LinearProgressIndicator(),

          Expanded(
            child: switch (dataAsync) {
              AsyncData(:final value) => DataView(value),
              _ => const Center(child: CircularProgressIndicator()),
            },
          ),
        ],
      ),
    );
  }
}
```

<div dir="rtl">

---

### 3. Shimmer Effect

يخلي المستخدم يحس إن الـ content بيتحمل. من [Flutter Official Docs](https://docs.flutter.dev/cookbook/effects/shimmer-loading).

</div>

```dart
// Shimmer widget implementation
class Shimmer extends StatefulWidget {
  const Shimmer({
    super.key,
    required this.child,
    this.gradient = const LinearGradient(
      colors: [Color(0xFFEBEBF4), Color(0xFFF4F4F4), Color(0xFFEBEBF4)],
      stops: [0.1, 0.3, 0.4],
      begin: Alignment(-1.0, -0.3),
      end: Alignment(1.0, 0.3),
    ),
  });

  final Widget child;
  final Gradient gradient;

  @override
  State<Shimmer> createState() => _ShimmerState();
}

class _ShimmerState extends State<Shimmer> with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 1500),
    )..repeat();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: _controller,
      child: widget.child,
      builder: (context, child) {
        return ShaderMask(
          shaderCallback: (bounds) {
            return widget.gradient.createShader(
              Rect.fromLTWH(
                -bounds.width * _controller.value,
                0,
                bounds.width * 3,
                bounds.height,
              ),
            );
          },
          blendMode: BlendMode.srcATop,
          child: child,
        );
      },
    );
  }
}

// Usage - Product card skeleton
class ProductCardSkeleton extends StatelessWidget {
  const ProductCardSkeleton({super.key});

  @override
  Widget build(BuildContext context) {
    return Shimmer(
      child: Card(
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Image placeholder
            Container(
              width: double.infinity,
              height: 200,
              color: Colors.white,
            ),
            const SizedBox(height: 8),
            // Title placeholder
            Container(
              width: double.infinity,
              height: 20,
              margin: const EdgeInsets.symmetric(horizontal: 16),
              color: Colors.white,
            ),
            const SizedBox(height: 8),
            // Subtitle placeholder
            Container(
              width: 150,
              height: 16,
              margin: const EdgeInsets.symmetric(horizontal: 16),
              color: Colors.white,
            ),
            const SizedBox(height: 16),
          ],
        ),
      ),
    );
  }
}

// In Riverpod
class ProductsScreen extends ConsumerWidget {
  const ProductsScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final productsAsync = ref.watch(productsProvider);

    return switch (productsAsync) {
      // Show shimmer while loading
      AsyncLoading() => GridView.builder(
          gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
            crossAxisCount: 2,
            childAspectRatio: 0.7,
          ),
          itemCount: 6, // Show 6 skeleton cards
          itemBuilder: (context, index) => const ProductCardSkeleton(),
        ),

      AsyncData(:final value) => GridView.builder(
          gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
            crossAxisCount: 2,
            childAspectRatio: 0.7,
          ),
          itemCount: value.length,
          itemBuilder: (context, index) => ProductCard(value[index]),
        ),

      AsyncError(:final error) => ErrorMessage(error),
    };
  }
}
```

<div dir="rtl">

---

### 4. Skeleton Loader (using skeletonizer package)

أسهل طريقة لعمل skeleton loaders! من [pub.dev/packages/skeletonizer](https://pub.dev/packages/skeletonizer).

</div>

```dart
// Add dependency
// pubspec.yaml:
// dependencies:
//   skeletonizer: ^1.0.0

import 'package:skeletonizer/skeletonizer.dart';

// Automatic skeleton - just wrap your widget!
class UserProfile extends ConsumerWidget {
  const UserProfile({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider);

    return switch (userAsync) {
      // Skeletonizer automatically creates skeleton from your widget!
      AsyncLoading() => Skeletonizer(
          enabled: true,
          child: UserCard(
            user: User.mock(), // Use mock data
          ),
        ),

      AsyncData(:final value) => Skeletonizer(
          enabled: false, // Disable when data is ready
          child: UserCard(user: value),
        ),

      AsyncError(:final error) => ErrorMessage(error),
    };
  }
}

// Custom skeleton with Bone widgets
class ProductCard extends StatelessWidget {
  final Product product;
  final bool isLoading;

  const ProductCard({
    super.key,
    required this.product,
    this.isLoading = false,
  });

  @override
  Widget build(BuildContext context) {
    return Skeletonizer(
      enabled: isLoading,
      child: Card(
        child: Column(
          children: [
            // Image
            Bone(
              width: double.infinity,
              height: 200,
              child: Image.network(product.imageUrl),
            ),

            // Title
            Bone.text(
              words: 3,
              child: Text(product.title),
            ),

            // Price
            Bone.text(
              words: 1,
              child: Text('\$${product.price}'),
            ),
          ],
        ),
      ),
    );
  }
}

// Usage with Riverpod
class ProductsGrid extends ConsumerWidget {
  const ProductsGrid({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final productsAsync = ref.watch(productsProvider);

    return switch (productsAsync) {
      AsyncLoading() => GridView.builder(
          gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
            crossAxisCount: 2,
          ),
          itemCount: 6,
          itemBuilder: (context, index) => ProductCard(
            product: Product.mock(),
            isLoading: true, // Show skeleton
          ),
        ),

      AsyncData(:final value) => GridView.builder(
          gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
            crossAxisCount: 2,
          ),
          itemCount: value.length,
          itemBuilder: (context, index) => ProductCard(
            product: value[index],
            isLoading: false, // Show real data
          ),
        ),

      AsyncError(:final error) => ErrorMessage(error),
    };
  }
}
```

<div dir="rtl">

**مميزات skeletonizer:**
- ✅ Automatic - مش محتاج تكتب skeleton UI يدوياً
- ✅ Customizable - ممكن تستخدم Bone widgets للـ custom skeletons
- ✅ Performance - optimized بشكل جيد
- ✅ Dark mode support

---

### 5. Pull-to-Refresh

من [Riverpod Official Docs](https://riverpod.dev/docs/how_to/pull_to_refresh).

</div>

```dart
// Provider
@riverpod
Future<List<Product>> products(ProductsRef ref) async {
  return await api.getProducts();
}

// UI - Basic pull-to-refresh
class ProductsScreen extends ConsumerWidget {
  const ProductsScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final productsAsync = ref.watch(productsProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('Products')),
      body: RefreshIndicator(
        // Use ref.refresh with .future
        onRefresh: () => ref.refresh(productsProvider.future),

        child: switch (productsAsync) {
          AsyncData(:final value) => ListView.builder(
              itemCount: value.length,
              itemBuilder: (context, index) => ProductTile(value[index]),
            ),

          AsyncError(:final error) => ListView(
              children: [ErrorMessage(error)],
            ),

          _ => const Center(child: CircularProgressIndicator()),
        },
      ),
    );
  }
}

// Advanced - with refresh indicator at top
class ProductsScreenAdvanced extends ConsumerWidget {
  const ProductsScreenAdvanced({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final productsAsync = ref.watch(productsProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('Products')),
      body: Column(
        children: [
          // Show linear indicator when refreshing
          if (productsAsync.isRefreshing)
            const LinearProgressIndicator(),

          Expanded(
            child: RefreshIndicator(
              onRefresh: () => ref.refresh(productsProvider.future),
              child: switch (productsAsync) {
                AsyncData(:final value) => ListView.builder(
                    itemCount: value.length,
                    itemBuilder: (context, index) => ProductTile(value[index]),
                  ),

                AsyncError(:final error) => ListView(
                    children: [
                      ErrorMessage(error),
                      ElevatedButton(
                        onPressed: () => ref.invalidate(productsProvider),
                        child: const Text('Retry'),
                      ),
                    ],
                  ),

                _ => const Center(child: CircularProgressIndicator()),
              },
            ),
          ),
        ],
      ),
    );
  }
}
```

<div dir="rtl">

**ملاحظات مهمة:**
- استخدم `ref.refresh(provider.future)` مع FutureProvider
- RefreshIndicator محتاج scrollable widget (ListView, GridView, etc.)
- لو محتاج empty state، استخدم `ListView(children: [...])`

---

### 6. Infinite Scroll

</div>

```dart
// Paginated data model
class PaginatedData<T> {
  final List<T> items;
  final int page;
  final bool hasMore;

  PaginatedData({
    required this.items,
    required this.page,
    required this.hasMore,
  });

  PaginatedData<T> copyWith({
    List<T>? items,
    int? page,
    bool? hasMore,
  }) {
    return PaginatedData(
      items: items ?? this.items,
      page: page ?? this.page,
      hasMore: hasMore ?? this.hasMore,
    );
  }
}

// Provider
@riverpod
class PaginatedProducts extends _$PaginatedProducts {
  @override
  Future<PaginatedData<Product>> build() async {
    final products = await api.getProducts(page: 1);
    return PaginatedData(
      items: products,
      page: 1,
      hasMore: products.length >= 20, // Assuming 20 items per page
    );
  }

  Future<void> loadMore() async {
    if (!state.hasValue) return;

    final currentData = state.value!;
    if (!currentData.hasMore) return;

    // Set loading (keeps current data)
    state = AsyncValue.data(currentData);

    // Load next page
    final result = await AsyncValue.guard(() async {
      final newProducts = await api.getProducts(page: currentData.page + 1);
      return currentData.copyWith(
        items: [...currentData.items, ...newProducts],
        page: currentData.page + 1,
        hasMore: newProducts.length >= 20,
      );
    });

    state = result;
  }
}

// UI with scroll detection
class ProductsScreen extends ConsumerStatefulWidget {
  const ProductsScreen({super.key});

  @override
  ConsumerState<ProductsScreen> createState() => _ProductsScreenState();
}

class _ProductsScreenState extends ConsumerState<ProductsScreen> {
  final _scrollController = ScrollController();

  @override
  void initState() {
    super.initState();

    // Listen to scroll
    _scrollController.addListener(() {
      if (_scrollController.position.pixels >=
          _scrollController.position.maxScrollExtent * 0.9) {
        // Reached 90% of scroll - load more
        ref.read(paginatedProductsProvider.notifier).loadMore();
      }
    });
  }

  @override
  void dispose() {
    _scrollController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final productsAsync = ref.watch(paginatedProductsProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('Products')),
      body: switch (productsAsync) {
        AsyncData(:final value) => ListView.builder(
            controller: _scrollController,
            itemCount: value.items.length + (value.hasMore ? 1 : 0),
            itemBuilder: (context, index) {
              // Show product
              if (index < value.items.length) {
                return ProductTile(value.items[index]);
              }

              // Show loading indicator at bottom
              if (productsAsync.isReloading) {
                return const Center(
                  child: Padding(
                    padding: EdgeInsets.all(16),
                    child: CircularProgressIndicator(),
                  ),
                );
              }

              // "Load more" button
              return Center(
                child: Padding(
                  padding: const EdgeInsets.all(16),
                  child: ElevatedButton(
                    onPressed: () {
                      ref.read(paginatedProductsProvider.notifier).loadMore();
                    },
                    child: const Text('Load More'),
                  ),
                ),
              );
            },
          ),

        AsyncError(:final error) => ErrorMessage(error),
        _ => const Center(child: CircularProgressIndicator()),
      },
    );
  }
}

// Alternative - using NotificationListener
class ProductsScreenAlt extends ConsumerWidget {
  const ProductsScreenAlt({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final productsAsync = ref.watch(paginatedProductsProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('Products')),
      body: NotificationListener<ScrollNotification>(
        onNotification: (notification) {
          // Check if reached near bottom
          if (notification.metrics.pixels >=
              notification.metrics.maxScrollExtent * 0.9) {
            // Load more
            ref.read(paginatedProductsProvider.notifier).loadMore();
          }
          return false;
        },
        child: switch (productsAsync) {
          AsyncData(:final value) => CustomScrollView(
              slivers: [
                SliverList(
                  delegate: SliverChildBuilderDelegate(
                    (context, index) => ProductTile(value.items[index]),
                    childCount: value.items.length,
                  ),
                ),

                // Loading indicator
                if (value.hasMore && productsAsync.isReloading)
                  const SliverToBoxAdapter(
                    child: Center(
                      child: Padding(
                        padding: EdgeInsets.all(16),
                        child: CircularProgressIndicator(),
                      ),
                    ),
                  ),
              ],
            ),

          AsyncError(:final error) => ErrorMessage(error),
          _ => const Center(child: CircularProgressIndicator()),
        },
      ),
    );
  }
}
```

<div dir="rtl">

---

## 🎭 isRefreshing vs isReloading vs isLoading

### الفرق بينهم

| Property | Meaning | UI Pattern |
|----------|---------|------------|
| **isLoading** | أي loading state | Show main loading indicator |
| **isRefreshing** | بنحدث data موجودة | Show small indicator (linear bar) |
| **isReloading** | Parameter/dependency تغير | Show small indicator |

### أمثلة عملية

</div>

```dart
// Example 1: Show different UI for each state
class DataScreen extends ConsumerWidget {
  const DataScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final dataAsync = ref.watch(dataProvider);

    return Scaffold(
      appBar: AppBar(
        title: const Text('Data'),
        actions: [
          // Show icon when refreshing
          if (dataAsync.isRefreshing)
            const Padding(
              padding: EdgeInsets.all(16),
              child: SizedBox(
                width: 20,
                height: 20,
                child: CircularProgressIndicator(strokeWidth: 2),
              ),
            ),
        ],
      ),
      body: Column(
        children: [
          // Linear indicator for reloading
          if (dataAsync.isReloading)
            const LinearProgressIndicator(),

          Expanded(
            child: switch (dataAsync) {
              // Initial loading - big spinner
              AsyncLoading() => const Center(
                  child: CircularProgressIndicator(),
                ),

              // Data with refresh/reload - show data with indicator
              AsyncData(:final value) => DataView(value),

              AsyncError(:final error) => ErrorMessage(error),
            },
          ),
        ],
      ),
    );
  }
}

// Example 2: Different messages
class StatusScreen extends ConsumerWidget {
  const StatusScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final dataAsync = ref.watch(dataProvider);

    String loadingMessage = '';
    if (dataAsync.isLoading && !dataAsync.hasValue) {
      loadingMessage = 'Loading data...';
    } else if (dataAsync.isRefreshing) {
      loadingMessage = 'Refreshing...';
    } else if (dataAsync.isReloading) {
      loadingMessage = 'Updating...';
    }

    return Scaffold(
      body: Column(
        children: [
          if (loadingMessage.isNotEmpty)
            Container(
              padding: const EdgeInsets.all(8),
              color: Colors.blue.shade100,
              child: Text(loadingMessage),
            ),

          Expanded(
            child: switch (dataAsync) {
              AsyncData(:final value) => DataView(value),
              AsyncError(:final error) => ErrorMessage(error),
              _ => const Center(child: CircularProgressIndicator()),
            },
          ),
        ],
      ),
    );
  }
}

// Example 3: Disable interactions during loading
class InteractiveScreen extends ConsumerWidget {
  const InteractiveScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final dataAsync = ref.watch(dataProvider);

    // Disable buttons when loading/reloading
    final isUpdating = dataAsync.isLoading || dataAsync.isReloading;

    return Scaffold(
      body: switch (dataAsync) {
        AsyncData(:final value) => Column(
            children: [
              DataView(value),
              ElevatedButton(
                // Disable button during update
                onPressed: isUpdating
                    ? null
                    : () {
                        // Update action
                      },
                child: isUpdating
                    ? const SizedBox(
                        width: 16,
                        height: 16,
                        child: CircularProgressIndicator(strokeWidth: 2),
                      )
                    : const Text('Update'),
              ),
            ],
          ),

        AsyncError(:final error) => ErrorMessage(error),
        _ => const Center(child: CircularProgressIndicator()),
      },
    );
  }
}
```

<div dir="rtl">

---

## 📋 Best Practices

### 1. اختر Loading Pattern المناسب

</div>

```dart
// ✅ GOOD - Different patterns for different scenarios

// Initial load: Full screen spinner or skeleton
AsyncLoading() => const Center(child: CircularProgressIndicator()),
// OR
AsyncLoading() => const ProductsSkeleton(),

// Refresh: Small indicator at top
if (asyncValue.isRefreshing)
  const LinearProgressIndicator(),

// Reload: Keep showing data with small indicator
if (asyncValue.isReloading)
  const LinearProgressIndicator(),

// Pagination: Spinner at bottom
if (hasMore && asyncValue.isReloading)
  const Center(child: CircularProgressIndicator()),
```

<div dir="rtl">

### 2. استخدم Skeleton Loaders للـ Content-Heavy UIs

</div>

```dart
// ✅ GOOD - Skeleton for complex content
AsyncLoading() => Skeletonizer(
  enabled: true,
  child: ProductGrid(products: Product.mockList()),
)

// ❌ BAD - Generic spinner for complex content
AsyncLoading() => const Center(child: CircularProgressIndicator())
```

<div dir="rtl">

### 3. Always Show Previous Data During Refresh

</div>

```dart
// ✅ GOOD - User sees old data while refreshing
return switch (asyncValue) {
  AsyncValue(:final value?) => Column(
      children: [
        if (asyncValue.isRefreshing)
          const LinearProgressIndicator(),
        DataWidget(value),
      ],
    ),
  _ => const LoadingScreen(),
};

// ❌ BAD - Screen goes blank during refresh
return switch (asyncValue) {
  AsyncLoading() => const LoadingScreen(),
  AsyncData(:final value) => DataWidget(value),
  _ => const ErrorScreen(),
};
```

<div dir="rtl">

### 4. Use Pull-to-Refresh للـ Lists

</div>

```dart
// ✅ GOOD - Easy refresh
RefreshIndicator(
  onRefresh: () => ref.refresh(dataProvider.future),
  child: ListView(...),
)
```

<div dir="rtl">

### 5. Show Loading Progress عند الإمكان

</div>

```dart
// ✅ GOOD - Show progress when available
if (downloadAsync case AsyncLoading(:final progress?)) {
  return LinearProgressIndicator(value: progress);
}

// Or with percentage
if (downloadAsync case AsyncLoading(:final progress?)) {
  return Column(
    children: [
      LinearProgressIndicator(value: progress),
      Text('${(progress * 100).toInt()}%'),
    ],
  );
}
```

<div dir="rtl">

### 6. Optimize Infinite Scroll

</div>

```dart
// ✅ GOOD - Load when 90% scrolled
if (_scrollController.position.pixels >=
    _scrollController.position.maxScrollExtent * 0.9) {
  loadMore();
}

// ❌ BAD - Load at exact bottom (might not trigger)
if (_scrollController.position.pixels ==
    _scrollController.position.maxScrollExtent) {
  loadMore();
}
```

<div dir="rtl">

### 7. Prevent Multiple Simultaneous Loads

</div>

```dart
// ✅ GOOD - Check before loading more
Future<void> loadMore() async {
  // Don't load if already loading or no value or no more data
  if (state.isLoading || !state.hasValue) {
    return;
  }

  final currentData = state.value!;
  if (!currentData.hasMore) {
    return;
  }

  // Load next page
  // ...
}
```

<div dir="rtl">

---

## 🎓 مثال عملي كامل: E-Commerce Products Screen

</div>

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';
import 'package:skeletonizer/skeletonizer.dart';

part 'products.g.dart';

// Models
class Product {
  final String id;
  final String name;
  final double price;
  final String imageUrl;

  Product({
    required this.id,
    required this.name,
    required this.price,
    required this.imageUrl,
  });

  static Product mock() => Product(
        id: 'mock',
        name: 'Product Name',
        price: 99.99,
        imageUrl: 'https://via.placeholder.com/150',
      );

  static List<Product> mockList() => List.generate(6, (i) => mock());
}

// API
class ProductsApi {
  Future<List<Product>> getProducts({int page = 1}) async {
    await Future.delayed(const Duration(seconds: 2));
    return List.generate(
      20,
      (i) => Product(
        id: 'product_${page}_$i',
        name: 'Product ${page * 20 + i}',
        price: 10.0 + i * 5.0,
        imageUrl: 'https://via.placeholder.com/150',
      ),
    );
  }
}

final productsApiProvider = Provider((ref) => ProductsApi());

// Paginated products provider
class PaginatedData<T> {
  final List<T> items;
  final int page;
  final bool hasMore;

  PaginatedData({
    required this.items,
    required this.page,
    required this.hasMore,
  });

  PaginatedData<T> copyWith({
    List<T>? items,
    int? page,
    bool? hasMore,
  }) {
    return PaginatedData(
      items: items ?? this.items,
      page: page ?? this.page,
      hasMore: hasMore ?? this.hasMore,
    );
  }
}

@riverpod
class Products extends _$Products {
  @override
  Future<PaginatedData<Product>> build() async {
    final api = ref.watch(productsApiProvider);
    final products = await api.getProducts(page: 1);
    return PaginatedData(
      items: products,
      page: 1,
      hasMore: products.length >= 20,
    );
  }

  Future<void> loadMore() async {
    if (state.isLoading || !state.hasValue) {
      return;
    }

    final currentData = state.value!;
    if (!currentData.hasMore) {
      return;
    }

    final result = await AsyncValue.guard(() async {
      final api = ref.read(productsApiProvider);
      final newProducts = await api.getProducts(page: currentData.page + 1);
      return currentData.copyWith(
        items: [...currentData.items, ...newProducts],
        page: currentData.page + 1,
        hasMore: newProducts.length >= 20,
      );
    });

    state = result;
  }

  Future<void> refresh() async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      final api = ref.read(productsApiProvider);
      final products = await api.getProducts(page: 1);
      return PaginatedData(
        items: products,
        page: 1,
        hasMore: products.length >= 20,
      );
    });
  }
}

// UI Components
class ProductCard extends StatelessWidget {
  final Product product;
  final bool isLoading;

  const ProductCard({
    super.key,
    required this.product,
    this.isLoading = false,
  });

  @override
  Widget build(BuildContext context) {
    return Skeletonizer(
      enabled: isLoading,
      child: Card(
        clipBehavior: Clip.antiAlias,
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            AspectRatio(
              aspectRatio: 1,
              child: Image.network(
                product.imageUrl,
                fit: BoxFit.cover,
                errorBuilder: (_, __, ___) => const Icon(Icons.error),
              ),
            ),
            Padding(
              padding: const EdgeInsets.all(8),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    product.name,
                    style: const TextStyle(
                      fontWeight: FontWeight.bold,
                      fontSize: 16,
                    ),
                    maxLines: 2,
                    overflow: TextOverflow.ellipsis,
                  ),
                  const SizedBox(height: 4),
                  Text(
                    '\$${product.price.toStringAsFixed(2)}',
                    style: TextStyle(
                      color: Colors.green.shade700,
                      fontSize: 14,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}

// Main screen
class ProductsScreen extends ConsumerStatefulWidget {
  const ProductsScreen({super.key});

  @override
  ConsumerState<ProductsScreen> createState() => _ProductsScreenState();
}

class _ProductsScreenState extends ConsumerState<ProductsScreen> {
  final _scrollController = ScrollController();

  @override
  void initState() {
    super.initState();
    _scrollController.addListener(_onScroll);
  }

  @override
  void dispose() {
    _scrollController.removeListener(_onScroll);
    _scrollController.dispose();
    super.dispose();
  }

  void _onScroll() {
    if (_scrollController.position.pixels >=
        _scrollController.position.maxScrollExtent * 0.9) {
      ref.read(productsProvider.notifier).loadMore();
    }
  }

  @override
  Widget build(BuildContext context) {
    final productsAsync = ref.watch(productsProvider);

    return Scaffold(
      appBar: AppBar(
        title: const Text('Products'),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: () => ref.read(productsProvider.notifier).refresh(),
          ),
        ],
      ),
      body: Column(
        children: [
          // Linear indicator when refreshing
          if (productsAsync.isRefreshing && productsAsync.hasValue)
            const LinearProgressIndicator(),

          Expanded(
            child: RefreshIndicator(
              onRefresh: () => ref.read(productsProvider.notifier).refresh(),
              child: switch (productsAsync) {
                // Loading - show skeleton
                AsyncLoading() => GridView.builder(
                    gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
                      crossAxisCount: 2,
                      childAspectRatio: 0.75,
                      crossAxisSpacing: 8,
                      mainAxisSpacing: 8,
                    ),
                    padding: const EdgeInsets.all(8),
                    itemCount: 6,
                    itemBuilder: (context, index) => ProductCard(
                      product: Product.mock(),
                      isLoading: true,
                    ),
                  ),

                // Data - show products
                AsyncData(:final value) => CustomScrollView(
                    controller: _scrollController,
                    slivers: [
                      SliverPadding(
                        padding: const EdgeInsets.all(8),
                        sliver: SliverGrid(
                          gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
                            crossAxisCount: 2,
                            childAspectRatio: 0.75,
                            crossAxisSpacing: 8,
                            mainAxisSpacing: 8,
                          ),
                          delegate: SliverChildBuilderDelegate(
                            (context, index) => ProductCard(
                              product: value.items[index],
                            ),
                            childCount: value.items.length,
                          ),
                        ),
                      ),

                      // Loading more indicator
                      if (value.hasMore && productsAsync.isReloading)
                        const SliverToBoxAdapter(
                          child: Center(
                            child: Padding(
                              padding: EdgeInsets.all(16),
                              child: CircularProgressIndicator(),
                            ),
                          ),
                        ),

                      // End message
                      if (!value.hasMore)
                        SliverToBoxAdapter(
                          child: Center(
                            child: Padding(
                              padding: const EdgeInsets.all(16),
                              child: Text(
                                'You\'ve seen all ${value.items.length} products!',
                                style: TextStyle(color: Colors.grey.shade600),
                              ),
                            ),
                          ),
                        ),
                    ],
                  ),

                // Error
                AsyncError(:final error) => ListView(
                    children: [
                      const SizedBox(height: 100),
                      const Icon(Icons.error, size: 64, color: Colors.red),
                      const SizedBox(height: 16),
                      Center(child: Text('Error: $error')),
                      const SizedBox(height: 16),
                      Center(
                        child: ElevatedButton(
                          onPressed: () => ref.invalidate(productsProvider),
                          child: const Text('Retry'),
                        ),
                      ),
                    ],
                  ),
              },
            ),
          ),
        ],
      ),
    );
  }
}
```

<div dir="rtl">

---

## 🔗 الخطوة الجاية

في الملف الجاي هنتعلم **Refresh Strategies**:
- ref.refresh vs ref.invalidate
- Auto-refresh strategies
- Time-based refresh
- Focus-based refresh
- Caching strategies

جاهز؟ يلا نكمل! 🚀

---

## 📚 المصادر

- [Create a shimmer loading effect | Flutter](https://docs.flutter.dev/cookbook/effects/shimmer-loading)
- [skeletonizer package | pub.dev](https://pub.dev/packages/skeletonizer)
- [Implementing pull-to-refresh | Riverpod](https://riverpod.dev/docs/how_to/pull_to_refresh)
- [Flutter Placeholder Packages | Flutter Gems](https://fluttergems.dev/placeholder/)

</div>
