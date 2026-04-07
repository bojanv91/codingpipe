---
title: What I fix after the LLM is done
pubDatetime: 2026-04-07
description: "The design problems LLM-generated code produces, and how to fix them."
slug: what-i-fix-after-the-llm-is-done
tags: [practices, software-design]
draft: false
---

LLM-generated code can be correct and still be badly structured. It may compile and pass the tests, yet make future changes broader and messier than they need to be.

I deal with that code a lot, both when generating it and when cleaning it up afterward. The issues are repetitive enough to recognize them as familiar design problems.

Most of the failures come back to two design concerns: cohesion and coupling. Cohesion is about keeping behavior close to the data and responsibilities it belongs with. Coupling is about limiting how much one part of the system needs to know about another. LLM-generated code tends to miss both because generation is local. Each line follows sensibly from the previous one, while the design starts to weaken at the boundaries between classes.

---

Cohesion problems often show up as methods doing most of their work on another class's data.

```csharp
public async Task NotifyOverdue(Guid projectId)
{
    var tasks = await _db.Tasks
        .Where(t => t.ProjectId == projectId)
        .ToListAsync();

    var overdue = tasks
        .Where(t => t.DueDate < DateTime.UtcNow && t.Status != TaskStatus.Completed)
        .ToList();

    foreach (var task in overdue)
        await _notifier.Send(task.AssigneeId, $"Task '{task.Title}' is overdue");
}
```

What counts as "overdue" lives in a notification service. That rule can't be reused without copying it. And it will be copied — slightly differently every time. The fix comes with the duplication.

That copying produces duplication no search tool will catch — not grep, not your IDE, not an LLM scanning the codebase.

```csharp
// In TaskQueryService:
public async Task<List<ProjectTask>> GetOverdueTasksAsync(Guid projectId)
    => await _db.Tasks
        .Where(t => t.ProjectId == projectId && t.DueDate < DateTime.UtcNow)
        .ToListAsync();

// In TaskReminderJob:
var overdue = await _db.Tasks
    .Where(t => t.DueDate < DateTime.UtcNow && t.Status != TaskStatus.Completed)
    .ToListAsync();
```

Same concept, two implementations. One checks `DueDate < now`. The other adds `Status != Completed`. They've already diverged. When the business adds a grace period, one gets updated and the other doesn't. You won't find this with search because the code looks different. You only find it when you understand what both pieces actually mean.

```csharp
public bool IsOverdue => DueDate.HasValue
    && DueDate.Value < DateTime.UtcNow
    && Status != TaskStatus.Completed;
```

Keeping that rule in one place removes the duplication and lets the notification service rely on the task's own definition of overdue.

The Flutter version shows up in display logic:

```dart
// TaskTile:
final isUrgent = task.priority == Priority.p1;

// TaskDetailHeader:
final isHighPriority = task.priority == Priority.p1 || task.priority == Priority.p2;
```

Two widgets, two thresholds, same concept. Neither linter catches it.

```dart
extension TaskPriorityDisplay on Priority {
  bool get isUrgent => this == Priority.p1;
  bool get isElevated => this == Priority.p1 || this == Priority.p2;
  Color get flagColor => isUrgent ? Colors.red : Colors.grey;
}
```

---

Coupling problems are usually more expensive because they spread knowledge and decisions across the wrong places. One common case is caller code making decisions that belong to the object being acted on.

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

The caller is inspecting the task's state to decide what the task is allowed to do. That rule — what makes a task assignable — now lives in the caller. The next time I generate something similar, it comes out slightly differently. One checks `Completed`, another adds `Cancelled`, a third forgets `Archived`. Six months later there are three diverging definitions of the same concept scattered across the codebase.

The task knows its own state. Let it make the call:

```csharp
// Task entity:
public void AssignTo(Guid assigneeId)
{
    if (Status is TaskStatus.Completed or TaskStatus.Cancelled)
        throw new InvalidOperationException("Cannot assign a closed task");
    AssigneeId = assigneeId;
}

// Caller:
task.AssignTo(assigneeId);
```

If code has to inspect an object's state to decide what that object is allowed to do, the decision usually belongs inside the object.

The same thing shows up in Flutter in a way that's easy to miss because passing state down through the widget tree feels natural:

