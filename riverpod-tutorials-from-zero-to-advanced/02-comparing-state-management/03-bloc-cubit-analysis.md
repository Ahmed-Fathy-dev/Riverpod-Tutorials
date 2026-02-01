<div dir="rtl">

# تحليل شامل لـ BLoC و Cubit

**المستوى**: 🟠 متقدم

## 📌 المفاهيم الأساسية

في الملف ده هنتكلم عن:
- إيه هو BLoC Pattern وإيه الفرق بينه وبين Cubit
- إزاي بيشتغلوا من جوا
- كل الـ API المتاح
- المميزات والعيوب بالتفصيل
- مقارنة مع Riverpod

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تفهم BLoC و Cubit بعمق
- تعرف الفرق بينهم وامتى تستخدم كل واحد
- تفهم ليه شركات كبيرة بتستخدمهم
- تقرر لو BLoC مناسب لمشروعك ولا لأ

---

## 📖 إيه هو BLoC؟

حل BLoC اختصار لـ **Business Logic Component**. ده نمط معماري (Architecture Pattern) اتعمل بواسطة Google سنة 2018.

### الفكرة الأساسية

فصل تام بين:
- **UI Layer**: الـ Widgets
- **Business Logic Layer**: الـ BLoC
- **Data Layer**: Repositories

</div>

```dart
// The BLoC architecture
┌─────────────────────────────────────────┐
│           Presentation Layer            │
│         (Widgets / UI Code)             │
└──────────────┬──────────────────────────┘
               │
        Events │ ↓        ↑ States
               │
┌──────────────┴──────────────────────────┐
│         Business Logic Layer            │
│              (BLoC / Cubit)             │
└──────────────┬──────────────────────────┘
               │
               │ ↓        ↑ Data
               │
┌──────────────┴──────────────────────────┐
│            Data Layer                   │
│     (Repositories / Data Sources)       │
└─────────────────────────────────────────┘
```

<div dir="rtl">

### المبادئ الأساسية

**مبدأ 1:** كل حاجة Events و States

</div>

```dart
// User action → Event
// Event → BLoC
// BLoC processes → Emits State
// State → UI updates
```

<div dir="rtl">

**مبدأ 2:** استخدام Streams

</div>

```dart
// Events in (Sink)
// States out (Stream)
```

<div dir="rtl">

**مبدأ 3:** Single Responsibility

</div>

```dart
// Each BLoC handles ONE business concern
```

<div dir="rtl">

---

## 🎨 الفرق بين BLoC و Cubit

### حل Cubit (الأبسط)

حل Cubit هو نسخة مبسطة من BLoC - بدون Events.

</div>

```dart
// ==========================================
// Cubit: Direct method calls
// ==========================================
class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);

  void increment() => emit(state + 1);
  void decrement() => emit(state - 1);
  void reset() => emit(0);
}

// Usage
final cubit = CounterCubit();
cubit.increment(); // Direct call
```

<div dir="rtl">

**متى تستخدم Cubit:**
- Business Logic بسيط
- مش محتاج Event tracking
- عايز كود أقل

### حل BLoC (الكامل)

حل BLoC بيستخدم Events - كل action بيكون Event.

</div>

```dart
// ==========================================
// BLoC: Event-driven
// ==========================================

// 1. Define Events
abstract class CounterEvent {}

class IncrementPressed extends CounterEvent {}
class DecrementPressed extends CounterEvent {}
class ResetPressed extends CounterEvent {}

// 2. Define BLoC
class CounterBloc extends Bloc<CounterEvent, int> {
  CounterBloc() : super(0) {
    on<IncrementPressed>((event, emit) {
      emit(state + 1);
    });

    on<DecrementPressed>((event, emit) {
      emit(state - 1);
    });

    on<ResetPressed>((event, emit) {
      emit(0);
    });
  }
}

// Usage
final bloc = CounterBloc();
bloc.add(IncrementPressed()); // Add event
```

<div dir="rtl">

**متى تستخدم BLoC:**
- Business Logic معقد
- محتاج Event tracking (analytics, debugging)
- محتاج Event transformations
- عايز full audit trail

