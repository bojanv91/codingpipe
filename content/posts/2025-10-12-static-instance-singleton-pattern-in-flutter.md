---
title: "The Static-Instance Singleton Pattern in Flutter"
date: 2025-10-12
dateUpdated: Last Modified
permalink: /posts/static-instance-singleton-pattern-in-flutter/
tags:
  - Flutter
  - Software Design
layout: layouts/post.njk
---

I’ve found a pattern that creates really clean singleton APIs: use static methods for actions that change state, and instance properties for reading data. Session management in mobile apps is a good example of where this shines.

## The pattern

Static methods for state changes:

```dart
await UserSession.login(authToken: 'token', username: 'user');
await UserSession.logout();
await UserSession.setDataRegion(DataRegion.US);
```

Instance properties for data access:

```dart
final username = UserSession.current.username;
final isAuth = UserSession.current.isAuthenticated;
final region = UserSession.current.dataRegion;
```

## Why this separation works

I think of it like database operations:

- Static methods = Commands (insert, update, delete the singleton)
- Instance properties = Queries (select from current state)

## The mental model

This distinction makes the API intuitive:

- Class-level operations -> Actions that change what the singleton is
- Instance-level access -> Reading what the singleton contains

## Implementation

```dart
class UserSession {
  static UserSession? _instance;
  
  // ACTIONS: Modify the global singleton
  static Future<void> login({required String authToken, required String username}) async {
    _instance = UserSession._(authToken: authToken, username: username);
    await _saveToStorage();
  }
  
  static Future<void> logout() async {
    _instance = const UserSession._empty();
    await _clearStorage();
  }
  
  // ACCESS: Read from current singleton
  static UserSession? get current => _instance;
  
  // DATA: Current state properties
  String get username => /* current data */;
  bool get isAuthenticated => /* current status */;
}
```

This pattern creates intuitive APIs where class-level operations change what the singleton is, and instance-level access reads what it contains.
