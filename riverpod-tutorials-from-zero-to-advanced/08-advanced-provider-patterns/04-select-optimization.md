<div dir="rtl">

# Select Optimization - تقليل الـ Rebuilds

**المستوى**: 🔴 متقدم

## 📌 المشكلة

```dart
class User {
  final String name;
  final int age;
  final String email;

  User({required this.name, required this.age, required this.email});
}

@riverpod
class CurrentUser extends _$CurrentUser {
  @override
  User build() => User(name: 'Ahmed', age: 25, email: 'ahmed@example.com');

  void updateAge(int age) {
    state = User(name: state.name, age: age, email: state.email);
  }
}

// ❌ BAD - Rebuilds when name, age, OR email changes!
class UserNameWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final user = ref.watch(currentUserProvider);
    return Text(user.name); // Only uses name!
  }
}
```

---

## ✅ الحل: select

من [Riverpod Docs](https://riverpod.dev/docs/how_to/select):

> The select functionality allows `ref.watch` to return only the selected properties

```dart
// ✅ GOOD - Rebuilds only when name changes!
class UserNameWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final name = ref.watch(currentUserProvider.select((user) => user.name));
    return Text(name);
  }
}
```

---

## 🎯 Syntax

```dart
// Basic select
final name = ref.watch(userProvider.select((user) => user.name));

// Multiple selects (separate watchers)
final name = ref.watch(userProvider.select((user) => user.name));
final age = ref.watch(userProvider.select((user) => user.age));

// Select with async
final name = ref.watch(
  userProvider.select((async) => async.value?.name ?? ''),
);
```

---

## 📊 Performance Comparison

```dart
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() => state++;
}

// Without select - rebuilds every increment
class CounterDisplay extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);
    print('CounterDisplay rebuilt'); // Prints on every increment

    return Text('$count');
  }
}

// With select - rebuilds only when isEven changes
class IsEvenDisplay extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final isEven = ref.watch(counterProvider.select((count) => count.isEven));
    print('IsEvenDisplay rebuilt'); // Prints only when toggling even/odd

    return Text(isEven ? 'Even' : 'Odd');
  }
}

// Test:
// increment() → count = 1 → both rebuild
// increment() → count = 2 → both rebuild
// increment() → count = 3 → only CounterDisplay rebuilds!
// increment() → count = 4 → both rebuild
```

---

## 📋 Best Practices

### 1. Use for Specific Properties

```dart
// ✅ GOOD
final isAdmin = ref.watch(userProvider.select((u) => u.role == 'admin'));

// ❌ OVERKILL - just use regular watch
final user = ref.watch(userProvider.select((u) => u));
```

### 2. Don't Overuse

From the docs:

> Using select slightly slows down individual read operations and increases complexity

Use only when:
- ✅ The property changes rarely
- ✅ The widget is expensive to rebuild
- ✅ You're watching a large object but need one field

### 3. Benchmark First!

```dart
// Before optimizing, measure!
class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    print('MyWidget rebuilt'); // Count rebuilds first
    // ...
  }
}
```

---

## 🎓 Real Example: Form

```dart
class FormState {
  final String name;
  final String email;
  final String password;

  FormState({
    required this.name,
    required this.email,
    required this.password,
  });
}

@riverpod
class RegistrationForm extends _$RegistrationForm {
  @override
  FormState build() => FormState(name: '', email: '', password: '');

  void updateName(String name) {
    state = FormState(name: name, email: state.email, password: state.password);
  }

  void updateEmail(String email) {
    state = FormState(name: state.name, email: email, password: state.password);
  }

  void updatePassword(String password) {
    state = FormState(name: state.name, email: state.email, password: password);
  }
}

// Name field - only rebuilds when name changes
class NameField extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final name = ref.watch(
      registrationFormProvider.select((form) => form.name),
    );

    return TextField(
      value: name,
      onChanged: (value) {
        ref.read(registrationFormProvider.notifier).updateName(value);
      },
    );
  }
}

// Email field - only rebuilds when email changes
class EmailField extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final email = ref.watch(
      registrationFormProvider.select((form) => form.email),
    );

    return TextField(
      value: email,
      onChanged: (value) {
        ref.read(registrationFormProvider.notifier).updateEmail(value);
      },
    );
  }
}

// Now typing in name field doesn't rebuild email field! 🎉
```

---

## 📚 المصادر

- [How to reduce provider/widget rebuilds | Riverpod](https://riverpod.dev/docs/how_to/select)

</div>
