---
title: What I fix in code review
pubDatetime: 2026-04-27
description: "The specific structural problems that appear repeatedly in codebases, with code diffs showing the problem and the fix."
slug: what-i-fix-in-code-review
tags: [practices, software-design]
draft: false
---

Code that compiles and passes tests isn't necessarily well structured. The failures that make it hard to maintain are recognizable. They show up in the same forms across different codebases and teams, no matter what the technology is.

These are the ones I fix most often.

---

## Logic placed where it's convenient, not where it belongs

The rule ends up in whichever class was open at the time — not where the codebase needs it.

```csharp
void AssignTask(Guid taskId, Guid assigneeId)
{
    var task = _db.Tasks.Find(taskId);
    if (task.Status == TaskStatus.Completed || task.Status == TaskStatus.Cancelled)
        throw new InvalidOperationException("Cannot assign a closed task");
    task.AssigneeId = assigneeId;
    _db.SaveChanges();
}
```

What makes a task assignable lives in the caller. The next developer who needs the same check writes it slightly differently — one forgets `Cancelled`, another adds `Archived`. Six months later there are three diverging definitions of the same rule.

```csharp
public void AssignTo(Guid assigneeId)
{
    if (Status is TaskStatus.Completed or TaskStatus.Cancelled)
        throw new InvalidOperationException("Cannot assign a closed task");
    AssigneeId = assigneeId;
}
```

If code inspects an object's state to decide what that object is allowed to do, the decision belongs inside the object.

---

## The same concept written in two places

No search tool catches this. The names are different. You only find it when you understand what both pieces mean.

```csharp
// NotificationService:
var overdue = tasks.Where(t => t.DueDate < DateTime.UtcNow && !t.IsPaid);

// ReportService:
var unpaidPastDue = tasks.Where(t => t.DueDate < DateTime.UtcNow);
```

Both mean overdue. One checks `IsPaid`. The other doesn't. When the business adds a grace period, one gets updated and the other doesn't. The system then reports different overdue counts in different places — a bug that looks like a data problem until someone reads both queries side by side.

```csharp
public bool IsOverdue() => DueDate < DateTime.UtcNow && !IsPaid;
```

One definition. Every caller gets the same answer.

---

## One method doing too much

Each line follows naturally from the previous. The role of the whole never comes into scope.

```csharp
public async Task<TaskDto> Handle(AssignTaskCommand cmd, CancellationToken ct)
{
    if (cmd.TaskId == Guid.Empty)
        throw new ArgumentException("TaskId required");

    var member = await _db.Members.FirstOrDefaultAsync(...);
    if (member?.Role != MemberRole.Admin && member?.Role != MemberRole.Lead)
        throw new ForbiddenException("Only admins can assign tasks");

    var task = await _db.Tasks.FindAsync(cmd.TaskId, ct);
    task.AssigneeId = cmd.AssigneeId;
    task.Status = TaskStatus.InProgress;
    await _db.SaveChangesAsync(ct);

    await _notifier.SendAsync(cmd.AssigneeId, $"You've been assigned to '{task.Title}'");
    return task.ToDto();
}
```

Validation, authorization, domain state change, persistence, notification — all at the same indentation level. Two years and twenty requirements later, this is the method nobody wants to touch.

```csharp
public async Task<TaskDto> Handle(AssignTaskCommand cmd, CancellationToken ct)
{
    var task = await _taskRepo.GetAsync(cmd.TaskId, ct);
    task.AssignTo(cmd.AssigneeId);
    await _db.SaveChangesAsync(ct);
    await _mediator.Publish(new TaskAssigned(task, cmd.AssigneeId), ct);
    return task.ToDto();
}
```

A coordinator reads like a table of contents. Each line names an intent. The details live one level down, where they belong.

---

## Knowing too much about others

```csharp
var canAssign = _projectService.GetProject(projectId)
    .Members.First(m => m.UserId == currentUserId)
    .Role == MemberRole.Admin;
```

Coupled to `Project`, its `Members` collection, `Member` structure, and the `Role` enum — to get a yes-or-no answer. Renaming `Role`, moving `Members` to a separate service, changing how membership works — all break a line that only needed to know one thing.

```csharp
var canAssign = _projectService.CanUserAssignTasks(projectId, currentUserId);
```

The same problem compounds when a class has no clear boundary:

```csharp
public class TaskService
{
    void Create(Task task);
    void Cancel(Guid taskId);
    void Assign(Guid taskId, Guid assigneeId);
    void GenerateReport(Guid taskId);
    void SendReminder(Guid taskId);
    void CalculatePriority(Task task);
}
```

Every developer working on anything task-related depends on it. Changes to reporting risk breaking assignment. Testing a reminder requires constructing a class that also knows about priority.

```csharp
public class TaskLifecycleService { }    // Create, Cancel, Assign
public class TaskReportingService { }    // Reports
public class TaskNotificationService { } // Reminders
```

Each class answers one question.

---

These failures are recognizable once you know what to look for. The cost of each is low when caught early and compounds quickly when missed. For the broader picture, [what makes code unmaintainable](/posts/what-makes-code-unmaintainable/) covers the factors these failures fall under.
