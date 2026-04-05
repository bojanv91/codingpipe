---
title: "A loop that does five things is five problems"
pubDatetime: 2025-04-20
description: "How to simplify complex data processing loops by refactoring them into clear, maintainable transformation pipelines using functional programming concepts."
slug: complex-loops-to-transformation-pipelines
tags:
  - design
draft: false
---

**The problem:** A loop that calculates distance, filters GPS errors, smooths elevation, and classifies terrain is doing four jobs. Adding a fifth means reading all four first.

```csharp
public ActivitySummary ProcessGpxData(List<GpxPoint> gpxPoints)
{
    double totalDistance = 0;
    double totalElevationGain = 0;
    List<GpxPoint> processedPoints = new List<GpxPoint>();

    for (int i = 0; i < gpxPoints.Count; i++)
    {
        var point = gpxPoints[i];

        if (i > 0)
        {
            var prevPoint = gpxPoints[i - 1];

            double segmentDistance = CalculateHaversineDistance(prevPoint, point);
            double speed = CalculateSpeed(segmentDistance, point.Timestamp - prevPoint.Timestamp);

            if (speed > 30) continue;

            double elevationDelta = point.Elevation - prevPoint.Elevation;
            if (elevationDelta > 0.5)
                totalElevationGain += elevationDelta;

            totalDistance += segmentDistance;

            point.SegmentDistance = segmentDistance;
            point.Speed = speed;
            point.SmoothedElevation = CalculateSmoothedElevation(i, gpxPoints);
            point.TerrainType = DetermineTerrainType(elevationDelta, speed);
        }
        else
        {
            point.SegmentDistance = 0;
            point.Speed = 0;
            point.SmoothedElevation = point.Elevation;
            point.TerrainType = "flat";
        }

        processedPoints.Add(point);
    }

    return new ActivitySummary
    {
        ProcessedPoints = processedPoints,
        TotalDistance = totalDistance,
        TotalElevationGain = totalElevationGain
    };
}
```

Every new requirement becomes a merge conflict with the existing logic.

---

**The fix:** Break the loop into a pipeline. One function per step, each receiving and returning the same intermediate structure.

```csharp
public ActivitySummary ProcessGpxData(List<GpxPoint> gpxPoints)
{
    var data = MapToIntermediateData(gpxPoints);

    data = CalculateSegmentMetrics(data);
    data = FilterGpsGlitches(data);
    data = ProcessElevationData(data);
    data = ApplyDataSmoothing(data);
    data = ClassifyTerrainTypes(data);
    data = CalculateAccumulatedTotals(data);

    return GenerateActivitySummary(data);
}
```

Each step has one job. The sequence is explicit. The intermediate structure carries state between steps:

```csharp
private class GpxIntermediateData
{
    // Original data
    public double Latitude { get; set; }
    public double Longitude { get; set; }
    public double Elevation { get; set; }
    public DateTime Timestamp { get; set; }

    // Derived/accumulated
    public double SegmentDistance { get; set; }
    public double Speed { get; set; }
    public double ElevationDelta { get; set; }
    public bool IsGpsGlitch { get; set; }
    public double SmoothedElevation { get; set; }
    public string TerrainType { get; set; }
    public double AccumulatedDistance { get; set; }
    public double AccumulatedElevationGain { get; set; }
    public double AccumulatedElevationLoss { get; set; }
}
```

---

**Implementation:** Each step transforms the list and passes it forward.

```csharp
private List<GpxIntermediateData> MapToIntermediateData(List<GpxPoint> gpxPoints)
{
    return gpxPoints.Select(p => new GpxIntermediateData
    {
        Latitude = p.Latitude,
        Longitude = p.Longitude,
        Elevation = p.Elevation,
        Timestamp = p.Timestamp
    }).ToList();
}

private List<GpxIntermediateData> CalculateSegmentMetrics(List<GpxIntermediateData> points)
{
    for (int i = 1; i < points.Count; i++)
    {
        var current = points[i];
        var previous = points[i - 1];

        current.SegmentDistance = CalculateHaversineDistance(previous, current);
        current.Speed = CalculateSpeed(current.SegmentDistance,
                                       current.Timestamp - previous.Timestamp);
        current.ElevationDelta = current.Elevation - previous.Elevation;
    }

    return points;
}

private List<GpxIntermediateData> FilterGpsGlitches(List<GpxIntermediateData> points)
{
    foreach (var point in points)
        point.IsGpsGlitch = point.Speed > 30;

    return points.Where(p => !p.IsGpsGlitch).ToList();
}
```

---

**When to reach for this:** Any loop that mixes filtering, calculating, and accumulating. Booking workflows, sensor ingestion, reporting pipelines — the pattern fits wherever a single pass is doing the work of several.

---

**TLDR:** Complex loops grow because extending them feels cheaper than splitting them. A transformation pipeline forces the split upfront. Each step gets a name, a clear input, and a testable output — and adding a new requirement means adding a new step, not reading the whole thing again.
