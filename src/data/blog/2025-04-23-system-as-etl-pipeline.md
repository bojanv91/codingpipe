---
title: The hidden ETL pipeline in your system
pubDatetime: 2025-04-23
description: "Understanding how most software systems inherently function as ETL pipelines and how to leverage this perspective for better system design."
slug: system-as-etl-pipeline
tags:
  - architecture
  - data
  - patterns
draft: false
---

I worked on a connected devices project where mobile app communicated with hardware devices via Bluetooth, processed raw data sent from different device manufacturers and sent it to cloud services in a standardized format. During development, I realized one part of the system was functioning as an ETL pipeline, but it was not originally designed as one.

This reminded me of a security system I had worked on years earlier. Edge controllers collected data from card readers and door locks (also from different manufacturers), then sent it to the cloud for storage and real-time reporting. These controllers extracted data from different hardware types, transformed multiple formats into one standard format, and loaded everything to cloud services. We had deliberately designed this sub-system as an ETL pipeline.

Both systems followed the same pattern: extract, transform, load. The mobile app in our connected device project was performing the exact same role as those edge controllers.

Recognizing this pattern opened up opportunities to improve the existing system. The mobile app was our transformation engine. Local storage existed for reliability, not offline functionality. The data transformation rules were our most valuable code. This let us improve each part separately: extract data from devices reliably, transform it more efficiently, and optimize the load to servers.

It also clarified architectural decisions. Reporting belonged at the backend where data was already processed and aggregated, not on devices or mobile app.

While both systems sent data back down (settings and configurations), this was separate from the ETL process. Keeping these concerns apart made the design cleaner.

Seeing this as ETL opened up standard optimizations: batching data before sending, cleaning up glitches, removing duplicates, and compressing data for efficient transfer.

What looked like a unique technical challenge was actually a familiar problem in disguise. The key was recognizing the pattern and applying proven solutions.
