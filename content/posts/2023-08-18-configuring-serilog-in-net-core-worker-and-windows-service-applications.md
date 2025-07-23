---
title: "Serilog setup for .NET Core worker services"
date: 2023-08-18
dateUpdated: Last Modified
permalink: /posts/configuring-serilog-in-net-core-worker-and-windows-service-applications/
tags:
  - .NET
  - Serilog
layout: layouts/post.njk
---

Worker services that fail during startup often fail silently. The default .NET logging doesn't capture bootstrap errors, leaving you debugging blind when services won't start in production.

This Serilog setup provides bootstrap logging that captures startup failures and structured logging for runtime operations.

Install the essential packages:

```powershell
dotnet add package Serilog
dotnet add package Serilog.Extensions.Hosting
dotnet add package Serilog.Settings.Configuration
dotnet add package Serilog.Sinks.Console
```

Replace Program.cs with bootstrap logging:

```csharp
using Serilog;
using WorkerService1;

// Bootstrap logger captures startup errors before full configuration loads
Log.Logger = new LoggerConfiguration()
    .WriteTo.Console()
    .CreateBootstrapLogger();

Log.Information("Starting up");

try
{
    var builder = Host.CreateApplicationBuilder(args);
    
    builder.Services.AddSerilog((services, loggerConfiguration) => loggerConfiguration
        .ReadFrom.Configuration(builder.Configuration));
    
    builder.Services.AddHostedService<Worker>();

    var host = builder.Build();
    host.Run();

    Log.Information("Stopped cleanly");
}
catch (Exception ex)
{
    Log.Fatal(ex, "An unhandled exception occurred during bootstrapping");
}
finally
{
    Log.CloseAndFlush();
}
```

Configure appsettings.json:

```json
{
  "Serilog": {
    "MinimumLevel": "Information",
    "Override": {
      "Microsoft": "Warning",
      "System": "Warning"
    },
    "WriteTo": [
      {
        "Name": "Console",
        "Args": {
          "outputTemplate": "[{Timestamp:yyyy-MM-dd HH:mm:ss} {Level:u3}] {Message:lj}{NewLine}{Exception}"
        }
      }
    ],
    "Enrich": [ "FromLogContext" ]
  }
}
```

## Usage

Update your worker to use structured logging:

```csharp
using Serilog;
using ILogger = Serilog.ILogger;

namespace WorkerService1
{
    public class Worker : BackgroundService
    {
        private static readonly ILogger _logger = Log.ForContext<Worker>();

        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {
            while (!stoppingToken.IsCancellationRequested)
            {
                _logger.Information("Worker running at: {Time}", DateTimeOffset.Now);
                await Task.Delay(5000, stoppingToken);
            }
        }
    }
}
```

## Why this works

Bootstrap logging captures startup failures that would otherwise be invisible. The two-phase approach creates a simple console logger first, then switches to full configuration once the host builds.

Microsoft and System logs at Warning level reduce framework noise. `Log.ForContext<Worker>()` provides scoped loggers. Structured logging with {Time} creates searchable properties instead of string interpolation.
