---
layout: post
title: "EatGit - Part 4"
excerpt: Part 4 of a quick Git tutorial to teach you everything you need to know.
seo_description: Part 4 of the Git tutorial - Learn about remotes, merging, rebasing, conflict resolution, and branch management.
categories: git
author:
- Mohammad Rahimi
---

* Table of Contents
{:toc}

## Preface

We already know the most common Git commands from
[Part 1]({{ site.baseurl }}{% link _posts/2024-05-08-eat-git-part-1.md %}) and
have explored Git's internals in
[Part 2]({{ site.baseurl }}{% link _posts/2024-05-15-eat-git-part-2.md %}). In
[Part 3]({{ site.baseurl }}{% link _posts/2024-05-19-eat-git-part-3.md %}), we
learned about additional tools that a single developer can benefit from. Here in
Part 4, we will see how all our knowledge about patch files, commits, and
branches come together, enabling a team of developers to work together
coherently.

We'll start by explaining what remotes are. Then, I will introduce some useful
commands for managing local and remote branches. After that, we'll get to the
most important part of this tutorial: sharing work and resolving conflicts.

## Remotes

If you remember in the
[Repositories]({{ site.baseurl }}{% link _posts/2024-05-08-eat-git-part-1.md %}#repositories)
section, I mentioned that they consist of a *.git* directory where Git saves all
its data, and possibly a checked-out commit. A remote is essentially the same
thing as a repository. The only difference is how we treat them. Repositories
usually live on developers' machines and go through many changes, including
checking out branches, staging, committing, or undoing work. On the other hand,
remotes usually live on a designated machine and primarily undergo push and pull
operations. When you push your branch to a remote, your work on that branch
becomes visible to anyone with access to that remote. When you pull a branch
from a remote to your local repository, someone else's work becomes accessible
to you.

It's not necessary for a remote to be on a separate machine; it can even reside
in another directory next to your local repository. In this section, I will show
you remotes in their most basic form, without anything fancy or extra, so you
can appreciate their simplicity later.

Let's create two entirely separate local repositories. In one of them, we will
list MSNBC anchors, and in the other, Fox News anchors:

```bash
# MSNBC anchors
mkdir left
cd left

git init
git config --local user.name 'Mohammad Rahimi'
git config --local user.email 'rahimi.mhmmd@outlook.com'

echo 'Joe Scarborough' >> employees.md
echo 'Mika Brzezinski' >> employees.md
echo 'Willie Geist' >> employees.md

git add employees.md
git commit -m 'Add Joe, Mika and Willie'

# Fox News anchors
cd ..
mkdir right
cd right

git init
git config --local user.name 'Mohammad Rahimi'
git config --local user.email 'rahimi.mhmmd@outlook.com'

echo 'Steve Doocy' >> employees.md
echo 'Ainsley Earhardt' >> employees.md
echo 'Brian Kilmeade' >> employees.md

git add employees.md
git commit -m 'Add Steve, Ainsley and Brian'
```

Let's set up a remote repository that both the left and right repositories can
push to and pull from. This remote repository will be created in a separate
directory next to the existing left and right repositories.

```bash
cd ..
mkdir correspondents
cd correspondents

git init --bare

tree -L 1
# .
# ├── branches
# ├── config
# ├── description
# ├── HEAD
# ├── hooks
# ├── info
# ├── objects
# └── refs
#
# 5 directories, 3 files
```

When the `--bare` flag is used, Git treats the current directory as if it were a
`.git` directory. This flag tells Git that we don't want to check out any
branches here, so there is no need for a working directory. Instead, the
repository's contents will be stored directly in the current directory.

This is all that needs to be done to set up a remote.

### Push

Now, let's go to our local repositories and push some branches.

```bash
cd ../left
git remote add press-room ../correspondents/
git remote show press-room
# * remote press-room
#   Fetch URL: ../correspondents/
#   Push  URL: ../correspondents/
#   HEAD branch: (unknown)


cd ../right
git remote add campaign-trail ../correspondents/
git remote show campaign-trail
# * remote campaign-trail
#   Fetch URL: ../correspondents/
#   Push  URL: ../correspondents/
#   HEAD branch: (unknown)
```

With the `git remote add` command, we can add a remote to our local repository.
We need to provide this command with a name for the remote and an address. This
address can be over a network or in a shared folder. Git supports many protocols
, including *HTTP* and *SSH*.

Let's push our local branches to the remote repository:

```bash
cd ../left
git push --set-upstream press-room main:msnbc

cd ../right
git push --set-upstream campaign-trail main:fox-news
```

When working with remotes, the above command is crucial. In each local
repository, we pushed our `main` branch to the remote, but with different names.
The command `git push --set-upstream press-room main:msnbc` instructs Git to
push the `main` branch from the local repository to a remote named `press-room`
and rename the branch on the remote to `msnbc`. Additionally, it sets the local
`main` branch to track the upstream branch `press-room/msnbc`. By setting the
upstream once, you create a connection between your local and remote branches,
so any future pushes or pulls on the local branch will automatically interact
with the remote counterpart.

Now, let's put ourselves in the position of the remote and observe how it
perceives these operations:

```bash
cd ../correspondents

git branch
#   fox-news
#   msnbc

git log msnbc
# commit d8a282080e0c312cbf1dec8fc994a68de6ec9793 (msnbc)
# Author: Mohammad Rahimi <rahimi.mhmmd@outlook.com>
# Date:   Sat May 25 11:37:08 2024 +0800
#
#     Add Joe, Mika and Willie

git log fox-news 
# commit b9b4fd7d46619cc2af50b189887b04e57b3f8327 (fox-news)
# Author: Mohammad Rahimi <rahimi.mhmmd@outlook.com>
# Date:   Sat May 25 11:37:16 2024 +0800
#
#     Add Steve, Ainsley and Brian

git log --oneline --all --graph
# * b9b4fd7 (fox-news) Add Steve, Ainsley and Brian
# * d8a2820 (msnbc) Add Joe, Mika and Willie
```

Note the absence of *HEAD* on the remote.

In this example, we changed both branch names on the remote because both of our
local repositories had a `main` branch. This shows that you can push two
entirely different repositories to the same remote. While this is not a standard
practice, it illustrates that remotes are essentially just a collection of
branches.

While it's uncommon to place multiple repositories on a single remote, it's
quite common to add multiple remotes to a repository. This might be done for
backup purposes or to share with different audiences and varying access levels.

All we've learned so far is only half the story. The other half involves pulling
from a remote. There are two commands that bring information from a remote to
the local repository. `git fetch` and `git pull`.

### Fetch

Let's start simple. We'll create an empty repository and see how we can fetch
from a remote to this empty repository. To see the changes in our branches, we
use the branch management command `git branch --all -vv` to show every branch,
whether local or remote, along with their details.

```bash
cd ..
mkdir audience
cd audience

git init
git config --local user.name 'Mohammad Rahimi'
git config --local user.email 'rahimi.mhmmd@outlook.com'

git remote add tv ../correspondents/

git branch --all -vv
# nothing to show

git fetch tv

git branch --all -vv
#   remotes/tv/fox-news b9b4fd7 Add Steve, Ainsley and Brian
#   remotes/tv/msnbc    d8a2820 Add Joe, Mika and Willie
```

The `git fetch` command does not update the local repository. Instead, it
updates the remote status in the local repository. It is the responsibility of
`git pull` to update the local repository.

### Pull

Our remote knows about both branches. To create a local branch from a remote
one:

```bash
git checkout -b channel-1 tv/msnbc

git branch --all -vv
# * channel-1           d8a2820 [tv/msnbc] Add Joe, Mika and Willie
#   remotes/tv/fox-news b9b4fd7 Add Steve, Ainsley and Brian
#   remotes/tv/msnbc    d8a2820 Add Joe, Mika and Willie
```

This command is similar to the one we used before to create feature branches
from our *main* branch, but here we used a remote branch instead of *main*.

The `*` in `* channel-1 683cc74 [tv/msnbc] Add Joe, Mika and Willie` indicates
the checked-out branch, and [tv/msnbc] indicates the upstream for this branch.

Now let's go to our MSNBC repository and add another anchor:

```bash
cd ../left

echo 'Rachel Maddow' >> employees.md
git add employees.md
git commit -m 'Add Maddow'

git push press-room main:msnbc
```

If the remote branch name was the same as the local one, we could use
`git push press-room main`.

Let's pull updates from the *msnbc* branch of *tv* to our *audience* repository:

```bash
cd ../audience
git branch --all -vv
# * channel-1           d8a2820 [tv/msnbc] Add Joe, Mika and Willie
#   remotes/tv/fox-news b9b4fd7 Add Steve, Ainsley and Brian
#   remotes/tv/msnbc    d8a2820 Add Joe, Mika and Willie

git pull tv msnbc

git branch --all -vv
# * channel-1           7d2e842 [tv/msnbc] Add Maddow
#   remotes/tv/fox-news b9b4fd7 Add Steve, Ainsley and Brian
#   remotes/tv/msnbc    7d2e842 Add Maddow
```

Because we are on our *channel-1* branch, which is set to track *tv/msnbc*, any
work that exists on *tv/msnbc* will be brought to the *channel-1* branch.
Additionally, there is a fetch operation performed internally in every pull
command.

If you use `git pull tv` instead of `git pull tv msnbc`, it will not only pull
the *msnbc* branch but also fetch every other remote branch from the *tv*
remote.

### Force Push

This is the operation that I have been warning you about. If you change history
on your local branch and try to push it to the remote, it will result in an
error, and Git won't push any of your commits.

```bash
cd ../left

echo 'Andrea Mitchell' >> employees.md
git add employees.md
git commit --amend -m 'Add Maddow and Mitchell'

git push press-room main:msnbc 
# To ../correspondents/
#  ! [rejected]        main -> msnbc (non-fast-forward)
# error: failed to push some refs to '../correspondents/'
# hint: Updates were rejected because a pushed branch tip is behind its remote
# hint: counterpart. If you want to integrate the remote changes, use 'git pull'
# hint: before pushing again.
# hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

Because the *main* hash on the local branch is different from the *msnbc* hash
on the remote, Git assumes that someone else has pushed changes to the remote
branch that you don't have locally. It advises you to pull those changes first.
However, if this was indeed the case, it would lead to potential conflicts
because two people shouldn't push to the same branch simultaneously. In this
scenario, where you've amended the commit history using `git commit --amend`, if
you still need to push, you can force it using:

```bash
git push --force press-room main:msnbc
```

And on the other side pull the changes:

```bash
cd ../audience

git pull tv msnbc
# ...
# hint: You have divergent branches and need to specify how to reconcile them.
# hint: You can do so by running one of the following commands sometime before
# hint: your next pull:
# hint:
# hint:   git config pull.rebase false  # merge
# hint:   git config pull.rebase true   # rebase
# hint:   git config pull.ff only       # fast-forward only
# hint:
# hint: You can replace "git config" with "git config --global" to set a default
# hint: preference for all repositories. You can also pass --rebase, --no-rebase,
# hint: or --ff-only on the command line to override the configured default per
# hint: invocation.
# fatal: Need to specify how to reconcile divergent branches.
```

On this side, Git detects that there are differences between the remote and
local branches. It believes that there has been some work on the remote that the
local branch doesn't have, and vice versa. However, we know that the discrepancy
is due to the change in history and the force push we executed earlier. To
resolve this problem, we can reset our local branch to one commit before,
effectively removing the last commit, and then pull again:

```bash
git log --oneline 
# 7d2e842 (HEAD -> channel-1) Add Maddow
# d8a2820 Add Joe, Mika and Willie

git reset --hard HEAD~

git log --oneline 
# d8a2820 (HEAD -> channel-1) Add Joe, Mika and Willie

git pull tv msnbc

git branch --all -vv
# * channel-1           72970d2 [tv/msnbc] Add Maddow and Mitchell
#   remotes/tv/fox-news b9b4fd7 Add Steve, Ainsley and Brian
#   remotes/tv/msnbc    72970d2 Add Maddow and Mitchell
```

As you can see, as soon as we alter history or have multiple individuals pushing
to the same branch, things become complicated, and we're often compelled to
resort to risky Git operations. Git workflows like merge and rebase are designed
to reduce these complications. They provide a structured approach to working
with Git, reducing friction and minimizing potential conflicts.

## Branch Management

In the previous section, we learned that remotes are essentially repositories
containing collections of branches. This underscores the critical role that
branches play in Git. Nearly every problem that can occur in a repository is
somehow connected to branches, whether it's pushing to the wrong branch or
dealing with a tangled history.

Having a solid grasp of branches and their relationships can be quite beneficial
. You're already familiar with one branch management command. To list all local
branches in a repository, you can use:

```bash
git branch --all -vv
```

This command provides comprehensive information about the repository and its
remotes. It displays the currently checked-out branch, the last commit on each
branch, tracking branches (upstreams), and lists the remote branches.

## Merging

## Rebasing

## Conflicts
