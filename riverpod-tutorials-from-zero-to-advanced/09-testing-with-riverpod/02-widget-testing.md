<div dir="rtl">

# Widget Testing

**المستوى**: 🔴 متقدم

## 📌 الخلاصة

**Widget Testing** = اختبار الـ UI والتفاعل مع المستخدم.

```dart
testWidgets('displays counter', (tester) async {
  await tester.pumpWidget(
    const ProviderScope(child: MyApp()),
  );

  expect(find.text('0'), findsOneWidget);
});
```

---

## 🎯 tester.container()

من [Riverpod 3.0](https://riverpod.dev/docs/whats_new):

> Access ProviderContainer directly in widget tests

```dart
testWidgets('can access container', (tester) async {
  await tester.pumpWidget(const ProviderScope(child: MyWidget()));

  // ✅ NEW in Riverpod 3!
  final container = tester.container();

  expect(container.read(counterProvider), 0);
});
```

---

## 🧪 Complete Example

```dart
// App
class CounterApp extends StatelessWidget {
  const CounterApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('Counter')),
        body: const CounterScreen(),
      ),
    );
  }
}

class CounterScreen extends ConsumerWidget {
  const CounterScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);

    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Text('$count', style: const TextStyle(fontSize: 48)),
          Row(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              IconButton(
                icon: const Icon(Icons.remove),
                onPressed: () =>
                    ref.read(counterProvider.notifier).decrement(),
              ),
              IconButton(
                icon: const Icon(Icons.add),
                onPressed: () =>
                    ref.read(counterProvider.notifier).increment(),
              ),
            ],
          ),
        ],
      ),
    );
  }
}

// Tests
void main() {
  testWidgets('displays initial count', (tester) async {
    await tester.pumpWidget(const ProviderScope(child: CounterApp()));

    expect(find.text('0'), findsOneWidget);
  });

  testWidgets('increments counter', (tester) async {
    await tester.pumpWidget(const ProviderScope(child: CounterApp()));

    await tester.tap(find.byIcon(Icons.add));
    await tester.pump();

    expect(find.text('1'), findsOneWidget);
  });

  testWidgets('decrements counter', (tester) async {
    await tester.pumpWidget(const ProviderScope(child: CounterApp()));

    await tester.tap(find.byIcon(Icons.remove));
    await tester.pump();

    expect(find.text('-1'), findsOneWidget);
  });

  testWidgets('can access container', (tester) async {
    await tester.pumpWidget(const ProviderScope(child: CounterApp()));

    final container = tester.container();

    expect(container.read(counterProvider), 0);

    await tester.tap(find.byIcon(Icons.add));
    await tester.pump();

    expect(container.read(counterProvider), 1);
  });
}
```

---

## 🎭 Testing with Overrides

```dart
testWidgets('displays mocked user', (tester) async {
  await tester.pumpWidget(
    ProviderScope(
      overrides: [
        userProvider.overrideWith((ref) => User(name: 'Test User')),
      ],
      child: const UserApp(),
    ),
  );

  expect(find.text('Test User'), findsOneWidget);
});
```

---

## ⏱️ Testing Async States

```dart
testWidgets('shows loading then data', (tester) async {
  await tester.pumpWidget(
    const ProviderScope(child: UserApp()),
  );

  // Loading state
  expect(find.byType(CircularProgressIndicator), findsOneWidget);

  // Wait for async operation
  await tester.pumpAndSettle();

  // Data state
  expect(find.text('Ahmed'), findsOneWidget);
});
```

---

## 📋 Best Practices

### 1. Always Wrap in ProviderScope

```dart
// ✅ GOOD
await tester.pumpWidget(
  const ProviderScope(child: MyApp()),
);

// ❌ BAD - Will crash!
await tester.pumpWidget(const MyApp());
```

### 2. Use pumpAndSettle for Async

```dart
// ✅ GOOD
await tester.pumpAndSettle(); // Waits for all async

// ⚠️ CAREFUL
await tester.pump(); // Only one frame
```

### 3. Test User Interactions

```dart
// Tap button
await tester.tap(find.byIcon(Icons.add));
await tester.pump();

// Enter text
await tester.enterText(find.byType(TextField), 'Hello');
await tester.pump();

// Scroll
await tester.drag(find.byType(ListView), const Offset(0, -200));
await tester.pump();
```

---

## 📚 المصادر

- [Testing your providers | Riverpod](https://riverpod.dev/docs/how_to/testing)
- [Widget Testing | Flutter](https://docs.flutter.dev/cookbook/testing/widget/introduction)

</div>
