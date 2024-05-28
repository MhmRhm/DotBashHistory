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
. You're already familiar with one branch management command. To list all
branches in a repository, you can use:

```bash
git branch --all -vv
```

This command provides comprehensive information about the repository and its
remotes. It displays the currently checked-out branch, the last commit on each
branch, tracking branches (upstreams), and lists the remote branches.

You should always do your work on a feature branch, never on the *main* branch.
Feature branches are typically created from the *main* branch. Here is the
procedure I recommend:

```bash
# update main branch
git checkou main
git pull origin main

# create feature branch from main
git checkout -b feat main
```

If you create the feature branch from the remote *main* branch, i.e.,
*origin/main*, Git will set up the remote branch as the upstream for your local
feature branch. However, you might end up with the wrong remote as the tracking
branch. To remove the upstream from a branch and set another, follow these
steps:

```sh
git checkout feat

# remove upstream
git branch --unset-upstream

# set a new upstream
git branch --set-upstream-to=origin/feat
```

To delete a local branch:

```bash
git branch -D feat
```

To rename a local branch:

```bash
git branch --move old-name new-name
```

To delete a branch on remote:

```bash
git push origin --delete old-name
```

To rename a branch on remote:

```bash
# pull the old branch to your local repository with the new name
git fetch origin
git checkout -b new-name origin/old-name

# remove old tracking branch from new local branch
git branch --unset-upstream

# push the new branch to the remote repository
git push origin new-name

# delete the old branch from remote repository
git push origin --delete old-name
git branch -D new-name
```

Being able to manage your branches is crucial to maintaining a clear history.

## Merging

Merging and rebasing are quite simple procedures. What can make them difficult
are the conflicts that we face while performing a merge or rebase. In this
section, I will show you what merge commits are and discuss fast-forward merges.

In the following examples, I will append the version-controlled files with the
current time. This will help you follow where each change comes into the picture
and how they form the final history. I used two files to avoid conflicts.

Let's create a repository with one commit for each file in it:

```bash
cd ..
mkdir repo1
cd repo1

git init
git config --local user.name 'Mohammad Rahimi'
git config --local user.email 'rahimi.mhmmd@outlook.com'

msg=$(date +%T) && echo $msg >> file1 && git add file1 && git commit -m "$msg"
sleep 3
msg=$(date +%T) && echo $msg >> file2 && git add file2 && git commit -m "$msg"

cat file1 file2
# 20:12:37
# 20:12:40

git log
# commit c8ccc48d8c46477dac269d27177ab27500e5023c (HEAD -> main)
# Author: Mohammad Rahimi <rahimi.mhmmd@outlook.com>
# Date:   Sun May 26 20:12:40 2024 +0800
#
#     20:12:40
#
# commit c3541992bf61294bf0283edd32378235ad188f30
# Author: Mohammad Rahimi <rahimi.mhmmd@outlook.com>
# Date:   Sun May 26 20:12:37 2024 +0800
#
#     20:12:37
```

Create two branches from this commit:

```bash
git checkout -b feat1 main
git checkout -b feat2 main
```

Create two commits on *feat1* and then two commits on *feat2*. Wait a few
seconds between each commit to ensure the file content and commit messages
differ by a few seconds.

```bash
git checkout feat1
msg=$(date +%T) && echo $msg >> file1 && git add file1 && git commit -m "$msg"
sleep 3
msg=$(date +%T) && echo $msg >> file1 && git add file1 && git commit -m "$msg"
sleep 3

git checkout feat2
msg=$(date +%T) && echo $msg >> file2 && git add file2 && git commit -m "$msg"
sleep 3
msg=$(date +%T) && echo $msg >> file2 && git add file2 && git commit -m "$msg"

git log --oneline --all --graph
# * b09b046 (HEAD -> feat2) 20:14:07
# * dc3a6d3 20:14:04
# | * ef67eb8 (feat1) 20:14:01
# | * 885a51c 20:13:58
# |/  
# * c8ccc48 (main) 20:12:40
# * c354199 20:12:37
```

