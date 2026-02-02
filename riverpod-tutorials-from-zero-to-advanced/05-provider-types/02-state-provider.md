<div dir="rtl">

# StateProvider - State البسيط القابل للتغيير

**المستوى**: 🟢 مبتدئ

## 📌 المفاهيم الأساسية

في الملف ده هنتكلم عن:
- إيه هو StateProvider ومتى نستخدمه
- الفرق بينه وبين Provider
- أمثلة عملية شاملة
- متى نستخدم Notifier بدل StateProvider
- Best practices

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تستخدم StateProvider للـ state البسيط
- تعرف متى تستخدم StateProvider
- تفهم الفرق بينه وبين Provider و Notifier
- تتجنب الأخطاء الشائعة

---

## 🔍 إيه هو StateProvider؟

**StateProvider** هو provider للـ **simple mutable state**.

الفرق الأساسي عن Provider:
- **Provider**: Read-only (مش بتتغير مباشرة)
- **StateProvider**: Mutable (بتتغير!)

</div>

```dart
// StateProvider - can change!
final counterProvider = StateProvider<int>((ref) => 0);

// Change the value
ref.read(counterProvider.notifier).state++;
ref.read(counterProvider.notifier).state = 10;

// Read the value
final count = ref.watch(counterProvider);
```

<div dir="rtl">

**متى تستخدمه:**
- ✅ Simple state (int, bool, String, enum)
- ✅ UI state (selected tab, filter, etc.)
- ✅ Settings toggles
- ✅ Form fields (simple)

**متى ما تستخدموش:**
- ❌ Complex state (objects, lists) → use NotifierProvider
- ❌ محتاج business logic → use NotifierProvider
- ❌ Async operations → use AsyncNotifierProvider

---

## 🎨 الاستخدامات الأساسية

### 1. Counter (أبسط مثال)

</div>

```dart
// Define the provider
final counterProvider = StateProvider<int>((ref) => 0);

// Widget
class CounterScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Watch the value (rebuilds when changed)
    final count = ref.watch(counterProvider);

    return Scaffold(
      appBar: AppBar(title: Text('Counter')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(
              'Count: $count',
              style: TextStyle(fontSize: 48),
            ),
            SizedBox(height: 20),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                ElevatedButton(
                  onPressed: () {
                    // Decrement
                    ref.read(counterProvider.notifier).state--;
                  },
                  child: Text('-'),
                ),
                SizedBox(width: 20),
                ElevatedButton(
                  onPressed: () {
                    // Increment
                    ref.read(counterProvider.notifier).state++;
                  },
                  child: Text('+'),
                ),
              ],
            ),
            SizedBox(height: 20),
            ElevatedButton(
              onPressed: () {
                // Reset
                ref.read(counterProvider.notifier).state = 0;
              },
              child: Text('Reset'),
            ),
          ],
        ),
      ),
    );
  }
}
```

<div dir="rtl">

### 2. Boolean Toggles

</div>

```dart
// Dark mode toggle
final isDarkModeProvider = StateProvider<bool>((ref) => false);

// Show completed toggle
final showCompletedProvider = StateProvider<bool>((ref) => true);

// Is loading (simple loader)
final isLoadingProvider = StateProvider<bool>((ref) => false);

// Usage
class SettingsScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final isDarkMode = ref.watch(isDarkModeProvider);
    final showCompleted = ref.watch(showCompletedProvider);

    return ListView(
      children: [
        SwitchListTile(
          title: Text('Dark Mode'),
          value: isDarkMode,
          onChanged: (value) {
            ref.read(isDarkModeProvider.notifier).state = value;
          },
        ),
        SwitchListTile(
          title: Text('Show Completed Tasks'),
          value: showCompleted,
          onChanged: (value) {
            ref.read(showCompletedProvider.notifier).state = value;
          },
        ),
      ],
    );
  }
}
```

<div dir="rtl">

### 3. Selected Index / Tab

</div>

```dart
// Selected tab index
final selectedTabProvider = StateProvider<int>((ref) => 0);

// Selected category
final selectedCategoryProvider = StateProvider<String>((ref) => 'all');

// Selected filter
enum TodoFilter { all, active, completed }
final selectedFilterProvider = StateProvider<TodoFilter>((ref) => TodoFilter.all);

// Usage: Bottom Navigation Bar
class HomeScreen extends ConsumerWidget {
  final List<Widget> _screens = [
    HomeTab(),
    SearchTab(),
    ProfileTab(),
  ];

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final selectedIndex = ref.watch(selectedTabProvider);

    return Scaffold(
      body: _screens[selectedIndex],
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: selectedIndex,
        onTap: (index) {
          ref.read(selectedTabProvider.notifier).state = index;
        },
        items: [
          BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Home'),
          BottomNavigationBarItem(icon: Icon(Icons.search), label: 'Search'),
          BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Profile'),
        ],
      ),
    );
  }
}
```

