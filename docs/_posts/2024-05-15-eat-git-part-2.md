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
*Plumbing* commands. Git commands are divided into two sets: *Porcelain* and
*Plumbing*. *Porcelain* commands are the ones you use for everyday tasks, while
*Plumbing* commands are the lower-level commands that Git combines to perform
higher-level tasks.

## Objects

Continuing with the example project from
[Part 1]({{ site.baseurl }}{% link _posts/2024-05-08-eat-git-part-1.md %}), your
repository should look like this:

```bash
# to view the history
git log
# commit 181590f967efc5d6618616533a170a96e1eb3638 (HEAD -> main)
# Author: Mohammad Rahimi <rahimi.mhmmd@outlook.com>
# Date:   Fri May 17 07:03:15 2024 +0800
#
#     Add Berman and Bash


# to display the most recent commit:
git show
# commit 181590f967efc5d6618616533a170a96e1eb3638 (HEAD -> main)
# Author: Mohammad Rahimi <rahimi.mhmmd@outlook.com>
# Date:   Fri May 17 07:03:15 2024 +0800
#
#     Add Berman and Bash
#
# diff --git a/employees.md b/employees.md
# new file mode 100644
# index 0000000..745b5ef
# --- /dev/null
# +++ b/employees.md
# @@ -0,0 +1,2 @@
# +John Berman
# +Dana Bash
```

