---
title: "Refactoring complex loops with transformation pipelines"
date: 2025-04-20
dateUpdated: Last Modified
permalink: /posts/complex-loops-to-transformation-pipelines/
tags:
  - Code
  - Software Design
layout: layouts/post.njk
---

Complex loops that validate data, apply business rules, and generate statistics in one place become increasingly difficult to maintain. Each new requirement risks breaking existing logic, and testing these interdependent operations becomes challenging.

## The problem: complex loops

Complex loops typically start simple but grow organically as developers add "just one more condition" rather than refactoring. The example below started as basic GPS point processing but expanded with additional steps for elevation tracking, speed calculation, and terrain classification. With each new feature, the code became harder to maintain:

```csharp
public class ActivitySummary ProcessGpxData(List<GpxPoint> gpxPoints)
{
    // State variables
    double totalDistance = 0;
    double totalElevationGain = 0;
    List<GpxPoint> processedPoints = new List<GpxPoint>();
    
    for (int i = 0; i < gpxPoints.Count; i++)
    {
        var point = gpxPoints[i];
        
        if (i > 0)
        {
            var prevPoint = gpxPoints[i-1];
            
            // Calculate distance and speed
            double segmentDistance = CalculateHaversineDistance(prevPoint, point);
            double speed = CalculateSpeed(segmentDistance, point.Timestamp - prevPoint.Timestamp);
                
            // Skip GPS errors
            if (speed > 30) continue;
            
            // Track elevation changes
            double elevationDelta = point.Elevation - prevPoint.Elevation;
            if (elevationDelta > 0.5)
                totalElevationGain += elevationDelta;
                
            // Update accumulated values
            totalDistance += segmentDistance;
            
            // Update point with calculated metrics
            point.SegmentDistance = segmentDistance;
            point.Speed = speed;
            
            // Apply elevation smoothing (simplified)
            point.SmoothedElevation = CalculateSmoothedElevation(i, gpxPoints);
            
            // Classify terrain (multiple concerns mixed together)
            point.TerrainType = DetermineTerrainType(elevationDelta, speed);
        }
        else
        {
            // Initialize first point
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

These loops:

- Do too many things at once instead of separating concerns
- Resist changes as dependencies are hidden and unclear
- Make unit testing difficult by coupling unrelated operations together
- Create excessive mental load for developers trying to understand them

## The solution: transformation pipelines

The solution is to break down complex processing into a transformation pipeline where each step has a single responsibility:

```csharp
public ActivitySummary ProcessGpxData(List<GpxPoint> gpxPoints)
{
    // Convert raw GPS points to our working structure
    var intermediateData = MapToIntermediateData(gpxPoints);
    
    var withSegmentMetrics = CalculateSegmentMetrics(intermediateData); 
    var filteredPoints = FilterGpsGlitches(withSegmentMetrics);         
    var withElevationData = ProcessElevationData(filteredPoints);       
    var smoothedData = ApplyDataSmoothing(withElevationData);           
    var enrichedData = ClassifyTerrainTypes(smoothedData);              
    var processedPoints = CalculateAccumulatedTotals(enrichedData);     
    
    // Create final output with statistics
    return GenerateActivitySummary(processedPoints);                    
}
```

**Each step has a single responsibility and can be implemented, tested, and modified independently.** The key to making this work is an intermediate data structure that carries accumulated state between processing steps:

```csharp
private class GpxIntermediateData
{
    // Original data
    public double Latitude { get; set; }
    public double Longitude { get; set; }
    public double Elevation { get; set; }
    public DateTime Timestamp { get; set; }
    
    // Derived/accumulated data
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

## Implementation details

Let's see how some of the pipeline steps would be implemented:

```csharp
// Step 1: Map input to intermediate structure
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

// Step 2: Calculate metrics between adjacent points
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

// Step 3: Filter out GPS glitches based on unreasonable speeds
private List<GpxIntermediateData> FilterGpsGlitches(List<GpxIntermediateData> points)
{
    // Mark points with unreasonable speed as glitches
    foreach (var point in points)
    {
        point.IsGpsGlitch = point.Speed > 30; // 30 m/s threshold
    }
    
    // Filter out glitches
    return points.Where(p => !p.IsGpsGlitch).ToList();
}
```

## Benefits of the pipeline approach

- Each step can be tested independently
- Changes affect only single functions, not the entire process
- Pipeline steps clearly document the transformation sequence
- Issues can be isolated to specific steps
- Functions can be reused in other systems
- New steps can be added without changing existing code

Creating intermediate data structures improves code clarity, which is often worth the small performance cost in most applications. This same approach works for booking systems, commerce workflows, reporting pipelines, hardware sensor processing, or any multi-step data processing.

## Related patterns and principles

This approach leverages several established patterns:

- **Pipes and Filters.** Connected processing steps with data flowing between them
- **Single Responsibility Principle.** Each step has one reason to change
- **Map-Reduce.** Map raw data to workable form, transform it, reduce to output

Unlike distributed implementations of these patterns, our approach applies them at method level for everyday code.

## Conclusion

When you spot a loop doing too much (tracking state, applying business rules, and formatting output simultaneously), consider refactoring it to a data transformation pipeline.

Break problems into subproblems, solve them independently, then merge them in the final solution. This approach improves code quality and makes future maintenance easier.
