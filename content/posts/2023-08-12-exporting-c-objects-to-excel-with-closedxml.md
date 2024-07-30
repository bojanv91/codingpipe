---
title: "Excel Export with ClosedXML in C# and .NET Core"
date: 2023-08-12
dateUpdated: Last Modified
permalink: /posts/exporting-c-objects-to-excel-with-closedxml/
tags:
  - .NET Core
layout: layouts/post.njk
---

Used library: ClosedXML ([docs site](https://docs.closedxml.io); [github](https://github.com/ClosedXML/ClosedXML))

The simplest way to export any objects IEnumerable to Excel is by using ClosedXML's `IXLTable.InsertTable<T>(IEnumerable<T> data)` method.

```cs
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

In ASP.NET you can use the following code to return a downloadable Excel file:

```cs
// <-- the above code -->
using var stream = new MemoryStream();
wb.SaveAs(stream);
var content = stream.ToArray();
var contentType = "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet";
return File(content, contentType, filename);
```

The `AuthorDto` can be any C# class object. Here is this example's class:

```cs
public class AuthorDto
{
    public int Id { get; set; }
    public string? FirstName { get; set; }
    public string? LastName { get; set; }
    public string? ContactEmail { get; set; } = "";
}
```

And here is the generated Excel table:

![](/img/2023-08-12-exporting-to-excel-with-closedxml-in-dotnet/img01-excel-file-sample-table.png)

For further table customization check the [tables feature](https://docs.closedxml.io/en/latest/features/tables.html) in the official docs.
