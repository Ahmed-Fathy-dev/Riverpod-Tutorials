<div dir="rtl">

# تطبيق عملي كامل: Todo App

**المستوى**: 🟢 مبتدئ

## 📌 المفاهيم الأساسية

في الملف ده هنبني:
- تطبيق Todo كامل من الصفر
- استخدام كل اللي اتعلمناه (Classic Syntax)
- تطبيق Best Practices
- Project organization

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تبني تطبيق كامل بـ Riverpod Classic Syntax
- تطبق كل المفاهيم الأساسية
- تتبع Best Practices
- تفهم الـ code organization
- تستخدم Notifier للـ complex state

---

## 🎨 التطبيق اللي هنبنيه

### الميزات

</div>

```
Todo App Features:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Add todos
✅ Mark as complete
✅ Delete todos
✅ Filter (all/active/completed)
✅ Clear completed
✅ Counter for active todos
```

<div dir="rtl">

### Screenshots Preview

</div>

```
┌─────────────────────────┐
│  📝 My Todos (3)       │
├─────────────────────────┤
│ All | Active | Done    │
├─────────────────────────┤
│ ☐ Buy groceries        │
│ ☑ Study Riverpod       │
│ ☐ Write code           │
├─────────────────────────┤
│ [+] Add new todo...    │
└─────────────────────────┘
```

<div dir="rtl">

---

## 📁 هيكل المشروع

</div>

```
lib/
├── main.dart
├── models/
│   └── todo.dart
├── providers/
│   ├── todos_provider.dart
│   └── filter_provider.dart
├── screens/
│   └── todos_page.dart
└── widgets/
    ├── todo_item.dart
    ├── todo_list.dart
    ├── add_todo_field.dart
    └── filter_buttons.dart
```

<div dir="rtl">

---

## 🚀 الخطوة 1: Setup

### pubspec.yaml

</div>

```yaml
name: todo_app
description: Todo app with Riverpod

environment:
  sdk: '>=3.0.0 <4.0.0'

dependencies:
  flutter:
    sdk: flutter
  flutter_riverpod: ^3.0.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^3.0.0
```

<div dir="rtl">

### التثبيت

</div>

```bash
flutter pub get
```

<div dir="rtl">

---

## 📦 الخطوة 2: Model

### lib/models/todo.dart

</div>

```dart
// Todo model - represents a single todo item
class Todo {
  final String id;
  final String title;
  final bool completed;

  const Todo({
    required this.id,
    required this.title,
    this.completed = false,
  });

  // CopyWith for immutable updates
  Todo copyWith({
    String? id,
    String? title,
    bool? completed,
  }) {
    return Todo(
      id: id ?? this.id,
      title: title ?? this.title,
      completed: completed ?? this.completed,
    );
  }

  @override
  bool operator ==(Object other) {
    if (identical(this, other)) return true;

    return other is Todo &&
        other.id == id &&
        other.title == title &&
        other.completed == completed;
  }

  @override
  int get hashCode => id.hashCode ^ title.hashCode ^ completed.hashCode;

  @override
  String toString() => 'Todo(id: $id, title: $title, completed: $completed)';
}
```

<div dir="rtl">

---

## 🎯 الخطوة 3: Providers

### lib/providers/filter_provider.dart

</div>

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';

// Filter options enum
enum TodoFilter {
  all,
  active,
  completed,
}

