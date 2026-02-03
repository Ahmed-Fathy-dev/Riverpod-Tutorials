<div dir="rtl">

# Testing AsyncNotifier

**المستوى**: 🔴 متقدم

## 📌 الخلاصة

Testing **AsyncNotifier** requires handling async initialization and state changes.

من [Code with Andrea](https://codewithandrea.com/articles/unit-test-async-notifier-riverpod/):

> Test async initialization, mutations, and error states

---

## 🧪 Testing Initialization

```dart
@riverpod
class Todos extends _$Todos {
  @override
  Future<List<Todo>> build() async {
    final api = ref.watch(todosApiProvider);
    return await api.getTodos();
  }
}

test('loads todos on init', () async {
  when(() => mockApi.getTodos()).thenAnswer((_) async => [
        Todo(id: '1', title: 'Todo 1'),
      ]);

  final container = ProviderContainer.test(
    overrides: [
      todosApiProvider.overrideWithValue(mockApi),
    ],
  );

  // Read .future to wait for initialization
  final todos = await container.read(todosProvider.future);

  expect(todos.length, 1);
  verify(() => mockApi.getTodos()).called(1);
});
```

---

## 🔄 Testing Mutations

```dart
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
});
```

---

## 👂 Using Listeners

```dart
test('notifies on state change', () async {
  when(() => mockApi.getTodos()).thenAnswer((_) async => []);

  final container = ProviderContainer.test(
    overrides: [
      todosApiProvider.overrideWithValue(mockApi),
    ],
  );

  final states = <AsyncValue<List<Todo>>>[];

  container.listen(
    todosProvider,
    (previous, next) {
      states.add(next);
    },
    fireImmediately: true,
  );

  await container.read(todosProvider.future);

  // Should have: loading, then data
  expect(states.length, 2);
  expect(states[0].isLoading, true);
  expect(states[1].hasValue, true);
});
```

---

## ⚠️ Testing Errors

```dart
test('handles error on init', () async {
  when(() => mockApi.getTodos()).thenThrow(Exception('Failed'));

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

test('handles error on mutation', () async {
  when(() => mockApi.getTodos()).thenAnswer((_) async => []);
  when(() => mockApi.addTodo(any())).thenThrow(Exception('Failed'));

  final container = ProviderContainer.test(
    overrides: [
      todosApiProvider.overrideWithValue(mockApi),
    ],
  );

  await container.read(todosProvider.future);

  await container.read(todosProvider.notifier).addTodo('New Todo');

  final state = container.read(todosProvider);
  expect(state.hasError, true);
});
```

---

## 📋 Best Practices

### 1. Always Await Initialization

```dart
// ✅ GOOD
await container.read(todosProvider.future);

// ❌ BAD - Might not be initialized
container.read(todosProvider.notifier).addTodo('...');
```

### 2. Test All States

```dart
test('goes through all states', () async {
  final states = <AsyncValue<List<Todo>>>[];

  container.listen(todosProvider, (_, next) => states.add(next));

  // Test loading
  expect(states[0].isLoading, true);

  // Test data
  await container.read(todosProvider.future);
  expect(states[1].hasValue, true);

  // Test error
  when(() => mockApi.addTodo(any())).thenThrow(Exception());
  await container.read(todosProvider.notifier).addTodo('...');
  expect(states.last.hasError, true);
});
```

### 3. Mock Dependencies, Not State

```dart
// ✅ GOOD
final container = ProviderContainer.test(
  overrides: [
    apiProvider.overrideWithValue(mockApi),
  ],
);

// ❌ AVOID
final container = ProviderContainer.test(
  overrides: [
    todosProvider.overrideWith(() => AsyncValue.data([...])),
  ],
);
```

---

## 📚 المصادر

- [How to Unit Test AsyncNotifier | Code with Andrea](https://codewithandrea.com/articles/unit-test-async-notifier-riverpod/)
- [Testing Riverpod Providers](https://article.temiajiboye.com/comprehensive-guide-to-testing-riverpod-providers)

</div>
