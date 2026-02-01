<div dir="rtl">

# الفرق بين Local State و Global State

**المستوى**: 🟢 مبتدئ

## 📌 المفاهيم الأساسية

في الملف ده هنفهم:
- إيه هو Local State (الحالة المحلية)
- إيه هو Global State (الحالة العامة)
- الفرق الجوهري بينهم
- امتى نستخدم كل واحد

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تفرق بين State المحلي والعام
- تحدد أنسب نوع لكل حالة
- تتجنب الأخطاء الشائعة
- تصمم State بشكل أفضل

---

## 🏠 Local State (الحالة المحلية)

### التعريف

**Local State** هو state خاص بـ Widget واحد بس، ومحدش غيره محتاجه.

### الخصائص

- ✅ خاص بـ Widget واحد فقط
- ✅ ما بيتشاركش مع Widgets تانية
- ✅ بيتدمر لما الـ Widget يتشال
- ✅ سهل في الإدارة

### أمثلة شائعة

</div>

```dart
// Example 1: TextField state
class SearchField extends StatefulWidget {
  @override
  State<SearchField> createState() => _SearchFieldState();
}

class _SearchFieldState extends State<SearchField> {
  // Local State: Only this widget needs it
  final TextEditingController _controller = TextEditingController();
  bool _isFocused = false;

  @override
  Widget build(BuildContext context) {
    return TextField(
      controller: _controller,
      onTap: () => setState(() => _isFocused = true),
      onSubmitted: (_) => setState(() => _isFocused = false),
      decoration: InputDecoration(
        hintText: _isFocused ? 'Type to search...' : 'Search',
      ),
    );
  }
}
```

<div dir="rtl">

**التحليل:**
- الـ `_controller` و `_isFocused` دول local state
- محدش غير الـ Widget ده محتاجهم
- لما الـ Widget يتشال، الـ state يروح معاه

---

</div>

```dart
// Example 2: Password visibility toggle
class PasswordField extends StatefulWidget {
  @override
  State<PasswordField> createState() => _PasswordFieldState();
}

class _PasswordFieldState extends State<PasswordField> {
  // Local State: Just for showing/hiding password
  bool _isPasswordVisible = false;

  @override
  Widget build(BuildContext context) {
    return TextField(
      obscureText: !_isPasswordVisible,
      decoration: InputDecoration(
        suffixIcon: IconButton(
          icon: Icon(
            _isPasswordVisible ? Icons.visibility_off : Icons.visibility,
          ),
          onPressed: () {
            setState(() => _isPasswordVisible = !_isPasswordVisible);
          },
        ),
      ),
    );
  }
}
```

<div dir="rtl">

**التحليل:**
- حالة إظهار/إخفاء الباسورد local
- مفيش Widget تاني محتاج يعرف الباسورد ظاهر ولا مخفي
- لو الـ Widget اتقفل وفتح تاني، يرجع للحالة الافتراضية (مخفي)

---

## 🌍 Global State (الحالة العامة)

### التعريف

**Global State** هو state محتاج أكتر من Widget يوصل له، أو بيأثر على أجزاء كتير من التطبيق.

### الخصائص

- ✅ متاح لعدة Widgets
- ✅ بيتشارك بين شاشات مختلفة
- ✅ بيفضل موجود حتى لو الـ Widget اتشال
- ✅ محتاج State Management

### أمثلة شائعة

</div>

```dart
// Example 1: User authentication
// Multiple screens need to know if user is logged in
class AuthState {
  final User? currentUser;
  final bool isAuthenticated;
  final String? token;

  // Needed by: AppBar, ProfilePage, SettingsPage, etc.
}
```

<div dir="rtl">

**ليه Global؟**
- الـ AppBar محتاج يعرض اسم المستخدم
- الـ ProfilePage محتاج بيانات المستخدم
- كل الشاشات محتاجة تعرف المستخدم login ولا لأ

---

</div>

```dart
// Example 2: Shopping cart
// Multiple screens need to access and modify cart
class CartState {
  final List<CartItem> items;
  final double totalPrice;
  final int itemCount;

  // Needed by: ProductPage, CartPage, CheckoutPage, AppBar badge
}
```

<div dir="rtl">

**ليه Global؟**
- صفحة المنتج محتاجة تضيف للعربة
- صفحة العربة محتاجة تعرض المحتويات
- الـ AppBar محتاج يعرض عدد العناصر
- صفحة الدفع محتاجة المجموع

---

## ⚖️ المقارنة التفصيلية

### جدول المقارنة

