---
title: "Typed AppSettings Configuration in .NET Core"
date: 2023-01-08
dateUpdated: Last Modified
permalink: /posts/strongly-typed-appsettings-configuration-in-net-core-with-validation/
tags:
  - .NET
layout: layouts/post.njk
---

The default way to access .NET app configuration is by using the `IConfiguration` object.

```csharp
public class Service
{
  private readonly IConfiguration _config;
  public Service(IConfiguration config) => _config = config;
  
  public void DoSomething()
  {
    var configValue = _config["ConnectionString"];
    // ...
  }
}
```

This approach can be error-prone in larger codebases because it uses magic-string keys. Common issues are:
- incorrect casing of the key names (e.g. `connectionString` instead of `ConnectionString`)
- typos in the key names (e.g. `ConectiongString`)
- no intellisense in the IDE for ease of access
- you must check the config JSON structure whenever you want to use a configuration

## Introducing the strongly typed configuration class "AppSettings"
To solve this, create a strongly-typed class that will serve as a model for your app configuration:
```csharp
public class AppSettings
{
  public string ConnectionString { get; set; }
  public string AppName { get; set; }
  public int AppVersion { get; set; }
}
```

Then, configure it on app startup:
```csharp
var appSettings = new AppSettings();
configuration.Bind(appSettings); // copy IConfiguration key-value pairs to appSettings
services.AddSingleton(appSettings); // register it in the DI
```

And finally use it in your app code:
```csharp
public class Service
{
  private readonly AppSettings _appSettings;
  public Service(AppSettings appSettings) => _appSettings = appSettings;
  
  public void DoSomething()
  {
    var configValue = _appSettings.ConnectionString;
    // ...
  }
}
```

## Accessing nested configurations
Let's say your configuration JSON contains the following structure:
```json
{
  "Smtp": {
    "Host": "...",
    "Port": "..."
  }
}
```

You can access the nested configurations through `IConfiguration` like this:
```csharp
// IConfiguration _config = ...
var configValue = _config["Smtp:Host"];
```

To make this work in your strongly-typed access approach, change the `AppSettings` class into this:
```csharp
public class AppSettings
{
  public SmtpSection Smtp { get; set; } = new SmtpSection();
  // ...

  public class SmtpSection
  {
    public string Host { get; set; }
    public int Port { get; set; }
  }
}
```

And then use it like this in your app code:
```csharp
// AppSettings _appSettings = ...
var configValue = _appSettings.Smtp.Host;
```

## Validating strongly typed configuration
Call the `appSettings.Validate()` method early in the app startup. It will trigger self validation, throwing an exception on errors.
```csharp
var appSettings = new AppSettings();
configuration.Bind(appSettings);
appSettings.Validate(); // NEW: add this here
services.AddSingleton(appSettings);
```

Here is a generic implementation to detect missing configurations. 
NOTE: tweak, extend, and change it as per your configuration validation needs.

```csharp
public class AppSettings
{
  public string Property1 { get; set; }
  public string Property2 { get; set; }
  // ...
  
  /// <summary>
  /// Ensures all configuration properties have a value that is not null and not empty.
  /// It does not validate if the format of each property value is correct. 
  /// </summary>
  /// <exception cref="InvalidOperationException"></exception>
  public void Validate()
  {
    var missingProperties = DoValidate(this);
  
    if (missingProperties.Any())
    {
      const string msg = "AppSettings - missing configs:";
      throw new InvalidOperationException($"{msg} {string.Join(", ", missingProperties)}");
    }
  }
  
  /// <summary>
  /// Find any missing configuration values with recursion.
  /// </summary>
  /// <param name="obj"></param>
  /// <returns></returns>
  private List<string> DoValidate(object obj)
  {
    var missingProperties = new List<string>();
  
    var props = obj.GetType().GetProperties();
    foreach (var prop in props)
    {
      var value = prop.GetValue(obj, null);
      if (value == null || value.ToString() == "")
      {
        missingProperties.Add(prop.Name);
      }
      else
      {
        // If the property value is a complex object, then check it's
        // nested properties. 
        if (value is object and not (string or IEnumerable))
        {
          var nestedMissingProperties = DoValidate(value);
          foreach (var missingProperty in nestedMissingProperties)
          {
            missingProperties.Add($"{prop.Name}.{missingProperty}");
          }
        }
      }
    }
  
    return missingProperties;
  }
}
```
