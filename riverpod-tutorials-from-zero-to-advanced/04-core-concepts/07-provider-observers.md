<div dir="rtl">

# Provider Observers - مراقبة الـ Providers

**المستوى**: 🟡 متوسط

## 📌 المفاهيم الأساسية

في الملف ده هنتكلم عن:
- إيه هو ProviderObserver
- إزاي تعمل logging للـ state changes
- Debugging و troubleshooting
- Analytics integration
- Performance monitoring
- Best practices

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تستخدم ProviderObserver للـ logging
- تتبع state changes
- تعمل debug للـ providers
- تدمج analytics
- تراقب الـ performance

---

## 🔍 إيه هو ProviderObserver؟

**ProviderObserver** هو class بيسمحلك تراقب كل التغييرات اللي بتحصل في الـ providers.

</div>

```dart
// Custom observer
class MyObserver extends ProviderObserver {
  @override
  void didAddProvider(
    ProviderBase provider,
    Object? value,
    ProviderContainer container,
  ) {
    print('Provider added: ${provider.name ?? provider.runtimeType}');
    print('Initial value: $value');
  }

  @override
  void didUpdateProvider(
    ProviderBase provider,
    Object? previousValue,
    Object? newValue,
    ProviderContainer container,
  ) {
    print('Provider updated: ${provider.name ?? provider.runtimeType}');
    print('Previous: $previousValue');
    print('New: $newValue');
  }

  @override
  void didDisposeProvider(
    ProviderBase provider,
    ProviderContainer container,
  ) {
    print('Provider disposed: ${provider.name ?? provider.runtimeType}');
  }

  @override
  void providerDidFail(
    ProviderBase provider,
    Object error,
    StackTrace stackTrace,
    ProviderContainer container,
  ) {
    print('Provider failed: ${provider.name ?? provider.runtimeType}');
    print('Error: $error');
  }
}

// Usage
void main() {
  runApp(
    ProviderScope(
      observers: [MyObserver()],
      child: MyApp(),
    ),
  );
}
```

<div dir="rtl">

---

## 🎨 Observer Methods

### didAddProvider - عند إنشاء Provider

</div>

```dart
class LoggerObserver extends ProviderObserver {
  @override
  void didAddProvider(
    ProviderBase provider,
    Object? value,
    ProviderContainer container,
  ) {
    print('➕ Created: ${provider.name}');
    print('   Type: ${provider.runtimeType}');
    print('   Initial value: $value');
    print('   Time: ${DateTime.now()}');
  }
}

// Output when provider is created:
// ➕ Created: counterProvider
//    Type: CounterProvider
//    Initial value: 0
//    Time: 2024-01-15 10:30:45.123
```

<div dir="rtl">

### didUpdateProvider - عند تحديث State

</div>

```dart
class StateChangeObserver extends ProviderObserver {
  @override
  void didUpdateProvider(
    ProviderBase provider,
    Object? previousValue,
    Object? newValue,
    ProviderContainer container,
  ) {
    print('🔄 Updated: ${provider.name}');
    print('   Previous: $previousValue');
    print('   New: $newValue');
    print('   Changed: ${previousValue != newValue}');
  }
}

// Output when state changes:
// 🔄 Updated: counterProvider
//    Previous: 5
//    New: 6
//    Changed: true
```

<div dir="rtl">

### didDisposeProvider - عند تدمير Provider

</div>

```dart
class DisposalObserver extends ProviderObserver {
  @override
  void didDisposeProvider(
    ProviderBase provider,
    ProviderContainer container,
  ) {
    print('🗑️  Disposed: ${provider.name}');
    print('   Type: ${provider.runtimeType}');
    print('   Time: ${DateTime.now()}');
  }
}

// Output when provider is disposed:
// 🗑️ Disposed: counterProvider
//    Type: CounterProvider
//    Time: 2024-01-15 10:35:20.456
```

<div dir="rtl">

### providerDidFail - عند حدوث Error

</div>