Now we can see the exact times when each of these four commits were made and the
diff introduced in each commit. Whether you choose to use merge or rebase, your
repository will initially look like above. A few feature branches waiting to be
integrated into the *main* branch.

Let's merge *feat1* into *main*:

```bash
git checkout main
git merge feat1

git log --oneline --all --graph
# * b09b046 (feat2) 20:14:07
# * dc3a6d3 20:14:04
# | * ef67eb8 (HEAD -> main, feat1) 20:14:01
# | * 885a51c 20:13:58
# |/  
# * c8ccc48 20:12:40
# * c354199 20:12:37
```

This was a fast-forward merge because the merge operation simply moved the
*main* pointer to the same commit as *feat1*. Now, *HEAD*, *main*, and *feat1*
all point to the same commit.

Merge the other branch into *main*:

```bash
git merge feat2
```

Accept the commit message as is. You are prompted to enter a commit message
because Git had to create a merge commit, indicating that a fast-forward merge
was not possible; in other words, we merged two diverged branches.

```bash
git log --oneline --all --graph
# *   c1e2259 (HEAD -> main) Merge branch 'feat2'
# |\  
# | * b09b046 (feat2) 20:14:07
# | * dc3a6d3 20:14:04
# * | ef67eb8 (feat1) 20:14:01
# * | 885a51c 20:13:58
# |/  
# * c8ccc48 20:12:40
# * c354199 20:12:37
```

A merge commit is a commit with two parents and usually has no diff unless
conflicts arise.

```bash
git log --patch -1
# commit c1e225981e8ab85fb540ae76088761362db56c89 (HEAD -> main)
# Merge: ef67eb8 b09b046
# Author: Mohammad Rahimi <rahimi.mhmmd@outlook.com>
# Date:   Sun May 26 20:16:34 2024 +0800
#
#     Merge branch 'feat2'

git cat-file -p HEAD
# tree 2f74f78d47dd07601049cad50cc9eb66bc9f518f
# parent ef67eb8e83f5cc23c8e74bd51cf0467435b9a26f
# parent b09b046ace1008eba63b7e1a937b1e7b5926ad31
# author Mohammad Rahimi <rahimi.mhmmd@outlook.com> 1716725794 +0800
# committer Mohammad Rahimi <rahimi.mhmmd@outlook.com> 1716725794 +0800
#
# Merge branch 'feat2'
```

In practice, developers never merge directly into the *main* branch to avoid
difficulties that may arise when multiple people push changes to the same
branch. Instead, in a real-world scenario, you would typically have a remote
repository hosted on a Git server like [Gitea][gitea] or [GitHub][github]. You
would work on feature branches and push these branches to the remote repository.
Then, you would ask other team members to review your work. If they have valid
comments, you would address them by making new changes to your local branch and
pushing those changes to the remote repository again. Once everyone is on the
same page and your changes are approved, the person responsible for the project
would merge your changes into the *main* branch, or the server might merge them
automatically after approval. The key is that only one entity can merge changes
into the *main* branch. Git servers often provide options to protect a branch
and refuse any pushes to that branch.

Running `git log --oneline --graph --all`, the line containing the *885a51c*
commit shows two vertical lines representing the *feat2* and *main* branches. In
large projects like the Linux kernel, you may see many more branches, so many
that they fill the entire screen width with these vertical lines. In such
projects, to show only commits with actual work, you can use the following
command:

```bash
git log --oneline --all --no-merges
# b09b046 (feat2) 20:14:07
# dc3a6d3 20:14:04
# ef67eb8 (feat1) 20:14:01
# 885a51c 20:13:58
# c8ccc48 20:12:40
# c354199 20:12:37
```

If your work on a feature branch will take a long time, other changes from your
teammates may get integrated into the *main* branch in the meantime. Some of
these changes might conflict with yours. To keep your feature branch up to date
and resolve conflicts gradually, it’s better to regularly merge the *main*
branch into your feature branch while working on it. We will discuss conflicts
shortly.

## Rebasing

