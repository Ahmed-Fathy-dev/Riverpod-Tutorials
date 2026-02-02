<div dir="rtl">

# نظرة عامة على Code Generation

**المستوى**: 🟢 متوسط

## 📌 المفاهيم الأساسية

في Section ده هنتعلم:
- إيه هو Code Generation في Riverpod
- ليه نستخدم Code Generation بدل Classic Syntax
- إزاي نعمل Setup للـ build_runner و riverpod_generator
- المقارنة بين الطريقتين (Classic vs Generated)
- إزاي نعمل Migration من Classic لـ Generated

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تفهم الفرق بين Classic Syntax و Code Generation
- تعرف متى تستخدم كل طريقة
- تعمل Setup للـ code generation في مشروعك
- تحول الـ providers من classic لـ generated syntax
- تفهم الـ `@riverpod` annotation وإزاي بيشتغل

---

## 📖 إيه هو Code Generation؟

**Code Generation** في Riverpod معناه إنك بتكتب كود بسيط، والـ **build_runner** بيولد (يعمل generate) للـ provider code تلقائياً.

### المقارنة السريعة:

</div>

```dart
// ❌ Classic Syntax (Sections 03-05) - Manual
class CounterNotifier extends Notifier<int> {
  @override
  int build() => 0;

  void increment() => state++;
}

// You manually declare the provider
final counterProvider = NotifierProvider<CounterNotifier, int>(
  () => CounterNotifier(),
);

// ✅ Code Generation (Section 06+) - Auto-generated
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'counter.g.dart';  // Generated file

@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() => state++;
}

// The provider is AUTO-GENERATED in counter.g.dart!
// You just use: ref.watch(counterProvider)
```

<div dir="rtl">

**الفرق الأساسي:**
- **Classic:** انت بتكتب الـ provider declaration يدوياً
- **Generated:** الـ build_runner بيولد الـ provider تلقائياً

---

## 🌟 ليه نستخدم Code Generation؟

### ميزة 1: أقل Boilerplate

</div>

```dart
// Classic: 10 lines of boilerplate
class TodosNotifier extends AsyncNotifier<List<Todo>> {
  @override
  Future<List<Todo>> build() async {
    return await fetchTodos();
  }
}

final todosProvider = AsyncNotifierProvider<TodosNotifier, List<Todo>>(
  () => TodosNotifier(),
);

// Generated: 7 lines only! (30% less code)
@riverpod
class Todos extends _$Todos {
  @override
  Future<List<Todo>> build() async {
    return await fetchTodos();
  }
}
// Provider is auto-generated!
```

<div dir="rtl">

### ميزة 2: Type Safety أفضل

الـ code generation بيولد types دقيقة جداً، فالـ IDE بيساعدك أكتر.

### ميزة 3: Parameters بدون Family!

</div>

```dart
// ❌ Classic: Need .family modifier
final todoProvider = FutureProvider.family<Todo, String>((ref, id) async {
  return await api.getTodo(id);
});

// Usage
final todo = ref.watch(todoProvider('123'));

// ✅ Generated: Just use parameters directly!
@riverpod
Future<Todo> todo(TodoRef ref, String id) async {
  return await api.getTodo(id);
}

// Usage (same!)
final todo = ref.watch(todoProvider('123'));
```

<div dir="rtl">

### ميزة 4: AutoDispose by Default

</div>

```dart
// Classic: Need to explicitly add .autoDispose
final messagesProvider = StreamProvider.autoDispose<List<Message>>((ref) {
  return chatService.messagesStream();
});

// Generated: AutoDispose is DEFAULT!
@riverpod
Stream<List<Message>> messages(MessagesRef ref) {
  return chatService.messagesStream();
}
// Automatically disposed when not used!

// Want to keep alive? Just add parameter:
@Riverpod(keepAlive: true)
Stream<List<Message>> messages(MessagesRef ref) {
  return chatService.messagesStream();
}
```

<div dir="rtl">

### ميزة 5: أسهل في الـ Refactoring

لو غيرت اسم الـ function، الـ provider name بيتغير تلقائياً!

---

## 🔄 متى تستخدم إيه؟

### استخدم Classic Syntax لو:
- بتتعلم Riverpod لأول مرة ✅
- عايز تفهم إزاي Riverpod بيشتغل من جوا ✅
- مش عايز تعقيد الـ build_runner setup ✅
- عندك مشروع بسيط جداً ✅

### استخدم Code Generation لو:
- **بتبدأ مشروع جديد** ⭐ (الأفضل!)
- عايز أقل boilerplate ممكن ✅
- محتاج parameters كتير في الـ providers ✅
- المشروع كبير ومحتاج scalability ✅
- عايز أفضل type safety ✅

---

## 📦 الملفات في Section ده

Section 06 فيه:

1. **00-overview.md** (الملف ده) - نظرة عامة
2. **01-setup-guide.md** - Setup خطوة بخطوة
3. **02-basic-syntax.md** - الـ syntax الأساسي
4. **03-notifier-with-codegen.md** - Notifier + AsyncNotifier مع code generation
5. **04-classic-vs-generated.md** - مقارنة تفصيلية
6. **05-migration-guide.md** - إزاي تحول من classic لـ generated
7. **06-common-patterns.md** - Patterns شائعة
8. **07-troubleshooting.md** - حل المشاكل الشائعة

---

## 🎓 نصيحة للمبتدئين

**لو انت لسه بتتعلم Riverpod:**
1. اتعلم Classic Syntax الأول (Sections 03-05) ✅
2. افهم الـ concepts الأساسية ✅
3. بعد كده تعال Section 06 ده ✅
4. استخدم Code Generation في مشاريعك الجديدة ✅

**ليه؟** لأنك لو فهمت Classic Syntax، هتفهم إيه اللي بيحصل وراء الكواليس في Code Generation!

---

## 💡 حقيقة مهمة

**Riverpod 3.0** (النسخة الحالية) بتشجع على استخدام **Code Generation** كـ default approach!

من [الـ documentation الرسمي](https://riverpod.dev/docs/concepts/about_code_generation):
> "Code generation is the recommended way to use Riverpod."

لكن **Classic Syntax ما زال supported بالكامل** - فاختار اللي يناسبك! 🎯

---

## 🔗 الخطوة الجاية

في الملف الجاي هنتعلم إزاي نعمل **Setup كامل** للـ code generation:
- Dependencies اللي محتاجها
- إعداد build_runner
- أول provider بالـ code generation

جاهز؟ يلا نكمل! 🚀

</div>
