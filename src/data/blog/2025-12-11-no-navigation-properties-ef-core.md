---
title: "EF navigation properties hide your queries"
pubDatetime: 2025-12-11
description: "How avoiding navigation properties in EF Core eliminates N+1 queries and makes data dependencies explicit."
slug: no-navigation-properties-ef-core
tags: [dotnet, data]
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

Navigation properties hide database calls. `order.Customer.Name` looks like property access, but triggers a query. `order.Customer.Status = "inactive"` looks like a field set — `SaveChanges()` updates two records. `Include()` on an order with 10 items and 3 payments returns 30 rows, order data duplicated across each.

The problem isn't performance alone. It's that none of this is visible at the call site.

---

Early in my career, I worked on a project where navigation properties were everywhere. N+1 problems accumulated silently and only surfaced under production load.

I've seen code like this more than once:

```csharp
public void RecalculatePrice() 
{
    foreach (var item in this.Items)      // query
    {
        var price = item.Product.Price;   // query per item
        item.Price = calculatedPrice;     // marks for update
    }
}
```

It looks like a clean domain method. It's a cascade of hidden queries. And it's untestable without a full object graph loaded.

---

With foreign key IDs, you load explicitly:

```csharp
var items = await _db.OrderItems
    .Where(i => i.OrderId == order.Id)
    .ToListAsync();

var productIds = items.Select(i => i.ProductId).ToList();

var prices = await _db.Products
    .Where(p => productIds.Contains(p.Id))
    .Select(p => new { p.Id, p.Price })
    .ToDictionaryAsync(p => p.Id, p => p.Price);
```

Two queries, both visible, both testable with a mocked `DbContext`. The data dependency is in the code, not in EF Core's change tracker.

---

In consulting, teams rotate. A new developer sees `order.Customer.Name` and has no idea what fires underneath. With explicit loads, they see exactly what hits the database and where.

Performance problems show in code review, not production.