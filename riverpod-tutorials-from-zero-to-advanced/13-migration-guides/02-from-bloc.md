<div dir="rtl">

# Migration from Bloc

**المستوى**: 🟡 متوسط

## 📌 Quick Comparison

| Bloc | Riverpod |
|------|----------|
| Bloc/Cubit | Notifier/AsyncNotifier |
| BlocProvider | NotifierProvider |
| BlocBuilder | ConsumerWidget |
| context.read<Bloc>() | ref.read(provider.notifier) |
| Events | Methods |
| States | state property |

---

## 🔄 Migration Examples

### Bloc → AsyncNotifier

```dart
// OLD: Bloc
class CounterBloc extends Bloc<CounterEvent, int> {
  CounterBloc() : super(0) {
    on<IncrementEvent>((event, emit) {
      emit(state + 1);
    });
  }
}

// NEW: Riverpod
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;
  
  void increment() => state++;
}
```

### BlocBuilder → ConsumerWidget

```dart
// OLD: Bloc
BlocBuilder<CounterBloc, int>(
  builder: (context, count) {
    return Text('$count');
  },
)

// NEW: Riverpod
class CounterWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);
    return Text('$count');
  }
}
```

### Events → Methods

```dart
// OLD: Bloc
context.read<CounterBloc>().add(IncrementEvent());

// NEW: Riverpod
ref.read(counterProvider.notifier).increment();
```

---

## 📋 Migration Steps

1. **Identify** Blocs/Cubits
2. **Convert** to Notifiers
3. **Replace** BlocProviders
4. **Update** UI (BlocBuilder → Consumer)
5. **Test** thoroughly

---

## 📚 المصادر

- [From Bloc to Riverpod Guide](https://codewithandrea.com/articles/flutter-state-management-riverpod/)

</div>
