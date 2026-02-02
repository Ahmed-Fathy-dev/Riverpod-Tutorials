<div dir="rtl">

# StreamProvider - البيانات المستمرة Real-time

**المستوى**: 🟡 متوسط

## 📌 المفاهيم الأساسية

في الملف ده هنتكلم عن:
- إيه هو StreamProvider ومتى نستخدمه
- الفرق بينه وبين FutureProvider
- Real-time data handling
- Firebase integration
- Best practices

## 🎯 الهدف

بعد ما تخلص القراءة، هتقدر:
- تستخدم StreamProvider للـ real-time data
- تتعامل مع Streams في Riverpod
- تدمج Firebase أو WebSocket
- تعرف متى تستخدم StreamProvider

---

## 🔍 إيه هو StreamProvider؟

**StreamProvider** هو provider للبيانات اللي بتتحدث **باستمرار** (continuous updates).

</div>

```dart
// StreamProvider
final messagesProvider = StreamProvider<List<Message>>((ref) {
  return chatService.messagesStream();
});

// Usage - Same as FutureProvider!
final messagesAsync = ref.watch(messagesProvider);

messagesAsync.when(
  data: (messages) => MessagesList(messages),
  loading: () => CircularProgressIndicator(),
  error: (error, stack) => Text('Error: $error'),
);
```

<div dir="rtl">

**متى تستخدمه:**
- ✅ Real-time data (chat, notifications)
- ✅ Continuous updates (location, sensors)
- ✅ Firebase Firestore/Realtime Database
- ✅ WebSocket connections
- ✅ Stream-based APIs

**متى ما تستخدموش:**
- ❌ One-time fetch → use FutureProvider
- ❌ محتاج methods → use StreamNotifierProvider

---

## 🎨 الاستخدامات الأساسية

### 1. Chat Messages (Real-time)

</div>

```dart
// Message model
class Message {
  final String id;
  final String senderId;
  final String text;
  final DateTime timestamp;

  Message({
    required this.id,
    required this.senderId,
    required this.text,
    required this.timestamp,
  });
}

// Chat service
class ChatService {
  Stream<List<Message>> messagesStream(String roomId) {
    // In real app, this could be Firebase or WebSocket
    return Stream.periodic(Duration(seconds: 2), (count) {
      return [
        Message(
          id: '$count',
          senderId: 'user1',
          text: 'Message $count',
          timestamp: DateTime.now(),
        ),
      ];
    });
  }
}

// Providers
final chatServiceProvider = Provider<ChatService>((ref) => ChatService());

final currentRoomIdProvider = StateProvider<String>((ref) => 'room1');

final messagesProvider = StreamProvider<List<Message>>((ref) {
  final service = ref.watch(chatServiceProvider);
  final roomId = ref.watch(currentRoomIdProvider);

  return service.messagesStream(roomId);
});

// Chat Screen
class ChatScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final messagesAsync = ref.watch(messagesProvider);

    return Scaffold(
      appBar: AppBar(title: Text('Chat')),
      body: messagesAsync.when(
        data: (messages) {
          if (messages.isEmpty) {
            return Center(child: Text('No messages yet'));
          }

          return ListView.builder(
            reverse: true,
            itemCount: messages.length,
            itemBuilder: (context, index) {
              final message = messages[index];
              return MessageBubble(message: message);
            },
          );
        },
        loading: () => Center(child: CircularProgressIndicator()),
        error: (error, stack) => Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Icon(Icons.error, color: Colors.red, size: 48),
              SizedBox(height: 16),
              Text('Error loading messages'),
              Text('$error', style: TextStyle(fontSize: 12)),
            ],
          ),
        ),
      ),
    );
  }
}

class MessageBubble extends StatelessWidget {
  final Message message;

  const MessageBubble({required this.message});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: EdgeInsets.symmetric(horizontal: 16, vertical: 8),
      child: Align(
        alignment: Alignment.centerLeft,
        child: Container(
          padding: EdgeInsets.all(12),
          decoration: BoxDecoration(
            color: Colors.grey[200],
            borderRadius: BorderRadius.circular(12),
          ),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Text(message.text),
              SizedBox(height: 4),
              Text(
                '${message.timestamp.hour}:${message.timestamp.minute}',
                style: TextStyle(fontSize: 10, color: Colors.grey),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

<div dir="rtl">

### 2. Location Updates

</div>

```dart
// Location model
class LocationData {
  final double latitude;
  final double longitude;
  final double accuracy;

