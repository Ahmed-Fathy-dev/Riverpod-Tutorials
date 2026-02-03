<div dir="rtl">

# Provider Lifecycle

**المستوى**: 🔴 متقدم جداً

## 📌 الخلاصة

```dart
@riverpod
Future<User> user(UserRef ref) async {
  print('1. Provider build() called');
  
  ref.onDispose(() {
    print('5. Provider disposed');
  });
  
  print('2. Fetching user...');
  final user = await api.getUser();
  
  print('3. Returning user');
  return user;
  // 4. State updated, listeners notified
}
```

**Lifecycle:**
1. Creation (first watch)
2. Build execution
3. State update
4. Listeners notification
5. Disposal (when no listeners)

---

## 🔍 Deep Dive

### Creation

```dart
// When first watched
final user = ref.watch(userProvider);

// Riverpod internally:
// 1. Checks if provider instance exists
// 2. If not, creates new ProviderElement
// 3. Calls build() method
// 4. Stores result
// 5. Subscribes widget to updates
```

### Disposal

```dart
// When last listener removed
// Widget unmounted or provider invalidated

// Riverpod internally:
// 1. Reference count reaches 0
// 2. Calls onDispose callbacks
// 3. Cleans up subscriptions
// 4. Removes provider from container
```

---

## 📚 المصادر

- [Riverpod Source Code](https://github.com/rrousselGit/riverpod)

</div>
