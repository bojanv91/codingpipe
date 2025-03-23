---
title: "What to Log in Apps"
date: 2024-01-07
dateUpdated: Last Modified
permalink: /posts/what-to-log/
tags:
  - Logging
layout: layouts/post.njk
---

Logging is a critical aspect of application development, but knowing exactly what to log can make the difference between a debugging nightmare and a smooth troubleshooting process.
Let's start by examining the key types of information you should consider logging in your applications

You could log:

- Important and useful information for developers, QAs, or support
- Unhandled exceptions
- System changes (startup, shutdown, restart, crash)
- Resource issues (disk space to be full, memory exhausted)
- Network connections to external services (failure, success)
- Critical changes (application, data)
- Critical workflows (e.g. booking process, new customer sign-up process)
- Auth and access

## What Properties to Log?

Use the following log properties where applicable:

| Property | Description |
| ---- | ---- |
| `Timestamp` | the log event's timestamp |
| `Level` | the log level, e.g. info, debug, warning, error, fatal |
| `SourceContext` | the full name of the class from where the log was added (the logger name of the log entry) |
| `Message` | the log event's message |
| `Exception` | the full exception message and stack trace (if there is any) |
| `RequestMethod` | the HTTP request method (e.g. POST, GET, PUT, DELETE) |
| `RequestPath` | the HTTP request path |
| `StatusCode` | the HTTP response status code |
| `ClientIP` | the request caller's IP address |
| `CorrelationId` | the ID for request or operation tracking across multiple services |
| `MachineName` | the name of the machine on which the service is running |
| `Version` | the version of the app |
| `UserAgent` | the HTTP user agent |
| `UserId` | the authenticated user's ID making the request |

## What NOT to Log?

Always consider the security and privacy implications of the information you log, especially in production environments. Implementing proper log management and reviewing your logging practices regularly can help ensure that your logs serve their intended purpose without compromising sensitive data.

Here's a list of things you should generally avoid logging:

- Sensitive user information, such as passwords, credit card numbers, or other personally identifiable information (PII)
- Authentication tokens, secrets, and API keys
- Full request or response payloads - log only relevant parts or use automatic tools to obfuscate sensitive data
- Database connection strings
- Excessive debug information or internal implementation details
- Redundant information that doesn't contribute to the diagnosis of issues or monitoring of the system
- Production environment secrets
- Customer-specific names and information
