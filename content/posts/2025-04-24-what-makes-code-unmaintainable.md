---
title: 8 factors that make code unmaintainable
date: 2025-04-24
dateUpdated: Last Modified
permalink: /posts/what-makes-code-unmaintainable/
tags:
  - Software
  - Design
  - Code
layout: layouts/post.njk
---

Software systems inevitably change. The challenge is we rarely know when or where changes will be needed. Experience in a domain helps make better design decisions, especially for systems in production where you need frequent updates without breaking functionality.

## 1. Excessive coupling

Code coupled with many dependencies creates a chain reaction of changes. Modifying one class requires changes in other dependent classes, which triggers more changes elsewhere. This "dependency hell" is why simple changes often take days instead of minutes.

I've seen teams spend entire sprints on what should have been simple configuration changes because they required updates across dozens of tightly coupled components. When you can't change anything without changing everything, maintenance becomes a nightmare.

## 2. Solving imaginary future problems

Code that tries to solve future or speculative problems that haven't happened yet quickly becomes unmaintainable. We often don't know what tomorrow's problems will be, yet many developers build "just in case" features.

The key is making things right for today while enabling tomorrow's changes without implementing them. Follow YAGNI: don't add functionality until you need it.

## 3. Side effects and unclear intentions

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

From the method name "DepositAmount", would you expect it to upgrade the account tier and send emails? Methods should do what their name suggests, nothing more. When code does unexpected things, debugging becomes a puzzle where every method call might trigger hidden behavior across the system.

## 4. Knowledge duplication

When code is duplicated, changing requirements means updating multiple places. Miss one, and you create bugs. Business rules duplicated across the codebase often drift apart over time.

I follow the Rule of Three: tolerate duplication twice, but refactor on the third occurrence. Just ensure the duplicated code truly represents the same concept. Sometimes similar-looking code serves different purposes and should remain separate.

## 5. Inconsistent coding style

When code follows different conventions and patterns, developers waste mental energy switching contexts. Inconsistency goes beyond formatting: it includes naming, error handling, and how concepts are represented. The codebase should look like it was written by one person.

## 6. Poor structure

Code becomes unmaintainable when it's difficult to find where to make changes. Since developers spend about 10 times more time reading code than writing it, structure matters. The fundamental rule: code that changes together should be placed together. When related functionality is scattered across the codebase, even simple additions require modifying many files, increasing the risk of overlooked side effects."

## 7. Over-engineering

The worst over-engineering happens in the name of extensibility. Code designed to handle every possible future scenario often can't handle the actual changes that occur.

Design for changes you've already seen. If configuration values changed three times last year, make them configurable. If the core algorithm has been stable for two years, leave it alone. The paradox: 'flexible' code often becomes inflexible for real-world changes.

## 8. Under-engineering

Under-engineering creates technical debt through copy-paste coding, magic numbers, missing error handling, and functions doing too much. What seems faster initially slows all future development.

## Focus on what changes most often

Git can identify your most frequently changed files with:

```shell
git log --format=format: --name-only | grep -v '^$' | sort | uniq -c | sort -nr | head -20. 
```

Files appearing in more than 25% of commits deserve extra attention for maintainability.

## Final thoughts

The true test of maintainable code isn't whether it works today, but whether a new team member can understand and modify it six months from now. As Joel Spolsky noted, "It's harder to read code than to write it" -- which is why our job isn't just writing code that works, but code that others can easily understand and change.

Every decision that makes code harder to understand compounds over time, eventually creating what Michael Feathers calls "legacy code": code we're afraid to change. When reviewing or writing code, ask yourself if you're introducing any of these maintainability problems. Awareness alone can help you avoid the most common pitfalls.

***EDIT (2025):** Even AI tools work better when the codebase is maintainable. AI assistants can more accurately suggest improvements and detect bugs when code follows consistent patterns, has clear naming, and maintains proper separation of concerns. Conversely, the maintainability issues described above often confuse AI tools just as they confuse human developers.
