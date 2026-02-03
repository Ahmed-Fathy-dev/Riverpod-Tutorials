<div dir="rtl">

# Repository Pattern

**المستوى**: 🔴 متقدم

## 📌 الخلاصة

**Repository Pattern** = واجهة موحدة للوصول للبيانات من مصادر مختلفة.

```dart
// Domain layer - Interface
abstract class ProductsRepository {
  Future<List<Product>> getProducts();
}

// Data layer - Implementation  
class ProductsRepositoryImpl implements ProductsRepository {
  final ProductsRemoteDataSource remoteDataSource;
  final ProductsLocalDataSource localDataSource;
  
  @override
  Future<List<Product>> getProducts() async {
    try {
      // Try remote first
      final products = await remoteDataSource.getProducts();
      // Cache locally
      await localDataSource.cacheProducts(products);
      return products.map((m) => m.toEntity()).toList();
    } catch (e) {
      // Fallback to cache
      final cached = await localDataSource.getCachedProducts();
      return cached.map((m) => m.toEntity()).toList();
    }
  }
}
```

---

## 🎯 Multiple Data Sources

```dart
// Remote data source (API)
@riverpod
ProductsRemoteDataSource productsRemoteDataSource(
  ProductsRemoteDataSourceRef ref,
) {
  final client = ref.watch(httpClientProvider);
  return ProductsRemoteDataSourceImpl(client);
}

// Local data source (Database/Cache)
@riverpod
ProductsLocalDataSource productsLocalDataSource(
  ProductsLocalDataSourceRef ref,
) {
  final database = ref.watch(databaseProvider);
  return ProductsLocalDataSourceImpl(database);
}

// Repository combines both
@riverpod
ProductsRepository productsRepository(ProductsRepositoryRef ref) {
  return ProductsRepositoryImpl(
    remoteDataSource: ref.watch(productsRemoteDataSourceProvider),
    localDataSource: ref.watch(productsLocalDataSourceProvider),
  );
}
```

---

## 📚 المصادر

- [Repository Pattern Guide](https://codewithandrea.com/articles/flutter-repository-pattern/)

</div>
