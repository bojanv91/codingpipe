---
title: "What Makes Code Unmaintainable?"
date: 2025-04-24
dateUpdated: Last Modified
permalink: /posts/what-makes-code-unmaintainable/
tags:
  - Programming
layout: layouts/post.njk
---

***From the archives. Originally written in 2016, edited in 2025.***

Software systems inevitably change. The challenge is we rarely know when or where changes will be needed. Experience in a domain helps make better design decisions, especially for systems in production where you need frequent updates without breaking functionality.

Before judging any code, we need to understand what problem it solves and which trade-offs were considered. However, certain patterns consistently make code harder to maintain:

## 8 Factors That Make Code Unmaintainable

### 1. Excessive Coupling

Code coupled with many dependencies creates a chain reaction of changes. Modifying one class requires changes in other dependent classes, which triggers more changes elsewhere. This "dependency hell" is why simple changes often take days instead of minutes.

I've seen teams spend entire sprints making what should have been a simple configuration change, as it required updates across dozens of tightly coupled components.

Excessive coupling means you can't change anything without changing everything, turning maintenance into a nightmare.

### 2. Solving Imaginary Future Problems

Code that tries to solve future or speculative problems that haven't happened yet quickly becomes unmaintainable. We often don't know what tomorrow's problems will be, yet many developers build "just in case" features.

The key is finding the balance: make things right for today while enabling (not implementing) tomorrow's changes.

As the YAGNI principle states: don't add functionality until you actually need it.

### 3. Side Effects and Unclear Intentions

Consider this example:

```csharp
void DepositAmount(Account account, decimal amount)
{
    account.Balance += amount;
    account.LastActivityDate = DateTime.Now;
    account.UpgradeAccountTier();  // Side effect!
    SendPromotionalEmails(account);  // Another unexpected side effect!
}
```

From the method name `DepositAmount`, would you expect it to upgrade the account tier and send emails? This violates two key principles:

- **Side-effect free:** Methods should do what their name suggests, nothing more
- **Intention revealing:** Names should clearly indicate what the code does

Hidden side effects create bugs and make debugging difficult. When code does more than its name suggests, understanding system behavior becomes a puzzle.

### 4. Knowledge Duplication

When code is duplicated, changing requirements means updating multiple places. If you forget to update everything, you create inconsistent behavior and bugs.

When business rules are duplicated, they might be implemented slightly differently in multiple places. When requirements change, you must find every instance.

I follow the [Rule of Three](https://en.wikipedia.org/wiki/Rule_of_three_(computer_programming))  for refactoring duplicated code: duplicate once if needed, but when you see it a third time, it's time to abstract.

### 5. Inconsistent Coding Style

When code follows different conventions and patterns, developers waste mental energy switching context. The codebase should look like it was written by a single person.

Inconsistency goes beyond formatting - it includes naming, error handling, and how concepts are represented. When similar operations are implemented differently throughout the codebase, understanding becomes unnecessarily difficult.

### 6. Poor Structure

Code where it's difficult to find where to make changes quickly becomes unmaintainable. Developers spend about 10 times more time reading code than writing it.

The fundamental rule for structuring code is: **Code that changes together should be placed together.**

When related functionality is scattered across the codebase, even simple feature additions require modifying many files, increasing the risk of overlooked side effects.

### 7. Over-Engineering

The worst over-engineering happens in the name of extensibility, yet often achieves the opposite. Over-engineered code becomes rigid and complex despite intentions to make it flexible.

Signs of over-engineering include:

- Generic solutions for specific problems
- Design patterns used without clear necessity
- Excessive abstraction layers
- Complex inheritance hierarchies

The paradox is that code designed to be "flexible" often becomes inflexible for the changes that actually happen.

### 8. Under-Engineering

Under-engineering leads to "careless-driven development" and creates technical debt. Under-engineered code lacks proper abstractions, error handling, and consideration of edge cases.

Signs include:

- Copy-paste coding
- Magic numbers and hardcoded values
- Missing error handling
- Functions that do too many things at once

Under-engineered code seems faster to write initially but slows down all future development.

## The Thin Line of Maintainability

Not all code changes equally. The parts that change most frequently should be the easiest to change, so focus your maintainability efforts there.

You can use Git commands like `git log --stat` to identify which files change most frequently. By focusing on high-churn areas, you get the best return on your maintainability investment.

## Focus on What Changes Most Often

Maintainability isn't binary across the whole codebase. The parts that change most frequently should be the easiest to change. Not all code changes often, so focus your maintainability efforts on high-churn areas.

You can use Git to analyze how your codebase changes over time to identify these areas. Commands like `git log --stat` or specialized git history analyzers can reveal which files change most frequently, giving you clear targets for maintainability improvements. By focusing your attention where it matters most, you get the best return on your maintainability investment.

## Final Thoughts

Hardware is hard to change. Software should remain "soft", easy to modify. When teams fear making changes, the software has hardened into something more like hardware.

As Joel Spolsky said, "It's harder to read code than to write it." Our job isn't just writing code, it's writing code that others (including our future selves) can easily understand and change.

The next time you're reviewing code or writing your own, consider these factors. Are you introducing any of these maintainability problems? The awareness alone can help you avoid the most common pitfalls.

***EDIT (2025):** Even AI tools work better when the codebase is maintainable.*