In the previous section, both *feat1* and *feat2* started from the same commit,
performed independent work, and then merged into *main*. Now, let's consider a
different scenario. First *feat1* starts and is merged into *main* using a
fast-forward merge. Then *feat2* starts from the updated *main* branch and, when
merged, also uses a fast-forward merge. This way, we maintain a linear history.
This is the goal that rebase aims to achieve.

With rebase, you can start both feature branches simultaneously and work on them
in parallel. However, when it comes to integrate your work into the *main*
branch, before using the merge command, use rebase. Rebase takes the work
you've done in your feature branch and replays it onto *main*, making it seem as
though your feature branches always started from the latest commit on *main*.
The merge will be always a fast-forward operation, resulting in a clean, linear
history. Let's see how this works in practice.

We will continue with our previous example. Create another branch and add some
commits to it and the *main* branch:

```bash
git checkout -b feat3 main

git checkout main
msg=$(date +%T) && echo $msg >> file1 && git add file1 && git commit -m "$msg"
sleep 3
msg=$(date +%T) && echo $msg >> file1 && git add file1 && git commit -m "$msg"
sleep 3

git checkout feat3
msg=$(date +%T) && echo $msg >> file2 && git add file2 && git commit -m "$msg"
sleep 3
msg=$(date +%T) && echo $msg >> file2 && git add file2 && git commit -m "$msg"

git log --oneline --all --graph
# * 643b769 (HEAD -> feat3) 07:21:00
# * 555ce45 07:20:57
# | * 4e0cc3f (main) 07:20:52
# | * 3c31ef2 07:20:49
# |/  
# *   c1e2259 Merge branch 'feat2'
```

To rebase *feat3* onto *main*:

```bash
git checkout feat3
git rebase main

git log --oneline --all --graph
# * d168509 (HEAD -> feat3) 07:21:00
# * 619cfa8 07:20:57
# * 4e0cc3f (main) 07:20:52
# * 3c31ef2 07:20:49
# *   c1e2259 Merge branch 'feat2'
```

Notice that all commit hashes on *feat3* have changed. This is because the
parent for the first commit on *feat3* has changed. Also, notice how everything
is now aligned on one line.

In a merge workflow, you can use the rebase operation. However, in a rebase
workflow, you should avoid using the merge operation to update your feature
branches.

If, while you are working on your feature branch, other work gets integrated
into *main*, your branch will diverge from *main*, and you will need to repeat
the rebase. Doing this will bring up any possible conflicts and ensure that,
after the rebase, no conflicts remain in your code. We will discuss conflicts
shortly.

To integrate your work into the *main* branch:

```bash
git checkout main
git merge --ff-only feat3

git log --oneline --all --graph
# * d168509 (HEAD -> main, feat3) 07:21:00
# * 619cfa8 07:20:57
# * 4e0cc3f 07:20:52
# * 3c31ef2 07:20:49
# *   c1e2259 Merge branch 'feat2'
```

Notice that this operation should only be performed by the entity responsible
for the *main* branch.

If you have pushed your feature branch to a remote server, you will need to
force push the changes after each rebase since commit hashes are changed. In
this workflow, other team members should not branch out from your feature
branches because you are constantly changing history on your branch.

## Conflicts

Finally, you've made it here! So far, I've avoided any conflicts in the examples
because they deserve their own section. Here, we'll explore why conflicts occur
and how they manifest themselves in both merge and rebase workflows.

In this section I won't use any third-party tools for resolving conflicts
because I want you to see them in their true form. That being said, there are
some great tools available out there, and I always use them. Some of them
include Vimdiff, Meld, and P4Merge.

A conflict arises when two branches don't agree on a piece of code. Let's create
a conflict. In the following example, I will start from a clean repository.

```bash
cd ..
mkdir repo2
cd repo2

git init
git config --local user.name 'Mohammad Rahimi'
git config --local user.email 'rahimi.mhmmd@outlook.com'

git checkout main
msg=$(date +%T) && echo $msg >> t.md && git add t.md && git commit -m "$msg"
sleep 3

git checkout -b feat main
msg=$(date +%T) && echo $msg >> t.md && git add t.md && git commit -m "$msg"
sleep 3

git checkout main
msg=$(date +%T) && echo $msg >> t.md && git add t.md && git commit -m "$msg"
sleep 3

git checkout main
git merge feat
# Auto-merging t.md
# CONFLICT (content): Merge conflict in t.md
# Automatic merge failed; fix conflicts and then commit the result.
```