### المقارنة المباشرة

| الخاصية | Cubit | BLoC |
|---------|-------|------|
| **البساطة** | ⭐⭐⭐⭐⭐ أبسط | ⭐⭐⭐ معقد شوية |
| **Boilerplate** | ⭐⭐⭐⭐ قليل | ⭐⭐ كتير |
| **Event Tracking** | ❌ مفيش | ✅ كامل |
| **Transformations** | ❌ محدود | ✅ قوي جداً |
| **Testability** | ✅ سهل | ✅ سهل جداً |
| **مناسب لـ** | تطبيقات متوسطة | تطبيقات ضخمة |

---

## 🔍 Cubit بالتفصيل

خليني أبدأ بـ Cubit لأنه الأبسط:

### مثال كامل: Login Cubit

</div>

```dart
// 1. Define states
abstract class LoginState {}

class LoginInitial extends LoginState {}

class LoginLoading extends LoginState {}

class LoginSuccess extends LoginState {
  final User user;
  LoginSuccess(this.user);
}

class LoginFailure extends LoginState {
  final String error;
  LoginFailure(this.error);
}

// 2. Define Cubit
class LoginCubit extends Cubit<LoginState> {
  final AuthRepository authRepository;

  LoginCubit(this.authRepository) : super(LoginInitial());

  Future<void> login(String email, String password) async {
    emit(LoginLoading());

    try {
      final user = await authRepository.login(email, password);
      emit(LoginSuccess(user));
    } on NetworkException {
      emit(LoginFailure('Network error'));
    } on AuthException catch (e) {
      emit(LoginFailure(e.message));
    }
  }

  void logout() {
    emit(LoginInitial());
  }
}

// 3. Provide it
BlocProvider(
  create: (context) => LoginCubit(
    context.read<AuthRepository>(),
  ),
  child: LoginPage(),
);

// 4. Use it
class LoginPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocBuilder<LoginCubit, LoginState>(
      builder: (context, state) {
        if (state is LoginLoading) {
          return CircularProgressIndicator();
        }

        if (state is LoginSuccess) {
          return Text('Welcome ${state.user.name}');
        }

        if (state is LoginFailure) {
          return Text('Error: ${state.error}');
        }

        // LoginInitial
        return LoginForm();
      },
    );
  }
}

class LoginForm extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () {
        context.read<LoginCubit>().login(email, password);
      },
      child: Text('Login'),
    );
  }
}
```

<div dir="rtl">

---

## 🔍 BLoC بالتفصيل

دلوقتي خليني أوريك BLoC الكامل:

### مثال كامل: Todos BLoC

</div>

