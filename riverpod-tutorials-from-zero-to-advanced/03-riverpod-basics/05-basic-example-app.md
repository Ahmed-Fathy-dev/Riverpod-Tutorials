<div dir="rtl">

# تطبيق عملي كامل: Todo App

**المستوى**: 🟢 مبتدئ

## 📌 المفاهيم الأساسية

في الملف ده هنبني:
- تطبيق Todo كامل من الصفر
- استخدام كل اللي اتعلمناه
- Code generation
- Best practices

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تبني تطبيق كامل بـ Riverpod
- تطبق كل المفاهيم الأساسية
- تتبع Best Practices
- تفهم الـ code organization

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
│   ├── todos_provider.g.dart
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
version: 1.0.0

environment:
  sdk: '>=3.0.0 <4.0.0'

dependencies:
  flutter:
    sdk: flutter
  flutter_riverpod: ^2.5.0
  riverpod_annotation: ^2.3.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  riverpod_generator: ^2.4.0
  build_runner: ^2.4.0
  flutter_lints: ^3.0.0
```

<div dir="rtl">

### Install

</div>

```bash
flutter pub get
```

<div dir="rtl">

---

## 📦 الخطوة 2: Models

### lib/models/todo.dart

</div>

```dart
import 'package:flutter/foundation.dart';

@immutable
class Todo {
  final String id;
  final String title;
  final bool isCompleted;
  final DateTime createdAt;

  const Todo({
    required this.id,
    required this.title,
    this.isCompleted = false,
    required this.createdAt,
  });

  Todo copyWith({
    String? id,
    String? title,
    bool? isCompleted,
    DateTime? createdAt,
  }) {
    return Todo(
      id: id ?? this.id,
      title: title ?? this.title,
      isCompleted: isCompleted ?? this.isCompleted,
      createdAt: createdAt ?? this.createdAt,
    );
  }

  @override
  bool operator ==(Object other) {
    if (identical(this, other)) return true;

    return other is Todo &&
        other.id == id &&
        other.title == title &&
        other.isCompleted == isCompleted;
  }

  @override
  int get hashCode {
    return id.hashCode ^ title.hashCode ^ isCompleted.hashCode;
  }

  @override
  String toString() {
    return 'Todo(id: $id, title: $title, isCompleted: $isCompleted)';
  }
}
```

<div dir="rtl">

---

## 🎯 الخطوة 3: Providers

### lib/providers/filter_provider.dart

</div>

```dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'filter_provider.g.dart';

enum TodoFilter {
  all,
  active,
  completed,
}

@riverpod
class Filter extends _$Filter {
  @override
  TodoFilter build() {
    return TodoFilter.all;
  }

  void setFilter(TodoFilter newFilter) {
    state = newFilter;
  }
}
```

<div dir="rtl">

### lib/providers/todos_provider.dart

</div>

```dart
import 'package:riverpod_annotation/riverpod_annotation.dart';
import '../models/todo.dart';
import 'filter_provider.dart';

part 'todos_provider.g.dart';

@riverpod
class Todos extends _$Todos {
  @override
  List<Todo> build() {
    // Initial empty list
    return [];
  }

  void addTodo(String title) {
    if (title.trim().isEmpty) return;

    final newTodo = Todo(
      id: DateTime.now().millisecondsSinceEpoch.toString(),
      title: title.trim(),
      createdAt: DateTime.now(),
    );

    state = [...state, newTodo];
  }

  void toggleTodo(String id) {
    state = [
      for (final todo in state)
        if (todo.id == id)
          todo.copyWith(isCompleted: !todo.isCompleted)
        else
          todo,
    ];
  }

  void deleteTodo(String id) {
    state = state.where((todo) => todo.id != id).toList();
  }

  void clearCompleted() {
    state = state.where((todo) => !todo.isCompleted).toList();
  }
}

