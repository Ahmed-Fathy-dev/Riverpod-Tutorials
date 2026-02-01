<div dir="rtl">

# دليل الانتقال من BLoC إلى Riverpod

**المستوى**: 🟠 متقدم

## 📌 المفاهيم الأساسية

في الملف ده هنتكلم عن:
- استراتيجية الانتقال من BLoC لـ Riverpod
- تحويل BLoC/Cubit patterns لـ Riverpod
- التعايش المؤقت بين الحلين
- نصائح لانتقال سلس

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تقرر لو Migration مناسب ليك
- تحول BLoC/Cubit code لـ Riverpod
- تدير codebase مختلط
- تتجنب المشاكل الشائعة

---

## 🤔 لازم تهاجر؟

### امتى Migration مفيد:

</div>

```
✅ هاجر لو:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
- Boilerplate كتير وبيبطأ التطوير
- محتاج compile-time safety
- محتاج dependency injection أفضل
- عايز auto disposal تلقائي
- الفريق شايف إن Riverpod أبسط
- بتعمل refactoring كبير بالفعل

❌ متهاجرش لو:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
- BLoC شغال كويس ومفيش مشاكل
- الفريق كله comfortable مع BLoC
- مفيش وقت للـ migration
- المشروع stable ومش محتاج changes
- Event tracking مهم جداً ليك
```

<div dir="rtl">

---

## 🗺️ استراتيجيات الانتقال

### استراتيجية 1: Big Bang (مرة واحدة)

**مناسب لو:**
- المشروع صغير (< 20 BLoCs)
- عندك وقت كافي
- الفريق صغير

**الخطوات:**

</div>

```
Week 1: Planning
- تحديد كل BLoCs
- تقييم complexity
- كتابة migration plan

Week 2-3: Migration
- تحويل كل BLoCs
- Testing شامل

Week 4: Cleanup
- شيل BLoC dependencies
- Documentation
```

<div dir="rtl">

### استراتيجية 2: Gradual (تدريجي) - الموصى بها

**مناسب لو:**
- المشروع كبير (20+ BLoCs)
- مفيش وقت مخصص للـ migration
- عايز تقلل المخاطر

**الخطوات:**

</div>

```
Phase 1: Setup (أسبوع)
- Add Riverpod dependencies
- BLoC و Riverpod يشتغلوا مع بعض
- Migration أول BLoC بسيط (proof of concept)

Phase 2: New Features Only (شهر)
- كل feature جديد بـ Riverpod
- BLoC القديم زي ما هو

Phase 3: High-Priority Migration (شهرين)
- الـ screens الأكثر استخداماً
- الـ BLoCs اللي فيها bugs
- المناطق اللي محتاجة refactoring

Phase 4: Remaining Migration (حسب الوقت)
- باقي BLoCs بالراحة
- Testing مستمر

Phase 5: Cleanup (أسبوع)
- شيل BLoC dependencies
- Finalize documentation
```

<div dir="rtl">

---

## 🔄 تحويل Patterns

### Pattern 1: Simple Cubit → StateProvider

</div>

```dart
// ==========================================
// Before (Cubit)
// ==========================================
class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);

  void increment() => emit(state + 1);
  void decrement() => emit(state - 1);
}

// Provide
BlocProvider(
  create: (_) => CounterCubit(),
  child: MyApp(),
);

// Use
BlocBuilder<CounterCubit, int>(
  builder: (context, state) {
    return Text('$state');
  },
);

// ==========================================
// After (Riverpod)
// ==========================================
final counterProvider = StateProvider<int>((ref) => 0);

// No provider setup needed

// Use
Consumer(
  builder: (context, ref, child) {
    final count = ref.watch(counterProvider);
    return Text('$count');
  },
);

// Riverpod classic syntax
class Counter3Notifier extends Notifier<int> {
  @override
  int build() => 0;

  void increment() => state++;
  void decrement() => state--;
}

final counter3Provider = NotifierProvider<Counter3Notifier, int>(
  () => Counter3Notifier(),
);
```

