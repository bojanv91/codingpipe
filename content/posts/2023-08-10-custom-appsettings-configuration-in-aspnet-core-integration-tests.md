---
title: "Test-Specific AppSettings Configuration in ASP.NET Core integration tests"
date: 2023-08-10
dateUpdated: Last Modified
permalink: /posts/test-specific-appsettings-in-aspnet-core-integration-tests/
tags:
  - ASP.NET Core
  - Testing
layout: layouts/post.njk
---

When writing integration tests for ASP.NET Core applications, I've run into a common issue: the TestServer uses the main application's `appsettings.json` files by default. This becomes problematic when tests need different configuration values - like pointing to a test database instead of production.

Here's how I configure integration tests to use their own configuration files.

Let's create a test-specific `appsettings.json` file in your integration test project:

```plpaintext
Solution/
├── PlaygroundApi/
│   ├── appsettings.json          # Main app config
│   ├── appsettings.Development.json
│   └── Program.cs
└── PlaygroundApi.Tests/
    ├── appsettings.json          # Test-specific config
    └── IntegrationTests.cs
```

Main app appsettings.json:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=ProductionDb;Trusted_Connection=true;"
  },
  "ApiToken": "prod-api-key-123"
}
```

Test project appsettings.json:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Host=localhost;Port=5432;Database=TestDb;Username=postgres;Password=test123"
  },
  "ApiToken": "test-api-key-456"
}
```

Add this minimal API endpoint to return the configuration value:

```csharp
app.MapGet("/api-token", (IConfiguration config) => config["ApiToken"]);
```

For the test config file, set these properties in Visual Studio:

- **Build Action**: Content
- **Copy to output directory**: Copy if newer

This ensures the configuration file is available in your test output directory.

Now configure your `WebApplicationFactory` to use the test project's configuration:

```csharp
using Microsoft.AspNetCore.Mvc.Testing;
using Microsoft.Extensions.Configuration;

namespace PlaygroundApi.Tests
{
    public class IntegrationTests
    {
        private readonly WebApplicationFactory<PlaygroundApi.Program> _webAppFactory;

        public IntegrationTests()
        {
            _webAppFactory = new WebApplicationFactory<PlaygroundApi.Program>()
                .WithWebHostBuilder(builder =>
                {
                    builder.ConfigureAppConfiguration((context, config) =>
                    {
                        // Clear existing configuration to avoid conflicts
                        config.Sources.Clear();
                        
                        // Set the base path to the integration tests build output directory
                        // where the integration tests' config file will be copied into.
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

            // Assert that the test-specific configuration value is returned
            Assert.Equal("test-api-key-456", result);
        }
    }
}
```

I prefer this approach over environment variables or in-memory configuration because it keeps test settings explicit and version-controlled. You can easily see what configuration your tests are using, and different team members will have consistent test behavior.

The `config.Sources.Clear()` call ensures we don't inherit any configuration from the main application, and `SetBasePath(Directory.GetCurrentDirectory())` tells the configuration system to look in the test project's output directory.

## Using with Testcontainers

When using Testcontainers, you can dynamically override the connection string after the container starts:

```csharp
public class IntegrationTestsWithContainer : IAsyncLifetime
{
    private readonly PostgreSqlContainer _postgreSqlContainer = new PostgreSqlBuilder().Build();
    private WebApplicationFactory<PlaygroundApi.Program> _webAppFactory;

    public async Task InitializeAsync()
    {
        await _postgreSqlContainer.StartAsync();
        
        _webAppFactory = new WebApplicationFactory<PlaygroundApi.Program>()
            .WithWebHostBuilder(builder =>
            {
                builder.ConfigureAppConfiguration((context, config) =>
                {
                    config.Sources.Clear();
                    config.SetBasePath(Directory.GetCurrentDirectory());
                    config.AddJsonFile("appsettings.json");
                    
                    // Override connection string with container's connection string
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