<div dir="rtl">

### 4. Simple Form Fields

</div>

```dart
// Search query
final searchQueryProvider = StateProvider<String>((ref) => '');

// Selected sort option
enum SortOption { nameAsc, nameDesc, dateAsc, dateDesc }
final sortOptionProvider = StateProvider<SortOption>((ref) => SortOption.nameAsc);

// Page size
final pageSizeProvider = StateProvider<int>((ref) => 10);

// Usage: Search Bar
class SearchBar extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final query = ref.watch(searchQueryProvider);

    return TextField(
      decoration: InputDecoration(
        hintText: 'Search...',
        prefixIcon: Icon(Icons.search),
        suffixIcon: query.isNotEmpty
            ? IconButton(
                icon: Icon(Icons.clear),
                onPressed: () {
                  ref.read(searchQueryProvider.notifier).state = '';
                },
              )
            : null,
      ),
      onChanged: (value) {
        ref.read(searchQueryProvider.notifier).state = value;
      },
    );
  }
}
```

<div dir="rtl">

---

## 💻 أمثلة عملية كاملة

### مثال 1: Todo Filters

</div>

```dart
// Filter enum
enum TodoFilter { all, active, completed }

// Filter state
final todoFilterProvider = StateProvider<TodoFilter>((ref) => TodoFilter.all);

// All todos (from somewhere)
final allTodosProvider = StateProvider<List<Todo>>((ref) => [
  Todo(id: '1', title: 'Learn Riverpod', isCompleted: false),
  Todo(id: '2', title: 'Build app', isCompleted: true),
  Todo(id: '3', title: 'Deploy', isCompleted: false),
]);

// Filtered todos (computed)
final filteredTodosProvider = Provider<List<Todo>>((ref) {
  final todos = ref.watch(allTodosProvider);
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

// UI: Filter Chips
class TodoFilterChips extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final currentFilter = ref.watch(todoFilterProvider);

    return Row(
      children: [
        FilterChip(
          label: Text('All'),
          selected: currentFilter == TodoFilter.all,
          onSelected: (_) {
            ref.read(todoFilterProvider.notifier).state = TodoFilter.all;
          },
        ),
        SizedBox(width: 8),
        FilterChip(
          label: Text('Active'),
          selected: currentFilter == TodoFilter.active,
          onSelected: (_) {
            ref.read(todoFilterProvider.notifier).state = TodoFilter.active;
          },
        ),
        SizedBox(width: 8),
        FilterChip(
          label: Text('Completed'),
          selected: currentFilter == TodoFilter.completed,
          onSelected: (_) {
            ref.read(todoFilterProvider.notifier).state = TodoFilter.completed;
          },
        ),
      ],
    );
  }
}

// UI: Todo List
class TodoList extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final todos = ref.watch(filteredTodosProvider);

    return ListView.builder(
      itemCount: todos.length,
      itemBuilder: (context, index) {
        final todo = todos[index];
        return ListTile(
          title: Text(todo.title),
          leading: Checkbox(
            value: todo.isCompleted,
            onChanged: (value) {
              // Toggle todo (implementation depends on how todos are managed)
            },
          ),
        );
      },
    );
  }
}
```

<div dir="rtl">

### مثال 2: Theme Settings

</div>

