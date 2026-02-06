<div dir="rtl">

# دليل الانتقال من Provider إلى Riverpod

**المستوى**: 🟡 متوسط

## 📌 المفاهيم الأساسية

في الملف ده هنتكلم عن:
- خطة الانتقال التدريجي من Provider لـ Riverpod
- تحويل كل نوع من Provider types
- الحالات الشائعة والحلول
- نصائح لانتقال سلس

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تعمل migration بثقة من Provider لـ Riverpod
- تحول أي Provider code لـ Riverpod equivalent
- تدير migration تدريجي بدون مشاكل
- تتجنب الأخطاء الشائعة

---

## 🗺️ خطة الانتقال

### الخطوة 1: التقييم

قبل ما تبدأ، قيّم المشروع:

</div>

```
أسئلة التقييم:
━━━━━━━━━━━━━━━━━━
1. حجم المشروع؟
   - صغير (< 10 screens) → Migration سهل
   - متوسط (10-50 screens) → Migration تدريجي
   - كبير (50+ screens) → Migration على مراحل

2. عدد Providers؟
   - قليل (< 10) → يوم واحد
   - متوسط (10-30) → أسبوع
   - كتير (30+) → أسابيع

3. Testing coverage؟
   - عالي → Migration آمن
   - قليل → اعمل tests أول

4. الـ deadline؟
   - مفيش ضغط → Migration كامل
   - فيه deadline → Migration تدريجي
```

<div dir="rtl">

### الخطوة 2: Setup

أضف Riverpod للمشروع (يشتغل مع Provider!):

</div>

```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter

  # Keep Provider (for gradual migration)
  provider: ^6.0.0

  # Add Riverpod
  flutter_riverpod: ^2.5.0

dev_dependencies:
  # Optional but recommended
  riverpod_generator: ^2.4.0
  build_runner: ^2.4.0
```

<div dir="rtl">

### الخطوة 3: Wrap App

أضف ProviderScope (يشتغل مع Provider!):

</div>

```dart
// Before
void main() {
  runApp(
    MultiProvider(
      providers: [
        // Your providers
      ],
      child: MyApp(),
    ),
  );
}

// After (both work together!)
void main() {
  runApp(
    ProviderScope( // Add Riverpod
      child: MultiProvider( // Keep Provider
        providers: [
          // Your providers (still working!)
        ],
        child: MyApp(),
      ),
    ),
  );
}
```

<div dir="rtl">

---

## 🔄 تحويل Provider Types

### النوع 1: Provider (قيمة ثابتة)

</div>

```dart
// ==========================================
// Provider → Riverpod
// ==========================================

// Before (Provider)
final nameProvider = Provider<String>((ref) {
  return 'Ahmed';
});

// After (Riverpod) - EXACT SAME!
final nameProvider = Provider<String>((ref) {
  return 'Ahmed';
});

// Usage Before
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final name = context.watch<String>();
    return Text(name);
  }
}

// Usage After
class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final name = ref.watch(nameProvider);
    return Text(name);
  }
}
```

<div dir="rtl">

**التغييرات:**
- Provider definition: نفسه!
- Widget: `StatelessWidget` → `ConsumerWidget`
- Access: `context.watch<T>()` → `ref.watch(provider)`

### النوع 2: ChangeNotifierProvider

</div>

```dart
// ==========================================
// ChangeNotifierProvider → StateNotifierProvider
// ==========================================

// Before (Provider)
class CounterNotifier extends ChangeNotifier {
  int _count = 0;

  int get count => _count;

  void increment() {
    _count++;
    notifyListeners();
  }
}

final counterProvider = ChangeNotifierProvider((ref) {
  return CounterNotifier();
});

// After (Riverpod) - Better approach
class CounterNotifier extends Notifier<int> {
  @override
  int build() => 0;

  void increment() {
    state = state + 1; // No notifyListeners needed!
  }
}

final counterProvider = NotifierProvider<CounterNotifier, int>(
  () => CounterNotifier(),
);

// Or even simpler with StateProvider
final counterProvider = StateProvider<int>((ref) => 0);

// Usage Before
class CounterDisplay extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final counter = context.watch<CounterNotifier>();
    return Text('${counter.count}');
  }
}

// Usage After (StateNotifierProvider)
class CounterDisplay extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);
    return Text('$count');
  }
}

// Or (StateProvider)
class CounterDisplay extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);
    return Text('$count');
  }
}
```

