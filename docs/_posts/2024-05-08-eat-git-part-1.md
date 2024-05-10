---
layout: post
title: "Eat Git - Part 1"
categories: git
author:
- Mohammad Rahimi
---

## Preface

This tutorial on Git has five parts, and this is the first. The following shows discussed material in each part:

1. Getting to Know Git
    - Creating Repositories
    - Working Tree and Index
    - Commits History
    - Search and Compare Tools
2. Behind the Scene
    - Git Internals
3. Local Workflows
    - Debugging with Git
    - Patch Files
    - Cherry Picking
    - Interactive Rebase
4. Remote Workflows
    - Remotes
    - Branch Management
    - Merging and Merge Conflicts
    - Rebasing and Rebase Conflicts
5. Do and Don't
    - Commit on the main Branch
    - Merge in Rebase Workflow
    - Feature Branch That Tracks the main Branch
    - Checkouts While There is Work in Index
    - Commit Before Configuring User Name and Email

In the first part, we get to know Git and become comfortable using it. If you are curious and want to know why everyone is talking about Git, this part should be enough.

In the second part, I will explain how Git creates and stores the data structures that enable it to do what it does. This part, in my opinion, is necessary to become confident at using other tools introduced later.

In part 3, I will wrap up most of the things you need to know to use Git effectively. Although Branches are useful even in a solo developer setting, their introduction deferred to the next part.

In part 4, I will introduce Branches. They are essential if you are part of a team working on a shared code-base. If you have been working with Git but want to understand the utilization of what you know in a collaborative environment, this is the part you need to read.

The last part is where I share my experience with you. In my previous two positions, I introduced my colleagues to Git, and then we started using it in our day-to-day development. This part sums up the mistakes that me and my colleagues used to make and show how to fix things when something goes wrong.

Why another Git tutorial? I wanted to make a guide to Git that I wish I had when I first started learning. If you have time to read the [Pro Git](https://git-scm.com/book/en/v2) Book, please do so—it's a great resource that inspired much of this tutorial. However, if you're looking to learn Git thoroughly in about a week, spending 1 to 2 hours daily, and want to be able to assist your colleagues, I hope this tutorial can help you achieve that goal.

For this tutorial, all you need is to install Git, which I'll explain shortly. Installing Git is straightforward. However, as a recommendation, I suggest using Ubuntu in a Virtual Machine to keep things organized. While not essential, it's a good practice I follow for most of my development work. Using VMs helps keep my main computer clean and fast, and if anything goes wrong, I can quickly revert to a clean state by cloning the base image.

## Installation

Install Git on Ubuntu command-line:

```bash
sudo apt-get install git
```
