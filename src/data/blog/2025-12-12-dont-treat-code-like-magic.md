---
title: "Don't treat code like magic"
pubDatetime: 2025-12-12
description: "Why you should reproduce problems reliably, document your investigation, test assumptions, and map dependencies before implementing fixes."
slug: dont-treat-code-like-magic
tags: [practices]
---

I refuse to implement solutions I don't understand. When code behaves unexpectedly, I don't reach for quick fixes.

## Reproduce first

I can't fix what I can't reliably reproduce. Before investigating, verify that you and your teammates can reproduce the problem consistently. If reproduction is flaky, you're chasing a symptom, not the root cause.

## Document while you investigate

Debugging takes days sometimes. When you trace execution paths and test hypotheses, document your findings. Your future self and teammates need to understand why the system behaves this way.

## Test your assumptions

When you think you understand how code works, prove it with automated tests. If tests pass for the wrong reasons, you still don't understand the problem.

## Map the dependencies

Before implementing any fix, check what depends on the current behavior. What other components will your change affect? What assumptions in your codebase does this break?

## Conclusion

**There's no magic in code. Only logic you haven't understood yet.**

Related: Rich Hickey's ["Simple Made Easy"](https://www.youtube.com/watch?v=SxdOUGdseq4) explains why unfamiliar code feels hard but isn't necessarily complex. You can learn unfamiliar patterns, but you can only fix complexity by untangling it.