// Filtered todos based on current filter
@riverpod
List<Todo> filteredTodos(FilteredTodosRef ref) {
  final filter = ref.watch(filterProvider);
  final todos = ref.watch(todosProvider);

  switch (filter) {
    case TodoFilter.all:
      return todos;
    case TodoFilter.active:
      return todos.where((todo) => !todo.isCompleted).toList();
    case TodoFilter.completed:
      return todos.where((todo) => todo.isCompleted).toList();
  }
}

// Active todos count
@riverpod
int activeTodosCount(ActiveTodosCountRef ref) {
  final todos = ref.watch(todosProvider);
  return todos.where((todo) => !todo.isCompleted).length;
}

// Completed todos count
@riverpod
int completedTodosCount(CompletedTodosCountRef ref) {
  final todos = ref.watch(todosProvider);
  return todos.where((todo) => todo.isCompleted).length;
}
```

<div dir="rtl">

### Generate Code

</div>

```bash
flutter pub run build_runner watch
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

class TodoItem extends ConsumerWidget {
  final Todo todo;

  const TodoItem({required this.todo});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return Dismissible(
      key: Key(todo.id),
      background: Container(
        color: Colors.red,
        alignment: Alignment.centerRight,
        padding: EdgeInsets.only(right: 16),
        child: Icon(Icons.delete, color: Colors.white),
      ),
      direction: DismissDirection.endToStart,
      onDismissed: (_) {
        ref.read(todosProvider.notifier).deleteTodo(todo.id);

        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(
            content: Text('تم حذف "${todo.title}"'),
            action: SnackBarAction(
              label: 'تراجع',
              onPressed: () {
                // Could implement undo here
              },
            ),
          ),
        );
      },
      child: Card(
        margin: EdgeInsets.symmetric(horizontal: 16, vertical: 4),
        child: ListTile(
          leading: Checkbox(
            value: todo.isCompleted,
            onChanged: (_) {
              ref.read(todosProvider.notifier).toggleTodo(todo.id);
            },
          ),
          title: Text(
            todo.title,
            style: TextStyle(
              decoration: todo.isCompleted
                  ? TextDecoration.lineThrough
                  : TextDecoration.none,
              color: todo.isCompleted ? Colors.grey : Colors.black,
            ),
          ),
          subtitle: Text(
            _formatDate(todo.createdAt),
            style: TextStyle(fontSize: 12, color: Colors.grey),
          ),
          trailing: IconButton(
            icon: Icon(Icons.delete, color: Colors.red),
            onPressed: () {
              ref.read(todosProvider.notifier).deleteTodo(todo.id);
            },
          ),
        ),
      ),
    );
  }

  String _formatDate(DateTime date) {
    final now = DateTime.now();
    final difference = now.difference(date);

    if (difference.inDays == 0) {
      if (difference.inHours == 0) {
        return 'منذ ${difference.inMinutes} دقيقة';
      }
      return 'منذ ${difference.inHours} ساعة';
    } else if (difference.inDays == 1) {
      return 'أمس';
    } else {
      return 'منذ ${difference.inDays} يوم';
    }
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
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final todos = ref.watch(filteredTodosProvider);

    if (todos.isEmpty) {
      return Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(Icons.check_circle_outline, size: 80, color: Colors.grey),
            SizedBox(height: 16),
            Text(
              'مفيش مهام! 🎉',
              style: TextStyle(fontSize: 24, color: Colors.grey),
            ),
          ],
        ),
      );
    }

    return ListView.builder(
      itemCount: todos.length,
      itemBuilder: (context, index) {
        return TodoItem(todo: todos[index]);
      },
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
    final title = _controller.text;

    if (title.trim().isNotEmpty) {
      ref.read(todosProvider.notifier).addTodo(title);
      _controller.clear();
    }
  }

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: EdgeInsets.all(16),
      child: Row(
        children: [
          Expanded(
            child: TextField(
              controller: _controller,
              decoration: InputDecoration(
                hintText: 'أضف مهمة جديدة...',
                border: OutlineInputBorder(),
                prefixIcon: Icon(Icons.add_task),
              ),
              textInputAction: TextInputAction.done,
              onSubmitted: (_) => _addTodo(),
            ),
          ),
          SizedBox(width: 8),
          FloatingActionButton(
            onPressed: _addTodo,
            child: Icon(Icons.add),
          ),
        ],
      ),
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
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final currentFilter = ref.watch(filterProvider);

    return Padding(
      padding: EdgeInsets.symmetric(horizontal: 16, vertical: 8),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          _FilterChip(
            label: 'الكل',
            isSelected: currentFilter == TodoFilter.all,
            onTap: () => ref.read(filterProvider.notifier).setFilter(TodoFilter.all),
          ),
          SizedBox(width: 8),
          _FilterChip(
            label: 'نشطة',
            isSelected: currentFilter == TodoFilter.active,
            onTap: () => ref.read(filterProvider.notifier).setFilter(TodoFilter.active),
          ),
          SizedBox(width: 8),
          _FilterChip(
            label: 'مكتملة',
            isSelected: currentFilter == TodoFilter.completed,
            onTap: () => ref.read(filterProvider.notifier).setFilter(TodoFilter.completed),
          ),
        ],
      ),
    );
  }
}

