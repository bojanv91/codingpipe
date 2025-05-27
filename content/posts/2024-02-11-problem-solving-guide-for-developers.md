---
title: Problem-Solving Guide for Developers
date: 2024-02-11
dateUpdated: Last Modified
permalink: /posts/problem-solving-guide-for-developers/
tags:
  - Productivity
layout: layouts/post.njk
---
The most common mistake developers make when solving problems is immediately starting with writing code for a solution that is not properly thought out for a problem that is not clearly understood.

There are countless ways how to approach problem-solving. In this post, I describe one approach with a couple of tips for software developers.

The process is made of four steps:

1. **Analyze** - Understand the problem
2. **Plan** - Determine how to resolve the problem
3. **Implement** and test your solution incrementally
4. **Review** and refine your solution

## Step 1: Analyze - Understand the problem

General tips:
- Define the problem clearly. Sometimes, you need to rewrite the problem statement so you can better understand it.
- If the problem is too complex or too large, break it down into smaller, more manageable subproblems.
- Identify possible causes of the problem, and focus on finding the root cause. You can use techniques such as the 5-Whys, flowcharts, and hypothesis testing.
- Ask questions and clarify assumptions to avoid wasting time on irrelevant or incorrect solutions.
- Validate your thinking by describing your understanding with your team. Seek other perspectives.

If the problem is a bug caused by your code, try to reproduce it locally and in other environments as well. Is the bug consistently happening everywhere, or is it tied to a specific environment with a specific state of the database?

If it's a bug caused by the usage of a third-party service or a library (e.g., Stripe, SendGrid, RabbitMQ, EntityFramework, etc.), do research using the official documentation, discussion/support forum or the GitHub repository of the service/library. Also, you can research on the web or on chat AI tools (e.g., Bing Copilot) by using the specific error message or code.

And if it's a new feature request, open and read mindfully any related documents, specs, attachments, discussions, or comments for the given feature request. Then, explore the codebase in the area where the feature is being requested. Make sure you get familiar with the code, especially if it's unknown to you. This will help you understand the problem better and ask better questions.

Some problems require the majority of the problem-solving time spent on analysis, and others less so. Whatever the case, make sure you timebox this activity and avoid getting into analysis paralysis.

## Step 2: Plan - Determine how to resolve the problem

- Brainstorm and generate possible solutions. Involve your team in the process.
- Think about the possible solutions, their pros and cons, risks, and trade-offs. Include them in your planning and design documents. Also, try to specify short-term and long-term solution options as well.
- Evaluate and select the most suitable solution that would solve the problem. Try to think pragmatically, taking into account the greater business context (schedule, experience, team capacity, impact, cost, urgency, etc.).
- Remember that any chosen solution has trade-offs. Be aware of them and communicate them clearly with the rest of the team (ideally, this will be part of the technical design document).
- Design and create an action plan for the selected solution approach. Use techniques like pseudocode, prototypes, technical design documents, flowcharts, and sequence diagrams to plan and design your solution.

## Step 3: Implement and test your solution incrementally

- Execute the action plan and monitor the progress. Communicate and collaborate with the team and adjust the plan as needed.
- Start with the simplest or most important subproblem and write working code.
- Keep it small, keep it testable.
- Make changes in code incrementally until the entire problem is solved.
- Test your code in each iteration.
- Use debugging tools, logging, and unit/integration testing to find and fix errors in your implementation before anyone from QA or end-users finds them.

## Step 4: Review and refine your solution

- Ensure your solution solves the problem.
- Review your code in the same way that others will review it and make any necessary tweaks. Ensure you followed the agreed coding conventions and best practices.
- Refactor your code for readability, security, and performance.
- Be mindful and pragmatic when choosing what to optimize and how much time and energy you'll spend. Optimize "just enough".
