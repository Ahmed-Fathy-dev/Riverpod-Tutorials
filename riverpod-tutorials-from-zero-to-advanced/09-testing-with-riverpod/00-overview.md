<div dir="rtl">

# Testing with Riverpod - نظرة عامة

**المستوى**: 🔴 متقدم

## 🎯 الهدف من هذا القسم

Testing هو جزء أساسي من أي تطبيق احترافي. في **Section 09** هنتعلم:
- 🧪 Unit Testing للـ Providers
- 📱 Widget Testing مع Riverpod
- 🎭 Mocking Dependencies
- ⚡ Testing AsyncNotifier
- 📋 Best Practices

---

## 📚 محتوى القسم

### 1. Unit Testing Providers (01-unit-testing.md)
**المفاهيم:**
- `ProviderContainer.test()` في Riverpod 3
- Testing simple providers
- Testing with dependencies
- Listening to state changes

**مثال:**
```dart
test('counter increments', () {
  final container = ProviderContainer.test();

  expect(container.read(counterProvider), 0);

  container.read(counterProvider.notifier).increment();

  expect(container.read(counterProvider), 1);
});
```

---

### 2. Widget Testing (02-widget-testing.md)
**المفاهيم:**
- `ProviderScope` في tests
- `tester.container()` extension
- Testing UI interactions
- Verifying provider updates

**مثال:**
```dart
testWidgets('displays counter value', (tester) async {
  await tester.pumpWidget(
    const ProviderScope(child: MyApp()),
  );

  expect(find.text('0'), findsOneWidget);

  await tester.tap(find.byIcon(Icons.add));
  await tester.pump();

  expect(find.text('1'), findsOneWidget);
});
```

---

### 3. Mocking Dependencies (03-mocking-dependencies.md)
**المفاهيم:**
- Provider overrides
- Mocking repositories
- Using Mocktail/Mockito
- Test doubles

**مثال:**
```dart
class MockTodosApi extends Mock implements TodosApi {}

test('fetches todos', () async {
  final mockApi = MockTodosApi();
  when(() => mockApi.getTodos()).thenAnswer((_) async => []);

  final container = ProviderContainer.test(
    overrides: [
      todosApiProvider.overrideWithValue(mockApi),
    ],
  );

  await container.read(todosProvider.future);

  verify(() => mockApi.getTodos()).called(1);
});
```

---

### 4. Testing AsyncNotifier (04-testing-async-notifier.md)
**المفاهيم:**
- Testing async initialization
- Testing mutations
- Testing error states
- Using listeners

**مثال:**
```dart
test('adds todo', () async {
  final container = ProviderContainer.test(
    overrides: [todosApiProvider.overrideWithValue(mockApi)],
  );

  await container.read(todosProvider.notifier).addTodo('New Todo');

  final todos = await container.read(todosProvider.future);
  expect(todos.length, 1);
});
```

---

### 5. Best Practices (05-best-practices.md)
**المفاهيم:**
- Test structure
- Arrangeت Act, Assert pattern
- When to unit test vs widget test
- Coverage strategies

---

## 🆕 ما الجديد في Riverpod 3؟

من [Riverpod Testing Docs](https://riverpod.dev/docs/how_to/testing):

### ProviderContainer.test()

```dart
// Riverpod 2: Manual cleanup
test('my test', () {
  final container = ProviderContainer();
  addTearDown(container.dispose);
  // ...
});

// Riverpod 3: Automatic cleanup! ✅
test('my test', () {
  final container = ProviderContainer.test();
  // No manual dispose needed!
});
```

### tester.container()

```dart
// Riverpod 2: Manual access
testWidgets('my test', (tester) async {
  ProviderContainer? container;
  await tester.pumpWidget(
    ProviderScope(
      observer: ContainerObserver((c) => container = c),
      child: MyWidget(),
    ),
  );
  // Use container...
});

// Riverpod 3: Direct access! ✅
testWidgets('my test', (tester) async {
  await tester.pumpWidget(const ProviderScope(child: MyWidget()));
  final container = tester.container();
  // Use container directly!
});
```

---

## 🎯 Testing Strategy

### Pyramid of Tests

```
        /\
       /  \    E2E Tests (Few)
      /____\
     /      \  Widget Tests (Some)
    /________\
   /          \ Unit Tests (Many)
  /__________\
```

### What to Test

| Test Type | What | Example |
|-----------|------|---------|
| **Unit** | Business logic | Provider calculations, state changes |
| **Widget** | UI behavior | Button clicks, text display |
| **Integration** | Full features | Complete user flows |

---

## 📋 Quick Reference

### Unit Testing Checklist

- ✅ Test happy path
- ✅ Test error cases
- ✅ Test edge cases
- ✅ Test async operations
- ✅ Mock external dependencies
- ✅ Verify state changes
- ✅ Check side effects

### Widget Testing Checklist

- ✅ Test UI renders correctly
- ✅ Test user interactions
- ✅ Test loading states
- ✅ Test error states
- ✅ Test navigation
- ✅ Test form validation

---

## 🎓 Prerequisites

قبل ما تبدأ القسم ده، تأكد إنك فاهم:
- ✅ Dart testing basics (`test` package)
- ✅ Flutter widget testing (`flutter_test`)
- ✅ Riverpod providers (Sections 03-08)

---

## 📚 المصادر

- [Testing your providers | Riverpod](https://riverpod.dev/docs/how_to/testing)
- [How to Unit Test AsyncNotifier | Code with Andrea](https://codewithandrea.com/articles/unit-test-async-notifier-riverpod/)
- [Testing Riverpod Providers: Complete Guide](https://article.temiajiboye.com/comprehensive-guide-to-testing-riverpod-providers)

---

## 🚦 Let's Go!

جاهز لتعلم Testing مع Riverpod؟

**الخطوة الأولى:** افتح `01-unit-testing.md` 📖

</div>
