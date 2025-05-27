---
title: "Correlation IDs for ASP.NET Core background tasks"
date: 2025-03-23
dateUpdated: Last Modified
permalink: /posts/serilog-correlationid-background-task/
tags:
  - ASP.NET Core
  - Serilog
  - Logging
layout: layouts/post.njk
---

Correlation IDs enable you to trace related log entries across distributed systems by linking them with a unique identifier. This becomes essential when troubleshooting issues that span multiple services or execution contexts.

Background tasks in ASP.NET Core run outside the HTTP pipeline, so they can't access request correlation IDs, leaving their logs uncorrelated by default. The solution is creating a unique correlation ID for each background task execution cycle, allowing you to trace all logs from a single run.

## Setting up the logger extension

```csharp
public static class LoggerExtensions
{
    /// <summary>
    /// Creates a new logger instance with a unique correlation ID.
    /// Usage:
    ///   var serviceLogger = Log.ForContext<MyService>();
    ///   var contextLogger = serviceLogger.ForCorrelationIdContext();
    /// </summary>
    /// <param name="baseLogger">The base logger instance.</param>
    /// <returns>A new logger instance with a correlation ID.</returns>
    public static ILogger ForCorrelationIdContext(this ILogger baseLogger)
    {
        return baseLogger.ForContext("CorrelationId", Guid.NewGuid().ToString());
    }
}
```

## Using correlation IDs in background services

Here's how to implement correlation IDs in a background service:

```csharp
public class MyBackgroundTask : BackgroundService
{
    private static readonly ILogger _serviceLogger = Log.ForContext<MyBackgroundTask>();

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        _serviceLogger.Information("Background service started");

        while (!stoppingToken.IsCancellationRequested)
        {
            // Create a contextual logger with a unique correlation ID for this execution cycle
            var logger = _serviceLogger.ForCorrelationIdContext();
            logger.Information("Starting background task cycle");

            try
            {
                // Perform the actual work with the correlation ID attached to all logs
                await DoWorkAsync(logger);

                logger.Information("Completed background task cycle successfully");
            }
            catch (Exception ex)
            {
                // All exception logs will include the same correlation ID
                logger.Error(ex, "Error executing background task");
            }

            await Task.Delay(TimeSpan.FromMinutes(5), stoppingToken);
        }

        _serviceLogger.Warning("Background service is stopping");
    }

    private async Task DoWorkAsync(ILogger logger)
    {
        // This method simulates the work being done by the background task
        logger.Information("Starting to process items");
        
        // Simulate some processing time
        await Task.Delay(100);
        
        logger.Information("Processed 5 items successfully");
        
        // More processing
        await Task.Delay(50);
        
        logger.Debug("Finalizing processing cycle");
    }
}
```

## Configuring Serilog in Program.cs

Ensure Serilog is configured to include correlation IDs:

```csharp
builder.Host.UseSerilog((context, configuration) => 
    configuration
        .ReadFrom.Configuration(context.Configuration)
        .Enrich.WithCorrelationId()
        .WriteTo.Console(outputTemplate: 
            "[{Timestamp:HH:mm:ss} {Level:u3}] {CorrelationId} {Message:lj} {Properties:j}{NewLine}{Exception}")
);
```

## Example log output

Here's what the logs look like in your console:

**HTTP request logs (with correlation ID from HTTP header):**

```
[10:15:22 INF] 7b8d9e1f-c452-4daf-8e61-a9dc3be352a0 HTTP GET /api/products responded 200 in 45.6ms
[10:15:22 DBG] 7b8d9e1f-c452-4daf-8e61-a9dc3be352a0 Retrieved 10 products from database
[10:15:22 INF] 7b8d9e1f-c452-4daf-8e61-a9dc3be352a0 Request completed
```

**Background task logs (with generated correlation ID):**

```
[10:20:00 INF] Background service started
[10:20:00 INF] 42f1e3a8-5b7d-4c69-8f9a-a3c091e5d742 Starting background task cycle
[10:20:00 INF] 42f1e3a8-5b7d-4c69-8f9a-a3c091e5d742 Starting to process items
[10:20:00 INF] 42f1e3a8-5b7d-4c69-8f9a-a3c091e5d742 Processed 5 items successfully
[10:20:00 DBG] 42f1e3a8-5b7d-4c69-8f9a-a3c091e5d742 Finalizing processing cycle
[10:20:00 INF] 42f1e3a8-5b7d-4c69-8f9a-a3c091e5d742 Completed background task cycle successfully
```

Each background task execution gets its own unique correlation ID, and all logs from the same execution cycle share that same ID.

For the `.Enrich.WithCorrelationId()` extension you need to install:

```bash
dotnet add package Serilog.Enrichers.ClientInfo
```

This approach works for any background service where you need to correlate logs within execution cycles. Use it when debugging complex background operations or when you need to trace the flow of long-running processes.