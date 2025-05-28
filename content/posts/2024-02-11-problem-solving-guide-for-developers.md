---
title: Problem-solving guide for developers
date: 2024-02-11
dateUpdated: Last Modified
permalink: /posts/problem-solving-guide-for-developers/
layout: layouts/post.njk
---

Jumping straight into writing code without understanding the problem clearly often leads to wasted time and effort. A structured approach helps avoid this common pitfall.

A couple of years ago I discovered George Pólya's "How to Solve It." The ideas from his book helped me categorize and describe a similar process I was already using. Here's the four-step process:

1. **Analyze** - Understand the problem
2. **Plan** - Determine how to resolve the problem  
3. **Implement** and test your solution incrementally
4. **Review** and refine your solution

## Step 1: Analyze - understand the problem

I start by defining the problem clearly, often rewriting the problem statement until I truly understand it. For complex problems, I break them down into smaller, manageable pieces.

When dealing with bugs, I focus on reproduction first. Can I reproduce it locally? Is it environment-specific or tied to particular data state? For third-party library issues, I create prototypes and PoCs to reproduce problems in isolation, dive into official documentation and GitHub issues using specific error messages, and leverage Google searches and LLM tools for additional context.

For new features, I read all related specs and explore the existing codebase in that area. This helps me understand the problem better and ask informed questions.

The key is knowing when to stop analyzing and avoid analysis paralysis.

## Step 2: Plan - determine how to resolve the problem

I use prototyping (or you may call it spiking, PoC-ing, experimenting, investigating, etc.) as a key tool for learning and determining multiple approaches to find the best solution. I brainstorm possible solutions and involve my team in evaluating each option's pros, cons, and trade-offs within our business context.

After prototyping and evaluation, I create solution designs (tech specs) or brief implementation plans depending on team maturity and system complexity. Sometimes that's two paragraphs, sometimes a diagram, and sometimes a dozen pages with various diagrams and references.

## Step 3: Implement and test incrementally

I start with the simplest or most critical piece and write working code. Small, testable changes work better than big rewrites. I test each iteration and use debugging tools, logging, and automated tests to catch issues before QA or users do.

Communication with the team and adjusting the plan as needed keeps the implementation on track.

## Step 4: Review and refine

I ensure the solution actually solves the original problem, then review the code as if someone else wrote it. I refactor for readability and performance, but stay pragmatic about what needs optimization.

This process has saved me countless hours of debugging poorly planned solutions. The time invested upfront in understanding and planning pays dividends when the implementation goes smoothly.
