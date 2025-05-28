---
title: "Serilog setup for .NET worker services"
date: 2023-08-18
dateUpdated: Last Modified
permalink: /posts/configuring-serilog-in-net-core-worker-and-windows-service-applications/
tags:
  - ASP.NET Core
  - Serilog
  - Logging
layout: layouts/post.njk
---

Worker services are perfect for executing long-running or time-scheduled operations in the background. Whether running as console applications or deployed as services, proper logging is essential for monitoring and troubleshooting.

Here's how I set up Serilog with sensible defaults for worker service projects.

## Starting with the default template

The Worker Service template creates this basic structure:

```csharp
// Program.cs
using WorkerService1;

var builder = Host.CreateApplicationBuilder(args);
builder.Services.AddHostedService<Worker>();

var host = builder.Build();
host.Run();
```

```csharp
// Worker.cs
namespace WorkerService1
{
    public class Worker : BackgroundService
    {
        private readonly ILogger<Worker> _logger;

        public Worker(ILogger<Worker> logger)
        {
            _logger = logger;
        }

        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {
            while (!stoppingToken.IsCancellationRequested)
            {
                if (_logger.IsEnabled(LogLevel.Information))
                {
                    _logger.LogInformation("Worker running at: {time}", DateTimeOffset.Now);
                }
                await Task.Delay(1000, stoppingToken);
            }
        }
    }
}
```

## Installing Serilog packages

Install the essential Serilog packages:

```bash
dotnet add package Serilog
dotnet add package Serilog.Extensions.Hosting
dotnet add package Serilog.Settings.Configuration
dotnet add package Serilog.Sinks.Console
```

## Configuring Program.cs with Serilog

The key to reliable Serilog setup is using a bootstrap logger during startup, then replacing it with the full configuration:

```csharp
// Program.cs
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
    
    // This replaces the bootstrap logger with configuration from appsettings.json
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
    // Ensures all logs are written before shutdown
    Log.CloseAndFlush();
}
```

## Setting up appsettings.json

Configure Serilog through `appsettings.json`:

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

## Using Serilog in your worker

The Worker class continues to use dependency injection, but now gets Serilog's implementation:

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

This setup provides structured console logging with reduced noise from Microsoft and System namespaces. The bootstrap logger ensures startup errors are captured even if the configuration fails to load, while the main configuration provides full logging capabilities once the host is built.