---
title: "Build the throwaway prototype first"
description: "Before integrating unfamiliar code into production, explore it in a throwaway project first."
pubDatetime: 2025-04-21
slug: power-of-prototyping
tags:
  - practices
draft: false
---

Something I reach for before touching production in unfamiliar territory: a prototype project.

**When to prototype:** Unfamiliar SDK. Complex refactoring with unclear blast radius. Performance uncertainty. Any integration where you don't know what you don't know yet.

**Structure:** Keep prototypes in a dedicated folder inside your solution:

```plaintext
YourSolution/
└── Prototypes/
    ├── EmailApiTrial/
    ├── NewSdkExploration/
    ├── RefactoringProposal/
    └── PerformanceBenchmarks/
```

**Constraints:** Minimal, disposable, self-contained. Document what you learned, not how the code works. Don't carry production quality expectations in here.

**Bring concepts, not code.** The output of a prototype is understanding. When you're done, production code gets written from scratch using what you learned — not copy-pasted from the prototype.

**With AI:** LLMs accelerate prototyping significantly. For a new SDK: _"Generate a C# test project that exercises [SDK] authentication, main API calls, and error handling."_ You get scaffolding in seconds. From there, test error cases, check limits, understand the failure modes before committing to an abstraction.

**Why it saves time:** A few hours of prototyping surfaces technical limits and wrong assumptions before they become production bugs. Estimates get more accurate. Design decisions get grounded in something real rather than what you imagined the SDK would do.

**TLDR:** Create a throwaway project and explore there first. The prototype teaches you what to build. Then build it properly.
