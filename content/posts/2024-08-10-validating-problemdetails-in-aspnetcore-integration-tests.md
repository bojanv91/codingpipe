---
title: "Testing ProblemDetails responses in ASP.NET Core"
date: 2024-08-10
dateUpdated: Last Modified
permalink: /posts/testing-problemdetails-in-aspnetcore-integration-tests/
tags:
  - ASP.NET Core
  - Testing
layout: layouts/post.njk
---

When building APIs that return structured error responses using ProblemDetails, you need integration tests that validate both the HTTP status codes and the error response structure. I've found that testing only status codes misses important validation errors and business rule violations that clients depend on.

To read more on **ProblemDetails**, check [RFC9457](https://www.rfc-editor.org/rfc/rfc9457) and [ProblemDetails in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/error-handling?view=aspnetcore-8.0#problem-details).

## Setting up the test dependencies

First, ensure you have the necessary NuGet packages installed in your test project:

```bash
dotnet add package Microsoft.AspNetCore.Mvc.Testing
dotnet add package xunit
dotnet add package Shouldly
dotnet add package Flurl.Http
```

## Creating the test fixture

Implement the test fixture that starts the API project in memory within the integration tests:

```csharp
using Microsoft.AspNetCore.Mvc.Testing;
using Flurl.Http;
using Playground.WebApi;

namespace Playground.IntegrationTests 
{
  public class ApiTestFixture: WebApplicationFactory<Program>, IAsyncLifetime 
  {
    private readonly FlurlClient _client;

    public ApiTestFixture() 
    {
      _client = new FlurlClient(CreateClient());
    }

    public FlurlClient Client => _client;

    public async Task InitializeAsync() 
    {
      // Perform any initialization here if needed
      await Task.CompletedTask;
    }

    public new async Task DisposeAsync() 
    {
      _client.Dispose();
      await base.DisposeAsync();
    }
  }
}
```

## Writing the test cases

Now you can write test cases that validate both successful responses and ProblemDetails error responses:

```csharp
using Flurl.Http;
using Microsoft.AspNetCore.Mvc;
using Shouldly;
using System.Net;

namespace Playground.IntegrationTests.ApiTests 
{
  public class StudentTests: IClassFixture<ApiTestFixture>
  {
    private readonly FlurlClient _client;
    private const string BaseUrl = "http://localhost/api";

    public StudentTests(ApiTestFixture fixture)
    {
      _client = fixture.Client;
    }

    private IFlurlRequest Request(string url) => _client.Request(BaseUrl + url);
  {
    [Fact]
    public async Task Create_student_with_valid_input() 
    {
      // Arrange
      var studentData = new 
      {
        FirstName = "John",
        LastName = "Doe",
        EnrollmentDate = DateTime.UtcNow
      };

      // Act
      var response = await Request("/students").PostJsonAsync(studentData);

      // Assert
      response.StatusCode.ShouldBe((int) HttpStatusCode.OK);
    }

    [Fact]
    public async Task Create_student_fails_when_first_name_is_not_provided() 
    {
      // Arrange
      var studentWithEmptyFirstName = new 
      {
        FirstName = "",
        LastName = "A",
        EnrollmentDate = DateTime.UtcNow
      };

      // Act
      var exception = await Should.ThrowAsync<FlurlHttpException>(async() => {
        await Request("/students").PostJsonAsync(studentWithEmptyFirstName);
      });

      // Assert
      exception.StatusCode.ShouldBe((int) HttpStatusCode.BadRequest);
      var errorResponse = await exception.GetResponseJsonAsync<ValidationProblemDetails>();
      errorResponse.ShouldNotBeNull();
      errorResponse.Status.ShouldBe((int)HttpStatusCode.BadRequest);
      errorResponse.Title.ShouldBe("One or more validation errors occurred.");
      errorResponse.Errors.ShouldContainKey("FirstName");
      errorResponse.Errors["FirstName"].ShouldContain("The FirstName field is required.");
    }

    [Fact]
    public async Task Create_student_fails_with_submitted_reserved_name()
    {
      // Arrange
      var studentWithReservedName = new 
      {
        FirstName = "James",
        LastName = "Bond",
        EnrollmentDate = DateTime.UtcNow
      };

      // Act
      var exception = await Should.ThrowAsync<FlurlHttpException>(async() => {
        await Request("/students").PostJsonAsync(studentWithReservedName);
      });

      // Assert
      exception.StatusCode.ShouldBe((int) HttpStatusCode.BadRequest);
      var problemDetails = await exception.GetResponseJsonAsync<ValidationProblemDetails>();
      problemDetails.ShouldNotBeNull();
      problemDetails.Status.ShouldBe((int)HttpStatusCode.BadRequest);
      problemDetails.Title.ShouldBe("Cannot create a student named James Bond because it's a reserved name.");
    }

    [Fact]
    public async Task Create_student_fails_when_EnrollmentDate_is_too_far_in_the_past()
    {
      // Arrange
      var tooOldDate = DateTime.UtcNow.AddYears(-3); // 3 years ago
      var studentWithTooOldEnrollmentDate = new 
      {
        FirstName = "John",
        LastName = "Doe",
        EnrollmentDate = tooOldDate
      };

      // Act
      var exception = await Should.ThrowAsync<FlurlHttpException>(async() => {
        await Request("/students").PostJsonAsync(studentWithTooOldEnrollmentDate);
      });

      // Assert
      exception.StatusCode.ShouldBe((int) HttpStatusCode.BadRequest);
      var problemDetails = await exception.GetResponseJsonAsync<ValidationProblemDetails>();
      problemDetails.ShouldNotBeNull();
      problemDetails.Status.ShouldBe((int)HttpStatusCode.BadRequest);
      problemDetails.Title.ShouldBe("One or more validation errors occurred.");
      problemDetails.Errors.ShouldContainKey("EnrollmentDate");
      problemDetails.Errors["EnrollmentDate"].ShouldContain("Enrollment date cannot be more than 500 days in the past.");
    }
  }
```

This approach validates both the HTTP status codes and the structure of ProblemDetails error responses. Testing the complete error response structure helps catch issues where your API returns the wrong status code or malformed error details that break client error handling.
