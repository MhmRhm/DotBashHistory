---
layout: post
title: "EatGit - Part 5"
excerpt: Part 5 of a quick Git tutorial to teach you everything you need to know.
seo_description: Part 5 of the Git tutorial - Learn about common mistakes and how you can recover from them.
categories: git
author:
- Mohammad Rahimi
---

* Table of Contents
{:toc}

## Preface

In [Part 1]({{ site.baseurl }}{% link _posts/2024-05-08-eat-git-part-1.md %}),
where I introduced the most common Git commands, and in
[Part 2]({{ site.baseurl }}{% link _posts/2024-05-15-eat-git-part-2.md %}),
where we delved into Git internals, and
[Part 3]({{ site.baseurl }}{% link _posts/2024-05-19-eat-git-part-3.md %}),
which taught us about patch files and other useful tools, or
[Part 4]({{ site.baseurl }}{% link _posts/2024-05-23-eat-git-part-4.md %}), where
we completed our understanding of Git by studying remotes, branches, and how to
merge or rebase them, I've been warning you about common mistakes and how to
avoid them.

When we discussed
[Commits]({{ site.baseurl }}{% link _posts/2024-05-08-eat-git-part-1.md %}#commits)
, you learned how to address the issue of accidentally creating a commit without
setting up a username and email or committing with the wrong information.

After showing you how Git stores its data in the form of
[Objects]({{ site.baseurl }}{% link _posts/2024-05-15-eat-git-part-2.md %}#objects)
, we saw why it's a bad idea to change history on a branch shared with others.

We explored how to share our work independently of our branches by using
[Patch Files]({{ site.baseurl }}{% link _posts/2024-05-19-eat-git-part-3.md %}#patch-files)
. We also learned how to fix issues where our patch files don't apply due to
mismatches between our work and others.

While learning about
[Force Push]({{ site.baseurl }}{% link _posts/2024-05-23-eat-git-part-4.md %}#force-push)
, we discussed why having multiple people pushing to the same branch is a bad
practice. We also did some
[Branch Management]({{ site.baseurl }}{% link _posts/2024-05-23-eat-git-part-4.md %}#branch-management)
, removing incorrect tracking branches and setting appropriate ones. During our
first [Rebasing]({{ site.baseurl }}{% link _posts/2024-05-23-eat-git-part-4.md %}#rebasing)
, I explained how using merge to update feature branches in a rebase workflow
can lead to strange behaviors.

In this section, we will learn how to recover from committing to the wrong
branch and how to handle merges in a rebase workflow. Both scenarios require
using `git reset`.

## Undo Commits

## Commit to Wrong Branch

## Edits on Remote

## Merge in Rebase

## Erase from History

## Summary