<div dir="rtl">

**التغييرات الرئيسية:**
- `ChangeNotifier` → `StateNotifier<T>` (أو `StateProvider` للبسيط)
- مفيش `notifyListeners()` - تلقائي!
- الـ state immutable (بتبدله مش بتعدله)

### النوع 3: FutureProvider

</div>

```dart
// ==========================================
// FutureProvider → FutureProvider
// ==========================================

// Before (Provider)
final userProvider = FutureProvider<User>((ref) async {
  return await api.getUser();
});

// After (Riverpod) - EXACT SAME!
final userProvider = FutureProvider<User>((ref) async {
  return await api.getUser();
});

// Usage Before
class UserProfile extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final userAsync = context.watch<AsyncValue<User>>();

    return userAsync.when(
      data: (user) => Text(user.name),
      loading: () => CircularProgressIndicator(),
      error: (error, stack) => Text('Error: $error'),
    );
  }
}

// Usage After - ALMOST SAME
class UserProfile extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider);

    return userAsync.when(
      data: (user) => Text(user.name),
      loading: () => CircularProgressIndicator(),
      error: (error, stack) => Text('Error: $error'),
    );
  }
}
```

<div dir="rtl">

**التغييرات:**
- Provider definition: نفسه!
- Widget: `StatelessWidget` → `ConsumerWidget`
- Access: `context.watch` → `ref.watch`

### النوع 4: StreamProvider

</div>

```dart
// ==========================================
// StreamProvider → StreamProvider
// ==========================================

// Before (Provider)
final messagesProvider = StreamProvider<List<Message>>((ref) {
  return chatService.messagesStream();
});

// After (Riverpod) - Add autoDispose!
final messagesProvider = StreamProvider.autoDispose<List<Message>>((ref) {
  return chatService.messagesStream();
});

// Usage - SAME!
class MessagesList extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final messagesAsync = ref.watch(messagesProvider);

    return messagesAsync.when(
      data: (messages) => ListView.builder(/*...*/),
      loading: () => CircularProgressIndicator(),
      error: (error, stack) => ErrorWidget(error),
    );
  }
}
```

<div dir="rtl">

**التغييرات:**
- أضف `.autoDispose` لتنضيف تلقائي!

### النوع 5: StateProvider (قيمة بسيطة)

</div>

```dart
// ==========================================
// StateProvider → StateProvider
// ==========================================

// Before (Provider)
final counterProvider = StateProvider<int>((ref) => 0);

// After (Riverpod) - EXACT SAME!
final counterProvider = StateProvider<int>((ref) => 0);

// Usage Before
class CounterDisplay extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final counter = context.watch<StateController<int>>();
    return Text('${counter.state}');
  }
}

// Usage After
class CounterDisplay extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final counter = ref.watch(counterProvider);
    return Text('$counter');
  }
}
```

<div dir="rtl">

**التغييرات:**
- Provider definition: نفسه!
- Access أبسط: مباشرة القيمة (مش `state`)

---

## 🔧 تحويل Widget Types

### حالة 1: StatelessWidget

</div>

```dart
// Before
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final user = context.watch<User>();
    return Text(user.name);
  }
}

// After
class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final user = ref.watch(userProvider);
    return Text(user.name);
  }
}
```

<div dir="rtl">

### حالة 2: StatefulWidget

</div>

```dart
// Before
class MyWidget extends StatefulWidget {
  @override
  State<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  @override
  Widget build(BuildContext context) {
    final user = context.watch<User>();
    return Text(user.name);
  }
}

// After (Option 1: ConsumerStatefulWidget)
class MyWidget extends ConsumerStatefulWidget {
  @override
  ConsumerState<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends ConsumerState<MyWidget> {
  @override
  Widget build(BuildContext context) {
    final user = ref.watch(userProvider);
    return Text(user.name);
  }
}

// After (Option 2: Consumer widget)
class MyWidget extends StatefulWidget {
  @override
  State<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  @override
  Widget build(BuildContext context) {
    return Consumer(
      builder: (context, ref, child) {
        final user = ref.watch(userProvider);
        return Text(user.name);
      },
    );
  }
}
```

