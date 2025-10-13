---
title: "Property Getters vs Get Methods: When to Use Which"
date: 2025-10-14
dateUpdated: Last Modified
permalink: /posts/property-getters-vs-get-methods-when-to-use-which/
tags:
  - Software Design
layout: layouts/post.njk
---

I kept seeing inconsistent patterns in codebases - sometimes `obj.property`, sometimes `obj.getProperty()`. After refactoring across TypeScript, Dart, and C# projects, here's the simple rule I follow:

**Property getters** - computed values that feel like properties:

- Synchronous computation
- Pure function (returns the same value for the same state)
- Feels like accessing a property
- Fast/instant execution

**Get methods** - ****operations that do work:

- Async operations: needs async/await
- Might throw exceptions
- Has side effects (I/O, network, logging, system calls)
- Can return different values over time

This pattern keeps my code predictable - if it looks like property access, it acts like one. If it has parentheses, I know it's doing more work.

## Examples

Configuration/State Checks:

```tsx
// TypeScript - frontend config
class AppConfig {
  private _environment: string;
  
  get isDevelopment(): boolean {
    return this._environment === 'development';
  }
  
  get apiBaseUrl(): string {
    return this.isDevelopment 
      ? 'http://localhost:3000' 
      : 'https://api.production.com';
  }
}

// Usage
const config = new AppConfig();
console.log(config.apiBaseUrl);
if (config.isDevelopment) {  // No parentheses
  // Do something
}
```

Async Operations:

```dart
// Dart - platform channels
class AppConfig {
  Future<String> getBuildVersion() async {
    final info = await PackageInfo.fromPlatform();
    return '${info.version}+${info.buildNumber}';
  }
}

// Usage - must await
final version = await AppConfig.current.getBuildVersion();
setState(() => _buildVersion = version);
```

File/Network Operations:

```csharp
// C# - file I/O
public class LogReader 
{
    public async Task<IEnumerable<FileInfo>> GetLogFilesAsync(string directory) 
    {
        return await Task.Run(() => 
            new DirectoryInfo(directory)
                .GetFiles("*.log")
                .OrderByDescending(f => f.LastWriteTime));
    }
}

// Usage
var logs = await reader.GetLogFilesAsync("/var/logs");
```