```dart
// ==========================================
// 1. Define Events
// ==========================================
abstract class TodosEvent {}

class TodosLoadRequested extends TodosEvent {}

class TodoAdded extends TodosEvent {
  final String title;
  TodoAdded(this.title);
}

class TodoToggled extends TodosEvent {
  final String id;
  TodoToggled(this.id);
}

class TodoDeleted extends TodosEvent {
  final String id;
  TodoDeleted(this.id);
}

class TodosFilterChanged extends TodosEvent {
  final TodosFilter filter;
  TodosFilterChanged(this.filter);
}

// ==========================================
// 2. Define States
// ==========================================
enum TodosFilter { all, active, completed }

abstract class TodosState {}

class TodosInitial extends TodosState {}

class TodosLoading extends TodosState {}

class TodosLoaded extends TodosState {
  final List<Todo> todos;
  final TodosFilter filter;

  TodosLoaded({
    required this.todos,
    this.filter = TodosFilter.all,
  });

  List<Todo> get filteredTodos {
    switch (filter) {
      case TodosFilter.all:
        return todos;
      case TodosFilter.active:
        return todos.where((todo) => !todo.isCompleted).toList();
      case TodosFilter.completed:
        return todos.where((todo) => todo.isCompleted).toList();
    }
  }

  TodosLoaded copyWith({
    List<Todo>? todos,
    TodosFilter? filter,
  }) {
    return TodosLoaded(
      todos: todos ?? this.todos,
      filter: filter ?? this.filter,
    );
  }
}

class TodosError extends TodosState {
  final String message;
  TodosError(this.message);
}

// ==========================================
// 3. Define BLoC
// ==========================================
class TodosBloc extends Bloc<TodosEvent, TodosState> {
  final TodosRepository repository;

  TodosBloc(this.repository) : super(TodosInitial()) {
    // Handle TodosLoadRequested event
    on<TodosLoadRequested>((event, emit) async {
      emit(TodosLoading());

      try {
        final todos = await repository.getTodos();
        emit(TodosLoaded(todos: todos));
      } catch (e) {
        emit(TodosError(e.toString()));
      }
    });

    // Handle TodoAdded event
    on<TodoAdded>((event, emit) async {
      if (state is TodosLoaded) {
        final currentState = state as TodosLoaded;

        try {
          final newTodo = await repository.addTodo(event.title);
          emit(currentState.copyWith(
            todos: [...currentState.todos, newTodo],
          ));
        } catch (e) {
          emit(TodosError(e.toString()));
        }
      }
    });

    // Handle TodoToggled event
    on<TodoToggled>((event, emit) async {
      if (state is TodosLoaded) {
        final currentState = state as TodosLoaded;

        try {
          await repository.toggleTodo(event.id);

          final updatedTodos = currentState.todos.map((todo) {
            if (todo.id == event.id) {
              return todo.copyWith(isCompleted: !todo.isCompleted);
            }
            return todo;
          }).toList();

          emit(currentState.copyWith(todos: updatedTodos));
        } catch (e) {
          emit(TodosError(e.toString()));
        }
      }
    });

    // Handle TodoDeleted event
    on<TodoDeleted>((event, emit) async {
      if (state is TodosLoaded) {
        final currentState = state as TodosLoaded;

        try {
          await repository.deleteTodo(event.id);

          final updatedTodos = currentState.todos
              .where((todo) => todo.id != event.id)
              .toList();

          emit(currentState.copyWith(todos: updatedTodos));
        } catch (e) {
          emit(TodosError(e.toString()));
        }
      }
    });

    // Handle TodosFilterChanged event
    on<TodosFilterChanged>((event, emit) {
      if (state is TodosLoaded) {
        final currentState = state as TodosLoaded;
        emit(currentState.copyWith(filter: event.filter));
      }
    });
  }
}

// ==========================================
// 4. Provide it
// ==========================================
BlocProvider(
  create: (context) => TodosBloc(
    context.read<TodosRepository>(),
  )..add(TodosLoadRequested()), // Load on creation
  child: TodosPage(),
);

// ==========================================
// 5. Use it
// ==========================================
class TodosPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocBuilder<TodosBloc, TodosState>(
      builder: (context, state) {
        if (state is TodosLoading) {
          return Center(child: CircularProgressIndicator());
        }

        if (state is TodosError) {
          return Center(child: Text('Error: ${state.message}'));
        }

        if (state is TodosLoaded) {
          return Column(
            children: [
              // Filter buttons
              Row(
                children: [
                  FilterButton(
                    filter: TodosFilter.all,
                    currentFilter: state.filter,
                  ),
                  FilterButton(
                    filter: TodosFilter.active,
                    currentFilter: state.filter,
                  ),
                  FilterButton(
                    filter: TodosFilter.completed,
                    currentFilter: state.filter,
                  ),
                ],
              ),

              // Todos list
              Expanded(
                child: ListView.builder(
                  itemCount: state.filteredTodos.length,
                  itemBuilder: (context, index) {
                    final todo = state.filteredTodos[index];

                    return TodoTile(
                      todo: todo,
                      onToggle: () {
                        context.read<TodosBloc>().add(TodoToggled(todo.id));
                      },
                      onDelete: () {
                        context.read<TodosBloc>().add(TodoDeleted(todo.id));
                      },
                    );
                  },
                ),
              ),

              // Add button
              FloatingActionButton(
                onPressed: () {
                  context.read<TodosBloc>().add(TodoAdded('New Todo'));
                },
                child: Icon(Icons.add),
              ),
            ],
          );
        }

        return SizedBox.shrink();
      },
    );
  }
}
```

