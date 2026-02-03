<div dir="rtl">

# Dependency Injection with Riverpod

**المستوى**: 🔴 متقدم

## 📌 الخلاصة

**Riverpod = State Management + Dependency Injection**

```dart
// Step 1: Define providers for dependencies
@riverpod
http.Client httpClient(HttpClientRef ref) => http.Client();

@riverpod
ProductsRemoteDataSource productsRemoteDataSource(
  ProductsRemoteDataSourceRef ref,
) {
  final client = ref.watch(httpClientProvider);
  return ProductsRemoteDataSourceImpl(client);
}

@riverpod
ProductsRepository productsRepository(ProductsRepositoryRef ref) {
  final dataSource = ref.watch(productsRemoteDataSourceProvider);
  return ProductsRepositoryImpl(dataSource);
}

@riverpod
GetProductsUseCase getProductsUseCase(GetProductsUseCaseRef ref) {
  final repository = ref.watch(productsRepositoryProvider);
  return GetProductsUseCase(repository);
}

// Step 2: Use in UI
@riverpod
class Products extends _$Products {
  @override
  Future<List<Product>> build() async {
    final useCase = ref.watch(getProductsUseCaseProvider);
    return await useCase();
  }
}
```

**المميزات:**
- ✅ Compile-time safety
- ✅ Easy testing (override providers)
- ✅ No service locator
- ✅ Automatic dependency disposal

---

## 🧪 Testing Benefits

```dart
test('loads products', () async {
  final container = ProviderContainer.test(
    overrides: [
      // Mock the repository
      productsRepositoryProvider.overrideWithValue(MockProductsRepository()),
    ],
  );
  
  final products = await container.read(productsProvider.future);
  
  expect(products.length, 2);
});
```

---

## 📚 المصادر

- [Riverpod DI Guide](https://riverpod.dev/docs/concepts2/providers)

</div>
