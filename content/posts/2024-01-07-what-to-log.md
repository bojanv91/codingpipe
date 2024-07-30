---
title: "What to Log in Applications?"
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


-------------
Log levels and their appropriate use:

DEBUG: Detailed information for debugging
INFO: General information about application flow
WARN: Potentially harmful situations
ERROR: Error events that might still allow the application to continue running
FATAL: Severe errors that cause the application to abort


Essential information to include in log entries:

Timestamp (including timezone)
Log level
Thread ID or name
Class or module name
Method or function name
Line number in the source code
User ID or session ID (if applicable)
Relevant contextual data


Structured logging:

Using JSON or other machine-readable formats
Benefits for log parsing and analysis


Sensitive data handling:

Avoiding logging passwords, credit card numbers, or other sensitive information
Techniques for masking or obfuscating sensitive data when necessary


Performance considerations:

Asynchronous logging to minimize impact on application performance
Log rotation and archiving strategies


Contextual logging:

Using thread-local storage or similar mechanisms to maintain context across method calls
Correlation IDs for tracing requests across distributed systems


Error logging best practices:

Including stack traces
Logging the full exception details
Providing context around the error


Logging in different environments:

Adjusting log levels and verbosity for development, staging, and production


Integration with monitoring and alerting systems:

Using logs to trigger alerts
Centralized log management


Compliance and legal considerations:

Logging for audit trails
Data retention policies
GDPR and other regulatory requirements


Logging patterns and anti-patterns:

Do: Use descriptive log messages
Don't: Log in catch blocks without re-throwing the exception
Do: Use parameterized logging to prevent injection attacks
Don't: Overuse logging, which can lead to noise and performance issues


Testing logging implementations:

Unit tests for log output
Ensuring logs are useful for debugging in production scenarios