<div dir="rtl">

### Pattern 2: Complex Cubit → StateNotifierProvider

</div>

```dart
// ==========================================
// Before (Cubit)
// ==========================================
class TodosState {
  final List<Todo> todos;
  final bool isLoading;
  final String? error;

  TodosState({
    required this.todos,
    this.isLoading = false,
    this.error,
  });

  TodosState copyWith({
    List<Todo>? todos,
    bool? isLoading,
    String? error,
  }) {
    return TodosState(
      todos: todos ?? this.todos,
      isLoading: isLoading ?? this.isLoading,
      error: error ?? this.error,
    );
  }
}

class TodosCubit extends Cubit<TodosState> {
  final TodosRepository repository;

  TodosCubit(this.repository) : super(TodosState(todos: []));

  Future<void> loadTodos() async {
    emit(state.copyWith(isLoading: true, error: null));

    try {
      final todos = await repository.getTodos();
      emit(state.copyWith(todos: todos, isLoading: false));
    } catch (e) {
      emit(state.copyWith(error: e.toString(), isLoading: false));
    }
  }

  void addTodo(Todo todo) {
    emit(state.copyWith(todos: [...state.todos, todo]));
  }
}

// ==========================================
// After (Riverpod)
// ==========================================
@freezed
class TodosState with _$TodosState {
  const factory TodosState({
    required List<Todo> todos,
    @Default(false) bool isLoading,
    String? error,
  }) = _TodosState;
}

class TodosNotifier extends Notifier<TodosState> {
  @override
  TodosState build() {
    return const TodosState(todos: []);
  }

  Future<void> loadTodos() async {
    state = state.copyWith(isLoading: true, error: null);

    try {
      final todos = await ref.read(todosRepositoryProvider).getTodos();
      state = state.copyWith(todos: todos, isLoading: false);
    } catch (e) {
      state = state.copyWith(error: e.toString(), isLoading: false);
    }
  }

  void addTodo(Todo todo) {
    state = state.copyWith(todos: [...state.todos, todo]);
  }
}

final todosProvider = NotifierProvider<TodosNotifier, TodosState>(
  () => TodosNotifier(),
);
```

<div dir="rtl">

### Pattern 3: BLoC with Events → Notifier

</div>

