---
title: "Standardize Data at Write Time, Not Read Time"
pubDatetime: 2025-10-17
description: "Why I normalize data from multiple sources when saving to the database, not when displaying it. Using IoT temperature monitoring as a practical example."
slug: standardize-data-at-write-time-not-read-time
tags:
  - practices
  - databases
  - architecture
---

I've worked with systems where the same database field stores different value formats depending on which external API sent the data. For example, temperature readings might be 22.5 from one sensor, 72 from another, and 295.65 from a third. They mean entirely different things. Every query needs logic to convert based on source.

Converting for user preferences at display time is fine. Storing inconsistent formats from different sources is the problem. This is where the [Canonical Data Model](https://www.enterpriseintegrationpatterns.com/patterns/messaging/CanonicalDataModel.html) pattern applies: **standardize during writes to the database, not during reads. The database stores values in one standard format. Reads don't need to know how external sources formatted their data.**

## The Problem

I work with a system that aggregates sensor data from multiple IoT vendors. Each vendor sends temperature in their preferred format:

```plaintext
Wrong: Mixed units stored as-is
┌────────┬───────────┬───────┬──────────┐
│ id     │ sensor_id │ value │ unit     │
├────────┼───────────┼───────┼──────────┤
│ 1      │ A-101     │ 22.5  │ celsius  │
│ 2      │ B-202     │ 72    │ fahrenheit│
│ 3      │ C-303     │ 295.65│ kelvin   │
└────────┴───────────┴───────┴──────────┘
```

Now every component that reads this data needs conversion logic:

![The problem](/img/2025-10-18-standardize-data-at-write-time-not-read-time/the-problem.png)

Conversion logic scattered across dashboards, reports, and APIs. When aggregations are needed, the system converts in SQL with CASE statements before calculating which requires duplicating conversion logic.

## The Solution

Standardize at the ingestion boundary:

![The solution](/img/2025-10-18-standardize-data-at-write-time-not-read-time/the-solution.png)


Transform incoming readings to Celsius before saving:

```plaintext
Right: Standardized units
┌────────┬───────────┬───────┬─────────┐
│ id     │ sensor_id │ value │ unit    │
├────────┼───────────┼───────┼─────────┤
│ 1      │ A-101     │ 22.5  │ celsius │
│ 2      │ B-202     │ 22.2  │ celsius │
│ 3      │ C-303     │ 22.5  │ celsius │
└────────┴───────────┴───────┴─────────┘
```

Queries work correctly. Aggregations produce meaningful results. Conversion logic lives in one place. When a bug needs fixing, I update the ingestion layer, not dozens of components.

## Other Applications

I've seen the same pattern in fitness apps integrating multiple smartwatch providers. One provider sends pace as seconds per kilometer, another as decimal minutes per kilometer, a third as pace per mile. Storing these mixed formats means conversion logic scattered across leaderboards, personal records, and pace zone calculations. Standardizing to seconds per kilometer at ingestion makes everything downstream work correctly.

Same applies to any multi-source integration. Any ETL pipeline or system integration benefits from standardizing at write time, not read time.
