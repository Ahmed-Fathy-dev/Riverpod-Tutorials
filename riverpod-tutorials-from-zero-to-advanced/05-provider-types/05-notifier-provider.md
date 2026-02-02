<div dir="rtl">

# NotifierProvider - State المعقد (Synchronous)

**المستوى**: 🟡 متوسط

## 📌 المفاهيم الأساسية

في الملف ده هنتكلم عن:
- إيه هو NotifierProvider ومتى نستخدمه
- الفرق بينه وبين StateProvider
- إزاي نعمل methods للـ state
- Complex state management
- Best practices

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تستخدم NotifierProvider للـ complex state
- تعمل methods للتعامل مع الـ state
- تفهم متى تستخدم Notifier بدل StateProvider
- تدير state معقد بكفاءة

---

## 🔍 إيه هو NotifierProvider؟

**NotifierProvider** هو provider للـ **complex synchronous state** مع business logic.

</div>

```dart
// Notifier class
class CounterNotifier extends Notifier<int> {
  @override
  int build() {
    return 0; // Initial state
  }

  void increment() {
    state++;
  }

  void decrement() {
    state--;
  }

  void reset() {
    state = 0;
  }

  void add(int value) {
    state += value;
  }
}

// Provider
final counterProvider = NotifierProvider<CounterNotifier, int>(
  () => CounterNotifier(),
);

// Usage
ref.read(counterProvider.notifier).increment();
ref.read(counterProvider.notifier).add(5);
```

<div dir="rtl">

**متى تستخدمه:**
- ✅ Complex state (objects, lists)
- ✅ محتاج methods (add, remove, update)
- ✅ Business logic
- ✅ Synchronous operations

**متى ما تستخدموش:**
- ❌ Simple primitives → use StateProvider
- ❌ Async operations → use AsyncNotifierProvider
- ❌ Read-only → use Provider

---

## 🆚 StateProvider vs NotifierProvider

| Feature | StateProvider | NotifierProvider |
|---------|---------------|------------------|
| **State Type** | Primitives (int, bool) | Objects, Lists |
| **Methods** | لا | نعم ✅ |
| **Business Logic** | لا | نعم ✅ |
| **Update** | `.state = value` | `method()` |

</div>

```dart
// StateProvider - Simple
final counterProvider = StateProvider<int>((ref) => 0);
ref.read(counterProvider.notifier).state++; // Direct

// NotifierProvider - Complex with methods
class CounterNotifier extends Notifier<int> {
  @override
  int build() => 0;

  void increment() => state++;
  void decrement() => state--;
  void incrementBy(int value) => state += value;
}

final counterProvider = NotifierProvider<CounterNotifier, int>(
  () => CounterNotifier(),
);
ref.read(counterProvider.notifier).increment(); // Method
```

<div dir="rtl">

---

## 🎨 الاستخدامات الأساسية

### 1. Shopping Cart

</div>

