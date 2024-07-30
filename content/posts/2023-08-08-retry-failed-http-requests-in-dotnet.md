---
title: "HTTP Retry with Flurl and Polly in .NET Core"
date: 2023-08-08
dateUpdated: Last Modified
permalink: /posts/retry-failed-http-requests-in-dotnet/
tags:
  - .NET Core
layout: layouts/post.njk
---

HTTP requests may fail to many reasons - the API server could be temporary offline, a network glitch might occur, a mid-request deploy might happen, the server may be overloaded with requests, etc. 
Some failures go away if you just re-run the requests, and sometimes you'll need to wait a bit before doing that.

To overcome these type of glitches you can wrap your HTTP calls with a retry policy.

## .NET client code with Flurl and Polly

```cs
var response = await Policy
    .Handle<FlurlHttpException>(x =>  // Activate only on this exception, with filter:
        x.StatusCode >= 500      // on Server error
        || x.StatusCode == 408   // on Request Timeout
        || x.StatusCode == 429)  // on Too Many Requests
    .WaitAndRetryAsync(new[]      // Retry maximum 4 times, 
    {                             // but wait a specified duration between each retry.
        TimeSpan.FromSeconds(1),
        TimeSpan.FromSeconds(3),
        TimeSpan.FromSeconds(8),
        TimeSpan.FromSeconds(13)
    })
    .ExecuteAsync(() => "https://api.example.com/todos".GetJsonAsync<TodosResponse>());  // Execute the request.
```

TIP: You can abstract parts of this code to reduce the verboseness when applying policies across your projects to something like this:

```cs
var response = await ExecuteRequestAsync(() => "https://api.example.com/todos".GetJsonAsync<TodosResponse>());
```

## References
- https://github.com/App-vNext/Polly#wait-and-retry
- https://github.com/App-vNext/Polly/wiki/Retry
- https://learn.microsoft.com/en-us/azure/architecture/patterns/retry
- https://flurl.dev/docs/fluent-http/
