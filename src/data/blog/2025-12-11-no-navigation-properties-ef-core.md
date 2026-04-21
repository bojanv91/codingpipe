---
title: "Navigation properties make database calls invisible"
pubDatetime: 2025-12-11
description: "Navigation properties make database calls look like property access. The query fires, the change tracker updates records, the cartesian product bloats the result — none of it visible at the call site. This note covers why I use foreign key IDs instead."
slug: no-navigation-properties-ef-core
tags: [dotnet, data]
---

I use foreign key IDs on entities. Navigation properties stay off.

```csharp
// navigation property — avoid
public class Order
{
    public Customer Customer { get; set; }
}

// foreign key ID — use this
public class Order
{
    public int CustomerId { get; set; }
}
```

**What navigation properties hide:**

`order.Customer.Name` looks like property access, but triggers a query. `order.Customer.Status = "inactive"` looks like a field assignment, but `SaveChanges()` updates two records. `Include(o => o.Items).ThenInclude(i => i.Payments)` on an order with 10 items and 3 payments each returns 30 rows, order columns duplicated across every one.

None of this is visible at the call site.

**The failure mode in domain methods:**

```csharp
public void RecalculatePrice()
{
    foreach (var item in this.Items)      // query
    {
        var price = item.Product.Price;   // query per item
        item.Price = calculatedPrice;
    }
}
```

This method looks like it just iterates over in-memory collections. But it fires a query for `Items`, then one per item for `Product`. Untestable without a full object graph loaded in memory.

**Explicit loading:**

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

The query is the dependency. Navigation properties bury it inside the ORM, invisible to anyone reading the call site. Foreign key IDs put it in the code, where it can be reviewed and tested.