```dart
// ==========================================
// Before (BLoC)
// ==========================================

// Events
abstract class LoginEvent {}

class LoginEmailChanged extends LoginEvent {
  final String email;
  LoginEmailChanged(this.email);
}

class LoginPasswordChanged extends LoginEvent {
  final String password;
  LoginPasswordChanged(this.password);
}

class LoginSubmitted extends LoginEvent {}

// States
abstract class LoginState {}

class LoginInitial extends LoginState {}

class LoginInProgress extends LoginState {
  final String email;
  final String password;
  LoginInProgress(this.email, this.password);
}

class LoginLoading extends LoginState {}

class LoginSuccess extends LoginState {
  final User user;
  LoginSuccess(this.user);
}

class LoginFailure extends LoginState {
  final String error;
  LoginFailure(this.error);
}

// BLoC
class LoginBloc extends Bloc<LoginEvent, LoginState> {
  final AuthRepository authRepository;

  LoginBloc(this.authRepository) : super(LoginInitial()) {
    on<LoginEmailChanged>(_onEmailChanged);
    on<LoginPasswordChanged>(_onPasswordChanged);
    on<LoginSubmitted>(_onSubmitted);
  }

  void _onEmailChanged(LoginEmailChanged event, Emitter<LoginState> emit) {
    if (state is LoginInProgress) {
      final current = state as LoginInProgress;
      emit(LoginInProgress(event.email, current.password));
    } else {
      emit(LoginInProgress(event.email, ''));
    }
  }

  void _onPasswordChanged(LoginPasswordChanged event, Emitter<LoginState> emit) {
    if (state is LoginInProgress) {
      final current = state as LoginInProgress;
      emit(LoginInProgress(current.email, event.password));
    } else {
      emit(LoginInProgress('', event.password));
    }
  }

  Future<void> _onSubmitted(LoginSubmitted event, Emitter<LoginState> emit) async {
    if (state is! LoginInProgress) return;

    final current = state as LoginInProgress;
    emit(LoginLoading());

    try {
      final user = await authRepository.login(current.email, current.password);
      emit(LoginSuccess(user));
    } on NetworkException {
      emit(LoginFailure('Network error'));
    } on AuthException catch (e) {
      emit(LoginFailure(e.message));
    }
  }
}

// ==========================================
// After (Riverpod) - Much simpler!
// ==========================================

@freezed
class LoginState with _$LoginState {
  const factory LoginState.initial() = _Initial;
  const factory LoginState.loading() = _Loading;
  const factory LoginState.success(User user) = _Success;
  const factory LoginState.failure(String error) = _Failure;
}

class Login2Notifier extends Notifier<LoginState> {
  String _email = '';
  String _password = '';

  @override
  LoginState build() => const LoginState.initial();

  void updateEmail(String email) {
    _email = email;
    // No state change needed for form fields
  }

  void updatePassword(String password) {
    _password = password;
    // No state change needed for form fields
  }

  Future<void> submit() async {
    state = const LoginState.loading();

    try {
      final user = await ref.read(authRepositoryProvider).login(_email, _password);
      state = LoginState.success(user);
    } on NetworkException {
      state = const LoginState.failure('Network error');
    } on AuthException catch (e) {
      state = LoginState.failure(e.message);
    }
  }
}

final login2Provider = NotifierProvider<Login2Notifier, LoginState>(
  () => Login2Notifier(),
);
```

<div dir="rtl">

**الفرق:**
- BLoC: 90+ سطر، 9 classes
- Riverpod: 35 سطر، 1 class + 1 state

---

## 🔧 تحويل Widget Patterns

### BlocBuilder → Consumer/ConsumerWidget

</div>

```dart
// Before
class TodosList extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocBuilder<TodosBloc, TodosState>(
      builder: (context, state) {
        if (state is TodosLoading) {
          return CircularProgressIndicator();
        }

        if (state is TodosLoaded) {
          return ListView.builder(
            itemCount: state.todos.length,
            itemBuilder: (context, index) {
              return TodoTile(state.todos[index]);
            },
          );
        }

        return ErrorWidget('Error');
      },
    );
  }
}

// After (Option 1: Consumer)
class TodosList extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Consumer(
      builder: (context, ref, child) {
        final todosState = ref.watch(todosProvider);

        return todosState.when(
          loading: () => CircularProgressIndicator(),
          loaded: (todos) => ListView.builder(
            itemCount: todos.length,
            itemBuilder: (context, index) => TodoTile(todos[index]),
          ),
          error: (error) => ErrorWidget(error),
        );
      },
    );
  }
}

// After (Option 2: ConsumerWidget - cleaner)
class TodosList extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final todosState = ref.watch(todosProvider);

    return todosState.when(
      loading: () => CircularProgressIndicator(),
      loaded: (todos) => ListView.builder(
        itemCount: todos.length,
        itemBuilder: (context, index) => TodoTile(todos[index]),
      ),
      error: (error) => ErrorWidget(error),
    );
  }
}
```

<div dir="rtl">

### BlocListener → ref.listen

</div>

```dart
// Before
class LoginPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocListener<LoginBloc, LoginState>(
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
      child: BlocBuilder<LoginBloc, LoginState>(
        builder: (context, state) {
          return LoginForm();
        },
      ),
    );
  }
}

// After
class LoginPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Listen for side effects
    ref.listen(loginProvider, (previous, next) {
      next.whenOrNull(
        success: (user) {
          Navigator.pushReplacement(
            context,
            MaterialPageRoute(builder: (_) => HomePage()),
          );
        },
        failure: (error) {
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(content: Text(error)),
          );
        },
      );
    });

    // Build UI
    final loginState = ref.watch(loginProvider);

    return loginState.when(
      initial: () => LoginForm(),
      loading: () => CircularProgressIndicator(),
      success: (_) => SizedBox.shrink(),
      failure: (_) => LoginForm(),
    );
  }
}
```

