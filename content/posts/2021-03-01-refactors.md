---
template: post
title: Perils of code refactors
slug: refactors
draft: true
date: 2021-03-01T20:34:50.892Z
description: >-
  Code refactors are a tricky thing. It's easy when wearing an engineer hat to
  focus too much on the correctness of the code. Incorrect code feels like
  having a dirty house and refactoring is a way to make it feel clean again.
  We've got to be particularly careful that we don't refactor with a 'grass is
  greener on the other side' bias, as then all the work will have been for
  almost nothing. Does that new way of doing things really solve a problem
  you're frequently running into that significantly slowing you down? 
category: engineering
tags:
  - engineering
---
_This post is a work in progress_

__

Code refactors are a tricky thing. 

It's easy when wearing an engineer hat to focus too much on the correctness of the code. Incorrect code feels like having a dirty house and refactoring is a way to make it feel clean again. We've got to be particularly careful that we don't refactor with a 'grass is greener on the other side' bias, as then all the work will have been for almost nothing. Does that new way of doing things really solve a problem you're frequently running into that significantly slowing you down? 

Refactors often introduce new bugs and are much more work than anticipated. 



Some refactors are easy and can be done in a day or less. 

The problematic sort however is the refactor that will take weeks of work. 

You can choose to do it all at once or do it progressively over time. 



If you do it all at once you might end up with a two versions of the codebase each continuing to be worked on and that need to be merged. That's not a problem if the refactor doesn't touch much code, but when a refactor takes weeks it usually touches lots and lots of code, all across the project. 



Refactors this large are hard to estimate - they don't come along that often and so we usually don't have many comparable tasks from the past and how long they ended up taking. Refactors often introduce a new paradigm and one that we might be less familiar with, adding yet another hurdle to estimating it accurately. 

We don't know the unknown unknowns of the refactor - which things will have weird quirks, integration problems, dependencies that need updating, new bugs and so on. The scope can keep growing as we get deeper and deeper into the refactor. 



Progressive refactors have their own problems. We need to maintain two ways of doing things. We need to urge new work to use the new way, when for many people in the team the old way is more familiar and faster. If we're pressed for time we will choose the faster, old way rather than the new way. We should avoid progressive refactors if we can quickly refactor existing code to the new way. Sometimes it would only take 20 minutes, but it's 20 minutes of copy and pasting that many people would rather avoid. Sometimes these two ways of doing things stick around for a long time as people lose motivation or forget to change the way of doing things. 



Some learnings for avoiding the perils of refactors:

\- Choose opinionated frameworks with robust communities that are likely to satisfy future needs so that you won't have to refactor, or if you do that path will have been travelled by others before you

\- We proactive around preventing scope creep and make sure everyone knows there be dragons in refactor scope creep

\- Don't use high risk tools that you will possibly need to refactor away from. Tools can be considered high risk if they have opinionated styles tightly coupled to code in your codebase, with waning or small communities and slowing contributions. Possibly avoid complex tools that you don't understand or can't easily modify their behaviour.

\- Use codemods if possible to refactor large amounts of code at once

\- Look through past bugs, open tasks, issues, and their blockers and try to see if these match your expectations regarding what this refactor would unlock. Make the goal of the refactor clear, and define precisely how much time saved you might expect per month of work in the future. This can be hard but it forces us to see refactor work as an investment with an ROI for the customer and company. If you can show the value, then it should be easy to prioritize for any company looking to make good long term (1+ year) decisions

\- Don't prematurely optimize, refactoring to solve future problems such as scaling

\- Be careful about refactoring because you want to learn a framework. It may be interesting and a great learning opportunity for you, but it's often not what's best for the company

\- The popular tools that you read about on twitter are often those used and promoted by large companies like facebook and google that have very different constraints and resources than you. For their teams of thousands of engineers and many devops engineers a tool might be perfect - for your 3 person startup perhaps its too complex and too high maintenance. 

\- Beware of writing your own code for things there are libraries for, particularly if that part of the code is not your product's core differentiator (see wardley mapping). These libraries are built by professionals and it's surprisingly hard to get some things to work well. For example performant, accessible, documented and well designed dropdowns take time to create. While writing it yourself can give more flexibility it means more maintenance and more code. The goal is often to have less code - but not at the cost of readability. 

\- Beware of non headless UI libraries as often customizing them is hard and making them consistent with CSS and package updates can be a pain

\- Beware of updating packages to the newest version right away. You might run into bugs or compatibility issues and waiting a couple weeks will give someone else the opportunity to run into these and produce a solution.
