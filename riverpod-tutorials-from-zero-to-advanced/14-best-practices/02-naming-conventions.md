<div dir="rtl">

# Naming Conventions

**المستوى**: 🟡 متوسط

## 📝 Provider Naming

```dart
// ✅ GOOD - Clear, descriptive names
@riverpod
Future<User> currentUser(CurrentUserRef ref) async { ... }

@riverpod
List<Product> filteredProducts(FilteredProductsRef ref) { ... }

@riverpod
class Cart extends _$Cart { ... }

// ❌ BAD - Vague names
@riverpod
Future<User> user(UserRef ref) async { ... }  // Which user?

@riverpod
List<Product> products(ProductsRef ref) { ... }  // All or filtered?
```

---

## 🎯 Method Naming

```dart
class TodosController extends _$TodosController {
  // ✅ GOOD - Action verbs
  void addTodo(String title) { ... }
  void removeTodo(String id) { ... }
  void toggleTodo(String id) { ... }
  void clearCompleted() { ... }
  
  // ❌ BAD - Vague
  void update() { ... }
  void change() { ... }
}
```

---

## 📋 File Naming

```dart
// ✅ GOOD
products_provider.dart
auth_repository.dart
user_model.dart
login_screen.dart

// ❌ BAD
products.dart  // Too generic
authRepo.dart  // Use snake_case, not camelCase
UserModel.dart  // Use lowercase
```

</div>