```dart
// Cart Item model
class CartItem {
  final String id;
  final String productId;
  final String name;
  final double price;
  final int quantity;

  CartItem({
    required this.id,
    required this.productId,
    required this.name,
    required this.price,
    required this.quantity,
  });

  CartItem copyWith({
    String? id,
    String? productId,
    String? name,
    double? price,
    int? quantity,
  }) {
    return CartItem(
      id: id ?? this.id,
      productId: productId ?? this.productId,
      name: name ?? this.name,
      price: price ?? this.price,
      quantity: quantity ?? this.quantity,
    );
  }
}

// Shopping Cart Notifier
class ShoppingCartNotifier extends Notifier<List<CartItem>> {
  @override
  List<CartItem> build() {
    return [];
  }

  void addItem(String productId, String name, double price) {
    // Check if item already exists
    final existingIndex = state.indexWhere(
      (item) => item.productId == productId,
    );

    if (existingIndex >= 0) {
      // Increase quantity
      final updatedItem = state[existingIndex].copyWith(
        quantity: state[existingIndex].quantity + 1,
      );

      state = [
        ...state.sublist(0, existingIndex),
        updatedItem,
        ...state.sublist(existingIndex + 1),
      ];
    } else {
      // Add new item
      final newItem = CartItem(
        id: DateTime.now().toString(),
        productId: productId,
        name: name,
        price: price,
        quantity: 1,
      );

      state = [...state, newItem];
    }
  }

  void removeItem(String itemId) {
    state = state.where((item) => item.id != itemId).toList();
  }

  void updateQuantity(String itemId, int quantity) {
    if (quantity <= 0) {
      removeItem(itemId);
      return;
    }

    state = state.map((item) {
      if (item.id == itemId) {
        return item.copyWith(quantity: quantity);
      }
      return item;
    }).toList();
  }

  void clear() {
    state = [];
  }

  // Computed properties
  double get total {
    return state.fold(0.0, (sum, item) => sum + (item.price * item.quantity));
  }

  int get itemCount {
    return state.fold(0, (sum, item) => sum + item.quantity);
  }
}

// Provider
final shoppingCartProvider = NotifierProvider<ShoppingCartNotifier, List<CartItem>>(
  () => ShoppingCartNotifier(),
);

// Usage in widget
class CartScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final cart = ref.watch(shoppingCartProvider);
    final cartNotifier = ref.read(shoppingCartProvider.notifier);

    return Scaffold(
      appBar: AppBar(
        title: Text('Shopping Cart (${cartNotifier.itemCount})'),
        actions: [
          IconButton(
            icon: Icon(Icons.delete_sweep),
            onPressed: cart.isEmpty ? null : () {
              cartNotifier.clear();
            },
          ),
        ],
      ),
      body: cart.isEmpty
          ? Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Icon(Icons.shopping_cart_outlined, size: 64, color: Colors.grey),
                  SizedBox(height: 16),
                  Text('Your cart is empty'),
                ],
              ),
            )
          : Column(
              children: [
                Expanded(
                  child: ListView.builder(
                    itemCount: cart.length,
                    itemBuilder: (context, index) {
                      final item = cart[index];
                      return ListTile(
                        title: Text(item.name),
                        subtitle: Text('\$${item.price.toStringAsFixed(2)}'),
                        trailing: Row(
                          mainAxisSize: MainAxisSize.min,
                          children: [
                            IconButton(
                              icon: Icon(Icons.remove),
                              onPressed: () {
                                cartNotifier.updateQuantity(
                                  item.id,
                                  item.quantity - 1,
                                );
                              },
                            ),
                            Text('${item.quantity}'),
                            IconButton(
                              icon: Icon(Icons.add),
                              onPressed: () {
                                cartNotifier.updateQuantity(
                                  item.id,
                                  item.quantity + 1,
                                );
                              },
                            ),
                            IconButton(
                              icon: Icon(Icons.delete),
                              onPressed: () {
                                cartNotifier.removeItem(item.id);
                              },
                            ),
                          ],
                        ),
                      );
                    },
                  ),
                ),
                Container(
                  padding: EdgeInsets.all(16),
                  decoration: BoxDecoration(
                    color: Colors.grey[200],
                    border: Border(top: BorderSide(color: Colors.grey)),
                  ),
                  child: Row(
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      Text(
                        'Total: \$${cartNotifier.total.toStringAsFixed(2)}',
                        style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
                      ),
                      ElevatedButton(
                        onPressed: () {
                          // Checkout
                        },
                        child: Text('Checkout'),
                      ),
                    ],
                  ),
                ),
              ],
            ),
    );
  }
}
```

<div dir="rtl">

### 2. Form State Management

</div>

