<div dir="rtl">

# Complete Example - Auth Feature

**المستوى**: 🔴 متقدم

## 📌 الخلاصة

مثال كامل لـ **Auth feature** مع Clean Architecture.

```
features/auth/
├── data/
│   ├── data_sources/
│   │   └── auth_remote_data_source.dart
│   ├── models/
│   │   └── user_model.dart
│   └── repositories/
│       └── auth_repository_impl.dart
├── domain/
│   ├── entities/
│   │   └── user.dart
│   ├── repositories/
│   │   └── auth_repository.dart
│   └── use_cases/
│       ├── login_use_case.dart
│       └── logout_use_case.dart
└── presentation/
    ├── providers/
    │   └── auth_provider.dart
    └── screens/
        └── login_screen.dart
```

---

## 🏗️ Implementation

### Domain Layer

```dart
// domain/entities/user.dart
class User {
  final String id;
  final String email;
  final String name;
  
  User({required this.id, required this.email, required this.name});
}

// domain/repositories/auth_repository.dart
abstract class AuthRepository {
  Future<User> login(String email, String password);
  Future<void> logout();
  Future<User?> getCurrentUser();
}

// domain/use_cases/login_use_case.dart
class LoginUseCase {
  final AuthRepository repository;
  
  LoginUseCase(this.repository);
  
  Future<User> call(String email, String password) async {
    return await repository.login(email, password);
  }
}
```

### Data Layer

```dart
// data/models/user_model.dart
class UserModel {
  final String id;
  final String email;
  final String name;
  
  UserModel({required this.id, required this.email, required this.name});
  
  factory UserModel.fromJson(Map<String, dynamic> json) {
    return UserModel(
      id: json['id'],
      email: json['email'],
      name: json['name'],
    );
  }
  
  User toEntity() => User(id: id, email: email, name: name);
}

// data/repositories/auth_repository_impl.dart
class AuthRepositoryImpl implements AuthRepository {
  final AuthRemoteDataSource remoteDataSource;
  
  AuthRepositoryImpl(this.remoteDataSource);
  
  @override
  Future<User> login(String email, String password) async {
    final userModel = await remoteDataSource.login(email, password);
    return userModel.toEntity();
  }
  
  @override
  Future<void> logout() async {
    await remoteDataSource.logout();
  }
  
  @override
  Future<User?> getCurrentUser() async {
    final userModel = await remoteDataSource.getCurrentUser();
    return userModel?.toEntity();
  }
}
```

### Presentation Layer

```dart
// presentation/providers/auth_provider.dart
@riverpod
class Auth extends _$Auth {
  @override
  Future<User?> build() async {
    final repository = ref.watch(authRepositoryProvider);
    return await repository.getCurrentUser();
  }
  
  Future<void> login(String email, String password) async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      final useCase = ref.read(loginUseCaseProvider);
      return await useCase(email, password);
    });
  }
  
  Future<void> logout() async {
    state = const AsyncValue.loading();
    await ref.read(authRepositoryProvider).logout();
    state = const AsyncValue.data(null);
  }
}

// presentation/screens/login_screen.dart
class LoginScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final authAsync = ref.watch(authProvider);
    
    return authAsync.when(
      data: (user) => user != null 
          ? HomeScreen() 
          : LoginForm(),
      loading: () => const LoadingScreen(),
      error: (error, _) => ErrorScreen(error),
    );
  }
}
```

### Dependency Injection

```dart
// Providers for DI
@riverpod
AuthRemoteDataSource authRemoteDataSource(AuthRemoteDataSourceRef ref) {
  final client = ref.watch(httpClientProvider);
  return AuthRemoteDataSourceImpl(client);
}

@riverpod
AuthRepository authRepository(AuthRepositoryRef ref) {
  final dataSource = ref.watch(authRemoteDataSourceProvider);
  return AuthRepositoryImpl(dataSource);
}

@riverpod
LoginUseCase loginUseCase(LoginUseCaseRef ref) {
  final repository = ref.watch(authRepositoryProvider);
  return LoginUseCase(repository);
}
```

---

## 🧪 Testing

```dart
test('login succeeds', () async {
  final mockRepository = MockAuthRepository();
  when(() => mockRepository.login(any(), any()))
      .thenAnswer((_) async => User(id: '1', email: 'test@test.com', name: 'Test'));
  
  final container = ProviderContainer.test(
    overrides: [
      authRepositoryProvider.overrideWithValue(mockRepository),
    ],
  );
  
  await container.read(authProvider.notifier).login('test@test.com', 'password');
  
  final user = container.read(authProvider).requireValue;
  expect(user?.email, 'test@test.com');
});
```

---

## 📚 المصادر

- [Flutter Clean Architecture Examples](https://github.com/ssoad/flutter_riverpod_clean_architecture)

</div>
