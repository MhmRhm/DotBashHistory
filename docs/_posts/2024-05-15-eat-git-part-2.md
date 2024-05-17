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
uniquely identify the object. Note that auto-complete does not work for *Plumbing*
commands because they are not intended for end users. The command above will
confirm that the object is indeed a commit. To pretty-print the content of an
object:

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


[pro-git-book]: https://git-scm.com/book/en/v2
