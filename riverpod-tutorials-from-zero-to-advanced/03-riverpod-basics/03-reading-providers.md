<div dir="rtl">

# قراءة Providers

**المستوى**: 🟢 مبتدئ

## 📌 المفاهيم الأساسية

في الملف ده هنتكلم عن:
- الطرق المختلفة لقراءة providers
- ref.watch vs ref.read vs ref.listen
- امتى تستخدم كل واحدة
- أمثلة عملية

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تفهم الفرق بين watch, read, listen
- تختار الطريقة المناسبة
- تتجنب الأخطاء الشائعة
- تحسّن الـ performance

---

## 🎭 الطرق الثلاثة

حل Riverpod عنده 3 طرق لقراءة providers:

### نظرة سريعة

| الطريقة | Rebuilds؟ | الاستخدام |
|---------|-----------|----------|
| `ref.watch` | ✅ نعم | قراءة مع rebuild |
| `ref.read` | ❌ لا | قراءة بدون rebuild |
| `ref.listen` | ❌ لا | استماع للتغييرات (side effects) |

---

## 📖 ref.watch (القراءة التفاعلية)

### التعريف

حل `ref.watch` بيقرأ الـ provider و بيعمل rebuild للـ widget لما القيمة تتغير.

### الاستخدام

</div>

```dart
class CounterDisplay extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Watch = read + rebuild on change
    final count = ref.watch(counterProvider);

    // This widget rebuilds when count changes
    return Text('Count: $count');
  }
}
```

<div dir="rtl">

### متى تستخدمه؟

</div>

```
✅ استخدم ref.watch لو:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
- محتاج UI تتحدث لما الـ state يتغير
- في build() method
- في provider تاني (dependencies)

❌ متستخدموش لو:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
- في event handlers (onPressed, onChanged)
- في initState أو dispose
- مش محتاج rebuild
```

<div dir="rtl">

### مثال كامل

</div>

```dart
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() => state++;
}

class CounterPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // ✅ GOOD: watch in build
    final count = ref.watch(counterProvider);

    return Scaffold(
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Rebuilds when count changes
            Text(
              '$count',
              style: TextStyle(fontSize: 48),
            ),
            ElevatedButton(
              onPressed: () {
                // ❌ DON'T watch here!
                // ref.watch(counterProvider); // Wrong!

                // ✅ Use read instead
                ref.read(counterProvider.notifier).increment();
              },
              child: Text('Increment'),
            ),
          ],
        ),
      ),
    );
  }
}
```

<div dir="rtl">

### مثال: Provider يعتمد على provider

</div>

```dart
final firstNameProvider = StateProvider<String>((ref) => 'Ahmed');
final lastNameProvider = StateProvider<String>((ref) => 'Mohamed');

final fullNameProvider = Provider<String>((ref) {
  // ✅ GOOD: watch dependencies
  final firstName = ref.watch(firstNameProvider);
  final lastName = ref.watch(lastNameProvider);

  // Automatically updates when firstName or lastName changes
  return '$firstName $lastName';
});

class NameDisplay extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final fullName = ref.watch(fullNameProvider);

    return Text(fullName); // Updates automatically!
  }
}
```

<div dir="rtl">

---

## 🔍 ref.read (القراءة العادية)

### التعريف

حل `ref.read` بيقرأ القيمة الحالية بس بدون rebuild.

### الاستخدام

</div>

```dart
class IncrementButton extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return ElevatedButton(
      onPressed: () {
        // ✅ GOOD: read in event handler
        ref.read(counterProvider.notifier).increment();
      },
      child: Text('Increment'),
    );
  }
}
```

<div dir="rtl">

### متى تستخدمه؟

</div>

```
✅ استخدم ref.read لو:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
- في event handlers (onPressed, onTap)
- محتاج تستدعي method
- مش محتاج rebuild
- One-time access

❌ متستخدموش لو:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
- محتاج UI تتحدث
- في build() method (استخدم watch)
```

<div dir="rtl">

### مثال: Event Handlers

</div>

```dart
class TodosPage extends ConsumerWidget {
  final _controller = TextEditingController();

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // ✅ Watch for UI
    final todos = ref.watch(todosProvider);

    return Column(
      children: [
        TextField(
          controller: _controller,
          onSubmitted: (text) {
            // ✅ Read for actions
            ref.read(todosProvider.notifier).addTodo(text);
            _controller.clear();
          },
        ),
        ...todos.map((todo) => ListTile(
          title: Text(todo.title),
          trailing: IconButton(
            icon: Icon(Icons.delete),
            onPressed: () {
              // ✅ Read in onPressed
              ref.read(todosProvider.notifier).removeTodo(todo.id);
            },
          ),
        )),
      ],
    );
  }
}
```

<div dir="rtl">

### مثال: Navigation

</div>

```dart
class LoginButton extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return ElevatedButton(
      onPressed: () async {
        // ✅ Read for one-time access
        final authNotifier = ref.read(authProvider.notifier);

        final success = await authNotifier.login(email, password);

        if (success) {
          // ✅ Read again for navigation
          final user = ref.read(authProvider).value;
          Navigator.pushReplacement(
            context,
            MaterialPageRoute(
              builder: (_) => HomePage(user: user),
            ),
          );
        }
      },
      child: Text('Login'),
    );
  }
}
```

