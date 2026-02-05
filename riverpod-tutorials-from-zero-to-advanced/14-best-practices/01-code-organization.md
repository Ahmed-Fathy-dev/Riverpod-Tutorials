<div dir="rtl">

# Code Organization

**المستوى**: 🟡 متوسط

## 📁 Feature-based Structure

```
lib/
├── features/
│   ├── auth/
│   │   ├── data/
│   │   ├── domain/
│   │   ├── presentation/
│   │   └── providers/
│   └── products/
│       ├── data/
│       ├── domain/
│       ├── presentation/
│       └── providers/
├── core/
│   ├── network/
│   ├── errors/
│   └── constants/
└── shared/
    ├── widgets/
    ├── utils/
    └── models/
```

---

## 📋 Best Practices

### 1. Group Related Providers

```dart
// ✅ GOOD - All auth providers together
// features/auth/providers/auth_providers.dart
@riverpod
class Auth extends _$Auth { ... }

@riverpod
AuthRepository authRepository(AuthRepositoryRef ref) { ... }

@riverpod
LoginUseCase loginUseCase(LoginUseCaseRef ref) { ... }
```

### 2. Separate UI from Logic

```dart
// ✅ GOOD
features/products/
├── presentation/
│   ├── screens/
│   │   └── products_screen.dart
│   └── widgets/
│       └── product_card.dart
└── providers/
    └── products_provider.dart
```

### 3. Use Part Files for Generated Code

```dart
// products_provider.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'products_provider.g.dart';

@riverpod
class Products extends _$Products { ... }
```

</div>