<div dir="rtl">

---

## 🎯 الـ API الكامل

### BlocBuilder

للاستماع للـ state و rebuild:

</div>

```dart
// Basic usage
BlocBuilder<CounterBloc, int>(
  builder: (context, state) {
    return Text('Count: $state');
  },
);

// With buildWhen (rebuild condition)
BlocBuilder<TodosBloc, TodosState>(
  buildWhen: (previous, current) {
    // Only rebuild if todos actually changed
    return previous != current;
  },
  builder: (context, state) {
    // ...
  },
);
```

<div dir="rtl">

### BlocListener

للاستماع بدون rebuild (side effects):

</div>

```dart
BlocListener<LoginCubit, LoginState>(
  listener: (context, state) {
    if (state is LoginSuccess) {
      Navigator.pushReplacement(
        context,
        MaterialPageRoute(builder: (_) => HomePage()),
      );
    }

    if (state is LoginFailure) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text(state.error)),
      );
    }
  },
  child: LoginForm(),
);

// With listenWhen (condition)
BlocListener<CounterBloc, int>(
  listenWhen: (previous, current) {
    // Only listen when counter reaches 10
    return current == 10;
  },
  listener: (context, state) {
    showDialog(
      context: context,
      builder: (_) => AlertDialog(
        content: Text('You reached 10!'),
      ),
    );
  },
  child: CounterDisplay(),
);
```

<div dir="rtl">

### BlocConsumer

مزيج من BlocBuilder و BlocListener:

</div>

```dart
BlocConsumer<LoginCubit, LoginState>(
  listener: (context, state) {
    // Side effects
    if (state is LoginSuccess) {
      Navigator.pushReplacement(/*...*/);
    }
  },
  builder: (context, state) {
    // Build UI
    if (state is LoginLoading) {
      return CircularProgressIndicator();
    }
    return LoginForm();
  },
);
```

<div dir="rtl">

### BlocSelector

للاستماع لجزء معين من الـ State:

</div>

```dart
// Only rebuilds when name changes, not other user properties
BlocSelector<UserBloc, UserState, String>(
  selector: (state) {
    if (state is UserLoaded) {
      return state.user.name;
    }
    return '';
  },
  builder: (context, name) {
    return Text('Hello $name');
  },
);
```

<div dir="rtl">

### MultiBlocProvider

لعدة BLoCs:

</div>

```dart
MultiBlocProvider(
  providers: [
    BlocProvider(create: (context) => UserBloc()),
    BlocProvider(create: (context) => TodosBloc()),
    BlocProvider(create: (context) => ThemeBloc()),
  ],
  child: MyApp(),
);
```

<div dir="rtl">

---

## 🚀 ميزات متقدمة

### ميزة 1: Event Transformations

</div>

```dart
// Debounce search events
class SearchBloc extends Bloc<SearchEvent, SearchState> {
  SearchBloc() : super(SearchInitial()) {
    on<SearchQueryChanged>(
      (event, emit) async {
        emit(SearchLoading());
        final results = await api.search(event.query);
        emit(SearchLoaded(results));
      },
      transformer: debounce(Duration(milliseconds: 300)),
    );
  }
}

// Debounce transformer
EventTransformer<E> debounce<E>(Duration duration) {
  return (events, mapper) {
    return events.debounceTime(duration).flatMap(mapper);
  };
}

// Usage
bloc.add(SearchQueryChanged('flutter')); // Waits 300ms before processing
```

<div dir="rtl">

### ميزة 2: BlocObserver (Global monitoring)

</div>

```dart
// Monitor ALL BLoCs in the app
class AppBlocObserver extends BlocObserver {
  @override
  void onCreate(BlocBase bloc) {
    super.onCreate(bloc);
    print('BLoC created: ${bloc.runtimeType}');
  }

  @override
  void onEvent(Bloc bloc, Object? event) {
    super.onEvent(bloc, event);
    print('Event: $event in ${bloc.runtimeType}');
  }

  @override
  void onChange(BlocBase bloc, Change change) {
    super.onChange(bloc, change);
    print('State changed in ${bloc.runtimeType}: $change');
  }

  @override
  void onError(BlocBase bloc, Object error, StackTrace stackTrace) {
    super.onError(bloc, error, stackTrace);
    print('Error in ${bloc.runtimeType}: $error');
  }

  @override
  void onClose(BlocBase bloc) {
    super.onClose(bloc);
    print('BLoC closed: ${bloc.runtimeType}');
  }
}

// Setup in main
void main() {
  Bloc.observer = AppBlocObserver();
  runApp(MyApp());
}
```

