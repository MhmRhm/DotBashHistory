---
layout: post
title: "Eat Git - Part 2"
categories: git
author:
- Mohammad Rahimi
---

* Table of Contents
{:toc}

## Preface

In [Part 1]({{ site.baseurl }}{% link _posts/2024-05-08-eat-git-part-1.md %}),
we covered the main commands of Git, so now we know how to make a commit. In
this part, we’ll take a look behind the scenes and create a commit using Git
plumbing commands. Git commands are divided into two sets: Porcelain and
Plumbing. Porcelain commands are the ones you use for everyday tasks, while
Plumbing commands are the lower-level commands that Git combines to perform higher-level tasks.

## Objects



[pro-git-book]: https://git-scm.com/book/en/v2
