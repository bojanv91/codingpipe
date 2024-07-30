---
title: "Serilog Setup for .NET Core Worker Service Project"
date: 2023-08-18
dateUpdated: Last Modified
permalink: /posts/configuring-serilog-in-net-core-worker-and-windows-service-applications/
tags:
  - .NET Core
  - Logging
layout: layouts/post.njk
---

In this post, we'll add and configure the *Serilog* logger to a Windows Service app. This post assumes you've already [built a Windows Service app project](/building-windows-service-applications-in-net-core) and made sure your current app directory path is fixed as described in the referenced post.

## Add and configure *Serilog*

**First,** we need to install the following *Serilog* NuGet packages into our app:
- *Serilog* - the main package
- *Serilog.Extensions.Hosting* - enables the use of `IHostBuilder.UseSerilog()`
- *Serilog.Settings.Configuration* - enables reading log configuration from *appsettings.json*
- *Serilog.Sinks.Console* - enables writing to the Console
- *Serilog.Sinks.File* - enables writing to log files

```
dotnet add package Serilog
dotnet add package Serilog.Extensions.Hosting
dotnet add package Serilog.Settings.Configuration
dotnet add package Serilog.Sinks.Console
dotnet add package Serilog.Sinks.File
```

**Next**, in the _Program.cs_ file we wrap the startup code in a `try`/`catch` block to ensure that any configuration issues will be appropriately logged:

```diff-cs
using Serilog;

namespace PlaygroundWorkerService
{
    public class Program
    {
        public static void Main(string[] args)
        {
            Directory.SetCurrentDirectory(AppDomain.CurrentDomain.BaseDirectory);

            // The initial bootstrap logger is able to log errors during start-up.
            // It's fully replaced by the logger configured in `UseSerilog()`.
+            Log.Logger = new LoggerConfiguration()
+                .WriteTo.Console()
+                .CreateBootstrapLogger();

            Log.Information("Starting up");

            try
            {
                IHost host = Host.CreateDefaultBuilder(args)
                    .ConfigureServices(services =>
                    {
                        services.AddHostedService<Worker>();
                    })
+                    .UseSerilog((hostingContext, services, loggerConfiguration) => loggerConfiguration
+                        .ReadFrom.Configuration(hostingContext.Configuration))
                    .UseWindowsService()
                    .Build();
                    
                host.Run();

                Log.Information("Stopped cleanly");

            }
            catch (Exception ex)
            {
                Log.Fatal(ex, "An unhandled exception occured during bootstrapping");
            }
            finally
            {
                Log.CloseAndFlush();
            }
        }
    }
}
```

The `.UseSerilog()` call will redirect all log events through your *Serilog* pipeline.

**Next,** we'll configure the logger using JSON configuration strings placed in the *appsettings.json*:

```json
{
  "Serilog": {
    "Using": [ "Serilog.Sinks.Console", "Serilog.Sinks.File" ],
    "MinimumLevel": "Information",
    "Override": {
      "Microsoft": "Information",
      "System": "Warning"
    },
    "WriteTo:Async": {
      "Name": "Async",
      "Args": {
        "configure": [
          {
            "Name": "File",
            "Args": {
              "path": "logs/log.txt",
              "outputTemplate": "[{Timestamp:u} {Level:u3}] {Message:lj}{NewLine}{Exception}",
              "rollingInterval": "Day",
              "shared": true
            }
          }
        ]
      }
    },
    "WriteTo": [
      {
        "Name": "Console",
        "Args": {
          "outputTemplate": "[{Timestamp:u} {Level:u3}] {Message:lj}{NewLine}{Exception}"
        }
      }
    ],
    "Enrich": [ "FromLogContext", "WithMachineName", "WithThreadId" ],
    "Properties": {
      "Application": "Sample"
    }
  }
}
```

**Finally,** we can start our application and logs will show up in the app's */logs* directory, as well in the Console/Terminal if you start it as a console app.

## Use the *Serilog* logger

Here is how we can use the *Serilog* logger from our hosted service:

```cs
using Serilog;
using ILogger = Serilog.ILogger;

namespace PlaygroundWorkerService
{
    public class Worker : BackgroundService
    {
        private static readonly ILogger _logger = Log.ForContext<Worker>();

        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {
            while (!stoppingToken.IsCancellationRequested)
            {
                _logger.Information("Worker running at: {time}", DateTimeOffset.Now);
                await Task.Delay(1000, stoppingToken);
            }
        }
    }
}
```

## References

- https://github.com/serilog/serilog-extensions-hosting#inline-initialization
- https://github.com/serilog/serilog-settings-configuration
