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

Imagine you've done some work and want someone else to review it. If they have
access to your repository, they can use `git diff` and `git log` to see what
you've done. However, in the open-source community, this isn't always the case.
Patch files offer a way to share your work without relying on branches, making
them more flexible. You can store patch files on a USB stick or send them via
email. Creating patch files is quite easy. 

To demonstrate, let's continue with our CNN anchors example. Since you may have
conducted experiments in previous sections, I will remove the directory and
repopulate it with two branches.

If you seek examples with more substance, feel free to subscribe to one of the
channels at [vger.kernel.org](vger.kernel.org) and apply patches that contributors post to those
channels. In addition to the patch files, you also need to clone the
[Linux kernel][main-linux-repo] or [Git][main-git-repo] repositories.

```bash
rm -rf cnn/
mkdir cnn
cd cnn/

git init

git config --local user.name 'Mohammad Rahimi'
git config --local user.email 'rahimi.mhmmd@outlook.com'

echo 'Anderson Cooper' >> employees.md
echo 'Jake Tapper' >> employees.md
echo 'Chris Wallace' >> employees.md

git add employees.md
git commit -m 'Add Cooper, Tapper and Wallace'
```

We create another branch and add some other employees. If you don't get the
meaning behind these changes, Google their names ;)

```bash
git checkout -b woke main

echo 'Abby Phillip' >> employees.md
git add employees.md
git commit -m 'Add Phillip'

echo 'Audie Cornish' >> employees.md
echo 'Zain Asher' >> employees.md
git add employees.md
git commit -m 'Add Cornish and Asher'

git log --oneline --all --graph
# * 2dcbac8 (HEAD -> woke) Add Cornish and Asher
# * 6bc2c0a Add Phillip
# * a48e1bb (main) Add Cooper, Tapper and Wallace
```

Now, to create patch files that include the changes made on our *woke* branch
but are not present on the *main* branch:

```bash
git checkout woke
git format-patch main
# 0001-Add-Phillip.patch
# 0002-Add-Cornish-and-Asher.patch

cat 0001-Add-Phillip.patch
# From 6bc2c0aae0c3449b128a4f8434602dfe6dab9e1a Mon Sep 17 00:00:00 2001
# From: Mohammad Rahimi <rahimi.mhmmd@outlook.com>
# Date: Mon, 20 May 2024 09:10:33 +0800
# Subject: [PATCH 1/2] Add Phillip
#
# ---
#  employees.md | 1 +
#  1 file changed, 1 insertion(+)
#
# diff --git a/employees.md b/employees.md
# index 0d59cdb..6454a94 100644
# --- a/employees.md
# +++ b/employees.md
# @@ -1,3 +1,4 @@
#  Anderson Cooper
#  Jake Tapper
#  Chris Wallace
# +Abby Phillip
# -- 
# 2.45.GIT
#
```

Patch files are essentially text files. If you look inside one of them, you will
see it is strikingly similar to the output of the `git diff` command. In fact,
you can create a patch file by storing the output of git diff in a file:

```bash
git diff 6bc2c0a 2dcbac8 > hand-made-patch.txt
```

The `format-patch` command, in addition to the diff, includes the corresponding
commit hash, author's name, and timestamp of the modifications. You can email
these patch files to repository maintainers. If they're accepted, the
maintainers will incorporate the changes into the main repository by committing
them.

To apply a patch file onto a branch:

```bash
git checkout main

git apply 0001-Add-Phillip.patch

git status 
# On branch main
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git restore <file>..." to discard changes in working directory)
#         modified:   employees.md
#
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#         0001-Add-Phillip.patch
#         0002-Add-Cornish-and-Asher.patch
#         hand-made-patch.txt
#
# no changes added to commit (use "git add" and/or "git commit -a")
```

As you can see, the changes are not committed yet. You can add them to the
*Index* and then commit them, which will have your name as both the author and
committer. Alternatively, you can use the following command to commit the patch
file directly onto the main branch with the original author's name:

```bash
git restore employees.md

git config --local user.name "Maintainer's name"

git am 0001-Add-Phillip.patch

git config --local user.name 'Mohammad Rahimi'
```

To see the above change:

```bash
git log --oneline --all --graph
# * 548e193 (HEAD -> main) Add Phillip
# | * 2dcbac8 (woke) Add Cornish and Asher
# | * 6bc2c0a Add Phillip
# |/  
# * a48e1bb Add Cooper, Tapper and Wallace

git cat-file -p 548e193
# tree 03e53233c394eb4559d98e1a10a14eac4044f726
# parent a48e1bb85df53651fbc173d2aba47ebe3db4a38b
# author Mohammad Rahimi <rahimi.mhmmd@outlook.com> 1716167433 +0800
# committer Maintainer's name <rahimi.mhmmd@outlook.com> 1716169076 +0800
#
# Add Phillip
```


[main-git-repo]: https://git.kernel.org/pub/scm/git/git.git/
[main-linux-repo]: https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/