```dart
// Theme mode
final themeModeProvider = StateProvider<ThemeMode>((ref) => ThemeMode.system);

// Primary color seed
final primaryColorSeedProvider = StateProvider<Color>((ref) => Colors.blue);

// Use Material 3
final useMaterial3Provider = StateProvider<bool>((ref) => true);

// Font size multiplier
final fontSizeMultiplierProvider = StateProvider<double>((ref) => 1.0);

// Settings Screen
class ThemeSettingsScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final themeMode = ref.watch(themeModeProvider);
    final primaryColor = ref.watch(primaryColorSeedProvider);
    final useMaterial3 = ref.watch(useMaterial3Provider);
    final fontSizeMultiplier = ref.watch(fontSizeMultiplierProvider);

    return ListView(
      children: [
        // Theme Mode
        ListTile(
          title: Text('Theme Mode'),
          subtitle: Text(themeMode.name),
        ),
        RadioListTile<ThemeMode>(
          title: Text('Light'),
          value: ThemeMode.light,
          groupValue: themeMode,
          onChanged: (value) {
            ref.read(themeModeProvider.notifier).state = value!;
          },
        ),
        RadioListTile<ThemeMode>(
          title: Text('Dark'),
          value: ThemeMode.dark,
          groupValue: themeMode,
          onChanged: (value) {
            ref.read(themeModeProvider.notifier).state = value!;
          },
        ),
        RadioListTile<ThemeMode>(
          title: Text('System'),
          value: ThemeMode.system,
          groupValue: themeMode,
          onChanged: (value) {
            ref.read(themeModeProvider.notifier).state = value!;
          },
        ),

        Divider(),

        // Primary Color
        ListTile(
          title: Text('Primary Color'),
          trailing: CircleAvatar(backgroundColor: primaryColor),
        ),
        Wrap(
          spacing: 8,
          children: [
            Colors.blue,
            Colors.red,
            Colors.green,
            Colors.purple,
            Colors.orange,
          ].map((color) {
            return ChoiceChip(
              label: Text(''),
              selected: primaryColor == color,
              selectedColor: color,
              backgroundColor: color.withOpacity(0.3),
              onSelected: (_) {
                ref.read(primaryColorSeedProvider.notifier).state = color;
              },
            );
          }).toList(),
        ),

        Divider(),

        // Material 3
        SwitchListTile(
          title: Text('Use Material 3'),
          value: useMaterial3,
          onChanged: (value) {
            ref.read(useMaterial3Provider.notifier).state = value;
          },
        ),

        // Font Size
        ListTile(
          title: Text('Font Size'),
          subtitle: Slider(
            value: fontSizeMultiplier,
            min: 0.8,
            max: 1.5,
            divisions: 7,
            label: '${(fontSizeMultiplier * 100).toInt()}%',
            onChanged: (value) {
              ref.read(fontSizeMultiplierProvider.notifier).state = value;
            },
          ),
        ),
      ],
    );
  }
}
```

<div dir="rtl">

### مثال 3: Pagination Controls

</div>

```dart
// Current page
final currentPageProvider = StateProvider<int>((ref) => 1);

// Page size
final pageSizeProvider = StateProvider<int>((ref) => 10);

// Sort field
final sortFieldProvider = StateProvider<String>((ref) => 'name');

// Sort direction
final sortDirectionProvider = StateProvider<bool>((ref) => true); // true = ascending

// Pagination Widget
class PaginationControls extends ConsumerWidget {
  final int totalItems;

  const PaginationControls({required this.totalItems});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final currentPage = ref.watch(currentPageProvider);
    final pageSize = ref.watch(pageSizeProvider);

    final totalPages = (totalItems / pageSize).ceil();

    return Row(
      mainAxisAlignment: MainAxisAlignment.spaceBetween,
      children: [
        // Previous button
        IconButton(
          icon: Icon(Icons.chevron_left),
          onPressed: currentPage > 1
              ? () {
                  ref.read(currentPageProvider.notifier).state--;
                }
              : null,
        ),

        // Page indicator
        Text('Page $currentPage of $totalPages'),

        // Next button
        IconButton(
          icon: Icon(Icons.chevron_right),
          onPressed: currentPage < totalPages
              ? () {
                  ref.read(currentPageProvider.notifier).state++;
                }
              : null,
        ),

        // Page size selector
        DropdownButton<int>(
          value: pageSize,
          items: [10, 25, 50, 100].map((size) {
            return DropdownMenuItem(
              value: size,
              child: Text('$size per page'),
            );
          }).toList(),
          onChanged: (value) {
            ref.read(pageSizeProvider.notifier).state = value!;
            // Reset to page 1 when changing page size
            ref.read(currentPageProvider.notifier).state = 1;
          },
        ),
      ],
    );
  }
}
```

<div dir="rtl">

---

## ⚠️ أخطاء شائعة

### ❌ خطأ 1: استخدام StateProvider لـ Complex State

</div>

```dart
// ❌ WRONG - Complex object in StateProvider
final userProvider = StateProvider<User>((ref) {
  return User(name: 'Ahmed', email: 'ahmed@example.com');
});

// Updating is verbose and error-prone
ref.read(userProvider.notifier).state = User(
  name: 'Updated Name',
  email: ref.read(userProvider).email, // Have to copy other fields!
);

// ✅ CORRECT - Use NotifierProvider for complex state
class UserNotifier extends Notifier<User> {
  @override
  User build() {
    return User(name: 'Ahmed', email: 'ahmed@example.com');
  }

  void updateName(String name) {
    state = state.copyWith(name: name);
  }

  void updateEmail(String email) {
    state = state.copyWith(email: email);
  }
}

final userProvider = NotifierProvider<UserNotifier, User>(
  () => UserNotifier(),
);
```

<div dir="rtl">

### ❌ خطأ 2: استخدام StateProvider للـ Lists

</div>