  LocationData({
    required this.latitude,
    required this.longitude,
    required this.accuracy,
  });
}

// Location service
class LocationService {
  Stream<LocationData> locationStream() {
    // Using geolocator package
    return Geolocator.getPositionStream(
      locationSettings: LocationSettings(
        accuracy: LocationAccuracy.high,
        distanceFilter: 10, // Update every 10 meters
      ),
    ).map((position) {
      return LocationData(
        latitude: position.latitude,
        longitude: position.longitude,
        accuracy: position.accuracy,
      );
    });
  }
}

// Providers
final locationServiceProvider = Provider<LocationService>(
  (ref) => LocationService(),
);

final locationProvider = StreamProvider<LocationData>((ref) {
  final service = ref.watch(locationServiceProvider);
  return service.locationStream();
});

// Map Screen
class MapScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final locationAsync = ref.watch(locationProvider);

    return Scaffold(
      appBar: AppBar(title: Text('My Location')),
      body: locationAsync.when(
        data: (location) => Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(Icons.location_on, size: 64, color: Colors.blue),
            SizedBox(height: 16),
            Text(
              'Latitude: ${location.latitude.toStringAsFixed(6)}',
              style: TextStyle(fontSize: 18),
            ),
            Text(
              'Longitude: ${location.longitude.toStringAsFixed(6)}',
              style: TextStyle(fontSize: 18),
            ),
            SizedBox(height: 8),
            Text(
              'Accuracy: ${location.accuracy.toStringAsFixed(1)}m',
              style: TextStyle(fontSize: 14, color: Colors.grey),
            ),
          ],
        ),
        loading: () => Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              CircularProgressIndicator(),
              SizedBox(height: 16),
              Text('Getting your location...'),
            ],
          ),
        ),
        error: (error, stack) => Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Icon(Icons.location_off, size: 64, color: Colors.red),
              SizedBox(height: 16),
              Text('Location access denied'),
              Text('$error', style: TextStyle(fontSize: 12)),
            ],
          ),
        ),
      ),
    );
  }
}
```

<div dir="rtl">

### 3. Firebase Firestore Integration

</div>

```dart
// Todo model
class Todo {
  final String id;
  final String title;
  final bool isCompleted;
  final DateTime createdAt;

  Todo({
    required this.id,
    required this.title,
    required this.isCompleted,
    required this.createdAt,
  });

  factory Todo.fromFirestore(DocumentSnapshot doc) {
    final data = doc.data() as Map<String, dynamic>;
    return Todo(
      id: doc.id,
      title: data['title'],
      isCompleted: data['isCompleted'],
      createdAt: (data['createdAt'] as Timestamp).toDate(),
    );
  }
}

// Firestore provider
final firestoreProvider = Provider<FirebaseFirestore>((ref) {
  return FirebaseFirestore.instance;
});

// Current user ID
final currentUserIdProvider = Provider<String>((ref) {
  return FirebaseAuth.instance.currentUser?.uid ?? '';
});

// Todos stream provider
final todosStreamProvider = StreamProvider<List<Todo>>((ref) {
  final firestore = ref.watch(firestoreProvider);
  final userId = ref.watch(currentUserIdProvider);

  if (userId.isEmpty) {
    return Stream.value([]);
  }

  return firestore
      .collection('users')
      .doc(userId)
      .collection('todos')
      .orderBy('createdAt', descending: true)
      .snapshots()
      .map((snapshot) {
    return snapshot.docs.map((doc) => Todo.fromFirestore(doc)).toList();
  });
});