<div dir="rtl">

### BlocConsumer → ConsumerWidget + ref.listen

</div>

```dart
// Before
BlocConsumer<CounterBloc, int>(
  listener: (context, state) {
    if (state == 10) {
      showDialog(/*...*/);
    }
  },
  builder: (context, state) {
    return Text('$state');
  },
);

// After
class CounterDisplay extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    ref.listen(counterProvider, (previous, next) {
      if (next == 10) {
        showDialog(/*...*/);
      }
    });

    final count = ref.watch(counterProvider);
    return Text('$count');
  }
}
```

<div dir="rtl">

---

## 🏗️ التعايش المؤقت

### Setup للتعايش

</div>

```dart
// pubspec.yaml
dependencies:
  flutter_bloc: ^8.1.0  # Keep BLoC
  flutter_riverpod: ^2.5.0  # Add Riverpod

// main.dart
void main() {
  runApp(
    ProviderScope( // Riverpod
      child: MultiBlocProvider( // BLoC
        providers: [
          // Your BLoC providers
          BlocProvider(create: (_) => UserBloc()),
          BlocProvider(create: (_) => TodosBloc()),
        ],
        child: MyApp(),
      ),
    ),
  );
}
```

<div dir="rtl">

### استخدام BLoC من Riverpod

</div>

```dart
// Access BLoC from Riverpod provider
class SomeFeatureNotifier extends AsyncNotifier<Data> {
  @override
  Future<Data> build() async {
    // Can't directly access BLoC here
    // Use callback or pass as dependency
    return await fetchData();
  }

  Future<Data> fetchData() async {
    await Future.delayed(Duration(seconds: 1));
    return Data();
  }
}

final someFeatureProvider = AsyncNotifierProvider<SomeFeatureNotifier, Data>(
  () => SomeFeatureNotifier(),
);

// Better: Create bridge provider
final userBlocProvider = Provider<UserBloc>((ref) {
  // Get from context somehow
  throw UnimplementedError('Use Provider.of in widget');
});
```

<div dir="rtl">

### استخدام Riverpod من BLoC

</div>

```dart
// Not recommended, but possible
class SomeBloc extends Bloc<SomeEvent, SomeState> {
  final ProviderContainer container;

  SomeBloc(this.container) : super(InitialState()) {
    on<SomeEvent>((event, emit) {
      // Read Riverpod provider
      final data = container.read(someProvider);
      emit(NewState(data));
    });
  }
}

// Provide container
final container = ProviderContainer();

BlocProvider(
  create: (_) => SomeBloc(container),
  child: MyApp(),
);
```

<div dir="rtl">

**ملاحظة:** التعايش ده مؤقت فقط - الهدف الانتقال الكامل لـ Riverpod.

---

## 📋 Migration Checklist

</div>

```
Migration Checklist:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Planning:
□ حصر كل BLoCs/Cubits
□ تقييم complexity كل واحد
□ تحديد أولويات Migration
□ تقدير الوقت المطلوب

Setup:
□ إضافة Riverpod dependencies
□ Setup ProviderScope
□ BLoC و Riverpod يشتغلوا مع بعض

Conversion:
□ ابدأ بأبسط Cubit
□ حول Events لـ methods
□ حول States لـ Freezed/sealed classes
□ استخدم code generation
□ Test بعد كل conversion

Widgets:
□ BlocBuilder → ConsumerWidget
□ BlocListener → ref.listen
□ BlocConsumer → both
□ context.read<Bloc> → ref.read(provider)

Testing:
□ Unit tests للـ providers
□ Widget tests
□ Integration tests
□ Manual testing

Cleanup:
□ شيل BLoC dependencies
□ شيل event classes
□ نضف imports
□ Update documentation
```

