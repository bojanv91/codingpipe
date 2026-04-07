---
title: "Wrapping Google Sheets as a backend instead of migrating away from it"
pubDatetime: 2025-04-22
description: "A practical approach to using Google Sheets as a lightweight backend solution, preserving existing business logic while avoiding unnecessary rebuilding of systems."
slug: google-sheets-as-a-backend
tags:
  - design
draft: false
---

Years ago, I worked on a project with interconnected pricing spreadsheets and a tight deadline. The team had spent years perfecting complex calculations across these sheets, updating formulas weekly based on market conditions. The client needed their sales team to access these calculations via a web application, but rebuilding and testing all that Excel logic in code would take months.

So we didn't rebuild it.

When business experts have already built and validated complex calculations in spreadsheets, rewriting them is a trap. You spend weeks translating their work into code, introduce bugs in the translation, then build a change management process for every future update. The Google Sheets API offers a different path: build around the existing work instead of replacing it.

The architecture is straightforward. A template spreadsheet holds all the calculations — maintained by the business team, not developers. For each user session, the integration service creates a copy of that template, writes user inputs into the appropriate cells, reads the calculated results back out, then deletes the copy. The web interface handles input and display. The spreadsheet handles calculation.

In our case, the pricing team kept refining rules directly in the template. Sales agents used a web form to input deal parameters. The system generated quotes using the same calculations the team had been perfecting for years. When market conditions changed, the team updated the template — no deployment cycle, no ticket, no waiting.

This pattern has real constraints. Google's API rate limits make it unsuitable for high-throughput systems. Each API call adds latency. Session isolation via spreadsheet copies adds overhead. For internal tools with moderate traffic, none of these are dealbreakers. For anything requiring high concurrency, they are.

What matters here is who owns the business logic. In some systems, that ownership already sits with domain experts and the spreadsheets they rely on every day. The real opportunity is to recognize that and build around it.
