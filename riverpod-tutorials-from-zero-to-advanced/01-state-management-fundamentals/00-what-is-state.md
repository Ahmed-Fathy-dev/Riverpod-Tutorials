<div dir="rtl">

# إيه هو State بالظبط؟

**المستوى**: 🟢 مبتدئ

## 📌 المفاهيم الأساسية

في الملف ده هنفهم بدقة:
- تعريف State الصحيح
- أنواع State المختلفة
- الفرق بين State والبيانات العادية
- أمثلة عملية من تطبيقات حقيقية

## 🎯 الهدف

بعد ما تخلص الملف ده، هتقدر:
- تعرّف إيه هو State بدقة
- تفرق بين State والبيانات الثابتة
- تحدد امتى تحتاج State Management
- تفهم ازاي State بيتفاعل مع UI

---

## 🤔 إيه هو State؟

### التعريف الدقيق

**State** هو أي بيانات في التطبيق بتاعك **ممكن تتغير بمرور الوقت** وبيكون ليها **تأثير على شكل أو سلوك التطبيق**.

### تفكيك التعريف

خليني أوضحلك كل جزء:

#### 1. "بيانات في التطبيق"
يعني أي معلومة التطبيق محتفظ بيها - رقم، نص، قائمة، كائن، أي حاجة.

#### 2. "ممكن تتغير بمرور الوقت"
مش ثابتة - بتتعدل، بتزيد، بتنقص، بتتحدث خلال استخدام التطبيق.

#### 3. "ليها تأثير على شكل أو سلوك التطبيق"
لما تتغير، حاجة في UI بتتغير معاها - لون، نص، صورة، شاشة كاملة.

---

## 📱 أمثلة بسيطة من الحياة اليومية

### مثال 1: زر الإعجاب (Like Button)

تخيل زر الإعجاب في فيسبوك:

</div>

```dart
// State: Is the post liked?
bool isLiked = false;

// When user clicks the button
void toggleLike() {
  isLiked = !isLiked; // State changes
  // UI updates: button color changes from grey to blue
}
```

<div dir="rtl">

**التحليل:**
- **البيانات**: `isLiked` (true أو false)
- **التغيير**: من false لـ true (أو العكس)
- **التأثير**: لون الزر والعدد بيتغيروا

---

### مثال 2: عداد بسيط (Counter)

</div>

```dart
// State: Current count value
int count = 0;

// When user clicks increment
void increment() {
  count++; // State changes from 0 to 1, 1 to 2, etc.
  // UI updates: number displayed on screen changes
}
```

<div dir="rtl">

**التحليل:**
- **البيانات**: `count` (عدد صحيح)
- **التغيير**: بيزيد بواحد كل مرة
- **التأثير**: الرقم المعروض على الشاشة بيتحدث

---

### مثال 3: نموذج تسجيل الدخول (Login Form)

</div>

```dart
// State: Form fields and validation
class LoginState {
  String email = '';
  String password = '';
  bool isLoading = false;
  String? errorMessage;

  bool get isValid => email.isNotEmpty && password.length >= 6;
}
```

<div dir="rtl">

**التحليل:**
- **البيانات**: الإيميل، الباسورد، حالة التحميل، رسالة الخطأ
- **التغيير**: كل ما المستخدم يكتب أو يضغط submit
- **التأثير**: تفعيل/تعطيل زر Login، عرض رسالة خطأ، إظهار Loading

---

## 🎨 الفرق بين State والبيانات الثابتة

### البيانات الثابتة (Constants)

دي حاجات **مش بتتغير أبداً** خلال حياة التطبيق:

</div>

```dart
// ❌ NOT State - These never change
const String appName = "My Awesome App";
const String apiBaseUrl = "https://api.example.com";
const Color primaryColor = Colors.blue;
const int maxLoginAttempts = 3;

// Configuration that never changes during app lifetime
const List<String> supportedLanguages = ['ar', 'en', 'fr'];
```

<div dir="rtl">

### State (البيانات المتغيرة)

دي حاجات **بتتغير** خلال استخدام التطبيق:

</div>

```dart
// ✅ This is State - Changes over time
String currentLanguage = 'ar';        // User can change language
int loginAttempts = 0;                // Increments on failed login
bool isDarkMode = false;              // User toggles theme
List<String> recentSearches = [];    // Grows as user searches
```

<div dir="rtl">

### الفرق الجوهري