| الخاصية | Local State | Global State |
|---------|-------------|--------------|
| **النطاق** | Widget واحد | عدة Widgets أو التطبيق كله |
| **العمر** | يموت مع الـ Widget | يعيش طول ما التطبيق شغال |
| **المشاركة** | مش بيتشارك | بيتشارك بين Widgets كتير |
| **الإدارة** | `setState` عادي | محتاج State Management |
| **الأمثلة** | TextField value, checkbox | User data, cart, theme |
| **الصعوبة** | سهل | محتاج تنظيم |

---

## 🎨 أمثلة عملية مقارنة

### مثال: تطبيق Todo List

</div>

```dart
// Local State: Checkbox for single todo item
class TodoItem extends StatefulWidget {
  final Todo todo;

  @override
  State<TodoItem> createState() => _TodoItemState();
}

class _TodoItemState extends State<TodoItem> {
  // Local State: Is this specific item expanded?
  bool _isExpanded = false;

  @override
  Widget build(BuildContext context) {
    return Card(
      child: Column(
        children: [
          ListTile(
            title: Text(widget.todo.title),
            trailing: IconButton(
              icon: Icon(_isExpanded ? Icons.expand_less : Icons.expand_more),
              onPressed: () => setState(() => _isExpanded = !_isExpanded),
            ),
          ),
          if (_isExpanded)
            Padding(
              padding: EdgeInsets.all(16),
              child: Text(widget.todo.description),
            ),
        ],
      ),
    );
  }
}

// Global State: All todos in the app
class TodosState {
  final List<Todo> todos;
  final int completedCount;
  final TodoFilter currentFilter;

  // Needed by: TodoList, Statistics, Filters, AppBar
}
```

<div dir="rtl">

**التحليل:**
- **Local**: حالة expand/collapse لكل item (خاص بالـ item نفسه)
- **Global**: قائمة الـ todos كلها (محتاجة شاشات كتير)

---

### مثال: تطبيق ملاحظات

</div>

```dart
// Local State: Text editing in single note
class NoteEditor extends StatefulWidget {
  @override
  State<NoteEditor> createState() => _NoteEditorState();
}

class _NoteEditorState extends State<NoteEditor> {
  // Local State: Current text being edited
  final TextEditingController _titleController = TextEditingController();
  final TextEditingController _bodyController = TextEditingController();
  bool _isBold = false;
  bool _isItalic = false;

  // These are only needed while editing this specific note
}

// Global State: All saved notes
class NotesState {
  final List<Note> allNotes;
  final List<Note> favoriteNotes;
  final String searchQuery;
  final SortOption sortBy;

  // Needed by: NotesList, Search, Favorites, Statistics
}
```

<div dir="rtl">

**التحليل:**
- **Local**: النص اللي بيتكتب دلوقتي، formatting options
- **Global**: كل الملاحظات المحفوظة، البحث، الترتيب

---

## 🤔 إزاي تقرر: Local ولا Global؟

### أسئلة المساعدة

اسأل نفسك الأسئلة دي:

</div>

```
1. هل أكتر من Widget محتاج الـ State ده؟
   ✅ نعم → Global
   ❌ لا → Local

2. هل لو الـ Widget اتشال، محتاج الـ State يفضل موجود؟
   ✅ نعم → Global
   ❌ لا → Local

3. هل الـ State ده بيتأثر بأكشن من Widget تاني؟
   ✅ نعم → Global
   ❌ لا → Local

4. هل محتاج تحفظ الـ State ده لما التطبيق يتقفل ويفتح تاني؟
   ✅ نعم → Global (+ Persistence)
   ❌ لا → Local
```

<div dir="rtl">

---

## 💡 أمثلة من تطبيقات حقيقية

### تطبيق Twitter

</div>

```dart
class TwitterState {
  // ========== Global State ==========
  User currentUser;                    // Logged in user
  List<Tweet> timeline;                // User's timeline
  List<User> following;                // Who user follows
  int unreadNotifications;             // Notification count
  ThemeMode themeMode;                 // Dark/Light theme

  // ========== Local State (Examples) ==========
  // In TweetCard widget:
  bool isExpanded;                     // Show full tweet text?
  bool showReplies;                    // Show replies?

  // In ComposeTweet widget:
  String draftText;                    // Text being typed
  List<File> attachedImages;           // Images being uploaded

  // In ProfilePage widget:
  int selectedTabIndex;                // Tweets/Replies/Likes tab
}
```

<div dir="rtl">

---

### تطبيق Spotify

</div>

