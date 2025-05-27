---
title: "Testing Error Responses in ASP.NET Core using ProblemDetails"
date: 2024-08-10
dateUpdated: Last Modified
permalink: /posts/validating-problemdetails-in-aspnetcore-integration-tests/
tags:
  - ASP.NET Core
  - Testing
layout: layouts/post.njk
---

This post describes how to write xUnit integration tests for an ASP.NET Core API project that uses the **ProblemDetails** error response format.

To read more on **ProblemDetails**, check [RFC9457](https://www.rfc-editor.org/rfc/rfc9457) and [ProblemDetails in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/error-handling?view=aspnetcore-8.0#problem-details).

First, ensure you have the necessary NuGet packages installed in your test project. You can install them using the following .NET CLI commands in the command line:

```bash
dotnet add package Microsoft.AspNetCore.Mvc.Testing
dotnet add package xunit
dotnet add package Shouldly
dotnet add package Flurl.Http
```

Now, implement the test fixture that starts the API project in memory within the integration tests:

```csharp
using Microsoft.AspNetCore.Mvc.Testing;
using Flurl.Http;
using ContosoUniversity.WebApi;

namespace ContosoUniversity.IntegrationTests {
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

The above code uses **WebApplicationFactory<ContosoUniversity.WebApi.Program>** to start the API in-memory for testing.

Next, create a base class for your test classes that will configure the **Flurl.Http** client:

```csharp
using Flurl.Http;

namespace ContosoUniversity.IntegrationTests 
{
  public abstract class BaseIntegrationTest: IClassFixture<ApiTestFixture> 
  {
    protected readonly FlurlClient Client;
    protected const string BaseUrl = "http://localhost/api";

    protected IntegrationTestBase(ApiTestFixture fixture) 
    {
      Client = fixture.Client;
    }

    protected IFlurlRequest Request(string url) => Client.Request(BaseUrl + url);
  }
}
```

The **BaseIntegrationTest** class provides common functionality and configuration for all API test classes.

Now, write the test cases for managing students data:

```csharp
using Flurl.Http;
using Microsoft.AspNetCore.Mvc;
using Shouldly;
using System.Net;

namespace ContosoUniversity.IntegrationTests.ApiTests 
{
  public class StudentTests(ApiTestFixture fixture): BaseIntegrationTest(fixture)
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

      // Act`
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
    public async Task Create_student_fails_with_submited_reserved_name()
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
}
```

These tests check not only the status code but also the structure of the ProblemDetails error response where applicable.
