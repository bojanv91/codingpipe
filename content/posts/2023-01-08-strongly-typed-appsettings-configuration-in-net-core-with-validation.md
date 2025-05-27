---
title: "Type-Safe Configuration in .NET Core"
date: 2023-01-08
dateUpdated: Last Modified
permalink: /posts/type-safe-configuration-in-net-core/
tags:
  - ASP.NET Core
layout: layouts/post.njk
---

Working with configuration in .NET applications, I've seen too many runtime errors caused by typos in configuration keys or missing values that should have been caught earlier. Here's how I use strongly-typed configuration classes to catch these issues at compile time.

Let's create a configuration class that matches our **appsettings.json** structure:

```csharp
public class AppSettings
{
    public required ConnectionStringsConfig ConnectionStrings { get; init; }
    public required LoggingConfig Logging { get; init; }

    public class ConnectionStringsConfig
    {
        public required string DefaultConnection { get; init; }
    }

    public class LoggingConfig
    {
        public required string LogLevel { get; init; }
    }
}
```

In my projects, I structure nested configurations as inner classes like this because it keeps related settings grouped together and makes the configuration hierarchy obvious.

Which matches our **appsettings.json** structure:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=myserver;Database=mydb;"
  },
  "Logging": {
    "LogLevel": "Information"
  }
}
```

I prefer this registration approach in **Program.cs** because it gives you both `IOptions<T>` access and direct injection:

```csharp
builder.Services.Configure<AppSettings>(builder.Configuration);
builder.Services.AddSingleton(x => x.GetRequiredService<IOptions<AppSettings>>().Value);
```

Use in services:

```csharp
public class MyService(AppSettings _appSettings)
{
    public void DoWork()
    {
        var connectionString = _appSettings.ConnectionStrings.DefaultConnection;
        // ...
    }
}
```

Use in Blazor SSR Razor file:

```csharp
@inject AppSettings appSettings
```

Use in minimal APIs:

```csharp
app.MapGet("/api/status", (AppSettings settings) => 
{
    // Use settings.ConnectionStrings.DefaultConnection for database operations
    // Use settings.Logging.LogLevel for logging configuration
    return Results.Ok(new { Status = "Healthy" });
});
```

For simple apps with just a few settings, this approach might be overkill. But once you have nested configuration sections or multiple environments, the IntelliSense and compile-time checking become invaluable.

Using required properties means you'll get a clear error message at startup if configuration is missing, rather than discovering it when that code path executes.