```dart
// ❌ WRONG - List in StateProvider
final todosProvider = StateProvider<List<Todo>>((ref) => []);

// Adding item is verbose
ref.read(todosProvider.notifier).state = [
  ...ref.read(todosProvider),
  newTodo,
];

// ✅ CORRECT - Use NotifierProvider
class TodosNotifier extends Notifier<List<Todo>> {
  @override
  List<Todo> build() => [];

  void addTodo(Todo todo) {
    state = [...state, todo];
  }

  void removeTodo(String id) {
    state = state.where((todo) => todo.id != id).toList();
  }

  void toggleTodo(String id) {
    state = state.map((todo) {
      if (todo.id == id) {
        return todo.copyWith(isCompleted: !todo.isCompleted);
      }
      return todo;
    }).toList();
  }
}

final todosProvider = NotifierProvider<TodosNotifier, List<Todo>>(
  () => TodosNotifier(),
);
```

<div dir="rtl">

### ❌ خطأ 3: Forgetting .notifier when updating

</div>

```dart
// ❌ WRONG - Trying to update without .notifier
ref.read(counterProvider).state++; // ERROR! counterProvider returns int, not StateController

// ✅ CORRECT - Use .notifier to get StateController
ref.read(counterProvider.notifier).state++;
```

<div dir="rtl">

---

## 💡 Best Practices

### 1. استخدم StateProvider للـ Primitives فقط

</div>

```dart
// ✅ Good - Simple primitives
final counterProvider = StateProvider<int>((ref) => 0);
final isDarkModeProvider = StateProvider<bool>((ref) => false);
final selectedTabProvider = StateProvider<int>((ref) => 0);

// ❌ Bad - Complex types
final userProvider = StateProvider<User>((ref) => User());
final todosProvider = StateProvider<List<Todo>>((ref) => []);
```

<div dir="rtl">

### 2. استخدم Enums للـ Options

</div>

```dart
// ✅ Good - Type-safe with enum
enum SortOption { nameAsc, nameDesc, dateAsc, dateDesc }
final sortProvider = StateProvider<SortOption>((ref) => SortOption.nameAsc);

// ❌ Bad - String (error-prone)
final sortProvider = StateProvider<String>((ref) => 'name_asc');
```

<div dir="rtl">

### 3. دايماً استخدم .notifier عند التحديث

</div>

```dart
// ✅ Good - Using .notifier
ref.read(counterProvider.notifier).state++;

// ❌ Bad - Will error
ref.read(counterProvider).state++;
```

<div dir="rtl">

### 4. استخدم StateProvider للـ UI State فقط

</div>

```dart
// ✅ Good - UI state
final selectedTabProvider = StateProvider<int>((ref) => 0);
final searchQueryProvider = StateProvider<String>((ref) => '');

// ❌ Bad - Business logic (use Notifier instead)
final userAuthProvider = StateProvider<User?>((ref) => null);
```

<div dir="rtl">

---

## 🆚 StateProvider vs NotifierProvider

| Feature | StateProvider | NotifierProvider |
|---------|---------------|------------------|
| **Use Case** | Simple primitives | Complex state + logic |
| **Types** | int, bool, String, enum | Objects, Lists |
| **Methods** | No | Yes |
| **Update** | `.state = value` | `method()` calls |
| **Verbosity** | Low | Medium |

### متى تستخدم إيه؟

</div>

```dart
// StateProvider - Simple value
final counterProvider = StateProvider<int>((ref) => 0);
ref.read(counterProvider.notifier).state++;

// NotifierProvider - Complex state with methods
class CounterNotifier extends Notifier<int> {
  @override
  int build() => 0;

  void increment() => state++;
  void decrement() => state--;
  void reset() => state = 0;
  void add(int value) => state += value;
}

final counterProvider = NotifierProvider<CounterNotifier, int>(
  () => CounterNotifier(),
);
ref.read(counterProvider.notifier).increment();
```

<div dir="rtl">

**القاعدة الذهبية:**
- لو محتاج **methods** أو **business logic** → NotifierProvider
- لو **primitive بسيط** → StateProvider

---

## 📝 ملخص

**StateProvider** يستخدم لـ:
- ✅ Simple primitives (int, bool, String)
- ✅ UI state (selected tab, filters)
- ✅ Settings toggles
- ✅ Form fields (simple)

**مش يستخدم لـ:**
- ❌ Complex objects → use NotifierProvider
- ❌ Lists → use NotifierProvider
- ❌ محتاج methods → use NotifierProvider
- ❌ Async operations → use AsyncNotifierProvider

**Best Practices:**
- استخدمه للـ primitives فقط
- استخدم enums للـ options
- دايماً `.notifier` عند التحديث
- للـ UI state فقط

---

## 🔗 الخطوة الجاية

في الملف الجاي هنتكلم عن:
- **FutureProvider** - للـ async data اللي بتحصل مرة واحدة
- متى نستخدم FutureProvider
- أمثلة عملية

---

## 📚 المصادر

- [StateProvider - Riverpod Docs](https://riverpod.dev/docs/providers/state_provider)
- [When to use StateProvider](https://riverpod.dev/docs/concepts/providers)

</div>
