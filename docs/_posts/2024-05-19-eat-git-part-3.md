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
conducted experiments in previous part, I will remove the directory and
repopulate it with two branches.

If you seek examples with more substance, feel free to subscribe to one of the
channels at [vger.kernel.org](vger.kernel.org) and apply patches that
contributors post. In addition to the patch files, you also need to clone the
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
you can create a patch file by storing the output of `git diff` in a file:

```bash
# equivalent to 0002-Add-Cornish-and-Asher.patch
git diff 6bc2c0a 2dcbac8 > hand-made-patch
```

The `format-patch` command, in addition to the diff, includes the corresponding
commit hash, author's name, and timestamp of the modifications. You can email
these patch files to repository maintainers. If they're accepted, the
maintainers will incorporate the changes into the *main* branch by committing
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
#         hand-made-patch
#
# no changes added to commit (use "git add" and/or "git commit -a")
```

As you can see, the changes are not committed yet. You can add them to the
*Index* and then commit them, which will have your name as both the author and
committer. Alternatively, you can use the following command to commit the patch
file directly onto the *main* branch with the original author's name:

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

To apply the second commit on the *woke* branch, using the *hand-made-patch*
file:

```bash
git checkout main

git apply hand-made-patch

git add employees.md
git commit -m 'Add Cornish and Asher'
```

If you create a patch file using either of the two mentioned methods and send it
to someone, they might not be able to apply the patch cleanly if their file has
changed in the meantime. This is because the lines referenced in the patch
file's diff may have been deleted or moved. In such cases, Git will output an
error message and won't apply any of the patches.

```bash
# error: patch failed: <file>:<line>
# error: <file>: patch does not apply
```

To solve this problem, you need to update your *main* branch. Switch back to
your feature branch and either merge it with or rebase it onto the *main* branch
. Resolve any conflicts that arise and generate the patch files again. Merging
and rebasing will be explained in Part 4.

Patch files can be very handy and come to your rescue in times of need. They
also play a central role in contributing to major open-source projects. I
recommend getting comfortable using them. This concludes my introduction to
patch files.

## Cherry Picking

If you've done some work in a feature branch that turns out to be unnecessary,
you can still cherry-pick useful commits from that branch. Another use case for
cherry-picking is when you fix a bug in one branch and need the same fix in
another. Or perhaps you committed your work to the wrong branch by mistake. In
all these scenarios, cherry-picking can help.

To avoid conflicts in our example project, let's explore the last mentioned use
case. We'll create a commit but mistakenly place it on our *woke* branch.
Conflicts and resolving them will be covered in Part 4.

```bash
git checkout woke

echo 'Michael Smerconish' >> employees.md
git add employees.md
git commit -m 'Add Smerconish'

git log --oneline --all --graph
# * f892560 (HEAD -> woke) Add Smerconish
# * 2dcbac8 Add Cornish and Asher
# * 6bc2c0a Add Phillip
# | * cf560f0 (main) Add Cornish and Asher
# | * 548e193 Add Phillip
# |/
# * a48e1bb Add Cooper, Tapper and Wallace
```

Now, let's correct our mistake by cherry-picking the last commit to our main
branch:

```bash
git checkout main
git cherry-pick f892560