```dart
class ErrorObserver extends ProviderObserver {
  @override
  void providerDidFail(
    ProviderBase provider,
    Object error,
    StackTrace stackTrace,
    ProviderContainer container,
  ) {
    print('❌ Error in: ${provider.name}');
    print('   Error: $error');
    print('   Stack trace: $stackTrace');

    // Could send to crash reporting service
    // crashReporting.logError(error, stackTrace);
  }
}
```

<div dir="rtl">

---

## 🧪 أمثلة عملية

### مثال 1: Logger شامل

</div>

```dart
class ComprehensiveLogger extends ProviderObserver {
  final bool logCreation;
  final bool logUpdates;
  final bool logDisposal;
  final bool logErrors;

  ComprehensiveLogger({
    this.logCreation = true,
    this.logUpdates = true,
    this.logDisposal = true,
    this.logErrors = true,
  });

  @override
  void didAddProvider(
    ProviderBase provider,
    Object? value,
    ProviderContainer container,
  ) {
    if (!logCreation) return;

    final timestamp = DateTime.now().toIso8601String();
    print('[$timestamp] ➕ CREATED: ${_providerName(provider)}');
    print('   Initial value: ${_formatValue(value)}');
  }

  @override
  void didUpdateProvider(
    ProviderBase provider,
    Object? previousValue,
    Object? newValue,
    ProviderContainer container,
  ) {
    if (!logUpdates) return;

    final timestamp = DateTime.now().toIso8601String();
    print('[$timestamp] 🔄 UPDATED: ${_providerName(provider)}');
    print('   ${_formatValue(previousValue)} → ${_formatValue(newValue)}');
  }

  @override
  void didDisposeProvider(
    ProviderBase provider,
    ProviderContainer container,
  ) {
    if (!logDisposal) return;

    final timestamp = DateTime.now().toIso8601String();
    print('[$timestamp] 🗑️  DISPOSED: ${_providerName(provider)}');
  }

  @override
  void providerDidFail(
    ProviderBase provider,
    Object error,
    StackTrace stackTrace,
    ProviderContainer container,
  ) {
    if (!logErrors) return;

    final timestamp = DateTime.now().toIso8601String();
    print('[$timestamp] ❌ ERROR: ${_providerName(provider)}');
    print('   Error: $error');
    print('   Stack: ${stackTrace.toString().split('\n').take(3).join('\n')}');
  }

  String _providerName(ProviderBase provider) {
    return provider.name ?? provider.runtimeType.toString();
  }

  String _formatValue(Object? value) {
    if (value == null) return 'null';
    if (value is AsyncValue) {
      return value.when(
        data: (d) => 'AsyncData($d)',
        loading: () => 'AsyncLoading',
        error: (e, s) => 'AsyncError($e)',
      );
    }
    return value.toString();
  }
}

// Usage
void main() {
  runApp(
    ProviderScope(
      observers: [
        ComprehensiveLogger(
          logCreation: true,
          logUpdates: true,
          logDisposal: false, // Don't log disposal in production
          logErrors: true,
        ),
      ],
      child: MyApp(),
    ),
  );
}
```

<div dir="rtl">

### مثال 2: Analytics Integration

</div>

```dart
class AnalyticsObserver extends ProviderObserver {
  final AnalyticsService analytics;

  AnalyticsObserver(this.analytics);

  @override
  void didAddProvider(
    ProviderBase provider,
    Object? value,
    ProviderContainer container,
  ) {
    analytics.logEvent('provider_created', {
      'provider_name': provider.name ?? 'unknown',
      'provider_type': provider.runtimeType.toString(),
      'timestamp': DateTime.now().toIso8601String(),
    });
  }

  @override
  void didUpdateProvider(
    ProviderBase provider,
    Object? previousValue,
    Object? newValue,
    ProviderContainer container,
  ) {
    // Track specific provider updates
    if (provider.name?.contains('cart') ?? false) {
      analytics.logEvent('cart_updated', {
        'items_count': _extractItemsCount(newValue),
        'timestamp': DateTime.now().toIso8601String(),
      });
    }
  }

  @override
  void providerDidFail(
    ProviderBase provider,
    Object error,
    StackTrace stackTrace,
    ProviderContainer container,
  ) {
    analytics.logError('provider_error', {
      'provider_name': provider.name ?? 'unknown',
      'error_type': error.runtimeType.toString(),
      'error_message': error.toString(),
      'timestamp': DateTime.now().toIso8601String(),
    });
  }

  int _extractItemsCount(Object? value) {
    if (value is List) return value.length;
    return 0;
  }
}
```