<div dir="rtl">

### حالة 3: Consumer widget

</div>

```dart
// Before (Provider)
Consumer<CounterNotifier>(
  builder: (context, counter, child) {
    return Text('${counter.count}');
  },
);

// After (Riverpod)
Consumer(
  builder: (context, ref, child) {
    final count = ref.watch(counterProvider);
    return Text('$count');
  },
);
```

<div dir="rtl">

---

## 🎯 حالات شائعة

### حالة 1: Multiple Providers

</div>

```dart
// Before (Provider)
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final user = context.watch<User>();
    final cart = context.watch<Cart>();
    final theme = context.watch<ThemeNotifier>();

    return Column(
      children: [
        Text(user.name),
        Text('${cart.itemCount}'),
        Text('${theme.isDark}'),
      ],
    );
  }
}

// After (Riverpod)
class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final user = ref.watch(userProvider);
    final cart = ref.watch(cartProvider);
    final theme = ref.watch(themeProvider);

    return Column(
      children: [
        Text(user.name),
        Text('${cart.itemCount}'),
        Text('$theme'),
      ],
    );
  }
}
```

<div dir="rtl">

### حالة 2: context.read (لا rebuild)

</div>

```dart
// Before (Provider)
class IncrementButton extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () {
        context.read<CounterNotifier>().increment();
      },
      child: Text('Increment'),
    );
  }
}

// After (Riverpod)
class IncrementButton extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return ElevatedButton(
      onPressed: () {
        ref.read(counterProvider.notifier).increment();
      },
      child: Text('Increment'),
    );
  }
}
```

<div dir="rtl">

### حالة 3: context.select (selective rebuild)

</div>

```dart
// Before (Provider)
class UserName extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final name = context.select<User, String>((user) => user.name);
    return Text(name);
  }
}

// After (Riverpod) - use select
class UserName extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final name = ref.watch(userProvider.select((user) => user.name));
    return Text(name);
  }
}
```

<div dir="rtl">

### حالة 4: ProxyProvider (Dependencies)

</div>

```dart
// Before (Provider) - Complex!
ProxyProvider<ApiService, UserRepository>(
  update: (context, api, previous) {
    return UserRepository(api);
  },
);

ProxyProvider2<Database, ApiService, TodosRepository>(
  update: (context, db, api, previous) {
    return TodosRepository(db, api);
  },
);

// After (Riverpod) - Much simpler!
final apiProvider = Provider((ref) => ApiService());

final userRepositoryProvider = Provider((ref) {
  final api = ref.watch(apiProvider);
  return UserRepository(api);
});

final todosRepositoryProvider = Provider((ref) {
  final db = ref.watch(databaseProvider);
  final api = ref.watch(apiProvider);
  return TodosRepository(db, api);
});
```

<div dir="rtl">

---

## 📝 خطة Migration تدريجية

### الخطوة 1: ابدأ بالبسيط

ابدأ بأبسط providers:

</div>

```dart
// Week 1: Migrate simple providers
final themeProvider = StateProvider<ThemeMode>((ref) => ThemeMode.light);
final localeProvider = StateProvider<Locale>((ref) => Locale('ar'));
```

<div dir="rtl">

### الخطوة 2: Migrate الـ Screens الجديدة

أي screen جديد، اعمله بـ Riverpod:

</div>

```dart
// New feature - use Riverpod from day 1
final newFeatureProvider = NotifierProvider<NewFeatureNotifier, State>(
  () => NewFeatureNotifier(),
);
```

<div dir="rtl">

### الخطوة 3: Migrate حسب الأولوية

ركز على:
1. الـ screens الأكثر استخداماً
2. الـ providers اللي فيها bugs
3. الكود اللي محتاج refactoring

### الخطوة 4: Testing مستمر

بعد كل migration:

</div>

```dart
// Test that old and new work together
testWidgets('Provider and Riverpod coexist', (tester) async {
  await tester.pumpWidget(
    ProviderScope(
      child: MultiProvider(
        providers: [
          // Old Provider providers
        ],
        child: MyApp(),
      ),
    ),
  );

  // Test both systems
});
```

<div dir="rtl">

---

