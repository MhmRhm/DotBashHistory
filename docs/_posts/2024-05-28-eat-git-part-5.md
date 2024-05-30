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

We already know that branches in Git are essentially pointers to specific
commits. Using the `git reset` command, we can move a branch pointer to a
different commit. Let's create a repository and add some commits to see how this
works:

```bash
mkdir repo
cd repo

git init
git config --local user.name 'Mohammad Rahimi'
git config --local user.email 'rahimi.mhmmd@outlook.com'

msg=$(date +%T) && echo $msg >> t.md && git add t.md && git commit -m "$msg"
sleep 3
msg=$(date +%T) && echo $msg >> t.md && git add t.md && git commit -m "$msg"
sleep 3
msg=$(date +%T) && echo $msg >> t.md && git add t.md && git commit -m "$msg"
```

There are three versions of `git reset`:

1. `git reset --soft HEAD~`: Moves the current branch pointer to the parent
commit but leaves the *Working Directory* and the *Index* unchanged.
2. `git reset --mixed HEAD~`: Moves the current branch pointer to the parent
commit and updates the *Index* to match the previous commit, but leaves the
*Working Directory* unchanged.
3. `git reset --hard HEAD~`: Moves the current branch pointer to the parent
commit, updates the *Index* to match the previous commit, and updates the
*Working Directory* to match the previous commit.

Each version progressively changes more of the repository state, from just the
branch pointer with `--soft` to the entire *Working Directory* with `--hard`.

You also have the option to reset individual files. When you provide a path to
`git reset`, the branch pointer remains unchanged. To reset a file to a specific
version, use the following command:

```bash
git reset abc123 -- path/to/file
```

This command will update the specified file to the version from the commit
`abc123`, and the changes will be reflected in the *Index*.

Git almost never permanently deletes commits. If you perform a hard reset, you
can still recover the commits. First note down your last commit hash and then
follow these steps:

```bash
git log --oneline --all --graph
# * c346fee (HEAD -> main) 07:28:25
# * e703ac0 07:28:22
# * a80f490 07:28:19

git reset --hard a80f490

git log --oneline --all --graph
# * a80f490 (HEAD -> main) 07:28:19

git checkout -b recover c346fee

git log --oneline --all --graph
# * c346fee (HEAD -> recover) 07:28:25
# * e703ac0 07:28:22
# * a80f490 (main) 07:28:19
```

This ability to reset commits, along with using patch files, provides a way to
untangle a messed-up history if ever needed.

## Commit to Wrong Branch

Let's create a commit:

```bash
msg=$(date +%T) && echo $msg >> t.md && git add t.md && git commit -m "$msg"
git log --oneline --all --graph
# * 93eefd1 (HEAD -> recover) 07:52:52
# * c346fee 07:28:25
# * e703ac0 07:28:22
# * a80f490 (main) 07:28:19
```

We notice that instead of committing to the `main` branch, we made that commit
on the `recover` branch. We can use `git reset` and `git cherry-pick` to make
things right:

Switch to the `main` branch and cherry-pick the commit:

```bash
git checkout main
git cherry-pick 93eefd1
# Auto-merging t.md
# CONFLICT (content): Merge conflict in t.md
# error: could not apply 93eefd1... 07:52:52
# ...
```

Resolve the conflict so that `t.md` contains the following:

```bash
cat t.md
# 07:28:19
# 07:52:52
git add t.md
git cherry-pick --continue
```

After accepting the commit message, switch back to the `recover` branch and
undo the change:

```bash
git checkout recover
git reset --hard HEAD~
```

Verify the history to ensure the commit is now on the `main` branch:

```bash
git log --oneline --all --graph
# * 2482827 (main) 07:52:52
# | * c346fee (HEAD -> recover) 07:28:25
# | * e703ac0 07:28:22
# |/  
# * a80f490 07:28:19
```

This is one way to recover from committing on the wrong branch. By using
`git reset` and `git cherry-pick`, you can move the commit to the correct branch
and clean up the original branch.

## Edits on Remote

We've discussed why it's a bad idea to have multiple people committing on the
same branch. But what if one person commits to the same branch from different
repositories? This happens to me often. For example, after pushing my work to
GitHub, I sometimes notice a typo, fix it directly on the GitHub website, and
commit the change there. Later, when I return to my local repository and make
another commit before pulling the typo fix from GitHub, Git will complain that
there are changes on the remote that I don't have locally, and I cannot push to
the remote before merging these changes.

In such situations, there's a version of the git pull command that can help a
lot. Let's recreate this situation.

## Merge in Rebase

## Erase from History

## Summary