<div dir="rtl">

### ميزة 3: Hydrated BLoC (State Persistence)

</div>

```dart
// Automatic state persistence
class CounterBloc extends HydratedBloc<CounterEvent, int> {
  CounterBloc() : super(0) {
    on<Incremented>((event, emit) => emit(state + 1));
  }

  @override
  int? fromJson(Map<String, dynamic> json) {
    return json['counter'] as int;
  }

  @override
  Map<String, dynamic>? toJson(int state) {
    return {'counter': state};
  }
}

// Setup
void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  final storage = await HydratedStorage.build(
    storageDirectory: await getApplicationDocumentsDirectory(),
  );

  HydratedBlocOverrides.runZoned(
    () => runApp(MyApp()),
    storage: storage,
  );
}
```

<div dir="rtl">

---

## 🌟 المميزات

### ميزة 1: فصل تام (Separation of Concerns)

</div>

```dart
// UI knows NOTHING about business logic
// Business logic knows NOTHING about UI
// Perfect separation!

// UI just:
// - Sends events
// - Renders states

// BLoC just:
// - Receives events
// - Emits states
```

<div dir="rtl">

### ميزة 2: Testability ممتازة

</div>

```dart
// Testing is SUPER easy
test('login success flow', () {
  final bloc = LoginCubit(mockRepository);

  when(() => mockRepository.login(any(), any()))
      .thenAnswer((_) async => User(name: 'Ahmed'));

  bloc.login('test@example.com', 'password');

  expectLater(
    bloc.stream,
    emitsInOrder([
      isA<LoginLoading>(),
      isA<LoginSuccess>(),
    ]),
  );
});

test('login failure flow', () {
  final bloc = LoginCubit(mockRepository);

  when(() => mockRepository.login(any(), any()))
      .thenThrow(AuthException('Invalid credentials'));

  bloc.login('test@example.com', 'wrong');

  expectLater(
    bloc.stream,
    emitsInOrder([
      isA<LoginLoading>(),
      isA<LoginFailure>(),
    ]),
  );
});
```

<div dir="rtl">

### ميزة 3: Scalability للمشاريع الضخمة

الـ architecture واضح جداً - سهل يشتغل عليه فريق كبير.

### ميزة 4: Event Tracking كامل

</div>

```dart
// Every action is an event - perfect for:
// - Analytics
// - Debugging
// - Audit trail
// - Time-travel debugging

// You can see EXACTLY what happened:
// 1. User clicked login button → LoginRequested event
// 2. BLoC started loading → LoginLoading state
// 3. API returned success → LoginSuccess state
```

<div dir="rtl">

### ميزة 5: Community و Documentation قوية جداً

- مدعوم من Google
- Documentation ممتازة
- أمثلة كتيرة
- VS Code و Android Studio extensions

---

## ❌ العيوب

### عيب 1: Boilerplate كتير جداً

</div>

```dart
// For a simple boolean toggle, you need:

// 1. Events (2 classes minimum)
abstract class ToggleEvent {}
class TogglePressed extends ToggleEvent {}

// 2. States (2 classes minimum)
abstract class ToggleState {}
class ToggleOn extends ToggleState {}
class ToggleOff extends ToggleState {}

// 3. BLoC (1 class)
class ToggleBloc extends Bloc<ToggleEvent, ToggleState> {
  ToggleBloc() : super(ToggleOff()) {
    on<TogglePressed>((event, emit) {
      emit(state is ToggleOff ? ToggleOn() : ToggleOff());
    });
  }
}

// 4. Provider setup
BlocProvider(create: (_) => ToggleBloc());

// 5. UI code
BlocBuilder<ToggleBloc, ToggleState>(
  builder: (context, state) {
    return Switch(
      value: state is ToggleOn,
      onChanged: (_) {
        context.read<ToggleBloc>().add(TogglePressed());
      },
    );
  },
);

// Compare with Riverpod:
final toggleProvider = StateProvider((ref) => false);

Consumer(
  builder: (context, ref, child) {
    final isOn = ref.watch(toggleProvider);
    return Switch(
      value: isOn,
      onChanged: (value) => ref.read(toggleProvider.notifier).state = value,
    );
  },
);

// Much simpler!
```

