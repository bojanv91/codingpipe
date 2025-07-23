---
title: "Test-specific AppSettings configuration in ASP.NET Core integration tests"
date: 2023-08-10
dateUpdated: Last Modified
permalink: /posts/test-specific-appsettings-in-aspnet-core-integration-tests/
tags:
  - ASP.NET Core
  - Testing
layout: layouts/post.njk
---

Integration tests that use production configuration values are accidents waiting to happen. The TestServer defaults to your main application's `appsettings.json`, which means tests might hit production databases or external APIs.

Here's how to configure tests to use their own configuration files.

## Setup

Create a test-specific `appsettings.json` in your test project:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Host=localhost;Port=5432;Database=TestDb;Username=postgres;Password=test123"
  },
  "ApiToken": "test-api-key-456"
}
```

Set the file properties to **Build Action: Content** and **Copy to Output Directory: Copy if newer**.

Add this minimal API endpoint to your main app for testing:

```csharp
app.MapGet("/api-token", (IConfiguration config) => config["ApiToken"]);
```

Configure your `WebApplicationFactory` to use the test configuration:

```csharp
using Microsoft.AspNetCore.Mvc.Testing;
using Microsoft.Extensions.Configuration;

namespace PlaygroundApi.Tests
{
    public class IntegrationTests
    {
        private readonly WebApplicationFactory<Program> _webAppFactory;

        public IntegrationTests()
        {
            _webAppFactory = new WebApplicationFactory<Program>()
                .WithWebHostBuilder(builder =>
                {
                    builder.ConfigureAppConfiguration((context, config) =>
                    {
                        config.Sources.Clear();
                        config.SetBasePath(Directory.GetCurrentDirectory());
                        config.AddJsonFile("appsettings.json");
                    });
                });
        }

        [Fact]
        public async Task ShouldUseTestConfiguration()
        {
            var httpClient = _webAppFactory.CreateClient();
            var result = await httpClient.GetStringAsync("/api-token");
            
            Assert.Equal("test-api-key-456", result);
        }
    }
}
```

## Dynamic Configuration Override

For Testcontainers or other runtime configuration needs:

```csharp
public class ContainerizedTests : IAsyncLifetime
{
    private readonly PostgreSqlContainer _postgreSqlContainer = new PostgreSqlBuilder().Build();
    private WebApplicationFactory<Program> _webAppFactory;

    public async Task InitializeAsync()
    {
        await _postgreSqlContainer.StartAsync();
        
        _webAppFactory = new WebApplicationFactory<Program>()
            .WithWebHostBuilder(builder =>
            {
                builder.ConfigureAppConfiguration((context, config) =>
                {
                    config.Sources.Clear();
                    config.SetBasePath(Directory.GetCurrentDirectory());
                    config.AddJsonFile("appsettings.json");
                    
                    // Override with container connection string
                    config.AddInMemoryCollection(new Dictionary<string, string>
                    {
                        ["ConnectionStrings:DefaultConnection"] = _postgreSqlContainer.GetConnectionString()
                    });
                });
            });
    }

    [Fact]
    public async Task ShouldConnectToTestContainer()
    {
        var httpClient = _webAppFactory.CreateClient();
        
        // Your test logic here using the containerized database
    }

    public async Task DisposeAsync()
    {
        await _postgreSqlContainer.DisposeAsync();
        _webAppFactory?.Dispose();
    }
}
```

## Why This Works

`config.Sources.Clear()` prevents inheriting production configuration. `SetBasePath(Directory.GetCurrentDirectory())` points to the test output directory where your test `appsettings.json` gets copied. This approach keeps test configuration explicit and version-controlled rather than relying on environment variables or hardcoded values.