class _FilterChip extends StatelessWidget {
  final String label;
  final bool isSelected;
  final VoidCallback onTap;

  const _FilterChip({
    required this.label,
    required this.isSelected,
    required this.onTap,
  });

  @override
  Widget build(BuildContext context) {
    return FilterChip(
      label: Text(label),
      selected: isSelected,
      onSelected: (_) => onTap(),
      selectedColor: Theme.of(context).primaryColor.withOpacity(0.2),
    );
  }
}
```

<div dir="rtl">

---

## 🖥️ الخطوة 5: Screen

### lib/screens/todos_page.dart

</div>

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../providers/todos_provider.dart';
import '../widgets/add_todo_field.dart';
import '../widgets/filter_buttons.dart';
import '../widgets/todo_list.dart';

class TodosPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final activeTodosCount = ref.watch(activeTodosCountProvider);
    final completedTodosCount = ref.watch(completedTodosCountProvider);

    return Scaffold(
      appBar: AppBar(
        title: Text('مهامي ($activeTodosCount)'),
        actions: [
          if (completedTodosCount > 0)
            TextButton(
              onPressed: () {
                ref.read(todosProvider.notifier).clearCompleted();

                ScaffoldMessenger.of(context).showSnackBar(
                  SnackBar(
                    content: Text('تم حذف المهام المكتملة'),
                  ),
                );
              },
              child: Text(
                'حذف المكتملة',
                style: TextStyle(color: Colors.white),
              ),
            ),
        ],
      ),
      body: Column(
        children: [
          FilterButtons(),
          Divider(),
          Expanded(child: TodoList()),
          AddTodoField(),
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
    ProviderScope(
      child: MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Todo App',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        primarySwatch: Colors.blue,
        useMaterial3: true,
        appBarTheme: AppBarTheme(
          backgroundColor: Colors.blue,
          foregroundColor: Colors.white,
          elevation: 2,
        ),
      ),
      home: TodosPage(),
    );
  }
}
```

<div dir="rtl">

---

## ▶️ الخطوة 7: Run

</div>

```bash
# Generate code (if not already running watch)
flutter pub run build_runner build

# Run app
flutter run
```

<div dir="rtl">

---

## 🎯 المفاهيم اللي استخدمناها

### 1. Providers

</div>

```dart
✅ @riverpod class - للـ state management
✅ @riverpod function - للـ computed values
✅ Code generation - أقل boilerplate
```

<div dir="rtl">

### 2. Reading Providers

</div>

```dart
✅ ref.watch - للـ UI updates
✅ ref.read - للـ actions
✅ Computed providers - للقيم المحسوبة
```

