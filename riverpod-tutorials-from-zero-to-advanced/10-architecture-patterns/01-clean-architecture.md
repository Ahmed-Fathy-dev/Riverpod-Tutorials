<div dir="rtl">

# Clean Architecture

**المستوى**: 🔴 متقدم

## 📌 الخلاصة

**Clean Architecture** = فصل الكود لطبقات مستقلة عن بعضها.

من [Clean Architecture Guide](https://dev.to/ssoad/flutter-riverpod-clean-architecture-the-ultimate-production-ready-template-for-scalable-apps-gdh):

> Clean Architecture splits code into three primary layers: Presentation, Domain, and Data

---

## 🏗️ الطبقات الثلاثة

### 1. Presentation Layer (UI)

```dart
// presentation/providers/products_provider.dart
@riverpod
class Products extends _$Products {
  @override
  Future<List<Product>> build() async {
    final useCase = ref.watch(getProductsUseCaseProvider);
    return await useCase();
  }
}

// presentation/screens/products_screen.dart
class ProductsScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final productsAsync = ref.watch(productsProvider);
    
    return productsAsync.when(
      data: (products) => ProductsList(products),
      loading: () => const LoadingScreen(),
      error: (error, _) => ErrorScreen(error),
    );
  }
}
```

**المسؤولية:**
- عرض البيانات
- التفاعل مع المستخدم
- State management

---

### 2. Domain Layer (Business Logic)

```dart
// domain/entities/product.dart
class Product {
  final String id;
  final String name;
  final double price;
  
  Product({required this.id, required this.name, required this.price});
}

// domain/repositories/products_repository.dart
abstract class ProductsRepository {
  Future<List<Product>> getProducts();
  Future<Product> getProduct(String id);
}

// domain/use_cases/get_products_use_case.dart
@riverpod
GetProductsUseCase getProductsUseCase(GetProductsUseCaseRef ref) {
  return GetProductsUseCase(ref.watch(productsRepositoryProvider));
}

class GetProductsUseCase {
  final ProductsRepository repository;
  
  GetProductsUseCase(this.repository);
  
  Future<List<Product>> call() async {
    return await repository.getProducts();
  }
}
```

**المسؤولية:**
- Business rules
- Entities (models)
- Interfaces (abstractions)

---

### 3. Data Layer (Data Access)

```dart
// data/models/product_model.dart
class ProductModel {
  final String id;
  final String name;
  final double price;
  
  ProductModel({required this.id, required this.name, required this.price});
  
  factory ProductModel.fromJson(Map<String, dynamic> json) {
    return ProductModel(
      id: json['id'],
      name: json['name'],
      price: json['price'].toDouble(),
    );
  }
  
  Product toEntity() {
    return Product(id: id, name: name, price: price);
  }
}

// data/data_sources/products_remote_data_source.dart
abstract class ProductsRemoteDataSource {
  Future<List<ProductModel>> getProducts();
}

class ProductsRemoteDataSourceImpl implements ProductsRemoteDataSource {
  final http.Client client;
  
  ProductsRemoteDataSourceImpl(this.client);
  
  @override
  Future<List<ProductModel>> getProducts() async {
    final response = await client.get(Uri.parse('api/products'));
    
    if (response.statusCode == 200) {
      final List json = jsonDecode(response.body);
      return json.map((item) => ProductModel.fromJson(item)).toList();
    } else {
      throw ServerException();
    }
  }
}

// data/repositories/products_repository_impl.dart
class ProductsRepositoryImpl implements ProductsRepository {
  final ProductsRemoteDataSource remoteDataSource;
  
  ProductsRepositoryImpl(this.remoteDataSource);
  
  @override
  Future<List<Product>> getProducts() async {
    try {
      final models = await remoteDataSource.getProducts();
      return models.map((model) => model.toEntity()).toList();
    } catch (e) {
      throw ServerFailure();
    }
  }
}

// data/repositories/products_repository_provider.dart
@riverpod
ProductsRepository productsRepository(ProductsRepositoryRef ref) {
  final remoteDataSource = ref.watch(productsRemoteDataSourceProvider);
  return ProductsRepositoryImpl(remoteDataSource);
}
```

**المسؤولية:**
- API calls
- Database operations
- Caching
- Data transformation

---

## 🎯 Dependency Rule

```
Presentation → Domain ← Data

✅ Presentation can import Domain
✅ Data can import Domain
❌ Domain cannot import Presentation or Data
```

**مثال:**

```dart
// ✅ GOOD - Presentation imports Domain
import 'package:app/features/products/domain/entities/product.dart';
import 'package:app/features/products/domain/use_cases/get_products.dart';

// ❌ BAD - Domain imports Data
// domain/use_cases/get_products.dart
import 'package:app/features/products/data/models/product_model.dart'; // ❌ Wrong!
```

---

## 📋 Best Practices

### 1. Keep Domain Pure

```dart
// ✅ GOOD - No framework dependencies
class Product {
  final String id;
  final String name;
  
  Product({required this.id, required this.name});
}

// ❌ BAD - Using Flutter in domain
import 'package:flutter/material.dart';

class Product {
  final Color color; // ❌ Flutter dependency!
}
```

### 2. Use Interfaces in Domain

```dart
// ✅ GOOD - Abstract interface
abstract class ProductsRepository {
  Future<List<Product>> getProducts();
}

// ❌ BAD - Concrete implementation in domain
class ProductsRepository {
  final http.Client client; // ❌ Implementation detail!
}
```

### 3. Inject Dependencies

```dart
// ✅ GOOD - Constructor injection
class GetProductsUseCase {
  final ProductsRepository repository;
  
  GetProductsUseCase(this.repository);
}

// ❌ BAD - Direct instantiation
class GetProductsUseCase {
  final repository = ProductsRepositoryImpl(); // ❌ Tight coupling!
}
```

---

## 📚 المصادر

- [Flutter Riverpod Clean Architecture](https://ssoad.github.io/flutter_riverpod_clean_architecture/)
- [Clean Architecture Template](https://github.com/ssoad/flutter_riverpod_clean_architecture)

</div>
