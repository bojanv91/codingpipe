---
title: "Serilog Correlation IDs for background tasks in .NET Core"
date: 2025-03-23
dateUpdated: Last Modified
permalink: /posts/serilog-correlationid-background-task/
tags:
  - .NET Core
  - Logging
layout: layouts/post.njk
---

Background tasks that log without correlation IDs create debugging nightmares. When multiple execution cycles run concurrently, their logs interleave and become impossible to trace.

Here's how to correlate each execution cycle with a unique ID:

## Solution

Create a logger extension for correlation context:

```csharp
public static class LoggerExtensions
{
    public static ILogger ForExecutionContext(this ILogger baseLogger)
    {
        return baseLogger.ForContext("CorrelationId", Guid.NewGuid().ToString("N")[..8]);
    }
}
```

Update your background service:

```csharp
public class MyBackgroundTask : BackgroundService
{
    private static readonly ILogger _serviceLogger = Log.ForContext<MyBackgroundTask>();

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        _serviceLogger.Information("Background service started");

        while (!stoppingToken.IsCancellationRequested)
        {
            var logger = _serviceLogger.ForExecutionContext();
            logger.Information("Starting background task cycle");

            try
            {
                await DoWorkAsync(logger);
                logger.Information("Completed background task cycle successfully");
            }
            catch (Exception ex)
            {
                logger.Error(ex, "Error executing background task");
            }

            await Task.Delay(TimeSpan.FromMinutes(5), stoppingToken);
        }
    }

    private async Task DoWorkAsync(ILogger logger)
    {
        logger.Information("Starting to process items");
        logger.Information("Processed {ItemCount} items successfully", 5);
        logger.Debug("Finalizing processing cycle");
    }
}
```

Configure your output template to include correlation IDs:

```json
{
  "Serilog": {
    "WriteTo": [
      {
        "Name": "Console",
        "Args": {
          "outputTemplate": "[{Timestamp:HH:mm:ss} {Level:u3}] {CorrelationId} {Message:lj}{NewLine}{Exception}"
        }
      }
    ]
  }
}
```

## Result

Instead of interleaved chaos:

```plain
[10:20:00 INF] Starting background task cycle
[10:20:00 INF] Starting to process items
[10:20:00 INF] Processed 5 items successfully
[10:20:00 INF] Starting background task cycle
[10:20:00 INF] Starting to process items
[10:20:00 INF] Completed background task cycle successfully
```

You get traceable execution cycles:

```plain
[10:20:00 INF] a42f1e3a Starting background task cycle
[10:20:00 INF] a42f1e3a Starting to process items
[10:20:00 INF] a42f1e3a Processed 5 items successfully
[10:20:00 INF] 7b2c1e3b Starting background task cycle
[10:20:00 INF] 7b2c1e3b Starting to process items
[10:20:00 INF] a42f1e3a Completed background task cycle successfully
```

## Why This Works

Creating the correlation context at the cycle level (not per method) maintains the trace through the entire execution, including exceptions that bubble up. Short correlation IDs keep logs readable while providing unique identification for each run.