| البيانات الثابتة | State |
|-------------------|-------|
| محددة من البداية | بتتغير بمرور الوقت |
| نفس القيمة طول الوقت | قيمتها dynamic |
| مش محتاجة إدارة | محتاجة إدارة (Management) |
| مثال: اسم التطبيق | مثال: اسم المستخدم الحالي |

---

## 🔍 أنواع State

### 1️⃣ حالة واجهة المستخدم (UI State)

دي حاجات بتأثر على شكل UI بس (مش بيانات مهمة):

</div>

```dart
// UI State Examples
bool isMenuOpen = false;              // Is drawer open?
bool isPasswordVisible = false;       // Show/hide password?
int selectedTabIndex = 0;             // Which tab is selected?
bool isSearchFocused = false;         // Is search field focused?
```

<div dir="rtl">

**خصائصها:**
- ✅ مؤقتة (Temporary)
- ✅ خاصة بالشاشة الحالية
- ✅ مش محتاجة تتحفظ
- ✅ لو التطبيق اتقفل وفتح تاني، مفيش مشكلة ترجع للقيمة الافتراضية

---

### 2️⃣ حالة البيانات (Data State)

دي البيانات المهمة اللي جاية من API أو Database:

</div>

```dart
// Data State Examples
User? currentUser;                    // Logged in user
List<Product> products = [];          // Products from API
List<CartItem> cartItems = [];        // Items in shopping cart
Map<String, dynamic>? userProfile;    // User profile data
```

<div dir="rtl">

**خصائصها:**
- ✅ مهمة (Critical)
- ✅ غالباً محتاجة تتحفظ
- ✅ بتيجي من مصدر خارجي (API/Database)
- ✅ بتتشارك بين شاشات كتير

---

### 3️⃣ حالة التطبيق (App State)

دي إعدادات وبيانات بتأثر على التطبيق كله:

</div>

```dart
// App State Examples
bool isDarkMode = false;              // App theme
String currentLanguage = 'ar';        // App language
bool isAuthenticated = false;         // Is user logged in?
NotificationSettings notifications;   // App-wide settings
```

<div dir="rtl">

**خصائصها:**
- ✅ عامة (Global)
- ✅ بتأثر على التطبيق كله
- ✅ محتاجة تتحفظ بين الجلسات
- ✅ نادراً ما بتتغير

---

## 💡 أمثلة عملية من تطبيقات حقيقية

### مثال 1: تطبيق واتساب

</div>

```dart
class WhatsAppState {
  // Data State
  User currentUser;
  List<Chat> chats = [];
  List<Contact> contacts = [];

  // UI State
  Chat? selectedChat;
  bool isRecording = false;
  bool isTyping = false;

  // App State
  bool isDarkMode = false;
  String language = 'ar';
  NotificationSettings notifications;
}
```

<div dir="rtl">

**تحليل:**
- **Data State**: الرسائل، المحادثات، جهات الاتصال (من السيرفر)
- **UI State**: المحادثة المفتوحة، حالة التسجيل (مؤقت)
- **App State**: الثيم، اللغة، الإعدادات (عام)

---

### مثال 2: تطبيق YouTube

</div>

```dart
class YouTubeState {
  // Data State
  List<Video> homeVideos = [];
  List<Video> subscriptionVideos = [];
  Video? currentlyPlayingVideo;

  // UI State
  bool isPlayerFullScreen = false;
  bool isPlayerMuted = false;
  double playbackSpeed = 1.0;

  // App State
  User? currentUser;
  VideoQuality preferredQuality;
  bool autoPlayNext = true;
}
```

<div dir="rtl">

**تحليل:**
- **Data State**: الفيديوهات المتاحة (من API)
- **UI State**: حالة المشغل (مؤقت للفيديو الحالي)
- **App State**: تفضيلات المستخدم (بيتحفظ)

---

### مثال 3: تطبيق توصيل الطعام

</div>

```dart
class FoodDeliveryState {
  // Data State
  List<Restaurant> nearbyRestaurants = [];
  List<MenuItem> cart = [];
  Order? activeOrder;
  DeliveryLocation? currentLocation;

  // UI State
  Restaurant? selectedRestaurant;
  bool isFilterPanelOpen = false;
  String searchQuery = '';

  // App State
  User? currentUser;
  PaymentMethod? defaultPaymentMethod;
  Address? defaultAddress;
}
```

<div dir="rtl">

