---
title: "Status Fields - Why Enums Beat Booleans for Maintainability"
date: 2024-10-30
dateUpdated: Last Modified
permalink: /posts/use-enums-over-booleans/
tags:
  - Code
  - Software Design
layout: layouts/post.njk
---

In business applications, we often need to track the state of entities - users, orders, payments, etc. While boolean flags might seem simple, they often lead to maintenance headaches. Here's why you should consider using enums instead.

## 1: The meaning of false is not always clear

The meaning of false is not always clear when using Booleans for status fields.

Let's see this example:

```csharp
public class User
{
    public bool IsActive { get; set; }
}
```

What does it mean when IsActive is false? Does that mean the user has been banned? Or is the email not yet verified?
In most such situations, choosing Enum is a much better choice. Enums are more descriptive and flexible.

Let's refactor:

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

## 2: Expanding with a third option is a very challenging refactor

Adding a third state like “Banned,” using Booleans can get problematic. Let's see this example:

```csharp
public class User
{
  public bool IsActive { get; set; }  // from the original example
  public bool IsBanned { get; set; }  // newly added state for handling banned users
}
```

This approach creates four possible combinations, but only two are valid. Managing these relationships can become complicated very quickly. Can an active user be banned? What happens then - what field takes precedence?

Additionally, what about existing data? We need to change the DB schema and migrate a lot of data to the new schema, which may create downtime or technical hurdles, especially at scale.

With Enums, adding a new state is simple:

```csharp
public enum UserStatus
{
  Active,
  PendingVerification,
  Banned  // new state
}
```

## Rules of thumb

Here are some rules of thumb that I use:

1. Start with an enum if there’s any chance you’ll need more than two states in the future.
2. Use boolean flags only for clear yes/no scenarios that are unlikely to change.
3. Before adding a boolean flag, ask yourself: “Could this need more states down the road?”

Using enums over booleans early for potentially expandable status fields ensures easier maintenance and clearer code in the long run. I won't consider it as over-engineering.
