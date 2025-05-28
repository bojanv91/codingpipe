---
title: "Dynamic SQL queries: string concatenation vs SqlKata"
date: 2023-01-23
dateUpdated: Last Modified
permalink: /posts/building-dynamic-sql-queries-in-dotnet-using-sqlkata/
tags:
  - ASP.NET Core
  - Data Access
layout: layouts/post.njk
---

When building search features with multiple optional filters, you need dynamic SQL queries that adapt based on user input. I've encountered this requirement countless times - users want to search by name, filter by department, select multiple instructors, or combine any of these criteria.

Two approaches are commonly used: string concatenation with libraries like [Dapper](https://github.com/DapperLib/Dapper/blob/main/Readme.md), and expression-based builders like [SqlKata](https://sqlkata.com/). Here's how both approaches handle a typical search scenario with pagination and filtering.

## Query building with string concatenation

I'll start with the string concatenation approach using Dapper. We're building a search query that filters courses by optional parameters and returns paginated results with a total count:

```csharp
// This is the filter request object that the user passes to the query
public class QueryRequest
{
  public string? SearchText { get; set; }
  public int? DepartmentId { get; set; }
  public int[]? InstructorIds { get; set; }
  public int SkipCount { get; set; } // for offset
  public int TakeCount { get; set; } // for limit
}

public QueryResponse QueryCoursesStringConcat(QueryRequest request, IDbConnection dbConnection)
{
  // Build the base query that tries to search by all optional parameters if provided,
  // or returns all results. It's used in building the list and count queries bellow.
  var query = new StringBuilder();
  query.Append(@"
from Course
inner join Department on Department.ID = Course.DepartmentID
left join CourseAssignment on CourseAssignment.CourseID = Course.ID
left join Instructor on Instructor.ID = CourseAssignment.InstructorID
left join Enrollment on Enrollment.CourseID = Course.ID
WHERE 1 = 1
");
  if (!string.IsNullOrEmpty(request.SearchText))
    query.Append(" and (Course.Title ilike @SearchText or Instructor.FullName ilike @SearchText)");
  if (request.DepartmentId.HasValue)
    query.Append(" and Department.ID = @DepartmentID");
  if (request.InstructorIds?.Any() == true)
    query.Append(" and Instructor.ID = any(@InstructorIds)");

  // Built the list query with pagination on top of the base query.
  var listQuery = @"
select 
Course.Title as CourseTitle,
Course.Credits as CourseCredits,
Department.Name as DepartmentName,
Instructor.LastName as InstructorName "
+ query.ToString() + " OFFSET @SkipCount LIMIT @TakeCount";

  // Execute the list query using Dapper
  var items = dbConnection.Query<ResponseItem>(listQuery, new
  {
    SearchText = request.SearchText,
    DepartmentID = request.DepartmentId,
    InstructorIds = request.InstructorIds,
    SkipCount = request.SkipCount,
    TakeCount = request.TakeCount
  });

  // Build the count query.
  var countQuery = "select count(*) " + query;
  // Execute the count query using Dapper
  var totalCount = dbConnection.ExecuteScalar<int>(countQuery, new
  {
    SearchText = request.SearchText,
    DepartmentID = request.DepartmentId,
    InstructorIds = request.InstructorIds
  });

  return new QueryResponse
  {
    Items = items,
    TotalCount = totalCount
  };
}
```

I've learned to watch for several things when building queries this way: proper whitespace placement around concatenated strings, using `WHERE 1 = 1` pattern for cleaner conditional logic, and always using parameterized queries instead of string interpolation to prevent SQL injection.

## Query building with SqlKata

[SqlKata](https://sqlkata.com/) provides a fluent, expression-based approach. I've used it successfully across many projects for everything from simple search queries to complex report builders:

```csharp
public QueryResponse QueryCoursesSqlKata(QueryRequest request, QueryFactory dbQueryFactory)
{
  // Build the base query that tries to search by all optional parameters if provided,
  // or returns all results. It's used in building the list and count queries bellow.
  var query = dbQueryFactory.Query("Course")
    .Join("Department", "Department.ID", "Course.DepartmentID")
    .LeftJoin("CourseAssignment", "CourseAssignment.CourseID", "Course.ID")
    .LeftJoin("Instructor", "Instructor.ID", "CourseAssignment.InstructorID")
    .LeftJoin("Enrollment", "Enrollment.CourseID", "Course.ID");

  if (!string.IsNullOrEmpty(request.SearchText))
    query.Where(x => x
        .WhereLike("Course.Title", request.SearchText)
        .OrWhereLike("Instructor.FullName", request.SearchText));
  if (request.DepartmentId.HasValue)
    query.Where("Department.ID", request.DepartmentId);
  if (request.InstructorIds?.Any() == true)
    query.WhereIn("Instructor.ID", request.InstructorIds);

  // Build and execute the list query with pagination on top of the base query.
  var items = query.Clone()
    .Select(
      "Course.Title as CourseTitle",
      "Course.Credits as CourseCredits",
      "Department.Name as DepartmentName",
      "Instructor.LastName as InstructorName")
    .Offset(request.SkipCount)
    .Limit(request.TakeCount)
    .Get<ResponseItem>();

  // Build and execute the count query.
  var totalCount = query.Clone().Count<int>();

  return new QueryResponse
  {
    Items = items,
    TotalCount = totalCount
  };
}
```

Remember that SqlKata `Query` instances are mutable, so use `.Clone()` before modifying a query that you'll reuse for different purposes.

## When to use each approach

The main advantage of SqlKata is eliminating string concatenation errors, providing better IntelliSense support, and making complex conditional logic more readable. String concatenation requires careful attention to whitespace placement and can become unwieldy with complex dynamic conditions.

However, string concatenation gives you direct control over the generated SQL, which can be useful when you need to fine-tune specific query optimizations or use database-specific features that the query builder doesn't support.

For most dynamic query scenarios, I prefer SqlKata because it reduces bugs and makes the code more maintainable, especially as query complexity grows.

## SqlKata usage references

SqlKata's [documentation](https://sqlkata.com/docs) is rich with examples. There is also a [SqlKata Playground](https://sqlkata.com/playground) to quickly try out code samples and see their output for multiple database providers.

### Generating a SQL string from a SqlKata query

```csharp
var query = new Query("Course")
  .Where("Published", true)
  .Where("Credits", 5);

var compiler = new PostgresCompiler();
string sql = compiler.Compile(query).Sql;
// returns: SELECT * FROM "Course" WHERE "Published" = @p0 AND "Credits" = @p1
```

See the [compiler only examples](https://sqlkata.com/docs/#compile-only-example) for more details.

### Using [SelectRaw](https://sqlkata.com/docs/select#raw) for accessing Postgres JSONB columns

```csharp
var query = new Query("User")
  .Select("fullname as Fullname")
  .SelectRaw("data->'BillingAddress'->>'City' as BillingCity");
```

There are also similar accompanied methods: `WhereRaw`, `FromRaw`, etc.

### Nested conditions in the Where clause

```csharp
var query = new Query("User")
  .Where("Enabled", true)
  .Where(q => q.Where("Role", "Manager").OrWhere("Role", "Member"));

// SELECT * FROM "User" 
// WHERE "Enabled" = true AND ("Role" = 'Manager' OR "Role" = 'Member')
```