When a conflict happens, Git enters conflict mode. In this mode, you can edit
files, stage them, continue to the next round of conflicts, or abort the
operation altogether. Let's first take a look at our conflict.

```bash
cat t.md
# 20:07:03
# <<<<<<< HEAD
# 20:07:09
# =======
# 20:07:06
# >>>>>>> feat
```

Remember, we were on the *main* branch and were merging *feat* into *main*. In a
conflict, you'll see two sections in the conflicting file. The first section is
the *ours*, which is between `<<<<<<< HEAD` and `=======`. This represents the
content from the branch you were on and merging into, which is *main* in our
example. Different tools may refer to this section with names like *local*,
*current*, *source*, *mine*, or *Left*.

The second section is the *theirs*, found between `=======` and `>>>>>>> feat`.
This represents the incoming changes from the branch being merged, in this case,
*feat*. Other names for this section include *remote*, *incoming*, *target*, or
*right*.

Before we continue to resolving the conflict, let me introduce you to the third
section you might see based on your configuration.

```bash
git merge --abort

git config --local merge.conflictstyle diff3

git merge feat

cat t.md
# 20:07:03
# <<<<<<< HEAD
# 20:07:09
# ||||||| dec4cb3
# =======
# 20:07:06
# >>>>>>> feat
```

Anything between `|||||||` and `=======` is called the *base*. This section
shows the common ancestor of the conflicting branches, i.e., the state of the
file at the commit from which both branches diverged. In our case, the base
portion is empty.

To resolve the conflict, I will edit the file as follows:

```bash
# 20:07:03
# 20:07:06
# 20:07:09
```

Then stage the file:

```bash
git add t.md
```

And continue with the merge which will ask for a commit message:

```bash
git merge --continue

git log --oneline --all --graph 
# *   546e0ac (HEAD -> main) Merge branch 'feat'
# |\  
# | * e9b16ef (feat) 20:07:06
# * | 6574241 20:07:09
# |/  
# * dec4cb3 20:07:03
```

If you prefer not to edit files by hand and would rather use a more advanced
tool, after encountering a conflict, you can use one of Git's built-in conflict
resolution tools:

```bash
git merge feat
# Auto-merging t.md
# CONFLICT (content): Merge conflict in t.md
# Automatic merge failed; fix conflicts and then commit the result.

git mergetool
# This message is displayed because 'merge.tool' is not configured.
# See 'git mergetool --tool-help' or 'git help config' for more details.
# 'git mergetool' will now attempt to use one of the following tools:
# meld opendiff kdiff3 tkdiff xxdiff tortoisemerge gvimdiff diffuse diffmerge ecmerge p4merge araxis bc codecompare smerge emerge vimdiff nvimdiff
# Merging:
# t.md
#
# Normal merge conflict for 't.md':
#   {local}: modified file
#   {remote}: modified file
# Hit return to start merge resolution tool (bc):
```

At this point, I've covered all the possible operations you can perform during
a conflict.

### Conflicts in Rebase

When performing a merge, you will encounter conflicts only once at the point of
the merge. This is not the case with a rebase operation. During a rebase, Git
replays each commit from your branch onto the target branch, meaning you may
encounter conflicts at each commit that Git attempts to apply. This can
complicate the process of conflict resolution since you have to resolve
conflicts for each commit individually. However, in both cases, you will face
the same conflicts: all at once during a merge, or commit by commit during a
rebase.

Let me show you what I mean. In the following example, I will start from a clean
repository.

