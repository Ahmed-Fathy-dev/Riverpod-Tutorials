<div dir="rtl">

# Migration Strategy

**المستوى**: 🟡 متوسط

## 📌 الخلاصة

**Best approach: Incremental Migration**

---

## 🎯 Step-by-Step Strategy

### Phase 1: Setup (Week 1)

```yaml
# pubspec.yaml
dependencies:
  flutter_riverpod: ^3.0.0
  riverpod_annotation: ^3.0.0

dev_dependencies:
  riverpod_generator: ^3.0.0
  build_runner: ^2.4.0
```

```dart
// main.dart
void main() {
  runApp(
    ProviderScope(  // Add this!
      child: MyApp(),
    ),
  );
}
```

### Phase 2: Migrate One Feature (Week 2-3)

```
Pick smallest feature first!
✅ Settings screen
✅ Theme provider
✅ Simple counter
```

### Phase 3: Migrate Core Features (Week 4-6)

```
✅ Authentication
✅ Main data providers
✅ Navigation state
```

### Phase 4: Cleanup (Week 7)

```
✅ Remove old package
✅ Delete unused code
✅ Update tests
```

---

## 🔄 Coexistence Pattern

```dart
// Both can coexist!

// Old Provider code
final oldCounter = Provider.of<Counter>(context);

// New Riverpod code  
final newCounter = ref.watch(counterProvider);

// Gradually replace old with new
```

---

## ⚠️ Common Pitfalls

### 1. Don't migrate everything at once

```dart
// ❌ BAD
// Migrating entire app in one PR

// ✅ GOOD
// Migrate feature by feature
```

### 2. Test after each feature

```dart
// ✅ GOOD
- Migrate auth ✓
- Test auth ✓
- Migrate products ✓
- Test products ✓
```

### 3. Keep old code working

```dart
// ✅ GOOD
// Don't break existing features
// Add new Riverpod code alongside
// Remove old code when ready
```

---

## 📋 Checklist

- [ ] Add Riverpod dependencies
- [ ] Wrap app in ProviderScope
- [ ] Choose first feature to migrate
- [ ] Migrate feature
- [ ] Test thoroughly
- [ ] Repeat for next feature
- [ ] Remove old package when done

---

## 📚 المصادر

- [Migration Best Practices](https://riverpod.dev/docs/from_provider/quickstart)

</div>
