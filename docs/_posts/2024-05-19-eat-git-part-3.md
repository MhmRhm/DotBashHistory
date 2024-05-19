---
layout: post
title: "EatGit - Part 3"
excerpt: Part 3 of a quick Git tutorial to teach you everything you need to know.
seo_description: Part 3 of the Git tutorial - Learn about patch files, cherry-picking, and interactive rebasing.
categories: git
author:
- Mohammad Rahimi
---

* Table of Contents
{:toc}

## Preface

In [Part 1]({{ site.baseurl }}{% link _posts/2024-05-08-eat-git-part-1.md %}),
we covered the essentials of Git, including how to create commits and branches
and how to investigate the history of a project. In
[Part 2]({{ site.baseurl }}{% link _posts/2024-05-15-eat-git-part-2.md %}), we
explored how and where Git stores its data structures. We examined what goes
inside a commit and how branches are represented in Git. Additionally, we
created a commit using Git's *Plumbing* commands. In this part, I'll wrap up our
Git journey as a solo developer. We'll delve into more advanced tools that we
may need to use from time to time.

## Patch Files