// Todos Screen
class TodosScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final todosAsync = ref.watch(todosStreamProvider);

    return Scaffold(
      appBar: AppBar(title: Text('My Todos')),
      body: todosAsync.when(
        data: (todos) {
          if (todos.isEmpty) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Icon(Icons.inbox, size: 64, color: Colors.grey),
                  SizedBox(height: 16),
                  Text('No todos yet'),
                  SizedBox(height: 8),
                  ElevatedButton(
                    onPressed: () {
                      // Add todo
                    },
                    child: Text('Add Todo'),
                  ),
                ],
              ),
            );
          }

          return ListView.builder(
            itemCount: todos.length,
            itemBuilder: (context, index) {
              final todo = todos[index];
              return ListTile(
                title: Text(
                  todo.title,
                  style: TextStyle(
                    decoration: todo.isCompleted
                        ? TextDecoration.lineThrough
                        : null,
                  ),
                ),
                leading: Checkbox(
                  value: todo.isCompleted,
                  onChanged: (value) {
                    // Toggle todo in Firestore
                  },
                ),
                trailing: IconButton(
                  icon: Icon(Icons.delete),
                  onPressed: () {
                    // Delete todo from Firestore
                  },
                ),
              );
            },
          );
        },
        loading: () => Center(child: CircularProgressIndicator()),
        error: (error, stack) => Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Icon(Icons.error, color: Colors.red, size: 48),
              SizedBox(height: 16),
              Text('Error loading todos'),
            ],
          ),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          // Show add todo dialog
        },
        child: Icon(Icons.add),
      ),
    );
  }
}
```

<div dir="rtl">

---

## 💻 أمثلة عملية كاملة

### مثال 1: Live Notifications

</div>

```dart
// Notification model
class AppNotification {
  final String id;
  final String title;
  final String body;
  final DateTime timestamp;
  final bool isRead;

  AppNotification({
    required this.id,
    required this.title,
    required this.body,
    required this.timestamp,
    required this.isRead,
  });
}

// Notification service
class NotificationService {
  Stream<List<AppNotification>> notificationsStream() {
    // Simulate real-time notifications
    return Stream.periodic(Duration(seconds: 5), (count) {
      return [
        AppNotification(
          id: 'notif_$count',
          title: 'New Notification',
          body: 'You have a new notification #$count',
          timestamp: DateTime.now(),
          isRead: false,
        ),
      ];
    }).asyncExpand((newNotifications) async* {
      yield newNotifications;
    });
  }
}

// Providers
final notificationServiceProvider = Provider<NotificationService>(
  (ref) => NotificationService(),
);

final notificationsProvider = StreamProvider<List<AppNotification>>((ref) {
  final service = ref.watch(notificationServiceProvider);
  return service.notificationsStream();
});

// Unread count (computed)
final unreadCountProvider = Provider<int>((ref) {
  final notificationsAsync = ref.watch(notificationsProvider);

  return notificationsAsync.maybeWhen(
    data: (notifications) {
      return notifications.where((n) => !n.isRead).length;
    },
    orElse: () => 0,
  );
});

// Notification Bell Icon
class NotificationBellIcon extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final unreadCount = ref.watch(unreadCountProvider);

    return IconButton(
      icon: Badge(
        label: Text('$unreadCount'),
        isLabelVisible: unreadCount > 0,
        child: Icon(Icons.notifications),
      ),
      onPressed: () {
        Navigator.push(
          context,
          MaterialPageRoute(builder: (_) => NotificationsScreen()),
        );
      },
    );
  }
}

