<div dir="rtl">

# Migration from GetX

**المستوى**: 🟡 متوسط

## 📌 Quick Comparison

| GetX | Riverpod |
|------|----------|
| GetXController | Notifier |
| Get.put() | ProviderScope |
| Obx | Consumer |
| Get.find() | ref.read |
| GetBuilder | ConsumerWidget |

---

## 🔄 Migration Examples

### GetXController → Notifier

```dart
// OLD: GetX
class CounterController extends GetxController {
  var count = 0.obs;
  
  void increment() => count++;
}

// NEW: Riverpod
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;
  
  void increment() => state++;
}
```

### Obx → Consumer

```dart
// OLD: GetX
Obx(() => Text('${controller.count}'))

// NEW: Riverpod
Consumer(
  builder: (context, ref, child) {
    final count = ref.watch(counterProvider);
    return Text('$count');
  },
)
```

### Get.find → ref.read

```dart
// OLD: GetX
Get.find<CounterController>().increment();

// NEW: Riverpod
ref.read(counterProvider.notifier).increment();
```

---

## 📋 Migration Tips

- ✅ Remove `.obs` - Riverpod handles reactivity
- ✅ Remove Get.put() - use ProviderScope
- ✅ Replace Obx with Consumer
- ✅ No need for Get.find - use ref

---

## 📚 المصادر

- [GetX Alternatives](https://codewithandrea.com/articles/flutter-state-management-riverpod/)

</div>
