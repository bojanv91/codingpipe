---
title: "Building dynamic SQL queries: string concatenation vs SqlKata"
date: 2023-01-23
dateUpdated: Last Modified
permalink: /posts/building-dynamic-sql-queries-in-dotnet-using-sqlkata/
tags:
  - ASP.NET Core
  - Data Access
layout: layouts/post.njk
---

There are two common approaches to building dynamic SQL queries in C# application code. One uses string concatenation and executes the query using libraries such as [Dapper](https://github.com/DapperLib/Dapper/blob/main/Readme.md). The other uses expressions-based query builder libraries such as [SqlKata](https://sqlkata.com/).

In this post, we'll build a typical search query that can filter by any optional search parameters. Then, it'll return paginated results with a total count. We'll attach parameter-based filter conditions in the "where" clause only when valid values are passed for those parameters. We'll exclude `NULL` or empty array parameters since those values are invalid for our filter criteria.

Let's look at the typical implementations written in .NET application code.

## Query building with string concatenation

In this example, we're building a base query from which we'll build the list and count queries. Finally, we're using Dapper to execute both queries. We use [ContosoUniversity](https://learn.microsoft.com/en-us/aspnet/core/data/ef-mvc/complex-data-model/_static/diagram.png?view=aspnetcore-7.0) database as a referenced database example.

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

Here are a couple of things we're keeping an eye on while building such queries:
- making sure we're placing whitespaces in the correct places before and after concatenating the strings; it's easy to mess this up if not careful;
- opting in for using the `WHERE 1 = 1` pattern to which we are attaching parameter-based filter conditions with ease, instead of using the other common alternative option with `AND (@Param is NULL or Name = @Param)`
- making sure we're adding SQL parameters in the SQL query string (e.g.,`" and Name = @Param"`), and we're not adding the parameter's values into the string with concatenation (not like this: `$" and Name = '{request.Name}'"`) which opens our code to SQL injection.

## Query building with SqlKata

This example builds on top of the string concatenation example - you can view it as a rewritten version of it only by using only [SqlKata](https://sqlkata.com/).

SqlKata is an expression-based query-building library for C#. I've used it in many projects with great success. Use cases vary, from building ad-hoc dynamic SQL queries for reports and charts, to advanced user-generated reports and dashboard builders.

Let's look at the SqlKata-version of our search query.

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

  // Built and execute the list query with pagination on top of the base query.
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

SqlKata's `Query` instances are mutable. So, we have to remember to use `.Clone()` before we change a query for usage in many queries built on top of the original query. Just like how we've used it in our example above.

## SqlKata usage references

SqlKata's [documentation](https://sqlkata.com/docs) is rich with examples. There is also a [SqlKata Playground](https://sqlkata.com/playground) to quickly try out code samples and see their output for multiple database providers.

**Generating a SQL string from a SqlKata query**
```csharp
var query = new Query("Course")
  .Where("Published", true)
  .Where("Credits", 5);

var compiler = new PostgresCompiler();
string sql = compiler.Compile(query).Sql;
// returns: SELECT * FROM "Course" WHERE "Published" = @p0 AND "Credits" = @p1
```
See the [compiler only examples](https://sqlkata.com/docs/#compile-only-example) for more details.

**Using [SelectRaw](https://sqlkata.com/docs/select#raw) for accessing Postgres JSONB columns**
```csharp
var query = new Query("User")
  .Select("fullname as Fullname")
  .SelectRaw("data->'BillingAddress'->>'City' as BillingCity");
```
There are also similar accompanied methods: `WhereRaw`, `FromRaw`, etc.

**Nested conditions in the Where clause**
```csharp
var query = new Query("User")
  .Where("Enabled", true)
  .Where(q => q.Where("Role", "Manager").OrWhere("Role", "Member"));

// SELECT * FROM "User" 
// WHERE "Enabled" = true AND ("Role" = 'Manager' OR "Role" = 'Member')
```
