<div dir="rtl">

# Feature-based Structure

**المستوى**: 🔴 متقدم

## 📌 الخلاصة

**Feature-based** = تنظيم الكود حسب الـ features مش حسب الـ layers.

```
lib/
├── features/
│   ├── auth/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   ├── products/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   └── cart/
│       ├── data/
│       ├── domain/
│       └── presentation/
└── core/
    ├── network/
    ├── database/
    └── utils/
```

**المميزات:**
- ✅ Scalable - easy to add features
- ✅ Team-friendly - different teams work on different features
- ✅ Maintainable - all feature code in one place
- ✅ Testable - test features independently

---

## 📚 المصادر

- [Flutter Project Structure](https://codewithandrea.com/articles/flutter-project-structure/)

</div>
