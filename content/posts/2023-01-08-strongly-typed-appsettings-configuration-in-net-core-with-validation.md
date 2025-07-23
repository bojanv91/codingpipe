---
title: "Type-safe appsettings configuration in .NET Core"
date: 2023-01-08
dateUpdated: Last Modified
permalink: /posts/strongly-typed-appsettings-configuration-in-net-core-with-validation/
tags:
  - .NET
  - Configuration
layout: layouts/post.njk
---

Configuration errors have a special way of ruining your day. You deploy to production, everything looks fine, then a specific code path hits and boom - NullReferenceException because `appSettings["SomeKey"]` returned null or you mistyped the configuration e.g. `appSettings["Somkey"]`.

I've been burned by this enough times that I now default to strongly-typed configuration classes with required properties. This catches missing configuration at startup rather than runtime, and gives you IntelliSense when accessing values.

## Setup

Create a configuration class that mirrors your appsettings.json structure:

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

Corresponding appsettings.json:

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

Register both `IOptions<AppSettings>` and the settings object directly:

```csharp
builder.Services.Configure<AppSettings>(builder.Configuration);
builder.Services.AddSingleton(x => x.GetRequiredService<IOptions<AppSettings>>().Value);
```

The dual registration gives you clean injection in most cases, plus `IOptions<T>` when you need change notifications.

## Usage

Constructor injection:

```csharp
public class MyService(AppSettings appSettings)
{
    public void DoWork()
    {
        var connectionString = appSettings.ConnectionStrings.DefaultConnection;
        // IntelliSense works, no magic strings, no null checks needed
    }
}
```

Blazor pages:

```csharp
@inject AppSettings appSettings
```

Minimal APIs:

```csharp
app.MapGet("/api/status", (AppSettings settings) => 
{
    var connectionString = settings.ConnectionStrings.DefaultConnection;
    return Results.Ok(new { Status = "Healthy" });
});
```

## Why this works

Required properties ensure missing configuration fails fast at startup with clear error messages. Nested classes as inner classes keep related settings grouped and make the hierarchy obvious. The compiler catches configuration problems immediately instead of NullReferenceExceptions deep in production code paths. IntelliSense is available, so no typos in magic strings.
