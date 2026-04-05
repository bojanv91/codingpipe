---
title: "Git worktrees for parallel branch work"
pubDatetime: 2025-12-10
description: "Using git worktrees to work on multiple branches simultaneously without managing local file changes."
slug: git-worktrees-parallel-branch-work
tags: [devops]
---

I discovered git worktrees, a feature that's been in git for years but I only learned about recently.

I needed to switch to a release branch, but had local chaos in my main branch that I'd been working on for days. I also needed to switch back to main to finish other work. My only solution was manually cloning the project again. Then I found git worktrees - they let you switch between branches WITHOUT managing local file changes, and you can keep multiple branches open simultaneously.

I tested this by creating worktrees for different branches. Each worktree is a separate working directory pointing to the same repository. I had main open in one IDE window, a release branch in another, and a PR branch in a third - all at the same time.

## Some scenarios where this helps

**Switching to assist on another branch** - Open a worktree without messing with your current work.

**Testing on a stable branch** - Quickly verify something works on production without stashing or committing incomplete work.

**Code review and dev testing across multiple PRs** - Keep PR1, PR2, PR3 open simultaneously for comparison.

I can context-switch between branches instantly without git stash gymnastics. I now use worktrees whenever I need to touch more than one branch in a session.

Deeper dive: [Working on two git branches at once with git worktree](https://andrewlock.net/working-on-two-git-branches-at-once-with-git-worktree/)
