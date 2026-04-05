---
title: "The Static-Instance Singleton Pattern in Flutter"
pubDatetime: 2025-10-12
description: "Static methods for state changes, instance properties for reads. A singleton API pattern for Flutter."
slug: static-instance-singleton-pattern-in-flutter
tags:
  - design
draft: false
---

I use a pattern in Flutter where static methods own state changes and instance properties own reads. Session management is where it shows up most clearly.

**Static methods for state changes:**
```dart
await UserSession.login(authToken: 'token', username: 'user');
await UserSession.logout();
await UserSession.setDataRegion(DataRegion.US);
```

**Instance properties for reads:**
```dart
final username = UserSession.current.username;
final isAuth = UserSession.current.isAuthenticated;
final region = UserSession.current.dataRegion;
```

**Why the separation works:**

Think of it like database operations — static methods are commands (insert, update, delete), instance properties are queries (select from current state). Class-level operations change what the singleton is. Instance-level access reads what it contains.

**Implementation:**
```dart
class UserSession {
  static UserSession? _instance;

  static Future<void> login({required String authToken, required String username}) async {
    _instance = UserSession._(authToken: authToken, username: username);
    await _saveToStorage();
  }

  static Future<void> logout() async {
    _instance = const UserSession._empty();
    await _clearStorage();
  }

  static UserSession? get current => _instance;

  String get username => /* ... */;
  bool get isAuthenticated => /* ... */;
}
```

**TLDR:** Split singleton APIs by intent — static methods for commands, instance access for queries. The call site reads like the distinction matters, because it does.