```dart
// Form state
class RegistrationFormState {
  final String name;
  final String email;
  final String password;
  final String confirmPassword;
  final bool agreeToTerms;

  // Validation errors
  final String? nameError;
  final String? emailError;
  final String? passwordError;
  final String? confirmPasswordError;

  RegistrationFormState({
    this.name = '',
    this.email = '',
    this.password = '',
    this.confirmPassword = '',
    this.agreeToTerms = false,
    this.nameError,
    this.emailError,
    this.passwordError,
    this.confirmPasswordError,
  });

  RegistrationFormState copyWith({
    String? name,
    String? email,
    String? password,
    String? confirmPassword,
    bool? agreeToTerms,
    String? Function()? nameError,
    String? Function()? emailError,
    String? Function()? passwordError,
    String? Function()? confirmPasswordError,
  }) {
    return RegistrationFormState(
      name: name ?? this.name,
      email: email ?? this.email,
      password: password ?? this.password,
      confirmPassword: confirmPassword ?? this.confirmPassword,
      agreeToTerms: agreeToTerms ?? this.agreeToTerms,
      nameError: nameError != null ? nameError() : this.nameError,
      emailError: emailError != null ? emailError() : this.emailError,
      passwordError: passwordError != null ? passwordError() : this.passwordError,
      confirmPasswordError: confirmPasswordError != null ? confirmPasswordError() : this.confirmPasswordError,
    );
  }

  bool get isValid {
    return nameError == null &&
        emailError == null &&
        passwordError == null &&
        confirmPasswordError == null &&
        name.isNotEmpty &&
        email.isNotEmpty &&
        password.isNotEmpty &&
        agreeToTerms;
  }
}

// Registration Form Notifier
class RegistrationFormNotifier extends Notifier<RegistrationFormState> {
  @override
  RegistrationFormState build() {
    return RegistrationFormState();
  }

  void updateName(String name) {
    state = state.copyWith(
      name: name,
      nameError: () => _validateName(name),
    );
  }

  void updateEmail(String email) {
    state = state.copyWith(
      email: email,
      emailError: () => _validateEmail(email),
    );
  }

  void updatePassword(String password) {
    state = state.copyWith(
      password: password,
      passwordError: () => _validatePassword(password),
      confirmPasswordError: () => _validateConfirmPassword(password, state.confirmPassword),
    );
  }

  void updateConfirmPassword(String confirmPassword) {
    state = state.copyWith(
      confirmPassword: confirmPassword,
      confirmPasswordError: () => _validateConfirmPassword(state.password, confirmPassword),
    );
  }

  void toggleAgreeToTerms(bool value) {
    state = state.copyWith(agreeToTerms: value);
  }

  String? _validateName(String name) {
    if (name.isEmpty) return 'Name is required';
    if (name.length < 2) return 'Name must be at least 2 characters';
    return null;
  }

  String? _validateEmail(String email) {
    if (email.isEmpty) return 'Email is required';
    final emailRegex = RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$');
    if (!emailRegex.hasMatch(email)) return 'Invalid email format';
    return null;
  }

  String? _validatePassword(String password) {
    if (password.isEmpty) return 'Password is required';
    if (password.length < 8) return 'Password must be at least 8 characters';
    if (!password.contains(RegExp(r'[A-Z]'))) return 'Password must contain uppercase';
    if (!password.contains(RegExp(r'[0-9]'))) return 'Password must contain number';
    return null;
  }

  String? _validateConfirmPassword(String password, String confirmPassword) {
    if (confirmPassword.isEmpty) return 'Please confirm password';
    if (password != confirmPassword) return 'Passwords do not match';
    return null;
  }

  void reset() {
    state = RegistrationFormState();
  }
}

// Provider
final registrationFormProvider = NotifierProvider<RegistrationFormNotifier, RegistrationFormState>(
  () => RegistrationFormNotifier(),
);

// Usage in widget
class RegistrationForm extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final formState = ref.watch(registrationFormProvider);
    final formNotifier = ref.read(registrationFormProvider.notifier);

    return Form(
      child: Column(
        children: [
          TextFormField(
            decoration: InputDecoration(
              labelText: 'Name',
              errorText: formState.nameError,
            ),
            onChanged: formNotifier.updateName,
          ),
          SizedBox(height: 16),
          TextFormField(
            decoration: InputDecoration(
              labelText: 'Email',
              errorText: formState.emailError,
            ),
            onChanged: formNotifier.updateEmail,
          ),
          SizedBox(height: 16),
          TextFormField(
            decoration: InputDecoration(
              labelText: 'Password',
              errorText: formState.passwordError,
            ),
            obscureText: true,
            onChanged: formNotifier.updatePassword,
          ),
          SizedBox(height: 16),
          TextFormField(
            decoration: InputDecoration(
              labelText: 'Confirm Password',
              errorText: formState.confirmPasswordError,
            ),
            obscureText: true,
            onChanged: formNotifier.updateConfirmPassword,
          ),
          SizedBox(height: 16),
          CheckboxListTile(
            title: Text('I agree to terms and conditions'),
            value: formState.agreeToTerms,
            onChanged: (value) {
              formNotifier.toggleAgreeToTerms(value ?? false);
            },
          ),
          SizedBox(height: 24),
          ElevatedButton(
            onPressed: formState.isValid
                ? () {
                    // Submit form
                  }
                : null,
            child: Text('Register'),
          ),
        ],
      ),
    );
  }
}
```