git log --oneline --all --graph
# * c588da3 (HEAD -> main) Add Smerconish
# * cf560f0 Add Cornish and Asher
# * 548e193 Add Phillip
# | * f892560 (woke) Add Smerconish
# | * 2dcbac8 Add Cornish and Asher
# | * 6bc2c0a Add Phillip
# |/
# * a48e1bb Add Cooper, Tapper and Wallace
```

You can see the same commit from the *woke* branch now on the *main* branch.

## Interactive Rebase

Rebasing is a procedure that affects the history of a branch. As mentioned
several times before, changing history can lead to many complications if others
share that history with you. That being said, it is the most effective tool for
organizing your work and creating a clean history.

In this section, I will show you interactive rebasing. There is also a
contribution workflow that involves rebasing, which is different from this one.
Both use the `git rebase` command, but interactive rebasing also uses the `-i`
flag. I will explain merging and rebasing in the next part.

With interactive rebasing, you can specify a range of commits and an action at
each commit. The command goes through each one and applies the action. I
typically use interactive rebasing to combine my commits on feature branch into
one before publishing my work. This way, as time passes, the project will
maintain a clean history. You won't see commit messages like
*"Fix minor mistake"* or *"Rename variable"* after a meaningful commit like
*"Fix bug report #6623"*.

In our example project, before publishing our *main* branch, let's combine the
first two commits together (to minimize the chance of getting canceled ;)).

```bash
git checkout main
git rebase -i --root
```

The `git rebase -i` command expects a commit and will list all the commits from
HEAD up to but not including that commit. Because of this behavior, you cannot
edit the initial commit. That's why I used the `--root` flag. The list is
ordered from old at the top to new at the bottom.

Edit the file and change the action at the second commit from `pick` to `squash`
. Save and close the editor. The opened file also includes instructions on
commands that you can use at each step.

```bash
pick a48e1bb Add Cooper, Tapper and Wallace
squash 548e193 Add Phillip
pick cf560f0 Add Cornish and Asher
pick c588da3 Add Smerconish
```

The interactive rebase will show you one more editor for changing the commit
message after combining the first two commits. Provide a meaningful message.

Our new history should look like the following:

```bash
git log --oneline
# 08206a8 (HEAD -> main) Add Smerconish
# 301703c Add Cornish and Asher
# cd0bbc3 Add Cooper, Tapper, Wallace and Phillip
```

Although it can be scary, I recommend using interactive rebase in your daily
tasks. Keeping the history clean should be something that all team members
strive for.

## Stashes

Imagine you are working on a feature when your boss informs you that a customer
reported a bug after your last commit was published. You need to switch back to
the main branch immediately and start investigating if the bug is caused by your
changes. However, you have already made significant progress in your feature
branch, but it is not finished or in a state that deserves its own commit. If
you don’t want to lose your changes and also do not want to commit them, you can
stash the changes. Each stash contains uncommitted work, whether it is in the
*Working Tree* or the *Index*. Let's go through a slightly more complicated
scenario.

You are on a feature branch, refactoring some code. Your boss requests that you
postpone the refactoring and instead write some additional tests for your recent
module. To switch to another branch without losing the work you've already done:

```bash
git checkout -b refactor main

# do some work, stage and do more work

# boss asks for tests
git stash push --all -m 'Refactor code'
git checkout -b add-tests main

# write some tests, stage and write more
```

While you are working on the tests, a customer reports a critical bug in the
latest release, and you are assigned to fix it. To save your current work before
committing:

```bash
git stash push --all -m 'Add tests for module'
git checkout -b fix-bug main

# fix the bug, then
git commit -m 'Fix bug'
```

Now go back to writing tests:

```bash
git checkout add-tests
git stash pop --index

# finish rest of the tests, then
git commit -m 'Add tests for module'
```

After writing the tests, continue refactoring the code:

```bash
git checkout refactor
git stash pop --index
```

It is possible to stash your work on one branch and apply it to another branch.
Additionally, you can list all your stashes and apply any specific stash:

```bash
git stash list
# stash@{0}: On add-tests: Add tests
# stash@{1}: On refactor: Refactor code

