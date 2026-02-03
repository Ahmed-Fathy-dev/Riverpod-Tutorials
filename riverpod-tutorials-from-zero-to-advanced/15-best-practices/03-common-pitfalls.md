<div dir="rtl">

# Common Pitfalls

**المستوى**: 🟡 متوسط

## ⚠️ الأخطاء الشائعة

### 1. Using ref.watch in Methods

```dart
// ❌ BAD - ref.watch in method
class TodosController extends _$TodosController {
  void addTodo() {
    final userId = ref.watch(userIdProvider);  // ❌ Wrong!
  }
}

// ✅ GOOD - ref.watch in build, ref.read in method
class TodosController extends _$TodosController {
  @override
  Future<List<Todo>> build() async {
    final userId = ref.watch(userIdProvider);  // ✅ Correct
    return await api.getTodos(userId);
  }
  
  void addTodo() {
    final userId = ref.read(userIdProvider);  // ✅ Correct
  }
}
```

### 2. Forgetting to Use .notifier

```dart
// ❌ BAD - Missing .notifier
ref.read(counterProvider).increment();  // ❌ Won't work!

// ✅ GOOD
ref.read(counterProvider.notifier).increment();
```

### 3. Not Handling Async States

```dart
// ❌ BAD - No loading/error handling
final user = ref.watch(userProvider).value!;  // Can crash!

// ✅ GOOD
final userAsync = ref.watch(userProvider);
return userAsync.when(
  data: (user) => UserCard(user),
  loading: () => const LoadingScreen(),
  error: (error, _) => ErrorScreen(error),
);
```

### 4. Circular Dependencies

```dart
// ❌ BAD
@riverpod
int providerA(ProviderARef ref) {
  return ref.watch(providerBProvider) + 1;
}

@riverpod
int providerB(ProviderBRef ref) {
  return ref.watch(providerAProvider) + 1;  // ❌ Circular!
}

// ✅ GOOD - Extract shared state
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;
}

@riverpod
int doubleCounter(DoubleCounterRef ref) {
  return ref.watch(counterProvider) * 2;
}

@riverpod
int tripleCounter(TripleCounterRef ref) {
  return ref.watch(counterProvider) * 3;
}
```

### 5. Memory Leaks with Families

```dart
// ⚠️ CAREFUL - Can cause memory leak
@Riverpod(keepAlive: true)  // ❌ With family = memory leak!
Future<Product> product(ProductRef ref, String id) async { ... }

// ✅ GOOD - AutoDispose with families
@riverpod  // Default: AutoDispose
Future<Product> product(ProductRef ref, String id) async { ... }
```

</div>
