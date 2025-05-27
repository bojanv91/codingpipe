---
title: "Use enums over booleans for status fields"
date: 2024-10-30
dateUpdated: Last Modified
permalink: /posts/use-enums-over-booleans/
tags:
  - Code
  - Software Design
layout: layouts/post.njk
---

In business applications, we often need to track the state of entities - users, orders, payments, etc. I've encountered this maintenance issue across multiple projects: boolean status fields create ambiguity and become difficult to extend. Here's why enums provide better long-term maintainability.

## The meaning of FALSE is not always clear

The meaning of **false** is not always clear when using booleans for status fields.

```csharp
public class User
{
    public bool IsActive { get; set; }
}
```

What does it mean when **IsActive** is **false**? Does that mean the user has been banned? Or is the email not yet verified? Or an admin disabled the login for that user? Enums provide better clarity and flexibility.

Here's the refactored version:

```csharp
public enum UserStatus
{
  Active,
  PendingVerification
}
 
public class User
{
  public UserStatus Status { get; set; }
}
```

Now you know that the user can be in active or pending verification state. This approach makes the meaning of the user's states obvious.

## Adding states becomes complex with booleans

This ambiguity problem gets worse when you need to expand beyond two states. Adding a third state like "Banned" with booleans becomes problematic:

```csharp
public class User
{
  public bool IsActive { get; set; }  // from the original example
  public bool IsBanned { get; set; }  // newly added state for handling banned users
}
```

This approach creates four possible combinations, but only two are valid. Managing these relationships becomes complicated quickly. Can an active user be banned? What happens then - what field takes precedence?

Additionally, what about existing data? You need to change the DB schema and migrate data to the new schema, which may create downtime or technical hurdles, especially at scale.

With enums, adding a new state is simple:

```csharp
public enum UserStatus
{
  Active,
  PendingVerification,
  Banned  // new state
}
```

## Rules of thumb

Here are some guidelines I follow:

1. Start with an enum if there's any chance you'll need more than two states in the future.
2. Use boolean flags only for clear yes/no scenarios that are unlikely to change.
3. Before adding a boolean flag, ask yourself: "Could this need more states down the road?"

This pattern works well for order statuses (Draft, Pending, Shipped, Delivered), payment states, and any workflow where entities progress through multiple stages. Using enums early for potentially expandable status fields saves significant refactoring effort later.
