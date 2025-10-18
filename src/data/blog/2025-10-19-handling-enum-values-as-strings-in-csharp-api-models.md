---
title: "Handling Enum Values as Strings in C# API Models"
pubDatetime: 2025-10-19
description: "A practical pattern for working with enums in API request models - storing as strings while keeping type-safe enum logic throughout your codebase."
slug: handling-enum-values-as-strings-in-csharp-api-models
tags:
  - dotnet
  - api
  - practices
---

A common issue when building APIs is deciding how to represent enum values. Send them as integers and your requests become unreadable - is status `2` active or expired? Send them as strings and you end up with string comparisons scattered throughout your business logic. I use a pattern that solves both: store the enum as a string in the request model, but provide a conversion method that returns the strongly-typed enum.

## The Problem

Integer enums create two issues. First, they're not self-documenting - seeing `"status": 2` in a request tells you nothing. Second, when clients send unrecognized integers (from version mismatches or bad data), you can't tell if `5` is a new valid status or invalid data. With strings, I at least know what value was sent when logging errors.

Working with strings everywhere means losing type safety. You end up comparing magic strings throughout your domain logic instead of using compiler-checked enum values.

## The Solution

I store the enum as a string in the API model and add a conversion method:

```csharp
public class UpdateProductRequest
{
    // String value from API
    public string Status { get; set; }
    
    // Convert to enum for business logic
    public ProductStatus GetStatus()
    {
        if (Enum.TryParse<ProductStatus>(Status, out var result))
        {
            return result;
        }
        
        _logger.LogWarning("Invalid product status: {Status}", Status);
        throw new ArgumentException($"Invalid status: {Status}");
    }
}

public enum ProductStatus
{
    Active,
    Inactive,
    Expired,
    PendingReview
}
```

Now I work with the enum in my domain logic:

```csharp
var status = request.GetStatus();

if (status == ProductStatus.Expired)
{
    // Type-safe enum comparison
    NotifyExpiration();
}
```

API clients send readable values like `"Expired"` that are self-documenting. Inside my code, I get compiler-checked enum comparisons. When clients send invalid strings, the conversion method logs what was sent and throws a clear exception - I get visibility into typos, version mismatches, or malformed requests instead of mysterious integer errors.

This pattern keeps the API flexible while maintaining type safety in domain logic. The lazy conversion also means I can handle invalid values contextually - fail fast in critical paths, log and continue in others - without coupling validation to deserialization.