```bash
cd ..
mkdir repo3
cd repo3

git init
git config --local user.name 'Mohammad Rahimi'
git config --local user.email 'rahimi.mhmmd@outlook.com'
git config --local merge.conflictstyle diff3

msg=$(date +%T) && echo $msg >> t.md && git add t.md && git commit -m "$msg"
sleep 3

git checkout -b feat main
msg=$(date +%T) && echo $msg >> t.md && git add t.md && git commit -m "$msg"
sleep 3
git checkout main
msg=$(date +%T) && echo $msg >> t.md && git add t.md && git commit -m "$msg"
sleep 3

git checkout feat
msg=$(date +%T) && echo $msg >> t.md && git add t.md && git commit -m "$msg"
sleep 3
git checkout main
msg=$(date +%T) && echo $msg >> t.md && git add t.md && git commit -m "$msg"

git log --oneline --all --graph
# * 7f39578 (HEAD -> main) 08:26:37
# * 13daa1d 08:26:31
# | * 1bea4e6 (feat) 08:26:34
# | * cee92fe 08:26:28
# |/  
# * bfed5e1 08:26:25
```

We have distributed our commits between two branches, and we want the final
result after the rebase to reflect a chronological order of all commits.

Let's start the rebase. We expect Git to take commit *cee92fe* (08:26:28) and
place it after commit *7f39578* (08:26:37), and then do the same with commit
*1bea4e6* (08:26:34).

```bash
git checkout feat
git rebase main
# Auto-merging t.md
# CONFLICT (content): Merge conflict in t.md
# error: could not apply cee92fe... 08:26:28
# hint: Resolve all conflicts manually, mark them as resolved with
# hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
# hint: You can instead skip this commit: run "git rebase --skip".
# hint: To abort and get back to the state before "git rebase", run "git rebase --abort".
# hint: Disable this message with "git config advice.mergeConflict false"
# Could not apply cee92fe... 08:26:28
```

We encountered a conflict while rebasing commit *cee92fe* (08:26:28). Let's
resolve it by editing the file from:

```bash
# 08:26:25
# <<<<<<< HEAD
# 08:26:31
# 08:26:37
# ||||||| parent of cee92fe (08:26:28)
# =======
# 08:26:28
# >>>>>>> cee92fe (08:26:28)
```

to:

```bash
# 08:26:25
# 08:26:28
# 08:26:31
# 08:26:37
```

Stage the file and continue the rebase:

```bash
git add t.md
git rebase --continue
```

Accept the commit message and proceed to the next conflict. Edit the file from:

```bash
# 08:26:25
# 08:26:28
# <<<<<<< HEAD
# 08:26:31
# 08:26:37
# ||||||| parent of 1bea4e6 (08:26:34)
# =======
# 08:26:34
# >>>>>>> 1bea4e6 (08:26:34)
```

to:

```bash
# 08:26:25
# 08:26:28
# 08:26:31
# 08:26:34
# 08:26:37
```

Stage the file and continue. After accepting the commit message, your repository
will look like the following:

```bash
git add t.md
git rebase --continue

git log --oneline --all --graph
# * 0db41fa (HEAD -> feat) 08:26:34
# * 730a3ec 08:26:28
# * 7f39578 (main) 08:26:37
# * 13daa1d 08:26:31
# * bfed5e1 08:26:25
```

If you find yourself resolving the same conflicts repeatedly during each rebase,
consider using [Rerere][rerere]. 

Rerere stands for "reuse recorded resolution" and helps Git remember how you've
resolved conflicts in the past. This feature can save you time and effort by
automatically applying the same conflict resolutions when they occur again. To
enable it, run:

```bash
git config --local rerere.enabled true
```

This way, Git will record your conflict resolutions and reuse them in future
merges or rebases.

## Summary

Now you know everything you need to be proficient with Git. We explained remotes
and showed how they essentially hold branches. We learned some commands that
help us manage and keep track of our branches. We finally learned about merge
and rebase, gaining a solid understanding of how conflicts arise and how to
address them.

In [Part 5]({{ site.baseurl }}{% link _posts/2024-05-28-eat-git-part-5.md %}), I
will go through a few common mistakes that my colleagues and I have made while
working with Git and show you how to recover from them if you find yourself in a
similar situation.


[gitea]: https://about.gitea.com/
[github]: https://github.com/
[rerere]: https://git-scm.com/docs/git-rerere