```dart
class TaskListScreen extends StatelessWidget {
  Widget build(BuildContext context) {
    final tasks = context.watch<TaskListCubit>().state.tasks;
    final hasMultipleStatuses = tasks.map((t) => t.status).toSet().length > 1;
    return Column(children: [
      TaskFilterBar(isVisible: hasMultipleStatuses),
      TaskListView(),
    ]);
  }
}
```

`TaskListScreen` now has to understand when filtering makes sense. That's the filter bar's concern. If the rule changes — show filters when there are 5+ tasks, or when the user toggled a preference — I change the parent, not the widget that owns the behavior.

```dart
class TaskFilterBar extends StatelessWidget {
  Widget build(BuildContext context) {
    final tasks = context.watch<TaskListCubit>().state.tasks;
    final hasMultipleStatuses = tasks.map((t) => t.status).toSet().length > 1;
    if (!hasMultipleStatuses) return const SizedBox.shrink();
    return const _FilterChips();
  }
}

class TaskListScreen extends StatelessWidget {
  Widget build(BuildContext context) => Column(children: [
        TaskFilterBar(),
        TaskListView(),
      ]);
}
```

The widget observes and decides for itself. The parent composes.

---

A separate coupling problem appears when code depends on internal structure it does not need to know about.

```csharp
var canAssign = _projectService.GetProject(projectId)
    .Members.First(m => m.UserId == currentUserId)
    .Role == MemberRole.Admin;
```

This is coupled to `Project`, its `Members` collection, individual `Member` structure, and the `Role` enum — to get a yes-or-no answer. If `Members` moves to a separate service, or `Role` gets renamed, this line breaks even though it only needed to know whether this user can assign tasks.

```csharp
var canAssign = _projectService.CanUserAssignTasks(projectId, currentUserId);
```

This is Law of Demeter. Tell, Don't Ask is about behavior living in the wrong place; this is about knowing too much about the wrong structure.

The Flutter version is a service reaching directly into a cubit's state — a cubit being the state management object the widget tree observes:

```dart
class TaskReorderService {
  void moveToTop(TaskListCubit cubit, String taskId) {
    final index = cubit.state.tasks.indexWhere((t) => t.id == taskId);
    final task = cubit.state.tasks.removeAt(index);
    cubit.state.tasks.insert(0, task);
    cubit.emit(cubit.state.copyWith(tasks: cubit.state.tasks));
  }
}
```

`TaskReorderService` and `TaskListCubit` are effectively one class wearing two names. Any refactor of the cubit's internal state shape breaks the service.

```dart
// TaskListCubit exposes intent:
void moveTaskToTop(String taskId) {
  final tasks = List<Task>.from(state.tasks);
  final index = tasks.indexWhere((t) => t.id == taskId);
  final task = tasks.removeAt(index);
  tasks.insert(0, task);
  emit(state.copyWith(tasks: tasks));
}
```

Expose intent, hide implementation. If an outside class needs to manipulate your internals to get something done, that operation belongs inside you.

---

The most diagnostic question I ask about any generated class: what is this class?

```csharp
public async Task<TaskDto> Handle(AssignTaskCommand cmd, CancellationToken ct)
{
    if (cmd.TaskId == Guid.Empty)
        throw new ArgumentException("TaskId required");

    var member = await _db.Members.FirstOrDefaultAsync(...);
    if (member?.Role != MemberRole.Admin && member?.Role != MemberRole.Lead)
        throw new ForbiddenException("Only admins and leads can assign tasks");

    var task = await _db.Tasks.FindAsync(cmd.TaskId, ct);
    task.AssigneeId = cmd.AssigneeId;
    task.Status = TaskStatus.InProgress;
    await _db.SaveChangesAsync(ct);

    await _notifier.SendAsync(cmd.AssigneeId, $"You've been assigned to '{task.Title}'");
    return task.ToDto();
}
```

This class is handling validation, authorization, querying, and coordination at the same time. That mix is common in generated code because each line follows naturally from the previous one, even as the method accumulates unrelated responsibilities. The result may still work, but future changes to validation, authorization, domain rules, or side effects all end up touching the same method.

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

A class (or method) is easier to work with when it has one clear role. If describing it requires several unrelated responsibilities, the design is already doing too much.

---

These principles are guidelines, and real codebases always contain exceptions. Their value is in making tradeoffs visible, so exceptions stay intentional instead of becoming accidental design drift.