// Notifications Screen
class NotificationsScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final notificationsAsync = ref.watch(notificationsProvider);

    return Scaffold(
      appBar: AppBar(title: Text('Notifications')),
      body: notificationsAsync.when(
        data: (notifications) {
          if (notifications.isEmpty) {
            return Center(child: Text('No notifications'));
          }

          return ListView.builder(
            itemCount: notifications.length,
            itemBuilder: (context, index) {
              final notification = notifications[index];
              return ListTile(
                leading: CircleAvatar(
                  child: Icon(Icons.notifications),
                  backgroundColor: notification.isRead
                      ? Colors.grey
                      : Colors.blue,
                ),
                title: Text(
                  notification.title,
                  style: TextStyle(
                    fontWeight: notification.isRead
                        ? FontWeight.normal
                        : FontWeight.bold,
                  ),
                ),
                subtitle: Text(notification.body),
                trailing: Text(
                  _formatTime(notification.timestamp),
                  style: TextStyle(fontSize: 12, color: Colors.grey),
                ),
              );
            },
          );
        },
        loading: () => Center(child: CircularProgressIndicator()),
        error: (error, stack) => Center(child: Text('Error: $error')),
      ),
    );
  }

  String _formatTime(DateTime time) {
    final now = DateTime.now();
    final diff = now.difference(time);

    if (diff.inMinutes < 1) return 'Just now';
    if (diff.inMinutes < 60) return '${diff.inMinutes}m ago';
    if (diff.inHours < 24) return '${diff.inHours}h ago';
    return '${diff.inDays}d ago';
  }
}
```

<div dir="rtl">

### مثال 2: WebSocket Connection

</div>

```dart
// Stock price model
class StockPrice {
  final String symbol;
  final double price;
  final double change;
  final DateTime timestamp;

  StockPrice({
    required this.symbol,
    required this.price,
    required this.change,
    required this.timestamp,
  });
}

// WebSocket service
class StockWebSocketService {
  Stream<StockPrice> priceStream(String symbol) {
    // Connect to WebSocket
    final channel = IOWebSocketChannel.connect(
      'wss://api.example.com/stocks/$symbol',
    );

    return channel.stream.map((data) {
      final json = jsonDecode(data);
      return StockPrice(
        symbol: json['symbol'],
        price: json['price'].toDouble(),
        change: json['change'].toDouble(),
        timestamp: DateTime.now(),
      );
    });
  }
}

// Providers
final stockWebSocketServiceProvider = Provider<StockWebSocketService>(
  (ref) => StockWebSocketService(),
);

final selectedStockProvider = StateProvider<String>((ref) => 'AAPL');

final stockPriceProvider = StreamProvider<StockPrice>((ref) {
  final service = ref.watch(stockWebSocketServiceProvider);
  final symbol = ref.watch(selectedStockProvider);

  return service.priceStream(symbol);
});

