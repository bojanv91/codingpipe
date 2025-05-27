---
title: "The hidden ETL pipeline in your system"
date: 2025-04-23
dateUpdated: Last Modified
permalink: /posts/system-as-etl-pipeline/
tags:
  - Programming
layout: layouts/post.njk
---

A few months ago, I worked on a connected device project where mobile apps communicated with hardware via Bluetooth, processed raw data from different manufacturers and sent it to cloud services. During development, I realized one part of the system was functioning as an ETL pipeline, but it was not originally designed as one.

This reminded me of a security system I had worked on years earlier. Edge controllers collected data from card readers and door locks, then sent it to the cloud. These controllers extracted data from different hardware types, transformed multiple formats into one standard format, and loaded everything to cloud services. We had deliberately designed this as an ETL pipeline.

Both systems followed the same pattern: Extract, Transform, Load. The mobile app in our connected device project was performing the exact same role as those edge controllers.

Recognizing this pattern opened up opportunities to improve the system. The mobile app was our transformation engine. Local storage existed for reliability, not offline functionality. The data transformation rules were our most valuable code. This let us improve each part separately: extract better data from devices, transform it more efficiently, and optimize the load to servers.

It also clarified architectural decisions. Reporting belonged at the backend where data was already processed and aggregated, not on devices or mobile apps.

While both systems sent data back down (settings and configurations), this was separate from the ETL process. Keeping these concerns apart made the design cleaner.

Seeing this as ETL opened up standard optimizations: batching data before sending, cleaning up glitches, removing duplicates, and compressing data for efficient transfer.

What looked like a unique technical challenge was actually a familiar problem in disguise. The key was recognizing the pattern and applying proven solutions.
