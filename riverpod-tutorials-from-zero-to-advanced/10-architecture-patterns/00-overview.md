<div dir="rtl">

# Architecture Patterns - نظرة عامة

**المستوى**: 🔴 متقدم

## 🎯 الهدف

في **Section 11** هنتعلم كيف نبني تطبيقات كبيرة و maintainable باستخدام:
- 🏗️ Clean Architecture
- 📦 Repository Pattern
- 💉 Dependency Injection
- 📁 Feature-based Structure
- 🧪 Testable Architecture

---

## 📚 محتوى القسم

### 1. Clean Architecture (01-clean-architecture.md)
**المفاهيم:**
- الطبقات الثلاثة: Presentation, Domain, Data
- Dependency Rule
- Separation of Concerns
- Independent of Frameworks

### 2. Repository Pattern (02-repository-pattern.md)
**المفاهيم:**
- Data abstraction
- Multiple data sources
- Cache strategies
- Error handling

### 3. Dependency Injection (03-dependency-injection.md)
**المفاهيم:**
- DI مع Riverpod
- Constructor injection
- Provider injection
- Testing benefits

### 4. Feature-based Structure (04-feature-structure.md)
**المفاهيم:**
- Organizing by features
- Shared code
- Scalability
- Team collaboration

### 5. Complete Example (05-complete-example.md)
**مثال عملي:**
- Auth feature
- Products feature
- Cart feature
- Clean Architecture في action

---

## 🏗️ Clean Architecture Overview

```
┌─────────────────────────────────────┐
│         Presentation Layer          │
│   (UI, Widgets, State Management)   │
└────────────┬────────────────────────┘
             │ depends on
             ↓
┌─────────────────────────────────────┐
│          Domain Layer               │
│  (Entities, Use Cases, Interfaces)  │
└────────────┬────────────────────────┘
             │ depends on
             ↓
┌─────────────────────────────────────┐
│           Data Layer                │
│ (Repositories, Data Sources, DTOs)  │
└─────────────────────────────────────┘
```

**الـ Rule:** Dependencies point inward!

---

## 📁 Project Structure

```
lib/
├── features/
│   ├── auth/
│   │   ├── data/
│   │   │   ├── data_sources/
│   │   │   ├── models/
│   │   │   └── repositories/
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   ├── repositories/
│   │   │   └── use_cases/
│   │   └── presentation/
│   │       ├── providers/
│   │       ├── screens/
│   │       └── widgets/
│   └── products/
│       └── ... (same structure)
└── core/
    ├── constants/
    ├── errors/
    ├── network/
    └── utils/
```

---

## 🎯 المصادر

- [Flutter Riverpod Clean Architecture](https://ssoad.github.io/flutter_riverpod_clean_architecture/)
- [Flutter App Architecture with Riverpod](https://codewithandrea.com/articles/flutter-app-architecture-riverpod-introduction/)
- [Clean Architecture Guide](https://dev.to/ssoad/flutter-riverpod-clean-architecture-the-ultimate-production-ready-template-for-scalable-apps-gdh)

</div>
