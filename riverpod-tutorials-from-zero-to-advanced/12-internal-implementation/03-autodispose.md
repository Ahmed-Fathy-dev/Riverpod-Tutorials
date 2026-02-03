<div dir="rtl">

# AutoDispose Mechanism

**المستوى**: 🔴 متقدم جداً

## 📌 الخلاصة

**AutoDispose** = تنظيف تلقائي للـ providers عند عدم استخدامها.

```dart
@riverpod  // AutoDispose by default
Future<User> user(UserRef ref) async {
  return await api.getUser();
}

// Internally:
// - Reference counter starts at 0
// - Each watch() increments counter
// - Each unwatch() decrements counter
// - When counter reaches 0 → dispose!
```

---

## 🔍 Reference Counting

```dart
// Widget 1 mounts
ref.watch(userProvider);  // refCount: 0 → 1

// Widget 2 mounts
ref.watch(userProvider);  // refCount: 1 → 2

// Widget 1 unmounts
// refCount: 2 → 1 (still alive!)

// Widget 2 unmounts
// refCount: 1 → 0 (disposed!)
```

---

## 🛑 KeepAlive

```dart
@Riverpod(keepAlive: true)
Future<Config> appConfig(AppConfigRef ref) async {
  return await api.getConfig();
}

// Internally:
// - Reference counting DISABLED
// - Provider never disposed
// - Lives until app termination
```

---

## 📚 المصادر

- [AutoDispose Documentation](https://riverpod.dev/docs/concepts2/auto_dispose)

</div>
