---
title: Your system is an ETL pipeline
pubDatetime: 2025-04-23
description: "Understanding how most software systems inherently function as ETL pipelines and how to leverage this perspective for better system design."
slug: system-as-etl-pipeline
tags:
  - design
  - data
draft: false
---

Working on a connected devices project, I kept adjusting the mobile app's data handling — buffering reads from hardware, normalizing formats across manufacturers, batching uploads to the cloud. It felt like plumbing. It was plumbing. It took me longer than I'd like to admit to name what I was building: an ETL pipeline.

Years earlier I'd built one deliberately. Edge controllers in a security system collected data from card readers and door locks across different manufacturers, normalized everything into one format, and pushed it to the cloud for storage and reporting. Extract, transform, load. We called it an ETL pipeline from day one.

The mobile app was doing the same thing. I just hadn't named it.

Naming it changed how I thought about the system. Local storage wasn't an offline feature — it was the extraction buffer. The normalization code was the transformation layer, and the most valuable part of the codebase. The upload logic was the load step. Once I saw it that way, each piece had a clear job and could be improved independently.

It also resolved an argument we'd been having. The team wanted reporting on the device. I kept resisting without being able to say exactly why. The reason: reporting belongs where data is already aggregated — at the backend, after the load step. Putting it earlier means working with incomplete, un-normalized data.

There was bidirectional data too — settings and configuration pushed back down to devices. That's not ETL. Keeping it separate kept the design clean.

Once the pattern was named, standard ETL optimizations became obvious: batch before sending, deduplicate, compress. They were familiar solutions, applied once the problem came into focus.

Most data-moving systems are ETL pipelines. Many operate that way even when nobody names them that way in the design.