<div dir="rtl">

### 3. Todo List Management

</div>

```dart
// Todo model
class Todo {
  final String id;
  final String title;
  final String description;
  final bool isCompleted;
  final DateTime createdAt;

  Todo({
    required this.id,
    required this.title,
    this.description = '',
    this.isCompleted = false,
    required this.createdAt,
  });

  Todo copyWith({
    String? id,
    String? title,
    String? description,
    bool? isCompleted,
    DateTime? createdAt,
  }) {
    return Todo(
      id: id ?? this.id,
      title: title ?? this.title,
      description: description ?? this.description,
      isCompleted: isCompleted ?? this.isCompleted,
      createdAt: createdAt ?? this.createdAt,
    );
  }
}

// Todos Notifier
class TodosNotifier extends Notifier<List<Todo>> {
  @override
  List<Todo> build() {
    return [];
  }

  void addTodo(String title, {String description = ''}) {
    final newTodo = Todo(
      id: DateTime.now().toString(),
      title: title,
      description: description,
      createdAt: DateTime.now(),
    );

    state = [...state, newTodo];
  }

  void toggleTodo(String id) {
    state = state.map((todo) {
      if (todo.id == id) {
        return todo.copyWith(isCompleted: !todo.isCompleted);
      }
      return todo;
    }).toList();
  }

  void updateTodo(String id, {String? title, String? description}) {
    state = state.map((todo) {
      if (todo.id == id) {
        return todo.copyWith(
          title: title,
          description: description,
        );
      }
      return todo;
    }).toList();
  }

  void deleteTodo(String id) {
    state = state.where((todo) => todo.id != id).toList();
  }

  void clearCompleted() {
    state = state.where((todo) => !todo.isCompleted).toList();
  }

  void toggleAll() {
    final allCompleted = state.every((todo) => todo.isCompleted);

    state = state.map((todo) {
      return todo.copyWith(isCompleted: !allCompleted);
    }).toList();
  }

  // Computed properties
  int get activeCount => state.where((todo) => !todo.isCompleted).length;
  int get completedCount => state.where((todo) => todo.isCompleted).length;
}

// Provider
final todosProvider = NotifierProvider<TodosNotifier, List<Todo>>(
  () => TodosNotifier(),
);

// Filtered todos provider
final filteredTodosProvider = Provider<List<Todo>>((ref) {
  final todos = ref.watch(todosProvider);
  final filter = ref.watch(todoFilterProvider);

  switch (filter) {
    case TodoFilter.all:
      return todos;
    case TodoFilter.active:
      return todos.where((todo) => !todo.isCompleted).toList();
    case TodoFilter.completed:
      return todos.where((todo) => todo.isCompleted).toList();
  }
});
```

<div dir="rtl">

---