<div dir="rtl">

---

## 👂 ref.listen (الاستماع للتغييرات)

### التعريف

حل `ref.listen` بيستمع للتغييرات ويعمل side effects (navigation, snackbars, etc.)

### الاستخدام

</div>

```dart
class LoginPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // ✅ Listen for side effects
    ref.listen(authProvider, (previous, next) {
      next.whenOrNull(
        authenticated: (user) {
          // Navigate on success
          Navigator.pushReplacement(
            context,
            MaterialPageRoute(builder: (_) => HomePage()),
          );
        },
        error: (message) {
          // Show error
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(content: Text(message)),
          );
        },
      );
    });

    // Build UI
    return LoginForm();
  }
}
```

<div dir="rtl">

### متى تستخدمه؟

</div>

```
✅ استخدم ref.listen لو:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
- محتاج navigation
- محتاج snackbars/dialogs
- محتاج logging/analytics
- Side effects (مش UI)

❌ متستخدموش لو:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
- محتاج rebuild UI (استخدم watch)
- مفيش side effects
```

<div dir="rtl">

### مثال 1: Navigation

</div>

```dart
class SplashScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    ref.listen(authProvider, (previous, next) {
      if (next.isAuthenticated) {
        Navigator.pushReplacement(
          context,
          MaterialPageRoute(builder: (_) => HomePage()),
        );
      } else {
        Navigator.pushReplacement(
          context,
          MaterialPageRoute(builder: (_) => LoginPage()),
        );
      }
    });

    return Center(child: CircularProgressIndicator());
  }
}
```

<div dir="rtl">

### مثال 2: Show Snackbar

</div>

```dart
class TodosPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Listen for errors
    ref.listen(todosProvider, (previous, next) {
      next.whenOrNull(
        error: (error, stack) {
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(
              content: Text('Error: $error'),
              backgroundColor: Colors.red,
            ),
          );
        },
      );
    });

    final todosAsync = ref.watch(todosProvider);

    return todosAsync.when(
      data: (todos) => TodosList(todos),
      loading: () => CircularProgressIndicator(),
      error: (error, stack) => ErrorWidget(error),
    );
  }
}
```

<div dir="rtl">

### مثال 3: Logging/Analytics

</div>

```dart
class ProductDetailsPage extends ConsumerWidget {
  final String productId;

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Log page views
    ref.listen(productProvider(productId), (previous, next) {
      next.whenData((product) {
        analytics.logEvent('product_viewed', {
          'product_id': product.id,
          'product_name': product.name,
        });
      });
    });

    final productAsync = ref.watch(productProvider(productId));

    return productAsync.when(
      data: (product) => ProductDetails(product),
      loading: () => CircularProgressIndicator(),
      error: (error, stack) => ErrorWidget(error),
    );
  }
}
```

<div dir="rtl">

---

## ⚖️ المقارنة الشاملة

### جدول المقارنة

| الخاصية | ref.watch | ref.read | ref.listen |
|---------|-----------|----------|------------|
| **Rebuild** | ✅ نعم | ❌ لا | ❌ لا |
| **في build()** | ✅ ممتاز | ⚠️ ممكن | ✅ نعم |
| **في onPressed** | ❌ لا | ✅ ممتاز | ❌ لا |
| **For UI** | ✅ نعم | ❌ لا | ❌ لا |
| **For actions** | ❌ لا | ✅ نعم | ❌ لا |
| **For side effects** | ❌ لا | ❌ لا | ✅ نعم |

### متى تستخدم أيه؟

</div>

```dart
class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // ✅ WATCH: for UI that needs to rebuild
    final count = ref.watch(counterProvider);
    final user = ref.watch(userProvider);

    // ✅ LISTEN: for side effects
    ref.listen(authProvider, (previous, next) {
      if (next.isLoggedOut) {
        Navigator.pushReplacement(/*...*/);
      }
    });

    return Column(
      children: [
        // Display watched values
        Text('Count: $count'),
        Text('User: ${user.name}'),

        ElevatedButton(
          onPressed: () {
            // ✅ READ: in event handlers
            ref.read(counterProvider.notifier).increment();
          },
          child: Text('Increment'),
        ),
      ],
    );
  }
}
```

<div dir="rtl">

---

## ⚠️ أخطاء شائعة

### خطأ 1: watch في onPressed

</div>

```dart
// ❌ BAD: watch in event handler
ElevatedButton(
  onPressed: () {
    final count = ref.watch(counterProvider); // Wrong!
    print('Count: $count');
  },
  child: Text('Print'),
);

// ✅ GOOD: read in event handler
ElevatedButton(
  onPressed: () {
    final count = ref.read(counterProvider);
    print('Count: $count');
  },
  child: Text('Print'),
);
```

<div dir="rtl">

### خطأ 2: read في build (بدون سبب)

</div>

