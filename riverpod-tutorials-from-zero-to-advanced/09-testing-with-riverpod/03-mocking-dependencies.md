<div dir="rtl">

# Mocking Dependencies

**المستوى**: 🔴 متقدم

## 📌 الخلاصة

**Mocking** = استبدال الـ dependencies الحقيقية بـ fake implementations للاختبار.

```dart
class MockTodosApi extends Mock implements TodosApi {}

final container = ProviderContainer.test(
  overrides: [
    todosApiProvider.overrideWithValue(MockTodosApi()),
  ],
);
```

---

## 🎭 Using Mocktail

```yaml
# pubspec.yaml
dev_dependencies:
  mocktail: ^1.0.0
```

```dart
import 'package:mocktail/mocktail.dart';

// 1. Create mock class
class MockTodosApi extends Mock implements TodosApi {}

// 2. Setup mock behavior
test('test', () {
  final mockApi = MockTodosApi();

  when(() => mockApi.getTodos()).thenAnswer((_) async => []);

  // 3. Use mock
  final container = ProviderContainer.test(
    overrides: [
      todosApiProvider.overrideWithValue(mockApi),
    ],
  );
});
```

---

## 🧪 Complete Example

```dart
// API Interface
abstract class TodosApi {
  Future<List<Todo>> getTodos();
  Future<void> addTodo(String title);
  Future<void> deleteTodo(String id);
}

// Real Implementation
class RealTodosApi implements TodosApi {
  @override
  Future<List<Todo>> getTodos() async {
    // Real HTTP call
    final response = await http.get(Uri.parse('api/todos'));
    return (json.decode(response.body) as List)
        .map((json) => Todo.fromJson(json))
        .toList();
  }

  @override
  Future<void> addTodo(String title) async {
    await http.post(
      Uri.parse('api/todos'),
      body: json.encode({'title': title}),
    );
  }

  @override
  Future<void> deleteTodo(String id) async {
    await http.delete(Uri.parse('api/todos/$id'));
  }
}

// Provider
@riverpod
TodosApi todosApi(TodosApiRef ref) => RealTodosApi();

@riverpod
class Todos extends _$Todos {
  @override
  Future<List<Todo>> build() async {
    final api = ref.watch(todosApiProvider);
    return await api.getTodos();
  }

  Future<void> addTodo(String title) async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      final api = ref.read(todosApiProvider);
      await api.addTodo(title);
      return await api.getTodos();
    });
  }
}

// Mock
class MockTodosApi extends Mock implements TodosApi {}

// Tests
void main() {
  late MockTodosApi mockApi;

  setUp(() {
    mockApi = MockTodosApi();
  });

  group('Todos', () {
    test('loads empty list', () async {
      when(() => mockApi.getTodos()).thenAnswer((_) async => []);

      final container = ProviderContainer.test(
        overrides: [
          todosApiProvider.overrideWithValue(mockApi),
        ],
      );

      final todos = await container.read(todosProvider.future);

      expect(todos, isEmpty);
      verify(() => mockApi.getTodos()).called(1);
    });

    test('loads todos', () async {
      when(() => mockApi.getTodos()).thenAnswer((_) async => [
            Todo(id: '1', title: 'Todo 1'),
            Todo(id: '2', title: 'Todo 2'),
          ]);

      final container = ProviderContainer.test(
        overrides: [
          todosApiProvider.overrideWithValue(mockApi),
        ],
      );

      final todos = await container.read(todosProvider.future);

      expect(todos.length, 2);
      expect(todos[0].title, 'Todo 1');
    });

    test('adds todo', () async {
      when(() => mockApi.getTodos()).thenAnswer((_) async => []);
      when(() => mockApi.addTodo(any())).thenAnswer((_) async {});

      final container = ProviderContainer.test(
        overrides: [
          todosApiProvider.overrideWithValue(mockApi),
        ],
      );

      await container.read(todosProvider.future);

      when(() => mockApi.getTodos()).thenAnswer((_) async => [
            Todo(id: '1', title: 'New Todo'),
          ]);

      await container.read(todosProvider.notifier).addTodo('New Todo');

      verify(() => mockApi.addTodo('New Todo')).called(1);
    });

    test('handles error', () async {
      when(() => mockApi.getTodos())
          .thenThrow(Exception('Network error'));

      final container = ProviderContainer.test(
        overrides: [
          todosApiProvider.overrideWithValue(mockApi),
        ],
      );

      expect(
        () => container.read(todosProvider.future),
        throwsException,
      );
    });
  });
}
```

---

## 📋 Best Practices

### 1. Mock at the Right Level

```dart
// ✅ GOOD - Mock the API/Repository
final container = ProviderContainer.test(
  overrides: [
    apiProvider.overrideWithValue(MockApi()),
  ],
);

// ❌ AVOID - Mocking the provider itself
final container = ProviderContainer.test(
  overrides: [
    todosProvider.overrideWith(() => mockTodos),
  ],
);
```

### 2. Use Interfaces

```dart
// ✅ GOOD - Abstract interface
abstract class TodosApi {
  Future<List<Todo>> getTodos();
}

class RealTodosApi implements TodosApi { ... }
class MockTodosApi extends Mock implements TodosApi {}

// ❌ BAD - Concrete class (harder to mock)
class TodosApi {
  Future<List<Todo>> getTodos() { ... }
}
```

### 3. Verify Interactions

```dart
test('calls API once', () async {
  // ...
  verify(() => mockApi.getTodos()).called(1);
});

test('never calls delete', () async {
  // ...
  verifyNever(() => mockApi.deleteTodo(any()));
});
```

---

## 📚 المصادر

- [Testing your providers | Riverpod](https://riverpod.dev/docs/how_to/testing)
- [Mocktail Package](https://pub.dev/packages/mocktail)

</div>