git stash apply --index stash@{1}
```

To remove all the stashes use `git stash clear`.

If any of the modified or staged files have a different revision on the other
branch, the `git checkout` command will result in an error stating that your
modified files need to be committed or stashed before switching. However, if
checking out another branch won't overwrite any of those files, switching
between branches will maintain the state of both the *Index* and the
*Working Tree*.

## Miscellaneous

In this section, I'll introduce some Git tools that you might not use frequently
but are nonetheless useful.

### Bisect

The git bisect command is used to find the exact commit that introduced a bad
behavior. You begin by providing this command with a commit that has the issue
and a commit that you know has been good. To start the binary search in the
[it-tools][it-tools-repo] repository that we acquired in
[Part 1]({{ site.baseurl }}{% link _posts/2024-05-08-eat-git-part-1.md %}):

```bash
# commit with the tag v2024.5.10-33e5294 is bad
# commit with the tag v2023.12.21-5ed3693 is good
git bisect start v2024.5.10-33e5294 v2023.12.21-5ed3693
# Bisecting: 12 revisions left to test after this (roughly 4 steps)
# [a07806cd15fdbd24c88afaf618a2d0c16d66bb3f] refactor(home): lightened tool cards (#882)
```

You can tag a commit with an explanation, which is usually used to mark a
release version. The command can be something like:

```bash
git tag -a v2.0 abc123 -m "Version 2.0 LTS"
```

If you run `git log --oneline --all`, you will see that the `git bisect` command
has positioned *HEAD* at a commit in the middle of both tags. You can examine
the code and determine whether the issue exists or not. If it is a good commit,
continue with:

```bash
git bisect good
# Bisecting: 6 revisions left to test after this (roughly 3 steps)
# [221ddfa75c5731d7a5dc1f0b03663ba4fd9e7965] fix(language): English language cleanup (#1036)
```

And if it is a bad commit:

```bash
git bisect bad
# Bisecting: 2 revisions left to test after this (roughly 2 steps)
# [23f82d956a8af21e176f7268c9414244168bd4eb] fix(bcrypt tool): allow salt rounds up to 100 (#987)
```

You continue investigating commits until Git tells you that it has found the
commit that introduced the issue. You can stop the search midway with
`git bisect reset`.

If you have automated tests and some of them started to fail after an unknown
change, you can use `git bisect run my_script arguments` to automatically find
the problematic commit. The `my_script` or any other executable file should exit
with code 0 if the current source code is good and exit with a code between 1
and 127 (inclusive), except 125, if the current source code is bad.

### Blame

The `git show` command is used to display detailed information about a specific
commit, including the files it altered. Alternatively, you can use the
`git blame` command to see which commit introduced each line of a file. If you
find a bug and want to blame someone for it, you can use `git blame` to trace
each line back to its corresponding commit. To see which commit introduced each
line of code:

```bash
git blame employees.md
# ^cd0bbc3 (Mohammad Rahimi 2024-05-20 09:00:20 +0800 1) Anderson Cooper
# ^cd0bbc3 (Mohammad Rahimi 2024-05-20 09:00:20 +0800 2) Jake Tapper
# ^cd0bbc3 (Mohammad Rahimi 2024-05-20 09:00:20 +0800 3) Chris Wallace
# ^cd0bbc3 (Mohammad Rahimi 2024-05-20 09:00:20 +0800 4) Abby Phillip
# 301703c3 (Mohammad Rahimi 2024-05-20 19:05:45 +0800 5) Audie Cornish
# 301703c3 (Mohammad Rahimi 2024-05-20 19:05:45 +0800 6) Zain Asher
# 08206a8e (Mohammad Rahimi 2024-05-21 07:05:39 +0800 7) Michael Smerconish
```

The `^` symbol next to a commit hash indicates that the line of code was created
by the very first commit that introduced the file into the repository.

### Reflog

Whenever there is a change to *HEAD* or any other references in Git, the
reference log is updated. For example, when you switch branches, a new entry is
added to this log. This entry indicates the commit hash that *HEAD* now points
to:

```bash
git checkout 301703c
git reflog
# 301703c (HEAD) HEAD@{0}: checkout: moving from refactor to 3017
# 08206a8 (refactor, main, add-tests) HEAD@{1}: checkout: moving from add-tests to refactor
```

When attempting to untangle branches in your Git history, mistakes can lead to
losing commits. Without the hash of the lost commit, recovering it becomes
impossible. However, `git reflog` can help retrieve the lost hash. As long as
you have the hash of the last commit on a branch, you can do anything, except
running the garbage collector in Git. I won't cover the garbage collection
command in this tutorial.

To create a new branch from where *HEAD* used to be 3 modifications before:

```bash
git reflog -3
git checkout -b recovered HEAD@{2}
```

As a side note, there are two other syntaxes involving *HEAD* in Git. One of
them is `HEAD~`, which refers to the parent commit of *HEAD*, while `HEAD~2`
refers to the grandparent commit of *HEAD*. The other syntax, `HEAD^`, is less
commonly used and is primarily used to refer to the first parent of a merge
commit. We will explore merging in more detail in Part 4.

### Archive

To create an archive from your Git repository at a specific commit:

```bash
git archive v2024.5.13-a0bc346 --output v2024.5.13-a0bc346.zip
```

In addition to commit hashes, you can use tags, branch names, or *HEAD* as
references when creating an archive. Essentially, anything that points to a
specific commit can be used.

## Summary

In this part, we explored some useful tools offered by Git that can come in
handy in your daily use. We learned how to create patch files, cherry-pick
commits, and clean up our history with interactive rebase. Additionally, we used
stashes to temporarily store work that isn't ready to be committed yet, allowing
us to start new tasks. We also introduced some other less common tools.

In Part 4, we'll delve into what Git offers in a collaborative environment.
We'll take on the roles of different team members and explore how they can work
together to maintain a well-structured, functional codebase.


[main-git-repo]: https://git.kernel.org/pub/scm/git/git.git/
[main-linux-repo]: https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/
[it-tools-repo]: https://github.com/CorentinTh/it-tools
