---
title: "Don't use navigation properties in EF Core"
pubDatetime: 2025-12-11
description: "How avoiding navigation properties in EF Core eliminates N+1 queries and makes data dependencies explicit."
slug: no-navigation-properties-ef-core
tags: [dotnet, ef-core, practices]
---

I use foreign key IDs. Not navigation properties.

Instead of:

```csharp
public class Order 
{
    public Customer Customer { get; set; }
    public List<OrderItem> Items { get; set; }
}
```

I write:

```csharp
public class Order 
{
    public int CustomerId { get; set; }
}
```

## Why

Navigation properties hide database calls. `order.Customer.Name` looks like property access, but triggers a query.

You can accidentally modify related entities. Change `order.Customer.Status` and SaveChanges() updates both records. With foreign keys, you load and save explicitly.

Include() creates cartesian products. An order with 10 items and 3 payments returns 30 rows—order data duplicated in each row.

## The real problem

I've debugged code like this:

```csharp
public void RecalculatePrice() 
{
    foreach (var item in this.Items)          // query
    {
        var price = item.Product.Price;       // query
        // calculations
        item.Price = calculatedPrice;         // marks for update
    }
}
```

Looks clean. It's a cascade of hidden queries. Hard to test without full object graphs.

## What I learned

Early in my career, I've worked on a project where the team used navigation properties everywhere. The codebase accumulated N+1 problems that only appeared under production load. In consulting/agencies where team members rotate frequently, explicit code (queries in our case) causes fewer bugs. New developers see exactly what data loads and where.

Performance problems show in code review, not production.