## ⚠️ مشاكل شائعة والحلول

### مشكلة 1: ProviderNotFoundException

</div>

```dart
// ❌ Error: Provider not found
class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final user = ref.watch(userProvider);
    // Error if userProvider not in scope!
  }
}

// ✅ Solution: Make sure ProviderScope wraps everything
void main() {
  runApp(
    ProviderScope( // Must wrap app!
      child: MyApp(),
    ),
  );
}
```

<div dir="rtl">

### مشكلة 2: State not updating

</div>

```dart
// ❌ Wrong: Mutating state directly
final listProvider = StateProvider<List<int>>((ref) => []);

ref.read(listProvider.notifier).state.add(1); // Won't rebuild!

// ✅ Correct: Replace state
final list = ref.read(listProvider);
ref.read(listProvider.notifier).state = [...list, 1];
```

<div dir="rtl">

### مشكلة 3: Memory leaks

</div>

```dart
// ❌ Without autoDispose
final chatProvider = StreamProvider<List<Message>>((ref) {
  return chatService.messagesStream();
});
// Stream keeps running!

// ✅ With autoDispose
final chatProvider = StreamProvider.autoDispose<List<Message>>((ref) {
  return chatService.messagesStream();
});
// Automatically disposed when not watched!
```

<div dir="rtl">

---

## ✅ Checklist للـ Migration

</div>

```
Migration Checklist:
━━━━━━━━━━━━━━━━━━━━━━━

Setup:
□ أضفت flutter_riverpod للـ dependencies
□ لفيت الـ app بـ ProviderScope
□ Provider و Riverpod شغالين مع بعض

Providers:
□ حولت Provider → Provider (نفسه)
□ حولت ChangeNotifierProvider → StateNotifierProvider
□ حولت FutureProvider → FutureProvider
□ حولت StreamProvider → StreamProvider.autoDispose
□ حولت ProxyProvider → Provider with ref.watch

Widgets:
□ حولت StatelessWidget → ConsumerWidget
□ حولت StatefulWidget → ConsumerStatefulWidget
□ حولت context.watch → ref.watch
□ حولت context.read → ref.read
□ حولت context.select → ref.watch(...select)

Testing:
□ الـ unit tests شغالة
□ الـ widget tests شغالة
□ الـ integration tests شغالة
□ مفيش memory leaks

Cleanup:
□ شلت الـ Provider dependencies
□ شلت MultiProvider wrapper
□ نضفت imports
```

<div dir="rtl">

---

## 🎯 الخلاصة

### Migration سهلة لأن:

```
✅ Riverpod و Provider متشابهين جداً (نفس المطور!)
✅ ممكن يشتغلوا مع بعض
✅ الـ API قريب جداً
✅ Migration تدريجية ممكنة
```

### الفوائد بعد Migration:

```
✅ Compile-time safety
✅ مفيش BuildContext dependency
✅ Auto disposal تلقائي
✅ Dependency injection أسهل
✅ Testing أسهل
✅ Code أنضف وأقل
```

### الخطة الموصى بها:

</div>

```
Week 1: Setup + migrate simple providers
Week 2: Migrate new features only
Week 3: Migrate high-priority screens
Week 4: Migrate remaining screens
Week 5: Testing + cleanup
```

<div dir="rtl">

---

## 🔗 الخطوة الجاية

دلوقتي بعد ما فهمت Migration من Provider، وقت:
- **Decision Tree: متى تستخدم أيه** (الملف الجاي)
- **Performance Benchmarks**
- **أمثلة عملية كاملة**

---

## 📚 المصادر

- [Migrating from Provider - Official Guide](https://riverpod.dev/docs/from_provider/motivation)
- [Provider Package](https://pub.dev/packages/provider)
- [Riverpod Package](https://pub.dev/packages/flutter_riverpod)
- [Migration FAQ](https://riverpod.dev/docs/from_provider/quickstart)

---

## ✅ تأكد إنك فهمت

- [ ] تعرف تحول كل نوع من Provider types؟
- [ ] فاهم إزاي تعمل migration تدريجية؟
- [ ] تقدر تتجنب المشاكل الشائعة؟
- [ ] جاهز تبدأ migration في مشروعك؟

**جاهز للـ Decision Tree؟** 🌳

</div>
