<div dir="rtl">

# Unit Testing Providers

**المستوى**: 🔴 متقدم

## 📌 الخلاصة

**Unit Testing** = اختبار الـ logic بدون UI.

من [Riverpod Testing Docs](https://riverpod.dev/docs/how_to/testing):

> Create a ProviderContainer object which will enable your test to interact with providers

---

## 🎯 ProviderContainer.test()

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:riverpod/riverpod.dart';

@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() => state++;
  void decrement() => state--;
}

void main() {
  test('counter starts at 0', () {
    final container = ProviderContainer.test();

    expect(container.read(counterProvider), 0);
  });

  test('counter increments', () {
    final container = ProviderContainer.test();

    container.read(counterProvider.notifier).increment();

    expect(container.read(counterProvider), 1);
  });

  test('counter decrements', () {
    final container = ProviderContainer.test();

    container.read(counterProvider.notifier).increment();
    container.read(counterProvider.notifier).increment();
    container.read(counterProvider.notifier).decrement();

    expect(container.read(counterProvider), 1);
  });
}
```

---

## 🔍 Testing Async Providers

```dart
@riverpod
Future<User> user(UserRef ref) async {
  return await api.getUser();
}

void main() {
  test('loads user', () async {
    final container = ProviderContainer.test();

    // Use .future to wait for result
    final user = await container.read(userProvider.future);

    expect(user.name, 'Ahmed');
  });

  test('handles error', () async {
    final container = ProviderContainer.test(
      overrides: [
        // Mock API to throw error
        apiProvider.overrideWithValue(FailingApi()),
      ],
    );

    expect(
      () => container.read(userProvider.future),
      throwsException,
    );
  });
}
```

---

## 👂 Listening to Changes

```dart
test('notifies listeners', () {
  final container = ProviderContainer.test();
  final listener = Listener<int>();

  container.listen(
    counterProvider,
    listener.call,
    fireImmediately: true,
  );

  verify(() => listener(null, 0)).called(1);

  container.read(counterProvider.notifier).increment();

  verify(() => listener(0, 1)).called(1);
});

class Listener<T> extends Mock {
  void call(T? previous, T next);
}
```

---

## 🧪 Complete Example

```dart
// Model
class Todo {
  final String id;
  final String title;
  final bool completed;

  Todo({required this.id, required this.title, this.completed = false});

  Todo copyWith({bool? completed}) =>
      Todo(id: id, title: title, completed: completed ?? this.completed);
}

// API
abstract class TodosApi {
  Future<List<Todo>> getTodos();
  Future<void> addTodo(String title);
  Future<void> toggleTodo(String id);
}

// Provider
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

  Future<void> toggleTodo(String id) async {
    final previousState = state;

    state = state.whenData((todos) {
      return todos.map((todo) {
        if (todo.id == id) {
          return todo.copyWith(completed: !todo.completed);
        }
        return todo;
      }).toList();
    });

    try {
      final api = ref.read(todosApiProvider);
      await api.toggleTodo(id);
    } catch (error, stackTrace) {
      state = previousState;
      state = AsyncValue.error(error, stackTrace);
    }
  }
}

// Tests
void main() {
  late MockTodosApi mockApi;

  setUp(() {
    mockApi = MockTodosApi();
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
    verify(() => mockApi.getTodos()).called(1);
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

    final todos = await container.read(todosProvider.future);
    expect(todos.length, 1);
    expect(todos[0].title, 'New Todo');
    verify(() => mockApi.addTodo('New Todo')).called(1);
  });

  test('toggles todo', () async {
    when(() => mockApi.getTodos()).thenAnswer((_) async => [
          Todo(id: '1', title: 'Todo 1', completed: false),
        ]);
    when(() => mockApi.toggleTodo(any())).thenAnswer((_) async {});

    final container = ProviderContainer.test(
      overrides: [
        todosApiProvider.overrideWithValue(mockApi),
      ],
    );

    await container.read(todosProvider.future);

    await container.read(todosProvider.notifier).toggleTodo('1');

    final todos = container.read(todosProvider).requireValue;
    expect(todos[0].completed, true);
    verify(() => mockApi.toggleTodo('1')).called(1);
  });
}
```

---

## 📋 Best Practices

### 1. Use ProviderContainer.test()

```dart
// ✅ GOOD - Riverpod 3
test('my test', () {
  final container = ProviderContainer.test();
});

// ❌ OLD - Riverpod 2
test('my test', () {
  final container = ProviderContainer();
  addTearDown(container.dispose);
});
```

### 2. Mock Dependencies, Not Providers

```dart
// ✅ GOOD - Mock the API
test('test', () {
  final container = ProviderContainer.test(
    overrides: [
      apiProvider.overrideWithValue(MockApi()),
    ],
  );
});

// ❌ AVOID - Mocking the provider itself
test('test', () {
  final container = ProviderContainer.test(
    overrides: [
      todosProvider.overrideWith(() => mockTodos),
    ],
  );
});
```

### 3. Use .future for Async Providers

```dart
// ✅ GOOD
final user = await container.read(userProvider.future);

// ❌ VERBOSE
final userAsync = container.read(userProvider);
final user = userAsync.requireValue;
```

---

## 📚 المصادر

- [Testing your providers | Riverpod](https://riverpod.dev/docs/how_to/testing)
- [How to Unit Test AsyncNotifier | Code with Andrea](https://codewithandrea.com/articles/unit-test-async-notifier-riverpod/)

</div>