```dart
// ❌ BAD: read in build (UI won't update!)
@override
Widget build(BuildContext context, WidgetRef ref) {
  final count = ref.read(counterProvider); // Won't rebuild!

  return Text('Count: $count'); // Stale value!
}

// ✅ GOOD: watch in build
@override
Widget build(BuildContext context, WidgetRef ref) {
  final count = ref.watch(counterProvider); // Rebuilds!

  return Text('Count: $count'); // Always fresh!
}
```

<div dir="rtl">

### خطأ 3: watch provider نفسه بدل notifier

</div>

```dart
// ❌ BAD: watching entire provider
ElevatedButton(
  onPressed: () {
    final counter = ref.read(counterProvider);
    counter++; // Can't modify int!
  },
  child: Text('Increment'),
);

// ✅ GOOD: read the notifier
ElevatedButton(
  onPressed: () {
    ref.read(counterProvider.notifier).increment();
  },
  child: Text('Increment'),
);
```

<div dir="rtl">

### خطأ 4: listen بدون check

</div>

```dart
// ❌ BAD: listen without null check
ref.listen(userProvider, (previous, next) {
  Navigator.push(
    context,
    MaterialPageRoute(builder: (_) => ProfilePage(next)), // next might be null!
  );
});

// ✅ GOOD: check before using
ref.listen(userProvider, (previous, next) {
  if (next != null) {
    Navigator.push(
      context,
      MaterialPageRoute(builder: (_) => ProfilePage(next)),
    );
  }
});
```

<div dir="rtl">

---

## 🎯 أمثلة عملية

### مثال 1: Shopping Cart

</div>

```dart
@riverpod
class ShoppingCart extends _$ShoppingCart {
  @override
  List<CartItem> build() => [];

  void addItem(Product product) {
    state = [...state, CartItem(product)];
  }

  void removeItem(String id) {
    state = state.where((item) => item.id != id).toList();
  }

  double get total {
    return state.fold(0, (sum, item) => sum + item.price);
  }
}

class ProductCard extends ConsumerWidget {
  final Product product;

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // ✅ Watch for cart to check if product is added
    final cart = ref.watch(shoppingCartProvider);
    final isInCart = cart.any((item) => item.productId == product.id);

    return Card(
      child: Column(
        children: [
          Text(product.name),
          Text('\$${product.price}'),
          ElevatedButton(
            onPressed: isInCart
                ? null
                : () {
                    // ✅ Read for action
                    ref.read(shoppingCartProvider.notifier).addItem(product);

                    // ✅ Show snackbar (could use listen instead)
                    ScaffoldMessenger.of(context).showSnackBar(
                      SnackBar(content: Text('Added to cart')),
                    );
                  },
            child: Text(isInCart ? 'In Cart' : 'Add to Cart'),
          ),
        ],
      ),
    );
  }
}

class CartPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // ✅ Watch for cart items
    final cart = ref.watch(shoppingCartProvider);
    final total = ref.watch(shoppingCartProvider.notifier).total;

    // ✅ Listen for empty cart
    ref.listen(shoppingCartProvider, (previous, next) {
      if (previous != null && previous.isNotEmpty && next.isEmpty) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('Cart is empty')),
        );
      }
    });

    return Scaffold(
      appBar: AppBar(title: Text('Cart')),
      body: Column(
        children: [
          Expanded(
            child: ListView.builder(
              itemCount: cart.length,
              itemBuilder: (context, index) {
                final item = cart[index];
                return ListTile(
                  title: Text(item.name),
                  subtitle: Text('\$${item.price}'),
                  trailing: IconButton(
                    icon: Icon(Icons.delete),
                    onPressed: () {
                      // ✅ Read for action
                      ref.read(shoppingCartProvider.notifier).removeItem(item.id);
                    },
                  ),
                );
              },
            ),
          ),
          Padding(
            padding: EdgeInsets.all(16),
            child: Text(
              'Total: \$$total',
              style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
          ),
        ],
      ),
    );
  }
}
```

<div dir="rtl">

---

## 📊 ملخص: متى تستخدم أيه؟

</div>

```
ref.watch - للـ UI اللي محتاجة rebuild
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ في build() method
✅ لما محتاج UI تتحدث
✅ للـ dependencies بين providers

ref.read - للـ Actions
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ في event handlers (onPressed, onTap)
✅ لما محتاج تستدعي method
✅ لما مش محتاج rebuild

ref.listen - للـ Side Effects
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ للـ navigation
✅ للـ snackbars/dialogs
✅ للـ logging/analytics
✅ أي side effect
```

<div dir="rtl">

---

## 🔗 الخطوة الجاية

دلوقتي بعد ما فهمت إزاي تقرأ providers، وقت:
- **ProviderScope بالتفصيل** (الملف الجاي)
- **تطبيق عملي كامل**
- **Advanced topics**

---

## ✅ تأكد إنك فهمت

- [ ] فاهم الفرق بين watch, read, listen؟
- [ ] تعرف امتى تستخدم كل واحدة؟
- [ ] تقدر تتجنب الأخطاء الشائعة؟
- [ ] جربت الأمثلة بنفسك؟

**جاهز تفهم ProviderScope؟** 🎯

</div>