// Simple StateProvider for filter selection
final todoFilterProvider = StateProvider<TodoFilter>((ref) {
  return TodoFilter.all; // Default: show all todos
});
```

<div dir="rtl">

### lib/providers/todos_provider.dart

</div>

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../models/todo.dart';
import 'filter_provider.dart';

// Todos Notifier - manages the list of todos
class TodosNotifier extends Notifier<List<Todo>> {
  @override
  List<Todo> build() {
    // Initial state: empty list
    return [];
  }

  // Add a new todo
  void addTodo(String title) {
    if (title.trim().isEmpty) return;

    final newTodo = Todo(
      id: DateTime.now().toString(),
      title: title.trim(),
    );

    // Add to state immutably
    state = [...state, newTodo];
  }

  // Toggle todo completion
  void toggleTodo(String id) {
    state = [
      for (final todo in state)
        if (todo.id == id)
          todo.copyWith(completed: !todo.completed)
        else
          todo,
    ];
  }

  // Delete a todo
  void deleteTodo(String id) {
    state = state.where((todo) => todo.id != id).toList();
  }

  // Edit todo title
  void editTodo(String id, String newTitle) {
    if (newTitle.trim().isEmpty) return;

    state = [
      for (final todo in state)
        if (todo.id == id)
          todo.copyWith(title: newTitle.trim())
        else
          todo,
    ];
  }

  // Clear all completed todos
  void clearCompleted() {
    state = state.where((todo) => !todo.completed).toList();
  }

  // Mark all as completed
  void markAllComplete() {
    state = [
      for (final todo in state) todo.copyWith(completed: true),
    ];
  }

  // Mark all as incomplete
  void markAllIncomplete() {
    state = [
      for (final todo in state) todo.copyWith(completed: false),
    ];
  }
}

// Todos Provider
final todosProvider = NotifierProvider<TodosNotifier, List<Todo>>(
  () => TodosNotifier(),
);

// Filtered todos provider (computed)
final filteredTodosProvider = Provider<List<Todo>>((ref) {
  final filter = ref.watch(todoFilterProvider);
  final todos = ref.watch(todosProvider);

  switch (filter) {
    case TodoFilter.all:
      return todos;
    case TodoFilter.active:
      return todos.where((todo) => !todo.completed).toList();
    case TodoFilter.completed:
      return todos.where((todo) => todo.completed).toList();
  }
});

// Stats providers (computed)
final uncompletedTodosCountProvider = Provider<int>((ref) {
  final todos = ref.watch(todosProvider);
  return todos.where((todo) => !todo.completed).length;
});

final completedTodosCountProvider = Provider<int>((ref) {
  final todos = ref.watch(todosProvider);
  return todos.where((todo) => todo.completed).length;
});

final totalTodosCountProvider = Provider<int>((ref) {
  final todos = ref.watch(todosProvider);
  return todos.length;
});
```

<div dir="rtl">

---

## 🎨 الخطوة 4: Widgets

### lib/widgets/todo_item.dart

</div>

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../models/todo.dart';
import '../providers/todos_provider.dart';

class TodoItem extends ConsumerStatefulWidget {
  final Todo todo;

  const TodoItem({
    super.key,
    required this.todo,
  });

  @override
  ConsumerState<TodoItem> createState() => _TodoItemState();
}

class _TodoItemState extends ConsumerState<TodoItem> {
  bool _isEditing = false;
  late TextEditingController _controller;
  late FocusNode _focusNode;

  @override
  void initState() {
    super.initState();
    _controller = TextEditingController(text: widget.todo.title);
    _focusNode = FocusNode();
  }

  @override
  void dispose() {
    _controller.dispose();
    _focusNode.dispose();
    super.dispose();
  }

  void _startEditing() {
    setState(() => _isEditing = true);
    _focusNode.requestFocus();
  }

  void _stopEditing() {
    setState(() => _isEditing = false);
    final newTitle = _controller.text.trim();
    if (newTitle.isNotEmpty) {
      ref.read(todosProvider.notifier).editTodo(widget.todo.id, newTitle);
    } else {
      _controller.text = widget.todo.title;
    }
  }

  @override
  Widget build(BuildContext context) {
    return ListTile(
      leading: Checkbox(
        value: widget.todo.completed,
        onChanged: (_) {
          ref.read(todosProvider.notifier).toggleTodo(widget.todo.id);
        },
      ),
      title: _isEditing
          ? TextField(
              controller: _controller,
              focusNode: _focusNode,
              onSubmitted: (_) => _stopEditing(),
              decoration: const InputDecoration(
                border: InputBorder.none,
                contentPadding: EdgeInsets.zero,
              ),
            )
          : GestureDetector(
              onDoubleTap: _startEditing,
              child: Text(
                widget.todo.title,
                style: TextStyle(
                  decoration: widget.todo.completed
                      ? TextDecoration.lineThrough
                      : null,
                  color: widget.todo.completed
                      ? Colors.grey
                      : null,
                ),
              ),
            ),
      trailing: IconButton(
        icon: const Icon(Icons.delete, color: Colors.red),
        onPressed: () {
          ref.read(todosProvider.notifier).deleteTodo(widget.todo.id);
        },
      ),
    );
  }
}
```

<div dir="rtl">

### lib/widgets/add_todo_field.dart

</div>

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../providers/todos_provider.dart';

class AddTodoField extends ConsumerStatefulWidget {
  const AddTodoField({super.key});

  @override
  ConsumerState<AddTodoField> createState() => _AddTodoFieldState();
}

class _AddTodoFieldState extends ConsumerState<AddTodoField> {
  final _controller = TextEditingController();

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  void _addTodo() {
    final title = _controller.text.trim();
    if (title.isNotEmpty) {
      ref.read(todosProvider.notifier).addTodo(title);
      _controller.clear();
    }
  }

  @override
  Widget build(BuildContext context) {
    return TextField(
      controller: _controller,
      decoration: InputDecoration(
        hintText: 'What needs to be done?',
        prefixIcon: const Icon(Icons.add),
        border: OutlineInputBorder(
          borderRadius: BorderRadius.circular(8),
        ),
        suffixIcon: IconButton(
          icon: const Icon(Icons.send),
          onPressed: _addTodo,
        ),
      ),
      onSubmitted: (_) => _addTodo(),
    );
  }
}
```

