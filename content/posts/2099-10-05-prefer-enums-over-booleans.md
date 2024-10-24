---
title: "Prefer Enums Over Booleans"
date: 2099-10-05
dateUpdated: Last Modified
permalink: /posts/prefer-enums-over-booleans/
tags:
  - Programming
layout: layouts/post.njk
draft: true
---

Prefer enums over booleans for status/state fileds

>Enum for fixed choice lists, with little scope to change. Reference table for options that may have relationships. Boolean for binary (duh). Free text for unique user input (like comments, descriptions etc). That simple no?

## Problem

It is tempting to use `bool` to represent a choice between different things.

`bool` has only two values, and if our choices increase we can't add more values to it.
To add a third option, we need a second `bool` value, and the amount of possible states increases in powers of 2.

Example:

IsActive to active, inactive, paused, archived.
..IsPaused

...IsActive
...IsArchived
...IsBanned
...IsPendingVerification

Change IsActive to a Status field, so we can expand it down the line.

## Solution

When?

ut when we have the inviatable requirement change that we need to track if the Device is in unknown state. How easy is to change this?

Enums can accommodate additional states without chaning the existing code structure. If you need to add a third or more state later, it's much easier with an enum than a bool.

Why?

How many times you've been in the middle of a refactoring nighmare where you had to change User.IsActive to User.Status field. Initially **IsActive** worked well, but later when new states were needed it was hard to make the change. I've been bitten and I've seen many teams been bitten by this common use case, where we think that "just add a bool" will solve the job well. The **Status** field now has a third option, Active, Inactive, Archived, and it's starts to get confusing what Inactive means. So we decide to change the terms a bit: Active, Deactivated, Archived. Deactivated speaks better than Inactive, because Inactive can mean that the user is not using the app for some time and that's why their account is innactive. Deactivated speaks concisly that the account was deactivated. We added the thirdoption Archived, wich also may be mapped to IsActive=false the same way as IsActive=false maps to Deactivated. So using enums clears stuff up.

What's the actual process of changing from bool to enum in real codebase?

If we've designed this as enum with two fields in the beggining, it would be the same of there are 2 options, or more options, nothing to think about for the future possible changes.


When passing parameters to a function, it’s much clearer to use an enum. Consider something like, addItem(name, true) vs. addItem(name, APPROVED).