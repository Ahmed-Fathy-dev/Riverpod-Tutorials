<div dir="rtl">

# Testing Best Practices

**المستوى**: 🔴 متقدم

## 📌 الخلاصة

Best practices لكتابة tests احترافية مع Riverpod.

---

## 🎯 Test Structure: AAA Pattern

**Arrange, Act, Assert**

```dart
test('adds todo', () async {
  // Arrange - Setup
  final mockApi = MockTodosApi();
  when(() => mockApi.getTodos()).thenAnswer((_) async => []);
  when(() => mockApi.addTodo(any())).thenAnswer((_) async {});

  final container = ProviderContainer.test(
    overrides: [todosApiProvider.overrideWithValue(mockApi)],
  );

  await container.read(todosProvider.future);

  // Act - Perform action
  await container.read(todosProvider.notifier).addTodo('New Todo');

  // Assert - Verify result
  verify(() => mockApi.addTodo('New Todo')).called(1);
});
```

---

## 📊 When to Unit Test vs Widget Test

| Test Type | When | Example |
|-----------|------|---------|
| **Unit Test** | Pure logic, no UI | Provider calculations, API calls |
| **Widget Test** | UI + interactions | Button clicks, form validation |
| **Integration Test** | Full user flows | Login → Dashboard → Logout |

---

## 📋 Coverage Strategy

### What to Test

```dart
// ✅ Test business logic
test('calculates total correctly', () { ... });

// ✅ Test error handling
test('handles API error', () { ... });

// ✅ Test edge cases
test('handles empty list', () { ... });
test('handles null values', () { ... });

// ❌ Don't test framework code
test('provider exists', () { ... }); // Useless!

// ❌ Don't test trivial getters
test('user name getter returns name', () { ... }); // Useless!
```

---

## 🎭 Mocking Guidelines

### 1. Mock External Dependencies

```dart
// ✅ GOOD - Mock things you don't control
- HTTP clients
- Databases
- Third-party SDKs
- Platform channels

// ❌ DON'T mock your own code
- Your providers
- Your models
- Your business logic
```

### 2. Use Test Doubles

```dart
// Dummy - Passed but never used
class DummyLogger implements Logger {
  @override
  void log(String message) {}
}

// Stub - Returns canned responses
class StubTodosApi implements TodosApi {
  @override
  Future<List<Todo>> getTodos() async => [];
}

// Mock - Verifies interactions
class MockTodosApi extends Mock implements TodosApi {}

// Fake - Working implementation
class FakeTodosApi implements TodosApi {
  final _todos = <Todo>[];

  @override
  Future<List<Todo>> getTodos() async => _todos;

  @override
  Future<void> addTodo(String title) async {
    _todos.add(Todo(id: '${_todos.length}', title: title));
  }
}
```

---

## 🔧 Setup & Teardown

```dart
void main() {
  late MockTodosApi mockApi;
  late ProviderContainer container;

  setUp(() {
    mockApi = MockTodosApi();
    container = ProviderContainer.test(
      overrides: [
        todosApiProvider.overrideWithValue(mockApi),
      ],
    );
  });

  test('test 1', () async {
    // mockApi and container available
  });

  test('test 2', () async {
    // Fresh mockApi and container for each test
  });
}
```

---

## 📝 Test Naming

```dart
// ✅ GOOD - Descriptive names
test('increments counter when button is tapped', () { ... });
test('displays error message when API fails', () { ... });
test('filters todos by completion status', () { ... });

// ❌ BAD - Vague names
test('test1', () { ... });
test('it works', () { ... });
test('counter', () { ... });
```

---

## 🎓 Complete Example

```dart
// counter_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:riverpod/riverpod.dart';

void main() {
  group('Counter', () {
    late ProviderContainer container;

    setUp(() {
      container = ProviderContainer.test();
    });

    group('initialization', () {
      test('starts at zero', () {
        expect(container.read(counterProvider), 0);
      });
    });

    group('increment', () {
      test('increases count by one', () {
        container.read(counterProvider.notifier).increment();

        expect(container.read(counterProvider), 1);
      });

      test('can be called multiple times', () {
        container.read(counterProvider.notifier).increment();
        container.read(counterProvider.notifier).increment();
        container.read(counterProvider.notifier).increment();

        expect(container.read(counterProvider), 3);
      });
    });

    group('decrement', () {
      test('decreases count by one', () {
        container.read(counterProvider.notifier).decrement();

        expect(container.read(counterProvider), -1);
      });

      test('can go negative', () {
        container.read(counterProvider.notifier).decrement();
        container.read(counterProvider.notifier).decrement();

        expect(container.read(counterProvider), -2);
      });
    });

    group('reset', () {
      test('sets count back to zero', () {
        container.read(counterProvider.notifier).increment();
        container.read(counterProvider.notifier).increment();
        container.read(counterProvider.notifier).reset();

        expect(container.read(counterProvider), 0);
      });
    });
  });
}
```

---

## 📚 المصادر

- [Testing your providers | Riverpod](https://riverpod.dev/docs/how_to/testing)
- [Effective Dart: Testing](https://dart.dev/guides/language/effective-dart/testing)

</div>
