---
title: What makes code unmaintainable
pubDatetime: 2025-04-24
description: "Why code becomes hard to change: cohesion failures, coupling failures, leaked decisions, and other structural causes of unmaintainable software."
slug: what-makes-code-unmaintainable
tags:
  - practices
  - software-design
  - reference
draft: false
---

These are the structural causes I keep finding across projects. Not style issues. Not naming preferences. The things that make a codebase genuinely hard to change.

They group into two buckets: code that's in the wrong place, and code that knows too much about other code.

---

## Cohesion failures — code in the wrong place

**Feature Envy.** A method that spends most of its time on another class's data. A `TaskNotificationService` computing `DueDate < DateTime.UtcNow && Status != TaskStatus.Completed` to decide if a task is overdue. That logic belongs on `Task`. When it doesn't live there, it gets duplicated, diverges, and the definition of overdue becomes a matter of which file you happen to be reading.

**Semantic duplication.** Two implementations of the same concept with different names. One handler checks `DueDate < now`. Another checks `DueDate < now && Status != Completed`. Both mean overdue. No linter catches this because the names don't match — only domain knowledge does. The cost is paid later, when someone changes one and misses the other.

**Mixed abstraction levels.** A coordinator method that starts by calling `LoadOrder`, `ValidatePayment`, `ReserveInventory`, then suddenly drops into `if (order.Total > 1000 && customer.Status != Preferred)` halfway through. A coordinator should read like a table of contents. When it mixes orchestration with inline business rules, the rule becomes invisible to anyone navigating from the outside.

---

## Coupling failures — code that knows too much:

**Excessive chaining.** `_projectService.GetProject(id).Members.First(m => m.UserId == userId).Role` — coupled to `Project`, its `Members` collection, `Member`'s shape, and the `Role` enum, to answer a yes/no authorization question. Every link in that chain is a change that can break the caller. `_projectService.CanUserAssignTasks(projectId, userId)` answers the same question with one dependency.

**Leaked decisions.** The rule for what makes a task assignable written four different ways across four handlers. Each one someone's best guess at the moment they needed it.

```csharp
// In AssignTaskHandler
if (task.Status != TaskStatus.Completed && task.AssigneeId == null)
    task.AssigneeId = command.UserId;

// In BulkAssignHandler
if (task.AssigneeId == null)
    task.AssigneeId = command.UserId;
```

The decision belongs on `Task`:

```csharp
public bool TryAssign(Guid userId)
{
    if (Status == TaskStatus.Completed || AssigneeId != null) return false;
    AssigneeId = userId;
    return true;
}
```

One place and one rule. Every caller gets the same answer.

**Inappropriate intimacy.** External code that mutates another object's state directly — skipping the method that enforces invariants, writing to `task.AssigneeId` instead of calling `TryAssign`. The object loses control of itself. The caller now has to remember which fields are safe to change, and in what combination, to keep the object valid. This is the endpoint of leaked decisions left unfixed.

**Multiple roles.** A class whose purpose requires "and" to describe — validator and coordinator and notifier. Composer, policy checker, and side-effect runner. Each role it accumulates is a reason for an unrelated caller to depend on it. Changes to one role risk the others.

---

Maintainability breaks when logic drifts away from the data it operates on, or when a class accumulates knowledge about things outside its boundary. Both are detectable early. Both compound silently if left alone.