<div dir="rtl">

### مثال 3: Performance Monitoring

</div>

```dart
class PerformanceObserver extends ProviderObserver {
  final Map<ProviderBase, DateTime> _creationTimes = {};
  final Map<ProviderBase, int> _updateCounts = {};
  final Map<ProviderBase, Duration> _lifetimes = {};

  @override
  void didAddProvider(
    ProviderBase provider,
    Object? value,
    ProviderContainer container,
  ) {
    _creationTimes[provider] = DateTime.now();
    _updateCounts[provider] = 0;

    print('📊 Provider created: ${provider.name}');
  }

  @override
  void didUpdateProvider(
    ProviderBase provider,
    Object? previousValue,
    Object? newValue,
    ProviderContainer container,
  ) {
    _updateCounts[provider] = (_updateCounts[provider] ?? 0) + 1;

    // Warn if updated too frequently
    if (_updateCounts[provider]! > 100) {
      print('⚠️  Warning: ${provider.name} updated ${_updateCounts[provider]} times');
      print('   Consider optimization!');
    }
  }

  @override
  void didDisposeProvider(
    ProviderBase provider,
    ProviderContainer container,
  ) {
    final creationTime = _creationTimes[provider];
    if (creationTime != null) {
      final lifetime = DateTime.now().difference(creationTime);
      _lifetimes[provider] = lifetime;

      print('📊 Provider disposed: ${provider.name}');
      print('   Lifetime: ${lifetime.inSeconds}s');
      print('   Updates: ${_updateCounts[provider] ?? 0}');
    }

    _creationTimes.remove(provider);
    _updateCounts.remove(provider);
  }

  void printStatistics() {
    print('\n=== Performance Statistics ===');
    print('Active providers: ${_creationTimes.length}');
    print('Total disposed: ${_lifetimes.length}');

    if (_lifetimes.isNotEmpty) {
      final avgLifetime = _lifetimes.values
          .map((d) => d.inMilliseconds)
          .reduce((a, b) => a + b) / _lifetimes.length;

      print('Average lifetime: ${avgLifetime}ms');
    }
  }
}
```

<div dir="rtl">

### مثال 4: Debug-only Observer

</div>

```dart
class DebugObserver extends ProviderObserver {
  // Only active in debug mode
  final bool enabled = kDebugMode;

  @override
  void didAddProvider(
    ProviderBase provider,
    Object? value,
    ProviderContainer container,
  ) {
    if (!enabled) return;

    print('🐛 [DEBUG] Provider created: ${provider.name}');
    print('   Value: $value');
  }

  @override
  void didUpdateProvider(
    ProviderBase provider,
    Object? previousValue,
    Object? newValue,
    ProviderContainer container,
  ) {
    if (!enabled) return;

    print('🐛 [DEBUG] Provider updated: ${provider.name}');
    print('   Old: $previousValue');
    print('   New: $newValue');

    // Check for suspicious changes
    if (previousValue == newValue) {
      print('   ⚠️  Warning: Update with same value!');
    }
  }

  @override
  void providerDidFail(
    ProviderBase provider,
    Object error,
    StackTrace stackTrace,
    ProviderContainer container,
  ) {
    if (!enabled) return;

    print('🐛 [DEBUG] Provider error: ${provider.name}');
    print('   Error: $error');
    print('   Stack trace:');
    print(stackTrace);
  }
}

// Usage
void main() {
  runApp(
    ProviderScope(
      observers: [
        DebugObserver(), // Only logs in debug mode
      ],
      child: MyApp(),
    ),
  );
}
```

