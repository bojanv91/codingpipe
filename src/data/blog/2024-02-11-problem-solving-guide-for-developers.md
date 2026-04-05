---
title: Problem-solving guide for developers
pubDatetime: 2024-02-11
description: "The four-step process I use to understand programming problems, choose an approach, implement safely, and improve the result without wasting time."
slug: problem-solving-guide-for-developers
tags:
  - practices
draft: false
---

I used to lose a lot of time by starting implementation too early. I would make progress fast for an hour, then realize I had solved the wrong problem or picked an approach that made the rest of the work harder than it needed to be.

Over time I noticed I was following the same pattern whenever I solved hard problems well. It comes down to four steps:

1. **Analyze**: understand the problem
2. **Plan**: decide how to solve it
3. **Implement**: ship in small, testable steps
4. **Review**: check the result and refine it

## Step 1: Analyze - understand the problem

I start by restating the problem in my own words. If I cannot explain what is broken, missing, or risky in one or two plain sentences, I am usually not ready to code.

For bugs, reproduction comes first. Can I reproduce it locally? Does it happen only with certain data? Is it specific to one environment? Until I can answer those questions, everything else is guesswork.

For third-party library issues, I usually build a tiny prototype before touching production code. That tells me whether the problem is in my code, in my assumptions, or in the library itself. It also gives me a much better search query for docs, issue trackers, and source code.

For features, I read the spec, inspect the existing code in that area, and look for nearby patterns that the team already trusts. That helps me ask better questions before I start building.

The important part is to stop once the problem is clear enough. Analysis is useful until it becomes procrastination.

## Step 2: Plan - determine how to resolve the problem

I use prototypes to learn fast. Sometimes I need to answer a design question, sometimes I need to measure performance, and sometimes I just need to prove that an idea works before I commit to it.

Once I understand the options, I compare them against the constraints that actually matter: delivery time, complexity, failure modes, team familiarity, and long-term maintenance.

Then I write down a plan. Sometimes that is a short note with three implementation steps. Sometimes it is a larger design document with diagrams and trade-offs. The format matters less than the outcome: I want a plan that makes the next coding step obvious.

## Step 3: Implement and test incrementally

I start with the smallest slice that proves I am moving in the right direction. That might be a failing test, a narrow refactoring, or the most critical piece of behavior.

Small changes are easier to reason about, easier to review, and easier to roll back. They also reveal bad assumptions early, before those assumptions spread through the rest of the implementation.

As I go, I keep testing. Depending on the problem, that can mean automated tests, local debugging, targeted logging, or manual verification. I also keep the team in the loop when a discovery changes the plan.

## Step 4: Review and refine

Before I call the work done, I check whether I actually solved the original problem instead of just producing code that looks reasonable.

Then I review the code as if I were reading it for the first time. Is the intent obvious? Are the trade-offs acceptable? Did I leave behind unnecessary complexity? If performance matters, I verify it instead of assuming it is fine.

Rule of thumb: if I feel pressure to code immediately, that is usually a sign I should slow down and spend a few more minutes understanding the problem first.
