---
title: "Excel exports in .NET Core using ClosedXML"
date: 2023-08-12
dateUpdated: Last Modified
permalink: /posts/exporting-c-objects-to-excel-with-closedxml/
tags:
  - ASP.NET Core
  - Code Snippet
layout: layouts/post.njk
---

[ClosedXML](https://github.com/ClosedXML/ClosedXML) makes Excel exports straightforward in .NET without requiring Excel installation. Perfect for generating reports from any IEnumerable collection.

## Install package

```bash
dotnet add package ClosedXML
```

## Basic export to file

The simplest way to export any objects to Excel using ClosedXML's `InsertTable<T>()` method:

```csharp
IEnumerable<AuthorDto> items = GetAuthors();

using var wb = new XLWorkbook();
var ws = wb.AddWorksheet();

// Inserts the collection to Excel as a table with a header row.
ws.Cell("A1").InsertTable(items);

// Adjust column size to contents.
ws.Columns().AdjustToContents();

// Save to local file system.
var filename = $"Export - {DateTime.UtcNow:yyyyMMddHHmmss}.xlsx";
wb.SaveAs(filename);
```

## ASP.NET Core minimal API endpoint

Complete minimal API endpoint that returns a downloadable Excel file:

```csharp
app.MapGet("/export/authors", (IAuthorService authorService) =>
{
    var authors = authorService.GetAuthors();
    
    using var wb = new XLWorkbook();
    var ws = wb.AddWorksheet();
    ws.Cell("A1").InsertTable(authors);
    ws.Columns().AdjustToContents();

    using var stream = new MemoryStream();
    wb.SaveAs(stream);
    var content = stream.ToArray();
    var contentType = "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet";
    var filename = $"Authors - {DateTime.UtcNow:yyyyMMddHHmmss}.xlsx";

    return Results.File(content, contentType, filename);
});
```

## Example class

The `AuthorDto` can be any C# class object:

```csharp
public class AuthorDto
{
    public int Id { get; set; }
    public string? FirstName { get; set; }
    public string? LastName { get; set; }
    public string? ContactEmail { get; set; } = "";
}
```

This approach works well for simple exports. For advanced table customization, check the [tables feature](https://docs.closedxml.io/en/latest/features/tables.html) in the official docs.