<div dir="rtl">

---

## ⚠️ مشاكل شائعة

### مشكلة 1: Event Tracking Loss

</div>

```dart
// Problem: Lost event tracking from BLoC

// Solution 1: Add logging
class Todos2Notifier extends Notifier<TodosState> {
  @override
  TodosState build() => TodosState.initial();

  void addTodo(Todo todo) {
    // Log the action
    logger.info('Adding todo: ${todo.title}');
    analytics.logEvent('todo_added');

    state = state.copyWith(todos: [...state.todos, todo]);
  }
}

final todos2Provider = NotifierProvider<Todos2Notifier, TodosState>(
  () => Todos2Notifier(),
);

// Solution 2: Use BlocObserver equivalent
class RiverpodObserver extends ProviderObserver {
  @override
  void didUpdateProvider(
    ProviderBase provider,
    Object? previousValue,
    Object? newValue,
    ProviderContainer container,
  ) {
    logger.info('Provider ${provider.name} changed from $previousValue to $newValue');
  }
}

// Setup
ProviderScope(
  observers: [RiverpodObserver()],
  child: MyApp(),
);
```

<div dir="rtl">

### مشكلة 2: Complex Event Transformations

</div>

```dart
// Problem: BLoC had debounce/throttle on events

// BLoC Before
on<SearchQueryChanged>(
  _onQueryChanged,
  transformer: debounce(Duration(milliseconds: 300)),
);

// Riverpod Solution
class Search2Notifier extends Notifier<List<Result>> {
  Timer? _debounceTimer;

  @override
  List<Result> build() {
    ref.onDispose(() {
      _debounceTimer?.cancel();
    });
    return [];
  }

  void search(String query) {
    _debounceTimer?.cancel();
    _debounceTimer = Timer(Duration(milliseconds: 300), () {
      _performSearch(query);
    });
  }

  Future<void> _performSearch(String query) async {
    final results = await api.search(query);
    state = results;
  }
}

final search2Provider = NotifierProvider<Search2Notifier, List<Result>>(
  () => Search2Notifier(),
);
```

<div dir="rtl">

---

## 🎯 الخلاصة

### لو هتهاجر:

</div>

```
✅ افهم الفرق في الفلسفة:
   BLoC: Event-driven, strict separation
   Riverpod: Reactive, flexible

✅ ابدأ تدريجي:
   - أبسط Cubit أولاً
   - New features بـ Riverpod
   - Complex BLoCs في الآخر

✅ استخدم code generation:
   - أقل boilerplate
   - Type-safe
   - أسهل في الصيانة

✅ Testing مستمر:
   - بعد كل conversion
   - مفيش regression

✅ Documentation:
   - وثق القرارات
   - ساعد الفريق
```

<div dir="rtl">

### لو مش متأكد:

</div>

```
جرب Migration لـ feature واحدة صغيرة:
- شوف الفرق في الكود
- قيّم الـ DX
- اسأل الفريق
- قرر بعدها
```

<div dir="rtl">

---

## 🔗 الخطوة الجاية

دلوقتي بعد ما فهمت Migration من BLoC، وقت نبدأ نتعلم Riverpod من الصفر:
- **القسم 03: أساسيات Riverpod**
- **القسم 04: المفاهيم الأساسية**
- **القسم 05: أنواع Providers**

---

## 📚 المصادر

- [Riverpod Documentation](https://riverpod.dev)
- [BLoC Library](https://bloclibrary.dev)
- [Migration Discussions](https://github.com/rrousselGit/riverpod/discussions)

---

## ✅ تأكد إنك فهمت

- [ ] تعرف امتى Migration مفيد؟
- [ ] فاهم إزاي تحول BLoC/Cubit لـ Riverpod؟
- [ ] تقدر تدير codebase مختلط؟
- [ ] جاهز تبدأ تتعلم Riverpod؟

**جاهز نبدأ Riverpod من الصفر؟** 🚀

</div>
