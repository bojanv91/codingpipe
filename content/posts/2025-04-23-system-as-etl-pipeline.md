---
title: "How I found the same ETL pattern in two different systems"
date: 2025-04-23
dateUpdated: Last Modified
permalink: /posts/system-as-etl-pipeline/
tags:
  - Programming
layout: layouts/post.njk
---

I recently realized that two different systems I've worked on are actually doing the same thing - they're both a style of ETL (Extract, Transform, Load) pipelines.

For years, I worked on a security system where edge controllers collect data from card readers and door locks and push it to the cloud. These controllers do three main things:

- Extract data from many different hardware types
- Transform the data from many formats into one standard format
- Load the data to cloud services/databases

I always saw this as just a system with syncing capability. I never thought of that part of the system as an ETL.

Recently, I was working on a wearable device project with these parts:

- Mobile apps connect to wearables using Bluetooth
- The apps process raw device data from various manufacturers that we need to support
- The apps send the processed data to cloud services

I was struggling with some design problems in the this project. Then it hit me - this is the same pattern! The mobile app is doing exactly what the edge controller did: Extract, Transform, Load.

Using what I learned from the security project, I could then see:

- The mobile app is the transformation engine, just like the edge controller was
- Local storage is mainly for making transfers reliable, not for offline use
- The rules for transforming/mapping data formats are the most valuable part
- The data mainly flows one way (device → app → server)

By applying these ETL concepts from the security system to our wearable project, we are able to focus on improving each part separately: getting better data from devices, creating better rules for transforming/mapping data, and sending data to servers more efficiently.

This ETL view also clarifies other parts of the system:

1. **Reporting is simpler from the destination**: We don't need complex reporting capabilities on devices or in the mobile app. Reports are much easier to create from the already processed and aggregated data in the backend.

2. **Separating ETL from configuration**: Both systems do have some data flowing from cloud servers down to devices (like settings and configurations), but this is a separate concern from the ETL process. Keeping these separate makes both easier to manage.

3. **Optimization opportunities**: Seeing this as ETL opens up many ways to improve the pipeline:

    - Batching data before sending
    - Cleaning up glitches or unnecessary data points
    - Removing duplicate data entries
    - Compressing data for efficient transfer

Sometimes the best solutions come from seeing the patterns across different projects.