<div dir="rtl">

### مثال 5: State History Tracker

</div>

```dart
class StateHistoryObserver extends ProviderObserver {
  final Map<ProviderBase, List<StateChange>> _history = {};
  final int maxHistorySize;

  StateHistoryObserver({this.maxHistorySize = 50});

  @override
  void didUpdateProvider(
    ProviderBase provider,
    Object? previousValue,
    Object? newValue,
    ProviderContainer container,
  ) {
    if (!_history.containsKey(provider)) {
      _history[provider] = [];
    }

    final history = _history[provider]!;

    history.add(StateChange(
      timestamp: DateTime.now(),
      previousValue: previousValue,
      newValue: newValue,
    ));

    // Keep only recent history
    if (history.length > maxHistorySize) {
      history.removeAt(0);
    }
  }

  List<StateChange>? getHistory(ProviderBase provider) {
    return _history[provider];
  }

  void printHistory(ProviderBase provider) {
    final history = _history[provider];
    if (history == null || history.isEmpty) {
      print('No history for ${provider.name}');
      return;
    }

    print('=== History for ${provider.name} ===');
    for (var i = 0; i < history.length; i++) {
      final change = history[i];
      print('[$i] ${change.timestamp}');
      print('    ${change.previousValue} → ${change.newValue}');
    }
  }

  @override
  void didDisposeProvider(
    ProviderBase provider,
    ProviderContainer container,
  ) {
    _history.remove(provider);
  }
}

class StateChange {
  final DateTime timestamp;
  final Object? previousValue;
  final Object? newValue;

  StateChange({
    required this.timestamp,
    required this.previousValue,
    required this.newValue,
  });
}
```

<div dir="rtl">

---

## 🎯 Multiple Observers

يمكنك استخدام أكثر من observer في نفس الوقت:

</div>

```dart
void main() {
  runApp(
    ProviderScope(
      observers: [
        // Logger for development
        if (kDebugMode) DebugObserver(),

        // Analytics for production
        if (kReleaseMode) AnalyticsObserver(analyticsService),

        // Performance monitoring always
        PerformanceObserver(),

        // Error tracking always
        ErrorObserver(),
      ],
      child: MyApp(),
    ),
  );
}
```

<div dir="rtl">

---

## 🎨 Advanced Patterns

### Pattern 1: Conditional Logging

</div>

```dart
class ConditionalLogger extends ProviderObserver {
  final Set<String> watchedProviders;

  ConditionalLogger({required this.watchedProviders});

  bool _shouldLog(ProviderBase provider) {
    final name = provider.name ?? '';
    return watchedProviders.any((watched) => name.contains(watched));
  }

  @override
  void didUpdateProvider(
    ProviderBase provider,
    Object? previousValue,
    Object? newValue,
    ProviderContainer container,
  ) {
    if (!_shouldLog(provider)) return;

    print('🔍 Watched provider updated: ${provider.name}');
    print('   $previousValue → $newValue');
  }
}

// Usage: Only log cart and auth providers
final observer = ConditionalLogger(
  watchedProviders: {'cart', 'auth'},
);
```

<div dir="rtl">

### Pattern 2: Rate Limiting

</div>

```dart
class RateLimitedObserver extends ProviderObserver {
  final Map<ProviderBase, DateTime> _lastLog = {};
  final Duration minInterval;

  RateLimitedObserver({
    this.minInterval = const Duration(seconds: 1),
  });

  bool _shouldLog(ProviderBase provider) {
    final lastLog = _lastLog[provider];
    if (lastLog == null) return true;

    final elapsed = DateTime.now().difference(lastLog);
    return elapsed >= minInterval;
  }

  @override
  void didUpdateProvider(
    ProviderBase provider,
    Object? previousValue,
    Object? newValue,
    ProviderContainer container,
  ) {
    if (!_shouldLog(provider)) return;

    print('Updated: ${provider.name}');
    _lastLog[provider] = DateTime.now();
  }
}
```