<div dir="rtl">

### lib/widgets/filter_buttons.dart

</div>

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../providers/filter_provider.dart';

class FilterButtons extends ConsumerWidget {
  const FilterButtons({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final currentFilter = ref.watch(todoFilterProvider);

    return SegmentedButton<TodoFilter>(
      segments: const [
        ButtonSegment(
          value: TodoFilter.all,
          label: Text('All'),
          icon: Icon(Icons.list),
        ),
        ButtonSegment(
          value: TodoFilter.active,
          label: Text('Active'),
          icon: Icon(Icons.radio_button_unchecked),
        ),
        ButtonSegment(
          value: TodoFilter.completed,
          label: Text('Done'),
          icon: Icon(Icons.check_circle),
        ),
      ],
      selected: {currentFilter},
      onSelectionChanged: (Set<TodoFilter> newSelection) {
        ref.read(todoFilterProvider.notifier).state = newSelection.first;
      },
    );
  }
}
```

<div dir="rtl">

### lib/widgets/todo_list.dart

</div>

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../providers/todos_provider.dart';
import 'todo_item.dart';

class TodoList extends ConsumerWidget {
  const TodoList({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final todos = ref.watch(filteredTodosProvider);

    if (todos.isEmpty) {
      return const Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(Icons.inbox, size: 80, color: Colors.grey),
            SizedBox(height: 16),
            Text(
              'No todos yet!',
              style: TextStyle(
                fontSize: 18,
                color: Colors.grey,
              ),
            ),
          ],
        ),
      );
    }

    return ListView.builder(
      itemCount: todos.length,
      itemBuilder: (context, index) {
        final todo = todos[index];
        return TodoItem(
          key: ValueKey(todo.id),
          todo: todo,
        );
      },
    );
  }
}
```

<div dir="rtl">

---

## 📱 الخطوة 5: Screen

### lib/screens/todos_page.dart

</div>

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../providers/todos_provider.dart';
import '../providers/filter_provider.dart';
import '../widgets/add_todo_field.dart';
import '../widgets/filter_buttons.dart';
import '../widgets/todo_list.dart';

class TodosPage extends ConsumerWidget {
  const TodosPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final activeCount = ref.watch(uncompletedTodosCountProvider);
    final completedCount = ref.watch(completedTodosCountProvider);

    return Scaffold(
      appBar: AppBar(
        title: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Text('My Todos'),
            Text(
              '$activeCount active, $completedCount completed',
              style: const TextStyle(fontSize: 12),
            ),
          ],
        ),
        actions: [
          PopupMenuButton<String>(
            onSelected: (value) {
              switch (value) {
                case 'mark_all_complete':
                  ref.read(todosProvider.notifier).markAllComplete();
                  break;
                case 'mark_all_incomplete':
                  ref.read(todosProvider.notifier).markAllIncomplete();
                  break;
                case 'clear_completed':
                  ref.read(todosProvider.notifier).clearCompleted();
                  break;
              }
            },
            itemBuilder: (context) => [
              const PopupMenuItem(
                value: 'mark_all_complete',
                child: Text('Mark all complete'),
              ),
              const PopupMenuItem(
                value: 'mark_all_incomplete',
                child: Text('Mark all incomplete'),
              ),
              const PopupMenuItem(
                value: 'clear_completed',
                child: Text('Clear completed'),
              ),
            ],
          ),
        ],
      ),
      body: Column(
        children: [
          // Filter buttons
          Padding(
            padding: const EdgeInsets.all(16.0),
            child: Column(
              children: [
                const FilterButtons(),
                const SizedBox(height: 16),
                const AddTodoField(),
              ],
            ),
          ),
          const Divider(height: 1),
          // Todo list
          const Expanded(
            child: TodoList(),
          ),
        ],
      ),
    );
  }
}
```

<div dir="rtl">

---

## 🏁 الخطوة 6: Main

### lib/main.dart

</div>

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'screens/todos_page.dart';

void main() {
  runApp(
    const ProviderScope(
      child: MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Riverpod Todo App',
      theme: ThemeData(
        primarySwatch: Colors.blue,
        useMaterial3: true,
      ),
      darkTheme: ThemeData.dark(useMaterial3: true),
      home: const TodosPage(),
      debugShowCheckedModeBanner: false,
    );
  }
}
```

