---
title: "Don't use navigation properties in EF Core"
pubDatetime: 2025-12-11
description: "How avoiding navigation properties in EF Core eliminates N+1 queries and makes data dependencies explicit."
slug: no-navigation-properties-ef-core
tags: [dotnet, ef-core, practices]
---

I don't use navigation properties in my EF Core entities. I use foreign key IDs instead.

## Example

Instead of this:

```csharp
public class Order
{
    public int Id { get; set; }
    public Customer Customer { get; set; }     // Navigation property
    public List<OrderItem> Items { get; set; } // Navigation property
}

public class OrderItem
{
    public int Id { get; set; }
    public Order Order { get; set; }      // Navigation property
    public Product Product { get; set; }  // Navigation property
    public decimal Price { get; set; }
}
```

I do this:

```csharp
public class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }
}

public class OrderItem
{
    public int Id { get; set; }
    public int OrderId { get; set; }
    public int ProductId { get; set; }
    public decimal Price { get; set; }
}
```

## Why

Navigation properties hide database calls. `order.Customer.Name` looks simple but triggers a query. With foreign keys, every data fetch is explicit and visible in your code.

You also can't accidentally modify related entities. With navigation properties, changing `order.Customer.Status` updates both the customer and the order when you save. With foreign keys, you explicitly load and save what you want to change.

Collection navigations create cartesian explosions:

```csharp
var orders = await context.Orders
    .Include(o => o.Items)
    .Include(o => o.Payments)
    .ToListAsync();
```

This query duplicates order data for every item-payment combination. An order with 10 items and 3 payments returns 30 rows with the order duplicated in each. That's unnecessary data loaded into memory. With foreign keys, you load exactly what you need.

Following DDD with navigation properties creates hidden database operations. You build deep object graphs with methods that look clean but trigger numerous queries. I've fixed code like this:

```csharp
public class Order
{
    public void RecalculatePrice()
    {
        foreach (var item in this.Items) // DB query
        {
            var price = item.Product.Price; // DB query
            var discounts = this.AppliedDiscounts; // DB query
            // calculations...
            item.Price = calculatedPrice; // Marks for update
        }
    }
}
```

The method looks nice and DDD-flavored, but it's a train of database calls and entity updates. Hard to test since you don't have those object graphs set up in unit tests.

I've worked on a project early in my career where the team used navigation properties everywhere. The codebase accumulated N+1 problems that only appeared under production load. In a consulting agency where team members rotate frequently, new developers need to pick up the codebase quickly. I've observed over the years that explicit queries make fewer mistakes. Anyone can see exactly what data is being loaded.

Making queries explicit means performance problems show up during code review, not in production.