<div dir="rtl">

### 3. State Management

</div>

```dart
✅ Immutable state - باستخدام copyWith
✅ State updates - باستخدام state = ...
✅ Filters - باستخدام computed providers
```

<div dir="rtl">

### 4. Best Practices

</div>

```dart
✅ Feature-based organization
✅ Separate widgets
✅ ConsumerWidget للـ widgets
✅ ConsumerStatefulWidget عند الحاجة
```

<div dir="rtl">

---

## 🚀 تحسينات ممكنة

### تحسين 1: Persistence

</div>

```dart
// Save to SharedPreferences
@riverpod
class Todos extends _$Todos {
  @override
  List<Todo> build() {
    _loadFromStorage();
    return [];
  }

  Future<void> _loadFromStorage() async {
    final prefs = await SharedPreferences.getInstance();
    // Load and set state
  }

  void addTodo(String title) {
    // ... add todo
    _saveToStorage();
  }

  Future<void> _saveToStorage() async {
    final prefs = await SharedPreferences.getInstance();
    // Save state
  }
}
```

<div dir="rtl">

### تحسين 2: Categories

</div>

```dart
class Todo {
  final String category; // Work, Personal, etc.
  // ...
}

@riverpod
List<String> categories(CategoriesRef ref) {
  final todos = ref.watch(todosProvider);
  return todos.map((t) => t.category).toSet().toList();
}
```

<div dir="rtl">

### تحسين 3: Due Dates

</div>

```dart
class Todo {
  final DateTime? dueDate;
  // ...
}

@riverpod
List<Todo> overdueTodos(OverdueTodosRef ref) {
  final todos = ref.watch(todosProvider);
  final now = DateTime.now();

  return todos.where((todo) {
    final due = todo.dueDate;
    return due != null && due.isBefore(now) && !todo.isCompleted;
  }).toList();
}
```

<div dir="rtl">

### تحسين 4: Search

</div>

```dart
@riverpod
class SearchQuery extends _$SearchQuery {
  @override
  String build() => '';

  void setQuery(String query) {
    state = query;
  }
}

@riverpod
List<Todo> searchedTodos(SearchedTodosRef ref) {
  final todos = ref.watch(filteredTodosProvider);
  final query = ref.watch(searchQueryProvider).toLowerCase();

  if (query.isEmpty) return todos;

  return todos.where((todo) {
    return todo.title.toLowerCase().contains(query);
  }).toList();
}
```

<div dir="rtl">

---

## 📊 ملخص

</div>

```
ما بنيناه:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Todo app كامل
✅ Add/Edit/Delete todos
✅ Filter todos
✅ Counter للمهام
✅ UI جميل ومنظم

المفاهيم المستخدمة:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Providers (@riverpod)
✅ Code generation
✅ ref.watch/read
✅ Computed providers
✅ ConsumerWidget
✅ Best practices

تعلمنا:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Project structure
✅ State management
✅ UI organization
✅ Real-world patterns
```

<div dir="rtl">

---

## 🔗 الخطوة الجاية

دلوقتي بعد ما بنيت تطبيق كامل، جاهز للمستوى الجاي:
- **القسم 04: Core Concepts**
- **Advanced Providers**
- **Testing**

---

## 💡 تمرين

جرب تضيف الميزات دي:

</div>

```
□ Edit todo (double-tap to edit)
□ Categories/Tags
□ Due dates
□ Search
□ Sort (by date, alphabetically)
□ Dark mode
□ Persistence (SharedPreferences)
□ Animations
```

<div dir="rtl">

---

## ✅ تأكد إنك فهمت

- [ ] بنيت التطبيق وشغال؟
- [ ] فاهم كل جزء في الكود؟
- [ ] تقدر تضيف ميزات جديدة؟
- [ ] جاهز للمستوى المتقدم؟

**مبروك! خلصت Riverpod Basics** 🎉

</div>