## 💻 أمثلة عملية كاملة

### مثال: Multi-Step Wizard

</div>

```dart
// Wizard state
class WizardState {
  final int currentStep;
  final Map<String, dynamic> data;
  final List<String> completedSteps;

  WizardState({
    this.currentStep = 0,
    this.data = const {},
    this.completedSteps = const [],
  });

  WizardState copyWith({
    int? currentStep,
    Map<String, dynamic>? data,
    List<String>? completedSteps,
  }) {
    return WizardState(
      currentStep: currentStep ?? this.currentStep,
      data: data ?? this.data,
      completedSteps: completedSteps ?? this.completedSteps,
    );
  }

  bool get canGoNext => currentStep < 2; // 3 steps total
  bool get canGoPrevious => currentStep > 0;
  bool get isComplete => completedSteps.length >= 3;
}

// Wizard Notifier
class WizardNotifier extends Notifier<WizardState> {
  @override
  WizardState build() {
    return WizardState();
  }

  void nextStep() {
    if (!state.canGoNext) return;

    final stepKey = 'step_${state.currentStep}';
    final completedSteps = [...state.completedSteps];
    if (!completedSteps.contains(stepKey)) {
      completedSteps.add(stepKey);
    }

    state = state.copyWith(
      currentStep: state.currentStep + 1,
      completedSteps: completedSteps,
    );
  }

  void previousStep() {
    if (!state.canGoPrevious) return;

    state = state.copyWith(
      currentStep: state.currentStep - 1,
    );
  }

  void updateData(String key, dynamic value) {
    final updatedData = Map<String, dynamic>.from(state.data);
    updatedData[key] = value;

    state = state.copyWith(data: updatedData);
  }

  void reset() {
    state = WizardState();
  }
}

final wizardProvider = NotifierProvider<WizardNotifier, WizardState>(
  () => WizardNotifier(),
);
```

<div dir="rtl">

---

## 💡 Best Practices

### 1. استخدم Immutability

</div>

```dart
// ✅ Good - Immutable updates
void addItem(CartItem item) {
  state = [...state, item]; // Create new list
}

// ❌ Bad - Mutating state directly
void addItem(CartItem item) {
  state.add(item); // Don't do this!
  state = state; // Force update
}
```

<div dir="rtl">

### 2. Methods بدل Direct State Updates

</div>

```dart
// ✅ Good - Use methods
class CartNotifier extends Notifier<List<CartItem>> {
  void addItem(CartItem item) {
    state = [...state, item];
  }
}

// ❌ Bad - Direct state access
final cart = ref.read(cartProvider.notifier);
cart.state = [...cart.state, item]; // Don't expose state!
```

<div dir="rtl">

### 3. Computed Properties في الـ Notifier

</div>

```dart
// ✅ Good - Computed in Notifier
class CartNotifier extends Notifier<List<CartItem>> {
  double get total => state.fold(0.0, (sum, item) => sum + item.price);
  int get itemCount => state.length;
}

// Usage
final total = ref.read(cartProvider.notifier).total;
```

<div dir="rtl">

---

## 📝 ملخص

**NotifierProvider** يستخدم لـ:
- ✅ Complex state (objects, lists)
- ✅ محتاج methods
- ✅ Business logic
- ✅ Synchronous operations

**مش يستخدم لـ:**
- ❌ Simple primitives → StateProvider
- ❌ Async operations → AsyncNotifierProvider
- ❌ Read-only → Provider

**Best Practices:**
- استخدم immutability
- Methods بدل direct updates
- Computed properties في Notifier

---

## 🔗 الخطوة الجاية

في الملف الجاي هنتكلم عن:
- **AsyncNotifierProvider** - للـ complex async state
- CRUD operations
- Optimistic updates

---

## 📚 المصادر

- [NotifierProvider - Riverpod Docs](https://riverpod.dev/docs/providers/notifier_provider)
- [State Management Best Practices](https://riverpod.dev/docs/concepts/about_providers)

</div>