**تحليل:**
- **Data State**: المطاعم، العربة، الطلب النشط
- **UI State**: المطعم المفتوح، البحث
- **App State**: بيانات المستخدم، الإعدادات

---

## 🧪 تمرين عملي

خليني أديك مثال وانت تحدد نوع كل State:

</div>

```dart
class MusicPlayerState {
  // 1. What type of state is this?
  Song? currentSong;

  // 2. What type of state is this?
  bool isPlaying = false;

  // 3. What type of state is this?
  double volume = 0.5;

  // 4. What type of state is this?
  List<Song> queue = [];

  // 5. What type of state is this?
  bool isShuffleOn = false;
}
```

<div dir="rtl">

### الإجابات:

1. **`currentSong`** → **Data State** (الأغنية الحالية - بيانات مهمة)
2. **`isPlaying`** → **UI State** (حالة التشغيل - مؤقت)
3. **`volume`** → **App State** (مستوى الصوت - إعداد عام)
4. **`queue`** → **Data State** (قائمة التشغيل - بيانات)
5. **`isShuffleOn`** → **App State** (خاصية Shuffle - إعداد عام)

---

## ⚡ State Lifecycle (دورة حياة الحالة)

### مراحل حياة State

</div>

```dart
// 1. Initialization (التهيئة)
int counter = 0;

// 2. Update (التحديث)
void increment() {
  counter++; // State changes
}

// 3. Read (القراءة)
void displayCounter() {
  print('Counter: $counter'); // Reading state
}

// 4. Reset (إعادة التعيين)
void reset() {
  counter = 0; // Back to initial state
}

// 5. Disposal (التخلص)
// When widget is removed, state is disposed
```

<div dir="rtl">

---

## 🎯 القاعدة الذهبية

### متى الحاجة دي State؟

اسأل نفسك الأسئلة دي:

</div>

```
1. هل القيمة ممكن تتغير خلال حياة التطبيق؟
   ✅ نعم → State
   ❌ لا → Constant

2. هل تغيير القيمة بيأثر على UI؟
   ✅ نعم → State
   ❌ لا → Internal variable

3. هل محتاج Widget يتبني تاني لما تتغير؟
   ✅ نعم → State
   ❌ لا → Regular variable
```

<div dir="rtl">

### أمثلة تطبيقية

</div>

```dart
// ❌ NOT State - Never changes
const int maxUploadSize = 10 * 1024 * 1024; // 10 MB

// ✅ IS State - Changes and affects UI
int uploadProgress = 0; // 0 to 100

// ❌ NOT State - Doesn't affect UI
int _internalCounter = 0; // Private, not displayed

// ✅ IS State - Changes and affects UI
bool isUploading = false; // Shows/hides upload indicator
```

<div dir="rtl">

---

## 📊 ملخص سريع

### State هو:
- ✅ بيانات بتتغير بمرور الوقت
- ✅ بتأثر على UI أو سلوك التطبيق
- ✅ محتاجة إدارة (Management)
- ✅ ممكن تكون محلية أو عامة

### State مش:
- ❌ بيانات ثابتة (Constants)
- ❌ متغيرات داخلية مش بتأثر على UI
- ❌ حسابات مؤقتة
- ❌ إعدادات البرنامج الثابتة

---

## 🔗 الخطوة الجاية

دلوقتي بعد ما فهمت إيه هو State، وقت نفهم:
- **الفرق بين Local State و Global State** (الملف الجاي)
- **امتى نحتاج State Management**
- **أنواع حلول State Management**

---

## 📚 المصادر

- [Flutter State Management Introduction](https://docs.flutter.dev/data-and-backend/state-mgmt/intro)
- [Ephemeral vs App State](https://docs.flutter.dev/data-and-backend/state-mgmt/ephemeral-vs-app)
- [Declarative UI](https://docs.flutter.dev/data-and-backend/state-mgmt/declarative)

---

## ✅ تأكد إنك فهمت

قبل ما تكمل، تأكد إنك تقدر تجاوب على الأسئلة دي:

- [ ] تقدر تعرّف إيه هو State؟
- [ ] تعرف الفرق بين State والبيانات الثابتة؟
- [ ] تقدر تحدد نوع State (UI/Data/App)؟
- [ ] فاهم دورة حياة State؟
- [ ] تعرف امتى الحاجة تبقى State وامتى لأ؟

**لو كل الإجابات آه، تقدر تكمل للملف الجاي!** 🎉

</div>