<div dir="rtl">

### Pattern 3: Crash Reporting Integration

</div>

```dart
class CrashReportingObserver extends ProviderObserver {
  final CrashReportingService crashReporting;

  CrashReportingObserver(this.crashReporting);

  @override
  void providerDidFail(
    ProviderBase provider,
    Object error,
    StackTrace stackTrace,
    ProviderContainer container,
  ) {
    // Send to crash reporting
    crashReporting.recordError(
      error,
      stackTrace,
      context: {
        'provider': provider.name ?? 'unknown',
        'provider_type': provider.runtimeType.toString(),
        'timestamp': DateTime.now().toIso8601String(),
      },
    );

    // Also log locally
    print('Error reported: ${provider.name}');
  }

  @override
  void didUpdateProvider(
    ProviderBase provider,
    Object? previousValue,
    Object? newValue,
    ProviderContainer container,
  ) {
    // Set context for crash reports
    crashReporting.setCustomKey(
      'last_provider_update',
      provider.name ?? 'unknown',
    );

    crashReporting.setCustomKey(
      'last_update_time',
      DateTime.now().toIso8601String(),
    );
  }
}
```

<div dir="rtl">

---

## ⚠️ Best Practices

### 1. استخدم Debug Mode للـ Verbose Logging

</div>

```dart
class SmartLogger extends ProviderObserver {
  @override
  void didUpdateProvider(
    ProviderBase provider,
    Object? previousValue,
    Object? newValue,
    ProviderContainer container,
  ) {
    if (kDebugMode) {
      // Verbose logging in debug
      print('Updated: ${provider.name}');
      print('  Previous: $previousValue');
      print('  New: $newValue');
    } else {
      // Minimal logging in release
      // Maybe just analytics events
    }
  }
}
```

<div dir="rtl">

### 2. تجنب Heavy Operations في Observers

</div>

```dart
// ❌ WRONG - Expensive operation
class BadObserver extends ProviderObserver {
  @override
  void didUpdateProvider(/*...*/) {
    // DON'T DO THIS - blocks UI!
    _expensiveOperation();
  }

  void _expensiveOperation() {
    for (var i = 0; i < 1000000; i++) {
      // Heavy work
    }
  }
}

// ✅ CORRECT - Async operation
class GoodObserver extends ProviderObserver {
  @override
  void didUpdateProvider(/*...*/) {
    // Run async without blocking
    _logAsync();
  }

  Future<void> _logAsync() async {
    // Send to server without blocking
    await loggingService.log(/*...*/);
  }
}
```

<div dir="rtl">

### 3. Cleanup عند Disposal

</div>

```dart
class CleanupObserver extends ProviderObserver {
  final Map<ProviderBase, dynamic> _data = {};

  @override
  void didAddProvider(/*...*/) {
    _data[provider] = /* some data */;
  }

  @override
  void didDisposeProvider(
    ProviderBase provider,
    ProviderContainer container,
  ) {
    // Clean up stored data
    _data.remove(provider);
  }
}
```

<div dir="rtl">

---

## 📝 ملخص

**ProviderObserver:**
- ✅ Monitor all provider changes
- ✅ Log state updates
- ✅ Track errors
- ✅ Integrate analytics
- ✅ Monitor performance

**Methods:**
- `didAddProvider` → Provider created
- `didUpdateProvider` → State changed
- `didDisposeProvider` → Provider disposed
- `providerDidFail` → Error occurred

**Best Practices:**
- Use debug mode for verbose logging
- Don't block UI in observers
- Clean up on disposal
- Use multiple observers for different concerns
- Rate limit frequent updates

---

## 🔗 الخطوة الجاية

في الملف الجاي هنتكلم عن:
- **Error Handling Basics**
- AsyncValue error handling
- Retry strategies
- Error recovery

---

## 📚 المصادر

- [Provider Observers](https://riverpod.dev/docs/concepts/provider_observer)
- [Debugging](https://riverpod.dev/docs/concepts/reading#debugging)

</div>