<div dir="rtl">

---

## 🏃 تشغيل التطبيق

</div>

```bash
flutter run
```

<div dir="rtl">

---

## 🎯 شرح الأجزاء المهمة

### 1. Notifier للـ Complex State

</div>

```dart
class TodosNotifier extends Notifier<List<Todo>> {
  @override
  List<Todo> build() => [];

  void addTodo(String title) {
    state = [...state, newTodo]; // Immutable update
  }
}
```

<div dir="rtl">

**ليه استخدمنا Notifier؟**
- الـ state معقد (List of objects)
- محتاجين methods متعددة (add, delete, toggle, etc.)
- فيه business logic (validation, filtering)

**ليه مش StateProvider؟**
- StateProvider للـ state البسيط فقط (int, bool, String)
- مش مناسب للـ lists أو objects معقدة

### 2. Computed Providers (Provider)

</div>

```dart
final filteredTodosProvider = Provider<List<Todo>>((ref) {
  final filter = ref.watch(todoFilterProvider);
  final todos = ref.watch(todosProvider);

  switch (filter) {
    case TodoFilter.all: return todos;
    case TodoFilter.active: return todos.where((t) => !t.completed).toList();
    case TodoFilter.completed: return todos.where((t) => t.completed).toList();
  }
});
```

<div dir="rtl">

**ليه Provider؟**
- القيمة مُحسوبة من providers تانية
- مش بنعدل عليها مباشرة
- بتتحدث تلقائياً لما الـ dependencies تتغير

### 3. Separation of Concerns

**Models:** بيانات فقط (Todo class)
**Providers:** Business logic + State management
**Widgets:** UI فقط، مفيش logic

**الفايدة:**
- سهولة الاختبار
- إعادة الاستخدام
- صيانة أسهل

### 4. Immutability

</div>

```dart
// ✅ GOOD: Create new list
state = [...state, newTodo];

// ❌ BAD: Mutate existing list
state.add(newTodo); // Won't trigger rebuild!
```

<div dir="rtl">

**ليه Immutability مهم؟**
- Riverpod بيتابع التغييرات عن طريق object identity
- لو عدلت في نفس الـ object، مش هيعرف إنه اتغير
- لازم تعمل object جديد عشان Riverpod يعرف

---

## 💡 تحسينات ممكنة

### 1. Persistence (حفظ البيانات)

</div>

```dart
class TodosNotifier extends Notifier<List<Todo>> {
  @override
  List<Todo> build() {
    // Load from storage
    _loadTodos();
    return [];
  }

  Future<void> _loadTodos() async {
    final prefs = await SharedPreferences.getInstance();
    final todosJson = prefs.getString('todos');
    if (todosJson != null) {
      // Parse and set state
    }
  }

  Future<void> _saveTodos() async {
    final prefs = await SharedPreferences.getInstance();
    // Save state to storage
  }

  @override
  void addTodo(String title) {
    state = [...state, newTodo];
    _saveTodos(); // Save after each change
  }
}
```

<div dir="rtl">

### 2. Undo/Redo

</div>

```dart
class TodosNotifier extends Notifier<List<Todo>> {
  final List<List<Todo>> _history = [];
  int _historyIndex = -1;

  void _saveToHistory() {
    _history.add(List.from(state));
    _historyIndex++;
  }

  void undo() {
    if (_historyIndex > 0) {
      _historyIndex--;
      state = List.from(_history[_historyIndex]);
    }
  }

  void redo() {
    if (_historyIndex < _history.length - 1) {
      _historyIndex++;
      state = List.from(_history[_historyIndex]);
    }
  }
}
```

<div dir="rtl">

### 3. Search/Sort

</div>

