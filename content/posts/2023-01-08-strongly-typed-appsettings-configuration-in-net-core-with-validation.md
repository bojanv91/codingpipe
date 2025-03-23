---
title: "AppSettings: Typed Configuration in .NET Core using the Options Pattern"
date: 2023-01-08
dateUpdated: Last Modified
permalink: /posts/strongly-typed-appsettings-configuration-in-net-core-with-validation/
tags:
  - ASP.NET Core
layout: layouts/post.njk
---

Create a strongly typed configuration class for **appsettings.json** using **required** and **init** properties:

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

Match your **appsettings.json** structure:

```csharp
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=myserver;Database=mydb;"
  },
  "Logging": {
    "LogLevel": "Information"
  }
}
```

Register in **Program.cs**:

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

## Benefits of This Approach

This implementation provides several advantages over traditional string-based configuration access:

- Compile-time type checking prevents configuration key typos
- IntelliSense support improves developer productivity
- The **required** modifier ensures all necessary configuration values are provided
- **Init**-only properties prevent accidental configuration modifications
