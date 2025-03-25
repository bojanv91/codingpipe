---
title: "Correlation IDs with Serilog in ASP.NET Core Background Tasks"
date: 2025-03-23
dateUpdated: Last Modified
permalink: /posts/serilog-correlationid-background-task/
tags:
  - ASP.NET Core
  - Serilog
  - Logging
layout: layouts/post.njk
---

Background tasks in ASP.NET Core applications run independently of the HTTP pipeline, which means they don't have access to the HTTP request X-Correlation-Id header that's typically used for correlating logs in web applications. As a result, the correlation ID for background tasks is always null by default. To solve this problem, we should create a unique correlation ID for each background task execution cycle, allowing us to correlate all logs from a single execution run.

## Setting Up the Logger Extension

```csharp
public static class LoggerExtensions
{
    /// <summary>
    /// Creates a new logger instance with a unique correlation ID.
    /// Usage:
		///   var serviceLogger = Log.ForContext<MyService>();
		///   var contextLogger = serviceLogger.ForCorrelationIdContext(); // this one has a unique correlation ID
    /// </summary>
    /// <param name="baseLogger">The base logger instance.</param>
    /// <returns>A new logger instance with a correlation ID.</returns>
    public static ILogger ForCorrelationIdContext(this ILogger baseLogger)
    {
        return baseLogger.ForContext("CorrelationId", Guid.NewGuid().ToString());
    }
}
```

## Using Correlation IDs in Background Services

Here's a minimal example of implementing correlation IDs in a background service:

```csharp
public class MyBackgroundTask : BackgroundService
{
    private static readonly ILogger _serviceLogger = Log.ForContext<MyBackgroundTask>();

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        _serviceLogger.Information("Background service started");

        while (!stoppingToken.IsCancellationRequested)
        {
            // Create a contextual logger from the base logger with a unique
						// correlation ID for this execution cycle
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

## Example Log Output

Here's an example of what the logs might look like in your console:

**HTTP Request Logs (with correlation ID from HTTP header):**

```
[10:15:22 INF] 7b8d9e1f-c452-4daf-8e61-a9dc3be352a0 HTTP GET /api/products responded 200 in 45.6ms
[10:15:22 DBG] 7b8d9e1f-c452-4daf-8e61-a9dc3be352a0 Retrieved 10 products from database
[10:15:22 INF] 7b8d9e1f-c452-4daf-8e61-a9dc3be352a0 Request completed
```

**Background Task Logs (with generated correlation ID):**

```
[10:20:00 INF] Background service started
[10:20:00 INF] 42f1e3a8-5b7d-4c69-8f9a-a3c091e5d742 Starting background task cycle
[10:20:00 INF] 42f1e3a8-5b7d-4c69-8f9a-a3c091e5d742 Starting to process items
[10:20:00 INF] 42f1e3a8-5b7d-4c69-8f9a-a3c091e5d742 Processed 5 items successfully
[10:20:00 DBG] 42f1e3a8-5b7d-4c69-8f9a-a3c091e5d742 Finalizing processing cycle
[10:20:00 INF] 42f1e3a8-5b7d-4c69-8f9a-a3c091e5d742 Completed background task cycle successfully
```

Note how each execution of the background task gets its own correlation ID that's different from HTTP request correlation IDs, but all logs from the same background task execution cycle share the same ID.

## NuGet Dependencies

For a complete setup, you'll need to install the following NuGet packages:

* **Serilog.AspNetCore** - For core Serilog integration with ASP.NET Core
* **Serilog.Enrichers.CorrelationId** - For the `.Enrich.WithCorrelationId()` extension
* **Serilog.Sinks.Console** - For console output (if that's where you're sending logs)
