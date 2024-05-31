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

In such situations, there's a version of the `git pull` command that can help a
lot. Let's recreate this situation.

First, let's create a remote repository:

```bash
cd ..
mkdir remote
cd remote

git init -b no-name
```

Add the remote to our original repository and push to it:

```bash
git remote add origin ../remote/.git/
git push origin main
```

Switch to the remote repository and create a commit on the *main* branch:

```bash
cd ../remote

git config --local user.name 'Mohammad Rahimi'
git config --local user.email 'rahimi.mhmmd@outlook.com'

git log --oneline --all --graph
# * 2482827 (main) 07:52:52
# * a80f490 07:28:19

git checkout main
msg=$(date +%T) && echo $msg >> t.md && git add t.md && git commit -m "$msg"
```

At this point, we have a remote repository with a commit that the local
repository doesn't have. Now, go back to the local repository and create a
commit there:

```bash
cd ../repo

git checkout main
msg=$(date +%T) && echo $msg >> t.md && git add t.md && git commit -m "$msg"
```

Fetch the remote status:

```bash
git fetch origin

git log --oneline --all --graph
# * 3c2a116 (HEAD -> main) 03:55:41
# | * 3cd7b4b (origin/main) 03:53:14
# |/  
# * 2482827 07:52:52
# | * c346fee (recover) 07:28:25
# | * e703ac0 07:28:22
# |/  
# * a80f490 07:28:19
```

Now that we have recreated the scenario, let's pull in the changes from the
remote repository:

```bash
git pull origin main
# From ../remote/
#  * branch            main       -> FETCH_HEAD
# hint: You have divergent branches and need to specify how to reconcile them.
# hint: You can do so by running one of the following commands sometime before
# hint: your next pull:
# hint: 
# hint:   git config pull.rebase false  # merge (the default strategy)
# hint:   git config pull.rebase true   # rebase
# hint:   git config pull.ff only       # fast-forward only
# hint: 
# hint: You can replace "git config" with "git config --global" to set a default
# hint: preference for all repositories. You can also pass --rebase, --no-rebase,
# hint: or --ff-only on the command line to override the configured default per
# hint: invocation.
# fatal: Need to specify how to reconcile divergent branches.
```

Git is complaining about divergent branches. The solution to this situation is
to:

```bash
git pull --rebase origin main
```

After resolving the conflict to the following:

```bash
cat t.md
# 07:28:19
# 07:52:52
# 03:53:14
# 03:55:41
```

We will have:

```bash
git log --oneline --all --graph
# * 3a34917 (HEAD -> main) 03:55:41
# * 3cd7b4b (origin/main) 03:53:14
# * 2482827 07:52:52
# | * c346fee (recover) 07:28:25
# | * e703ac0 07:28:22
# |/  
# * a80f490 07:28:19
```

We saw how `git pull --rebase origin main` helped us to get the commits made on
the remote repository.

## Merge in Rebase

Mistakes can always happen. I've often had to fix a colleague's Git history
because they accidentally updated their feature branch by merging instead of
rebasing in a rebase workflow. The fix is quite easy. Just note down the head
hash and create a patch file before starting, and you will be safe. Here’s the
procedure:

1. Note down the head hash and create a patch file:

```bash
git checkout feat
git log -1 --format="%H"
git format-patch HEAD~
```

This shows the current head hash and creates a patch file from the last commit.

2. Hard reset to `HEAD~` on the feature branch to get rid of the merge commit:

```bash
git reset --hard HEAD~
```

3. Optionally apply the patch created from `git format-patch HEAD~`:

```bash
git am 0001-*.patch
```

4. Perform the rebase by running:

```bash
git rebase main
```

If you skip step 3, you may have to resolve conflicts again.

5. Force push to the remote repository:

```bash
git push --force origin feat
```

By following these steps, you can effectively clean up the history and ensure
the feature branch is correctly rebased onto the main branch. This process also
ensures that you can recover from any mistakes made during the fix.

## Erase from History

Another mistake that can happen is accidentally committing a large file or a
file that contains sensitive information into Git. Even if you remove the file
and commit again, others will still have to download it with each clone, and it
will be accessible through the project's history. To truly remove a file from
Git, you need to go over each commit and edit it so that the file is removed.

In the following example, I demonstrate replacing the text `07:28:19` with
`redacted` in each commit:

```bash
# perform the command in a testing branch
git checkout -b test main
git checkout main

git filter-branch --tree-filter '
if [ -f t.md ]; then
    sed -i 's/07:28:19/redacted/g' t.md
fi              
' test

git log --patch test
```

And after executing the steps outlined above, you will see that `07:28:19` has
been successfully replaced with `redacted` throughout the file in all commits.

```diff
commit bb83f638be36493f8a6c6dd8d8d143fbfc3f91ea (test)
Author: Mohammad Rahimi <rahimi.mhmmd@outlook.com>
Date:   Fri May 31 03:55:41 2024 +0800

    03:55:41

diff --git a/t.md b/t.md
index 8c6dc00..8441a33 100644
--- a/t.md
+++ b/t.md
@@ -1,3 +1,4 @@
 redacted
 07:52:52
 03:53:14
+03:55:41

commit 2be0694cf8c8f78ec3c7f752cacfaa9963fe2a34
Author: Mohammad Rahimi <rahimi.mhmmd@outlook.com>
Date:   Fri May 31 03:53:38 2024 +0800

    03:53:14

diff --git a/t.md b/t.md
index e100ac8..8c6dc00 100644
--- a/t.md
+++ b/t.md
@@ -1,2 +1,3 @@
 redacted
 07:52:52
+03:53:14

commit 5b7e5b494a6af0a44089bbeaca414bd05924fad8
Author: Mohammad Rahimi <rahimi.mhmmd@outlook.com>
Date:   Thu May 30 07:52:52 2024 +0800

    07:52:52

diff --git a/t.md b/t.md
index 58853f4..e100ac8 100644
--- a/t.md
+++ b/t.md
@@ -1 +1,2 @@
 redacted
+07:52:52

commit ff0d4203787c69d55c751fe32f917dd4c7c132d3
Author: Mohammad Rahimi <rahimi.mhmmd@outlook.com>
Date:   Thu May 30 07:28:19 2024 +0800

    07:28:19

diff --git a/t.md b/t.md
new file mode 100644
index 0000000..58853f4
--- /dev/null
+++ b/t.md
@@ -0,0 +1 @@
+redacted
```

## Summary

In this concluding section, I've discussed common mistakes I've observed with
Git and provided solutions to address each of them. By now, you should have all
the necessary tools to work with Git proficiently. Additionally, you should feel
confident in your understanding of Git, enabling you to resolve any issues your
colleagues may encounter.

If you found this information beneficial, please consider sharing it with
others. Your feedback is valuable, so if you have any suggestions for
improvement, please feel free to leave a comment. Thank you for your time and
attention.
