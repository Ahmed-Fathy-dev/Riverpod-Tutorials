<div dir="rtl">

# Migration from Provider

**المستوى**: 🟡 متوسط

## 📌 Quick Comparison

| Provider | Riverpod |
|----------|----------|
| ChangeNotifier | Notifier |
| ChangeNotifierProvider | NotifierProvider |
| Provider.of / context.watch | ref.watch |
| context.read | ref.read |
| Consumer | ConsumerWidget |
| MultiProvider | ProviderScope |

---

## 🔄 Migration Examples

### ChangeNotifier → Notifier

```dart
// OLD: Provider
class Counter extends ChangeNotifier {
  int _count = 0;
  int get count => _count;
  
  void increment() {
    _count++;
    notifyListeners();
  }
}

final counterProvider = ChangeNotifierProvider((ref) => Counter());

// NEW: Riverpod
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;
  
  void increment() => state++;
}
```

### Consumer → ConsumerWidget

```dart
// OLD: Provider
class CounterWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Consumer<Counter>(
      builder: (context, counter, child) {
        return Text('${counter.count}');
      },
    );
  }
}

// NEW: Riverpod
class CounterWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);
    return Text('$count');
  }
}
```

### context.watch → ref.watch

```dart
// OLD: Provider
final count = context.watch<Counter>().count;

// NEW: Riverpod
final count = ref.watch(counterProvider);
```

---

## 📋 Migration Steps

1. **Add Riverpod** to pubspec.yaml
2. **Wrap app** في ProviderScope
3. **Migrate one feature** at a time
4. **Test** بعد كل feature
5. **Remove Provider** لما تخلص

---

## 📚 المصادر

- [Provider vs Riverpod | Official Docs](https://riverpod.dev/docs/from_provider/provider_vs_riverpod)

</div>