Note that your commit hashes will be different from mine. Let's see if we can
find our commit among the Git object files. To list the contents of the
*.git/objects/* directory:

```bash
tree .git/objects/
# .git/objects/
# ├── 18
# │   └── 1590f967efc5d6618616533a170a96e1eb3638
# ├── 74
# │   └── 5b5efa7d5acb852d11cf14bc3a07c494bee605
# ├── 98
# │   └── 6e14542dcfbc7a3d13f16f8a8aeabbcda326de
# ├── d8
# │   └── 573240a1979328db1d568897ac1995754cddba
# ├── info
# └── pack
#
# 6 directories, 4 files
```

You might need to install `tree`:

```bash
sudo apt-get install tree
```

Git distributes objects into subdirectories named after the first two characters
of their hash. This helps speed up finding objects. Our commit is in the
*.git/objects/18* directory. Git provides a *Plumbing* command for inspecting
objects. To inspect type of an object:

```bash
git cat-file -t 1815
```

You don't need to use the entire hash, just enough characters for Git to
uniquely identify the object. Note that auto-complete does not work for
*Plumbing* commands because they are not intended for end users. The command
above will confirm that the object is indeed a commit. To pretty-print the
content of an object:

```bash
git cat-file -p 1815
# tree 986e14542dcfbc7a3d13f16f8a8aeabbcda326de
# author Mohammad Rahimi <rahimi.mhmmd@outlook.com> 1715900595 +0800
# committer Mohammad Rahimi <rahimi.mhmmd@outlook.com> 1715900595 +0800
#
# Add Berman and Bash
```

There are two pieces of information here that I want to highlight. First, you
can see both the author and the committer, each with an associated timestamp.
This is because you can make changes and send them to someone else to commit,
and they might commit those changes a few days or months later. The second piece
of information is `tree 986e14542dcfbc7a3d13f16f8a8aeabbcda326de`. The tree
represents the structure of your directory when you authored the changes. This
tree is also an object in Git that we can inspect.

```bash
git cat-file -t 986e
# tree
git cat-file -p 986e
# 100644 blob 745b5efa7d5acb852d11cf14bc3a07c494bee605    employees.md
```

This means that after the changes, our directory structure contained only one
file named *employees.md*. Let's inspect further:

```bash
git cat-file -t 745b
# blob
git cat-file -p 745b
# John Berman
# Dana Bash
```

Now you can see the file content. A Git blob (binary large object) is the object
type used to store the contents of each file in a repository.

Initial commits are different because they don't have any parent commits before
them. Let's create another commit and see how Git stores our changes.

```bash
# to make the change
echo 'Wolf Blitzer' >> employees.md
git status
# On branch main
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git restore <file>..." to discard changes in working directory)
#         modified:   employees.md
#
# no changes added to commit (use "git add" and/or "git commit -a")


# to create the commit
git add employees.md
git commit -m 'Add Blitzer'
# [main fed16ca] Add Blitzer
#  1 file changed, 1 insertion(+)


# to see the last changes
git show
# commit fed16ca7c469dcfcc1daeffa8b7255f87d61c026 (HEAD -> main)
# Author: Mohammad Rahimi <rahimi.mhmmd@outlook.com>
# Date:   Fri May 17 08:20:22 2024 +0800
#
#     Add Blitzer
#
# diff --git a/employees.md b/employees.md
# index 745b5ef..9ead660 100644
# --- a/employees.md
# +++ b/employees.md
# @@ -1,2 +1,3 @@
#  John Berman
#  Dana Bash
# +Wolf Blitzer


# to show the history
$ git log --oneline
# fed16ca (HEAD -> main) Add Blitzer
# 181590f Add Berman and Bash
```

The object we're interested in is our last commit, identified by *fed16ca*.

```bash
$ git cat-file -p fed1
# tree 081fb438ab187f1fd3f82d196bce3bc42c2b4d52
# parent 181590f967efc5d6618616533a170a96e1eb3638
# author Mohammad Rahimi <rahimi.mhmmd@outlook.com> 1715905222 +0800
# committer Mohammad Rahimi <rahimi.mhmmd@outlook.com> 1715905222 +0800
#
# Add Blitzer
```

You can see that the *parent* has our previous commit hash. If you change
something in the first commit, the file hash changes. Consequently, the commit
hash changes as well, and the parent commit in the following commits also
changes. Thus, every commit hash in your repository will change. If your
colleagues or other open-source contributors based their work on your commit,
meaning they have a *parent* commit somewhere that refers to a commit you made,
and you change something in the history, all your commit hashes will change.
Those who have your commit hash in their repository will be left disconnected
from your changes, and you will no longer share the same past. Furthermore,
there will be a new group of people who will base their work on your new hashes.
So, you cannot go back either. This can cause a divergence that's very difficult
, if not impossible, to fix. That's why it's advised not to change what you have
published. Git heavily relies on commit hashes to track changes in a repository
and determine what actions need to be taken.

## Branches

Here, I intend to show you how branches are represented internally in Git. To
understand that, I'll provide a very brief introduction. We will become
accustomed to using branches in Part 4.

Imagine your work is in a stable state and customers are happy with it. However,
you have some ideas about improving certain parts of your work. You don't want
to compromise the stable code. In Git, you can create a branch from a commit and
continue committing your changes in that branch. This is called a feature
branch.

We already have a branch called *main*. Let's create another branch and add a
commit to it. You don't need to understand these commands at the moment. To
create a branch called *feat* from *HEAD*:

```bash
git checkout -b feat HEAD
git log --oneline --all --graph
```

*HEAD* is a pointer to a commit or a branch. We will see that shortly. To create
a commit on the current branch:

```bash
echo 'Jamie Gangel' >> employees.md
git add employees.md

git commit -m 'Add Gangel'
git log --oneline --all --graph
# * 8975de0 (HEAD -> feat) Add Gangel
# * fed16ca (main) Add Blitzer
# * 181590f Add Berman and Bash
```

Now we have two branches that differ by one commit. First things first, what is
*HEAD*?

```bash
cat .git/HEAD
# ref: refs/heads/feat
```

The content of the file *.git/HEAD* is *ref: refs/heads/feat*. This means that
you are currently on the last commit of a branch called *feat*. If you make any
changes, stage them, and create a commit, the *parent* of that commit will be
what *refs/heads/feat* points to. Let's see where *refs/heads/feat* is pointing
to:

```bash
cat .git/refs/heads/feat
# 8975de012dc27004ec8bf33667a11c03ac251611
```

It should be familiar to you. It is the last commit we made after switching to
the *feat* branch. Let's see where *.git/refs/heads/main* is pointing to:

```bash
cat .git/refs/heads/main
# fed16ca7c469dcfcc1daeffa8b7255f87d61c026
```

It points to the second commit that we made in this repository. At that time, we
were on the *main* branch.

As you can see, branches are nothing but pointers to commits. Since each commit
also points to its parent(s), you can have multiple lines of work in your
repository simultaneously and switch between them. To switch back to the main
branch:

```bash
git checkout main
```

We can also check out a commit without creating a new branch. This state is
called a *detached HEAD* because *HEAD* is not pointing to a branch.
Occasionally, you may need to check out a specific commit to investigate if it
contains an issue. To check out a commit, use the following command:

```bash
git checkout 8975de0
# Note: switching to '8975de0'.
#
# You are in 'detached HEAD' state. You can look around, make experimental
# changes and commit them, and you can discard any commits you make in this
# state without impacting any branches by switching back to a branch.
#
# If you want to create a new branch to retain commits you create, you may
# do so (now or later) by using -c with the switch command. Example:
#
#   git switch -c <new-branch-name>
#
# Or undo this operation with:
#
#   git switch -
#
# Turn off this advice by setting config variable advice.detachedHead to false
#
# HEAD is now at 8975de0 Add Gangel

cat .git/HEAD
# 8975de012dc27004ec8bf33667a11c03ac251611
```

The purpose of this section was to help you understand that branches are nothing
but pointers to commits. This fact becomes important when, in Part 5, I explain
how to fix common issues that arise when working with Git.

## A Hand-made Commit

To create a commit using *Plumbing* commands, ensure that nothing is staged.
First, let's add a change:

```bash
echo 'Brianna Keilar' >> employees.md
```

Then, we add this change to the *index*. The *index* is a binary file located at
*.git/index*.

```bash
git update-index employees.md

git status
git diff --staged
```

We need our directory structure to be recorded in the commit. Let's create a
tree object from the *index*:

```bash
cat employees.md
# John Berman
# Dana Bash
# Wolf Blitzer
# Jamie Gangel
# Brianna Keilar

git hash-object employees.md
# c91fe675c5c094ff6437a8dc9f5e41bf362e93d4

git write-tree
# 63bce7388bfb114345c99b0af9cd44b7b7b3b148

git cat-file -p 63bc
# c91fe675c5c094ff6437a8dc9f5e41bf362e93d4
```

First, I printed the content of *employees.md*. Then, I calculated its SHA256
hash. This hash, with the same content, should always be the same. If you use
the `-w` flag with `git hash-object`, Git will actually write the object into
its internal files. Then, I created a tree object. This tree object will be
associated with our commit. Your tree object should also have the same hash as
mine.

Now is the time to create the actual commit:

```bash
git commit-tree 63bce7 -p 181590f -m 'Add Keilar'
# 7cbf3e032cfdce489a4fcfd8d2e486990b6460f3

git log --oneline --all --graph
```

We will not see our commit in the `git log` output because our commit has no
reference pointing to it. In other words, it is not on any branch. In the
`git commit-tree` command, I used my first commit as the *parent*, so it cannot
be associated with any existing branches, as its history is divergent. To create
a new branch from that commit:

```bash
git update-ref refs/heads/new-feat 7cbf3e

git log --oneline --all --graph
# * 7cbf3e0 (new-feat) Add Keilar
# | * 8975de0 (HEAD -> feat) Add Gangel
# | * fed16ca (main) Add Blitzer
# |/  
# * 181590f Add Berman and Bash

git show new-feat
# commit 7cbf3e032cfdce489a4fcfd8d2e486990b6460f3 (new-feat)
# Author: Mohammad Rahimi <rahimi.mhmmd@outlook.com>
# Date:   Sat May 18 12:05:29 2024 +0800
#
#     Add Keilar
#
# diff --git a/employees.md b/employees.md
# index 745b5ef..c91fe67 100644
# --- a/employees.md
# +++ b/employees.md
# @@ -1,2 +1,5 @@
#  John Berman
#  Dana Bash
# +Wolf Blitzer
# +Jamie Gangel
# +Brianna Keilar
```

You can also open a text editor and put the commit hash in *refs/heads/new-feat*
. However, this is not recommended because we should not alter any file in the
*.git* directory without Git's knowledge.

## Summary

In this part, we explored Git's *Plumbing* commands, inspected the *.git*
directory, created blob, tree, and commit objects, and gained an understanding
of how branches are represented internally in Git. At this point, we should have
the confidence required to tackle more advanced topics in Git.

In Part 3, I will introduce you to more useful tools that Git has to offer.
After Part 3, we will be able to continue our journey in Git at a higher
altitude and learn more about concepts that involve Git, rather than being stuck
with low-level everyday commands.


[pro-git-book]: https://git-scm.com/book/en/v2