```dart
class SpotifyState {
  // ========== Global State ==========
  User currentUser;                    // Logged in user
  Song? currentlyPlaying;              // Now playing song
  List<Song> queue;                    // Playback queue
  bool isPlaying;                      // Play/Pause state
  PlaybackMode mode;                   // Shuffle/Repeat

  // ========== Local State (Examples) ==========
  // In SearchBar widget:
  String searchText;                   // Search input
  bool isFocused;                      // Is field focused?

  // In SongList widget:
  int? expandedSongId;                 // Which song details shown
  bool isScrolling;                    // Auto-hide controls

  // In VolumeSlider widget:
  double tempVolume;                   // Volume while dragging
}
```

<div dir="rtl">

**الملاحظة المهمة:**
- الأغنية الحالية (currently playing) Global - كل التطبيق محتاجها
- النص في البحث Local - بس الـ SearchBar محتاجه

---

## ⚠️ أخطاء شائعة

### خطأ 1: جعل كل شيء Global

</div>

```dart
// ❌ BAD: Making everything global
class AppState {
  bool isPasswordVisible;        // Should be local to PasswordField
  bool isMenuOpen;               // Should be local to Menu widget
  String searchText;             // Should be local to SearchBar
  int selectedTabIndex;          // Depends: local or global?
  User currentUser;              // ✅ This is actually global
}
```

<div dir="rtl">

**المشكلة:**
- الـ state بيكبر من غير داعي
- صعب في الإدارة
- performance بيبقى أسوأ (كل حاجة بتتحدث مع كل التغييرات)

**الحل الصحيح:**

</div>

```dart
// ✅ GOOD: Only truly global state
class AppState {
  User currentUser;              // Really needs to be global
  ThemeMode theme;               // Really needs to be global
  List<Notification> notifications;  // Really needs to be global
}

// Local states stay in their widgets
class PasswordField extends StatefulWidget {
  // _isPasswordVisible stays here (local)
}

class SearchBar extends StatefulWidget {
  // searchText stays here (local)
}
```

<div dir="rtl">

---

### خطأ 2: جعل كل شيء Local

</div>

```dart
// ❌ BAD: Passing global data down through many levels
class HomePage extends StatefulWidget {
  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  User? currentUser;  // Should be global!

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        AppBar(user: currentUser),        // Passing down
        ProfileSection(user: currentUser), // Passing down
        SettingsButton(user: currentUser), // Passing down
      ],
    );
  }
}
```

<div dir="rtl">

**المشكلة:**
- الـ prop drilling (تمرير البيانات عبر مستويات كتير)
- صعب في الصيانة
- كل Widget في المنتصف لازم يعرف عن User حتى لو مش محتاجه

**الحل الصحيح:**

</div>

```dart
// ✅ GOOD: Make it global
final userProvider = StateProvider<User?>((ref) => null);

class HomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        AppBar(),         // Gets user from provider
        ProfileSection(), // Gets user from provider
        SettingsButton(), // Gets user from provider
      ],
    );
  }
}
```

<div dir="rtl">

---

## 🎯 القواعد الذهبية

### قاعدة 1: ابدأ Local

```
دايماً ابدأ بـ Local State، ولو لقيت Widgets تانية محتاجاه، حوله لـ Global.
```

### قاعدة 2: اسأل نفسك

```
"لو الـ Widget ده اتشال واترجع تاني، محتاج أفتكر الـ State ده؟"
✅ نعم → Global
❌ لا → Local
```

### قاعدة 3: البساطة أولاً

```
Local State أسهل في الإدارة، متستخدمش Global إلا لو فعلاً محتاج.
```

---

## 📊 ملخص سريع

### Local State:
- ✅ خاص بـ Widget واحد
- ✅ استخدم `setState`
- ✅ أمثلة: checkbox, text input, expand/collapse
- ✅ يموت مع الـ Widget

### Global State:
- ✅ مشترك بين Widgets كتير
- ✅ محتاج State Management (Riverpod مثلاً)
- ✅ أمثلة: user data, cart, theme
- ✅ يعيش طول ما التطبيق شغال

---

## 🔗 الخطوة الجاية

دلوقتي بعد ما فهمت الفرق بين Local و Global State، وقت نفهم:
- **ليه محتاجين State Management؟**
- **المشاكل اللي بيحلها**
- **أنواع الحلول المتاحة**

---

## 📚 المصادر

- [Ephemeral vs App State](https://docs.flutter.dev/data-and-backend/state-mgmt/ephemeral-vs-app)
- [State Management Options](https://docs.flutter.dev/data-and-backend/state-mgmt/options)

---

## ✅ تأكد إنك فهمت

- [ ] تعرف الفرق بين Local و Global State؟
- [ ] تقدر تحدد امتى تستخدم كل واحد؟
- [ ] فاهم ليه مش كل حاجة تكون Global؟
- [ ] تقدر تتجنب الأخطاء الشائعة؟

**جاهز للملف الجاي؟** 🚀

</div>