```dart
// Search provider
final searchQueryProvider = StateProvider<String>((ref) => '');

// Filtered and searched todos
final searchedTodosProvider = Provider<List<Todo>>((ref) {
  final todos = ref.watch(filteredTodosProvider);
  final query = ref.watch(searchQueryProvider).toLowerCase();

  if (query.isEmpty) return todos;

  return todos.where((todo) {
    return todo.title.toLowerCase().contains(query);
  }).toList();
});

// Sort option
enum TodoSort { alphabetical, dateAdded, priority }

final todoSortProvider = StateProvider<TodoSort>((ref) {
  return TodoSort.dateAdded;
});
```

<div dir="rtl">

---

## 🧪 اختبار التطبيق

### Test Example

</div>

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

void main() {
  test('adds todo correctly', () {
    final container = ProviderContainer();

    // Initial state
    expect(container.read(todosProvider), []);

    // Add todo
    container.read(todosProvider.notifier).addTodo('Test todo');

    // Verify
    final todos = container.read(todosProvider);
    expect(todos.length, 1);
    expect(todos.first.title, 'Test todo');
    expect(todos.first.completed, false);

    container.dispose();
  });

  test('toggles todo correctly', () {
    final container = ProviderContainer();

    // Add todo
    container.read(todosProvider.notifier).addTodo('Test todo');
    final todoId = container.read(todosProvider).first.id;

    // Toggle
    container.read(todosProvider.notifier).toggleTodo(todoId);

    // Verify
    final todo = container.read(todosProvider).first;
    expect(todo.completed, true);

    container.dispose();
  });

  test('filters work correctly', () {
    final container = ProviderContainer();

    // Add todos
    container.read(todosProvider.notifier).addTodo('Todo 1');
    container.read(todosProvider.notifier).addTodo('Todo 2');

    // Complete first todo
    final firstTodoId = container.read(todosProvider).first.id;
    container.read(todosProvider.notifier).toggleTodo(firstTodoId);

    // Test active filter
    container.read(todoFilterProvider.notifier).state = TodoFilter.active;
    expect(container.read(filteredTodosProvider).length, 1);

    // Test completed filter
    container.read(todoFilterProvider.notifier).state = TodoFilter.completed;
    expect(container.read(filteredTodosProvider).length, 1);

    // Test all filter
    container.read(todoFilterProvider.notifier).state = TodoFilter.all;
    expect(container.read(filteredTodosProvider).length, 2);

    container.dispose();
  });
}
```

<div dir="rtl">

---

## 📝 ملخص

**اللي عملناه:**
1. ✅ Model واضح ومنظم (Todo class)
2. ✅ Notifier للـ complex state management
3. ✅ Provider للـ computed values
4. ✅ StateProvider للـ simple state (filter)
5. ✅ Separation of concerns (Models, Providers, Widgets)
6. ✅ Immutability في كل التعديلات
7. ✅ تطبيق كامل وشغال

**المفاهيم المستخدمة:**
- **NotifierProvider**: للـ todos list (complex state)
- **Provider**: للـ filtered todos + stats (computed)
- **StateProvider**: للـ filter selection (simple state)
- **ref.watch()**: في build methods
- **ref.read()**: في event handlers
- **Immutable updates**: لكل تعديل على الـ state

**Best Practices المطبقة:**
- Single responsibility لكل widget
- Immutable state updates
- Proper disposal (controllers, focus nodes)
- Type-safe code
- Testable architecture

---

## 🔗 الخطوة الجاية

دلوقتي بعد ما عملت تطبيق كامل بالـ Classic Syntax:

**في Section 04** هنتعلم:
- ref object بالتفصيل
- Provider lifecycle
- Dependency injection patterns
- Family modifier
- AutoDispose

**في Section 06** هنتعلم:
- Code Generation باستخدام `@riverpod`
- إزاي نحول الكود ده لـ code generation
- الفرق والمميزات

**افتكر:** التطبيق ده استخدم Classic Syntax عشان تفهم الأساسيات. في Section 06 هتشوف إزاي Code Generation بيبسط الكود أكتر.

---

## 📚 المصادر

- [Riverpod Documentation](https://riverpod.dev)
- [Provider Types Guide](https://riverpod.dev/docs/providers/provider)
- [Notifier Guide](https://riverpod.dev/docs/providers/notifier_provider)
- [Testing Guide](https://riverpod.dev/docs/cookbooks/testing)

---

## ✅ Checklist

قبل ما تكمل، تأكد من:

- [ ] التطبيق بيشتغل بدون أخطاء
- [ ] فاهم الفرق بين Provider و StateProvider و NotifierProvider
- [ ] فاهم ليه استخدمنا Notifier للـ todos
- [ ] فاهم الـ immutability principle
- [ ] جربت تضيف features جديدة (search, persistence)

</div>
