---
title: How edit forms should load dropdown data
pubDatetime: 2026-04-21
description: "Edit forms with multiple dropdowns usually trigger one API call per dropdown. Two problems follow — the form stays blank until all calls resolve, and load time scales with dropdown count. This note covers how to fix both: embed current selections in the entity response, aggregate option lists into one scoped endpoint. Two requests total, regardless of dropdown count."
slug: how-edit-forms-should-load-dropdown-data
tags: [practices, software-design]
draft: false
---

Most projects I've joined had the same edit form setup. One API call per dropdown, leaving the form blank while they all resolve. The more dropdowns, the longer the form takes to load.

## Problem 1: The form can't render until all dropdown calls resolve

The ticket detail response usually looks like this:

```json
{
  "id": "ticket-99",
  "title": "Fix login bug",
  "statusId": "status-2",
  "priorityId": "priority-1",
  "assigneeId": "user-42"
}
```

The frontend has IDs but not labels. It stays blank until the dropdown calls resolve. It needs those payloads to know what to display as the current dropdown selection.

Embed the selection objects alongside their IDs in the detail payload:

```json
{
  "id": "ticket-99",
  "title": "Fix login bug",
  "projectId": "project-4",
  "status": { "id": "status-2", "name": "In Progress" },
  "priority": { "id": "priority-1", "name": "High" },
  "assignee": { "id": "user-42", "name": "John Doe" }
}
```

The form can render immediately. Dropdown calls still fire in parallel - they just no longer block the initial render.

## Problem 2: N dropdowns means N round trips

```plain
GET /api/statuses?projectId=123
GET /api/priorities?projectId=123
GET /api/members?projectId=123
```

That's three round trips. Page load time scales with dropdown count. Collapse them into one endpoint scoped to the form:

```plain
GET /api/tickets/{id}/edit-form-data
```

```json
{
  "statuses": [...],
  "priorities": [...],
  "assignees": [...]
}
```

Dropdown count no longer affects page load time. Populate each list in parallel on the backend:

```csharp
var (statuses, priorities, assignees) = await (
  _statusService.GetLookupAsync(ticket.ProjectId),
  _priorityService.GetLookupAsync(ticket.ProjectId),
  _memberService.GetLookupAsync(ticket.ProjectId)
);
```

## The LookupDto shape

Every option list in the app returns this shape.

```csharp
public record LookupDto(
    string Id,
    string Name,
    string? Icon = null,
    string? Group = null,
    bool IsDisabled = false,
    Dictionary<string, object>? Metadata = null
);
```

## How this fits together

Two requests fire simultaneously when the user clicks Edit:

1. `GET /api/tickets/{id}` — ticket details with embedded current selections. Form renders immediately.
2. `GET /api/tickets/{id}/edit-form-data` — all option lists scoped to the ticket. Dropdowns become interactive when this resolves.

The embedded value and the option list serve different purposes. `ticket.status` embeds `{ id, name }` so the form renders with the right value showing. `edit-form-data.statuses` returns the full `LookupDto[]` so the user can change it. They don't overlap.

## Autocomplete without duplication

When the assignee list is large, pre-loading all of them isn't practical. The aggregate endpoint handles the common case; a dedicated search endpoint handles type-ahead. Both call the same service method:

```csharp
public interface IMemberService
{
  Task<List<LookupDto>> GetLookupAsync(
        Guid projectId,
        Guid? currentSelectionId = null,
        string? search = null,
        int? page = null,
        int pageSize = 50);
}
```

The aggregate endpoint passes the current assignee ID. The method guarantees it appears in the result regardless of its position:

```csharp
assignees = await _memberService.GetLookupAsync(
  ticket.ProjectId,
    currentSelectionId: ticket.AssigneeId,
    pageSize: 50);
```

The autocomplete endpoint passes query parameters through:

```plain
GET /api/members/lookup?projectId=abc&search=jan&page=2
```

The frontend renders the pre-loaded list. If the user types, it switches to the search endpoint.

---

Two requests regardless of dropdown count. One for the entity, one for the options. The form renders immediately with the right values showing, and the dropdowns become interactive when the options arrive.