<div dir="rtl">

### عيب 2: Learning Curve عالي

</div>

```dart
// Concepts to learn:
// - Streams
// - Sinks
// - StreamControllers
// - Event transformations
// - State hierarchies
// - BLoC vs Cubit
// - When to use which widget (Builder/Listener/Consumer)

// Takes weeks to master!
```

<div dir="rtl">

### عيب 3: نفس مشاكل Provider

</div>

```dart
// ❌ BuildContext dependency
final bloc = context.read<CounterBloc>();

// ❌ Runtime errors
// If BLoC not provided, crashes at runtime

// ❌ Hard to use outside widgets
class MyService {
  void doSomething() {
    // How to access BLoC here?
  }
}
```

<div dir="rtl">

### عيب 4: Overkill للتطبيقات البسيطة

</div>

```dart
// For a todo app with 3 screens
// You'll write:
// - 10+ event classes
// - 10+ state classes
// - 5+ BLoC classes
// - Hundreds of lines of boilerplate

// When Riverpod would need:
// - 5 providers
// - Minimal code
```

<div dir="rtl">

---

## 📊 ملخص: BLoC vs Cubit vs Riverpod

| الجانب | Cubit | BLoC | Riverpod |
|--------|-------|------|----------|
| **سهولة التعلم** | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐ |
| **Boilerplate** | ⭐⭐⭐ | ⭐ | ⭐⭐⭐⭐ |
| **Type Safety** | ⭐⭐ Runtime | ⭐⭐ Runtime | ⭐⭐⭐⭐⭐ Compile |
| **Testing** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Scalability** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Event Tracking** | ❌ | ✅ ممتاز | ⭐⭐⭐ |
| **DX** | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| **BuildContext** | إلزامي | إلزامي | اختياري |

### متى تستخدم BLoC/Cubit؟

</div>

```
✅ استخدمهم لو:
- تطبيق enterprise ضخم جداً
- الفريق معتاد عليهم
- محتاج event tracking و audit trail
- محتاج separation of concerns صارم
- شركة كبيرة ومعايير صارمة

❌ متستخدمهمش لو:
- بتبدأ مشروع جديد (استخدم Riverpod أسهل)
- تطبيق صغير أو متوسط
- مش عايز boilerplate كتير
- محتاج compile-time safety
- محتاج dependency injection أسهل
```

<div dir="rtl">

---

## 🔗 الخطوة الجاية

دلوقتي بعد ما فهمت BLoC و Cubit بالتفصيل، وقت المقارنات المباشرة:
- **مقارنة Riverpod vs Provider** (الملف الجاي)
- **مقارنة Riverpod vs BLoC**
- **دليل Migration من BLoC لـ Riverpod**

---

## 📚 المصادر

- [BLoC Library](https://bloclibrary.dev)
- [BLoC Package](https://pub.dev/packages/flutter_bloc)
- [BLoC Architecture](https://bloclibrary.dev/#/architecture)
- [Cubit vs BLoC](https://bloclibrary.dev/#/coreconcepts?id=cubit-vs-bloc)

---

## ✅ تأكد إنك فهمت

- [ ] تعرف الفرق بين BLoC و Cubit؟
- [ ] فاهم Event-driven architecture؟
- [ ] تقدر تستخدم كل الـ widgets (Builder/Listener/Consumer)؟
- [ ] تعرف المميزات والعيوب؟
- [ ] فاهم امتى BLoC مناسب وامتى لأ؟

**جاهز للمقارنات المباشرة؟** ⚖️

</div>