// Stock Price Widget
class StockPriceWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final priceAsync = ref.watch(stockPriceProvider);
    final selectedStock = ref.watch(selectedStockProvider);

    return Card(
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            // Stock selector
            DropdownButton<String>(
              value: selectedStock,
              items: ['AAPL', 'GOOGL', 'MSFT', 'TSLA']
                  .map((symbol) => DropdownMenuItem(
                        value: symbol,
                        child: Text(symbol),
                      ))
                  .toList(),
              onChanged: (symbol) {
                if (symbol != null) {
                  ref.read(selectedStockProvider.notifier).state = symbol;
                }
              },
            ),
            SizedBox(height: 16),

            // Price display
            priceAsync.when(
              data: (price) => Column(
                children: [
                  Text(
                    '\$${price.price.toStringAsFixed(2)}',
                    style: TextStyle(fontSize: 48, fontWeight: FontWeight.bold),
                  ),
                  Row(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      Icon(
                        price.change >= 0
                            ? Icons.arrow_upward
                            : Icons.arrow_downward,
                        color: price.change >= 0 ? Colors.green : Colors.red,
                      ),
                      Text(
                        '${price.change >= 0 ? '+' : ''}${price.change.toStringAsFixed(2)}',
                        style: TextStyle(
                          fontSize: 20,
                          color: price.change >= 0 ? Colors.green : Colors.red,
                        ),
                      ),
                    ],
                  ),
                  SizedBox(height: 8),
                  Text(
                    'Updated: ${price.timestamp.hour}:${price.timestamp.minute}:${price.timestamp.second}',
                    style: TextStyle(fontSize: 12, color: Colors.grey),
                  ),
                ],
              ),
              loading: () => Column(
                children: [
                  CircularProgressIndicator(),
                  SizedBox(height: 8),
                  Text('Connecting...'),
                ],
              ),
              error: (error, stack) => Column(
                children: [
                  Icon(Icons.error, color: Colors.red),
                  Text('Connection failed'),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

<div dir="rtl">

---

## ⚠️ أخطاء شائعة

### ❌ خطأ 1: استخدام StreamProvider لـ One-time Fetch

</div>

```dart
// ❌ WRONG - Use FutureProvider instead!
final userProvider = StreamProvider<User>((ref) {
  return Stream.fromFuture(api.getUser()); // Converting Future to Stream!
});

// ✅ CORRECT - Use FutureProvider
final userProvider = FutureProvider<User>((ref) async {
  return await api.getUser();
});
```

<div dir="rtl">

### ❌ خطأ 2: Not disposing Stream properly

</div>

```dart
// ❌ WRONG - Stream may not close
final messagesProvider = StreamProvider<List<Message>>((ref) {
  final controller = StreamController<List<Message>>();

  // Stream started but never closed!
  chatService.listen((messages) {
    controller.add(messages);
  });

  return controller.stream;
});

// ✅ CORRECT - Dispose properly
final messagesProvider = StreamProvider<List<Message>>((ref) {
  final controller = StreamController<List<Message>>();

  final subscription = chatService.listen((messages) {
    controller.add(messages);
  });

  ref.onDispose(() {
    subscription.cancel();
    controller.close();
  });

  return controller.stream;
});
```

<div dir="rtl">

---

## 💡 Best Practices

### 1. استخدم StreamProvider للـ Real-time فقط

</div>

```dart
// ✅ Good - Real-time data
final chatMessagesProvider = StreamProvider<List<Message>>((ref) {
  return chatService.messagesStream();
});

// ❌ Bad - One-time (use FutureProvider)
final userProvider = StreamProvider<User>((ref) {
  return Stream.fromFuture(api.getUser());
});
```

<div dir="rtl">

### 2. دايماً نضف الـ Stream في onDispose

</div>

```dart
// ✅ Good - Cleanup
final locationProvider = StreamProvider<Position>((ref) {
  final controller = StreamController<Position>();

  final subscription = Geolocator.getPositionStream().listen(
    controller.add,
  );

  ref.onDispose(() {
    subscription.cancel();
    controller.close();
  });

  return controller.stream;
});
```

<div dir="rtl">

---

## 📝 ملخص

**StreamProvider** يستخدم لـ:
- ✅ Real-time data (chat, notifications)
- ✅ Continuous updates (location, sensors)
- ✅ Firebase Firestore/Realtime Database
- ✅ WebSocket connections

**مش يستخدم لـ:**
- ❌ One-time fetch → FutureProvider
- ❌ محتاج methods → StreamNotifierProvider

**Best Practices:**
- للـ real-time فقط
- دايماً cleanup في onDispose
- استخدم `.when` للتعامل مع الحالات

---

## 🔗 الخطوة الجاية

في الملف الجاي هنتكلم عن:
- **NotifierProvider** - للـ state المعقد (sync)
- متى نستخدمه بدل StateProvider
- Business logic patterns

---

## 📚 المصادر

- [StreamProvider - Riverpod Docs](https://riverpod.dev/docs/providers/stream_provider)
- [Working with Streams](https://dart.dev/tutorials/language/streams)

</div>
