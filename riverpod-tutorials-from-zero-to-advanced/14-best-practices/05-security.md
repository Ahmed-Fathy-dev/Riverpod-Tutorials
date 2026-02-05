<div dir="rtl">

# Security Best Practices

**المستوى**: 🔴 متقدم

## 🔒 Security Tips

### 1. Never Store Secrets in Providers

```dart
// ❌ BAD - Hardcoded secrets
@riverpod
String apiKey(ApiKeyRef ref) => 'sk_live_123456789';

// ✅ GOOD - Use environment variables
@riverpod
String apiKey(ApiKeyRef ref) => const String.fromEnvironment('API_KEY');
```

### 2. Validate User Input

```dart
// ✅ GOOD
Future<void> addTodo(String title) async {
  // Validate
  if (title.trim().isEmpty) {
    throw ValidationException('Title cannot be empty');
  }
  
  if (title.length > 100) {
    throw ValidationException('Title too long');
  }
  
  await api.addTodo(title.trim());
}
```

### 3. Handle Sensitive Data Carefully

```dart
// ✅ GOOD - Clear auth token on logout
Future<void> logout() async {
  await api.logout();
  
  // Clear sensitive data
  state = const AsyncValue.data(null);
  
  // Clear from secure storage
  await secureStorage.delete(key: 'auth_token');
}
```

### 4. Use HTTPS

```dart
// ✅ GOOD
final apiUrl = 'https://api.example.com';

// ❌ BAD
final apiUrl = 'http://api.example.com';  // Not secure!
```

### 5. Implement Proper Error Handling

```dart
// ✅ GOOD - Don't expose internal errors
try {
  await api.sensitiveOperation();
} catch (e) {
  // Log internally
  logger.error(e);
  
  // Show generic message to user
  throw Exception('Operation failed');  // Don't expose details!
}
```

</div>
