---
layout: post
title: "EatGit - Part 1"
excerpt: Part 1 of a quick Git tutorial to teach you everything you need to know.
seo_description: Part 1 of Git tutorial - Learn about Working Tree, Index, Commits, Branches and History.
categories: git
author:
- Mohammad Rahimi
---

* Table of Contents
{:toc}

## Preface

This tutorial on Git has five parts, and this is the first. The following shows
discussed material in each part:

1. Getting to Know Git
    - Creating Repositories
    - Working Tree and Index
    - Commits
    - Branches
    - History
2. Behind the Scene
    - Git Internals
        - Objects
        - Branches
        - Hand-made Commit
3. Local Workflows
    - Patch Files
    - Cherry Picking
    - Interactive Rebase
    - Debugging with Git
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

In Part 1, we get to know Git and become comfortable using it. If you're
curious and want to understand what everyone is talking about, this part should
be enough for you.

In [Part 2]({{ site.baseurl }}{% link _posts/2024-05-15-eat-git-part-2.md %}), I
will explain how Git creates and stores the data structures that enable it to do
what it does. This part, in my opinion, is necessary to become confident at
using other tools introduced later.

In [Part 3]({{ site.baseurl }}{% link _posts/2024-05-19-eat-git-part-3.md %}), I
will wrap up most of the things you need to know to use Git effectively as a
solo developer.

In Part 4, you will find essential information if you have been using Git and
want to learn how to apply your knowledge in a collaborative environment. I will
introduce three workflows that I have successfully used with my colleagues.

In Part 5 I share my experience with you. In my previous two positions, I
introduced my colleagues to Git, and then we started using it in our day-to-day
development. This part sums up the mistakes that me and my colleagues used to
make and show how to fix things when something goes wrong.

### Why Another Git Tutorial?

I wanted to make a guide to Git that I wish I had when I first started learning.
If you have time to read the [Pro Git][pro-git-book] Book, please do so—it's a
great resource that inspired much of this tutorial. However, if you're looking
to learn Git thoroughly in about a week, spending 1 to 2 hours daily, and want
to be able to assist your colleagues, I hope this tutorial can help you achieve
that goal.

For this tutorial, all you need is to install Git, which I'll explain shortly.
Installing Git is straightforward. However, as a recommendation, I suggest using
Ubuntu in a Virtual Machine to keep things organized. While not essential, it's
a good practice I follow for most of my development work. Using VMs helps keep
my main computer clean and fast, and if anything goes wrong, I can quickly
revert to a clean state by cloning the base image.

### Why Command Line?

There are some great graphical tools and addons for Git out there, like the
[GitLens][git-lens] addon for VSCode, which I personally use often. However,
these tools aren't always available, and they might not provide the depth of
insight you need when working with Git. Plus, using the command line is usually
faster, and it provides useful information in its output.

Whenever my colleagues encounter issues with Git, I find that the command line
is the most reliable tool for solving those cleanly and safely. It also offers a
consistent experience regardless of your development environment. The
[Git Reference][git-reference] covers the command line extensively. So, my
advice is to give the command line a try and use Git in that way. People often
resist learning to use the command line, but they don't regret it once they do.

For Linux and Mac, simply use the Terminal application. Ensure that
auto-complete is set up for your terminal. On Windows, use Git Bash. To set up
auto-complete on Mac, add `autoload -Uz compinit && compinit` to your
*~/.zprofile* file. For Linux, run `sudo apt-get install bash-completion`.

## Installation

The best way to install Git is to head to
[Git Downloads](https://git-scm.com/downloads).

To install Git on Ubuntu:

```bash
sudo apt-get install git
```

To install on Mac:

```bash
/bin/bash -c "$(curl -fsSL\
 https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

The above command will ask you for your password and install the Xcode command
line tools. When finished, it will ask you to run a few commands to make `brew`
accessible in your terminal. Copy-Paste those commands and execute them. Now,
you should be able to run:

```bash
brew install git
```

To install on Windows, choose one of the installers from
[Download for Windows][download-for-windows].

### Build from Source (for Later)

To build the latest Git on Ubuntu, you need to get the Git source code. There
are many ways to achieve that. Git is maintained in a Git repository! The
original repository is at [kernel.org][main-git-repo]. There, you can download
the code as a zip file. Those archives are usually a week or so behind the
latest changes. There is also a mirror on [GitHub][github-git-repo]. In the
GitHub repository, click on Clone at the top of the page and choose Download
Zip. That code may be a day or two behind the latest changes. To really get the
latest code for Git, you need Git!

To build and install Git:

```bash
# install all the dependencies
# In Ubuntu 22.04 to get this list:
#   sudo nano /etc/apt/sources.list
# uncomment lines starting with `deb-src`, then:
#   sudo apt-get build-dep git
# In Ubuntu 24.04 see Update 1

sudo apt-get install apache2-dev asciidoc asciidoc-base asciidoc-common\
 autoconf automake autopoint autotools-dev binutils binutils-aarch64-linux-gnu\
 binutils-common build-essential cvs cvsps debhelper debugedit dh-autoreconf\
 dh-exec dh-strip-nondeterminism docbook-xsl dpkg-dev dwz g++ g++-11 gcc gcc-11\
 gettext intltool-debian libapr1 libapr1-dev libaprutil1 libaprutil1-dev\
 libarchive-zip-perl libasan6 libbinutils libc-dev-bin libc6-dev libcc1-0\
 libcgi-pm-perl libcrypt-dev libctf-nobfd0 libctf0 libcurl4-gnutls-dev\
 libdbd-sqlite3-perl libdbi-perl libdebhelper-perl libdpkg-perl liberror-perl\
 libexpat1-dev libfile-stripnondeterminism-perl libgcc-11-dev libhwasan0\
 libio-pty-perl libitm1 libldap-dev libldap2-dev liblsan0 libnsl-dev\
 libpcre2-16-0 libpcre2-dev libpcre2-posix3 libsctp-dev libsctp1 libserf-1-1\
 libsigsegv2 libstdc++-11-dev libsub-override-perl libsvn-perl libsvn1\
 libtirpc-dev libtool libtsan0 libubsan1 libutf8proc2 libxml2-utils\
 libyaml-perl linux-libc-dev lto-disabled-list m4 make po-debconf rpcsvc-proto\
 subversion uuid-dev xmlto xsltproc zlib1g-dev

sudo apt-get install docbook2x

# if downloaded an archive
# replace <version> with approapriate value
tar -zxpvf git-<version>.tar.gz
cd git-<version>/

# if building from Git's Git repository
sudo apt-get install git
git --version
git clone --recurse-submodules --depth=1\
 https://git.kernel.org/pub/scm/git/git.git
cd git/

make configure
./configure --prefix=/usr
make -j $(nproc) all doc info
sudo make install install-doc install-html install-info

git --version
```

#### Update 1

The *sources.list* file's location and format have changed in Ubuntu 24.04. To
get the list for all dependencies:

```bash
sudo nano /etc/apt/sources.list.d/ubuntu.sources

# after copying and pasting every entry, update
# `Types: deb` to `Types: deb-src` for the copied text

sudo apt-get update
sudo apt-get build-dep git
```

Now we are ready to learn Git.

## Repositories

Git is a version control system, which means that as you make changes to your
files, you can decide which changes to record in Git and when to record them.
Each time you save something in Git, you create a snapshot of your project at
that point in time, which you can later revisit. Git stores these time-points in
a directory called *.git*. While you can explore these files, it's important not
to modify them directly without Git's knowledge.

A Git repository consists of this *.git* directory. Additionally, a repository
can have a specific time-point, checked out and visible.

Now, let's create a project and start working on it. In this initial example,
I'll create a list of anchors currently working at CNN.

```bash
mkdir cnn && cd cnn
touch employees.md

# you can use Vim, Nano or any other text editors
echo 'John Berman' >> employees.md
echo 'Dana Bash' >> employees.md
ls -al
```

Before running any Git commands, for consistency, let's set the default branch
name to *main* by executing the following command:

```bash
git config --global init.defaultBranch 'main'
```

Branches in Git are like timelines. We will explore this concept and the
`git config` command shortly.

Before we delve deeper, it's a good idea to set the default editor that Git
uses. For this tutorial, Nano is more than enough. To set Nano as the
default editor for Git:

```bash
git config --global core.editor "nano"
```

You can also set Notepad++ as the default editor by providing the command above
with the path to the Notepad++ executable. Now, let's begin version controlling
in this directory. To initialize a Git repository:

```bash
git init

# to list all files in a directory
ls -al
```

Note the *.git* directory. Git always keeps the status of the repository. To see
the current status:

```bash
git status
# On branch main
#
# No commits yet
#
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#         employees.md
#
# nothing added to commit but untracked files present (use "git add" to track)
```

This includes several important pieces of information. For now, focus on the
*Untracked files* section. These are files that Git doesn't care about. Any
changes you make to these files won't appear in the history. To let Git know
about the presence of the *employees.md* file:

```bash
git add employees.md
git status
# On branch main
#
# No commits yet
#
# Changes to be committed:
#   (use "git rm --cached <file>..." to unstage)
#         new file:   employees.md
#
```

The section named *Changes to be committed* lists files that will be included in
the next time-point. These time-points are referred to as *commits*. The phrase
*No commits yet* indicates that the repository is new and nothing has been
committed to it yet. The first line always indicates the current branch you are
on.

## Working Tree and Index

The *Working Tree* refers to your directory under version control. It's called a
"tree" because the structure of a directory and its sub-directories can be
represented as a tree. You put files from *Working Tree* into the *Index*. The
*Index* or *Staging area* is similar to a waiting hall before your flight takes
off (commits being created).

If you've made changes to multiple files but only a few of them are relevant to
a task, you can add only those files to the *Index* and create a commit. To add
all changed files from the *Working Tree* to the *Staging area*:

```bash
git add --all
# or
git add -A
# or
git add .
```

Caution! Always use `git status` before running this command. Many applications
generate supporting cache files, and we don't want to version control them
because they hold no value and can be recreated.

If you add a *.gitignore* file to a directory under Git version control, Git
will ignore any files or directories listed in the *.gitignore* file. You can
place this file next to the *.git* directory. You can find *.gitignore* files
tailored for various development environments in this
[GitHub][git-ignore-collection] repository.

## Commits

Each commit in Git contains six pieces of information:
1. **Changes**: This includes the files and the changes made to them that we
want to commit.
2. **Author**: The name and email address of the person who made the changes in
the files.
3. **Committer**: The person who made the commit. You can make changes to files
and send your modifications to someone with the authority to perform the commit
on your behalf. This individual is referred to as the *Committer*. Typically,
the *Author* and *Committer* are the same person.
4. **Time**: The timestamp of when the changes were authored and committed.
5. **Parent**: The ID of the commit before the one we are creating. This becomes
important when explaining Rebasing in Part 4.
6. **Message**: An explanation of why and how some changes were made. This
message is crucial for understanding the purpose of the commit, and it can be
read later by you and others instead of inspecting each file to figure out what
was done in a commit.

Git needs to know your name and email address because the person who has changed
the file becomes part of the history, and others may need to contact that person
. To inform Git about your information:

```bash
git config --local user.name 'Mohammad Rahimi'
git config --local user.email 'rahimi.mhmmd@outlook.com'
```

The `--local` option sets this information for only this repository.
Alternatively, you have the option to use `--global`. The downside of using
`--local` is that you may forget to set it before making a commit. I suggest to
use `--local` when you are working on different projects and using different
email addresses for them, which for me is always.

To commit our changes with a single-line message:

```bash
git commit -m 'Add Barman and Bash'
```

To commit our changes with a multi-line message:

```bash
git commit
```

Git combines all the mentioned pieces and creates a hash. If even a single bit
changes, the entire hash will change. Every object in Git has its own unique
hash. That's why you should not change anything in the *.git* directory.

The command above opens a text editor, which can be Vim or Nano. If you exit the
editor without saving, the commit is canceled.

To exit Nano without saving, press *Ctrl+x*, then the *N* key.

To exit Vim without saving, press *Esc*, then type `:q!`, and confirm with the
*Enter* or *return* key. Don't forget the `:` in `:q!`.

To save and exit in Nano, press *Ctrl+x*, then *Y*, and confirm the file name
with the *Enter* or *return* key.

In Vim, press *Esc*, then type `:wq`, and confirm with the *Enter* or *return*
key.

Multi-line messages are preferred. In your editor, on the first line, type a
short explanation. Add an empty line, and on the third line, you can explain as
much as you want. You can have paragraphs of text! Just keep the first line
short. The first line is the part that will be shown later when we see the
history with the `git log --oneline` command.

You should take commit messages seriously, as they reflect your professionalism
to others. Refer to [How to Write a Git Commit Message][commit-message-guide]
for a detailed explanation on effective commit messages.

Let's take a look at the status now:

```bash
git status
# On branch main
# nothing to commit, working tree clean
```

Congratulations! We've made our first commit.

If you take a closer look at the commit message, you will notice there is a
typo. In situations like this, when you forget something or there is a mistake
in your previous changes, you don't need to create another commit to fix it. You
can simply amend the last commit. To change the last commit:

```bash
git status

git commit --amend

git status
```

This will open an editor, allowing you to change the commit message. Note that
the commit hash is changed. This can have consequences. If you have already
published your work and others have based their changes on your commits, you
cannot go back and change the history. These hashes are an important part of the
history. As a rule of thumb, never change what has been published. We'll learn
about publishing our work in Part 4.

It happens quite often that I forget to set my name and email in a fresh
repository. To change the author information for the last commit:

```bash
# git config --local user.name 'Mohammad Rahimi'
# git config --local user.email 'rahimi.mhmmd@outlook.com'

git commit --amend --reset-author
```

Let's explore a few other useful commands related to the *Working Tree* and
*Index*. We'll make some changes and stage them before committing:

```bash
echo 'Wolf Blitzer' >> employees.md
git add -A
git status
```

To remove files from the Staging area:

```bash
git restore --staged employees.md
git status

# to remove everything
git restore --staged .
```

The command above does not alter the *Working Tree*. To undo changes in the
*Working Tree*:

```bash
git restore employees.md
git status

# to undo everything
git restore .
```

You can undo a commit, and I'll cover that in Part 5.

For now, let's make another change to the *employees.md*:

```bash
echo 'Anderson Cooper' >> employees.md
git add -A
echo 'John King' >> employees.md
```

To view the difference between the *Working Tree* and the *Index*:

```bash
git diff
```

To see the difference between the *Working Tree* and the last commit:

```bash
git diff HEAD
```

*HEAD* typically points to a branch, and a branch is also a pointer to a commit.
We'll delve into this further in
[Part 2]({{ site.baseurl }}{% link _posts/2024-05-15-eat-git-part-2.md %}).

To see the difference between the Index and the last commit:

```bash
git diff --staged
```

## Branches

Imagine your work is in a stable state and customers are happy with it. However,
you have some ideas about improving certain parts of your work. You don't want
to compromise the stable code. In Git, you can create a branch from a commit and
continue committing your changes in that branch. This new branch is usually
called a feature branch.

Each branch is created from a commit or another branch. To properly demonstrate
branches, let's make more commits to our project.

```bash
echo 'Anderson Cooper' >> employees.md
git add -A
git commit -m 'Add Cooper'

echo 'John King' >> employees.md
git add -A
git commit -m 'Add King'

git log --oneline --all --graph
# * 6139f8f (HEAD -> main) Add King
# * 68d9628 Add Cooper
# * 8c61dd6 Add Berman and Bash
```

To create a branch from another branch:

```bash
# to create feat1 from main
git checkout -b feat1 main
```

To create a branch from a commit:

```bash
# to create feat2 from second commit
git checkout -b feat2 68d9628
```

Lets make a commit on each:

```bash
git checkout feat1
echo 'Abby Phillip' >> employees.md
git add -A
git commit -m 'Add Phillip'
# dc51eef474daf310773d6a4181f14dee34db0444

git checkout feat2
echo 'Jake Tapper' >> employees.md
git add -A
git commit -m 'Add Tapper'
# a37e3d4bcbf80a3a00a450385b804d3bb610de48
```

Let's take a look at the history:

```bash
git log --oneline --all --graph
# * a37e3d4 (HEAD -> feat2) Add Tapper
# | * dc51eef (feat1) Add Phillip
# | * 6139f8f (main) Add King
# |/  
# * 68d9628 Add Cooper
# * 8c61dd6 Add Berman and Bash
```

If we run `git diff feat2 feat1`, we can see the changes required to go from the
*feat2* branch to the *feat1* branch, indicating the differences between them.

```diff
diff --git a/employees.md b/employees.md
index 1785192..7659b36 100644
--- a/employees.md
+++ b/employees.md
@@ -1,4 +1,5 @@
 John Berman
 Dana Bash
 Anderson Cooper
-Jake Tapper
+John King
+Abby Phillip
```

In [Part 3]({{ site.baseurl }}{% link _posts/2024-05-19-eat-git-part-3.md %})
and Part 4, we will explore how to share work from one branch to another branch
within the same Git repository or in someone else's repository. For now, we have
learned how to create branches, commit on them, switch between them, and compare
them.

## History

The main purpose of version controlling a directory is to maintain a history,
serving as a reference, documentation, and even a debugging tool. This history
helps you pinpoint the exact changes that caused an anomaly.

The command to access the history in Git is `git log`. To demonstrate the full
potential of this command, we need a project with a rich history. To download
such a project, or in Git terms, to clone a project:

```bash
git clone https://github.com/CorentinTh/it-tools.git
cd it-tools
```

The basic form of `git log` lists commit hashes, authors, timestamps, and full
commit messages. However, sometimes you only need a quick overview. In such
cases, you can use `git log --oneline`, which displays abbreviated commit hashes
and short commit messages. Alternatively, if you want to see the history
including the changes themselves, you can use `git log --patch`.


This command is a powerful tool. To customize the output of `git log`:

```bash
# to limit the output to last 10 commits
git log -10

# to list files that were changed by commits
git log --name-only

# to list commits that changed some files
git log -- filename1 filename2

# to search among commit messages
git log --grep 'keyword'

# to list commits that changed occurrences of an expression
git log -S 'expression'

# to list commits that changed inside a function
git log -L :function:file

# to include all branches
git log --all --graph --oneline

# to list commits since last week
git log --since="one week ago"
```

I frequently use the following command:

```bash
git log\
 --pretty=format:"%C(Red)%d %C(Yellow)%h %C(Cyan)%ch %C(Green)%cn %C(White)%s"\
 --graph --all
```

However, it's quite lengthy, so I've added an alias for it in Git. To add
aliases to Git commands:

```bash
git config --global alias.clog 'log --pretty=format:"%C(Red)%d %C(Yellow)%h %C(Cyan)%ch %C(Green)%cn %C(White)%s" --graph --all'
```

Now we can run the following command to get a colorful history:

```bash
git clog
```

Let me wrap this part up with a revisit to `git diff` command:

```bash
# to diff against any commit
git diff abc123

# to list files that were changed between two commits
git diff abc123 def456 --name-only

# to diff the feat branch against its common base on main
git diff main...feat
```

You can instruct Git to use an external difftool for comparing files. Some
options include Meld, Vimdiff, and P4Merge. Here is the original `git diff`
output:

```diff
diff --git a/.github/workflows/docker-nightly-release.yml b/.github/workflows/docker-nightly-release.yml
index 81a0898..41dbb15 100644
--- a/.github/workflows/docker-nightly-release.yml
+++ b/.github/workflows/docker-nightly-release.yml
@@ -32,7 +32,7 @@ jobs:
       - run: corepack enable
       - uses: actions/setup-node@v3
         with:
-          node-version: 16
+          node-version: 20
           cache: 'pnpm'
 
       - name: Install dependencies
```

In a diff, the first few lines determine the sides being compared. When you use
`git diff abc123 def456`, you're comparing *def456* against *abc123*. On the "a"
side, you'll have *abc123*, and on the "b" side, you'll have *def456*.

If a file was deleted, you would see:

```diff
--- a/filename
+++ /dev/null
```

If a file was renamed, you would see the new name. The `@@ -32,7 +32,7 @@ jobs:`
part indicates the location of the change in the file. Here, it means at line 32
, in the `jobs:` block, starting with `- run: corepack enable`, we had 7 lines
of code. After the change, also at the same location, we still have 7 lines of
code. The `jobs:` in `@@ -32,7 +32,7 @@ jobs:` is intended to be the name of the
function or block to which the changes belong and can span many lines before the
`- run: corepack enable`. Git guesses this part based on indentation, and it is
not always accurate. The lines containing the "+" and "-" symbols show what is
removed and what is added.

You can create a patch file by redirecting the output of `git diff` into a file.
These patch files become important in
[Part 3]({{ site.baseurl }}{% link _posts/2024-05-19-eat-git-part-3.md %}) of
this tutorial.

```bash
# creating a patch file from the Working Tree
git diff HEAD > changes.patch
```

## Summary

In this part, we created a directory, performed some work in it, initiated
version control, learned how to stage files, and create commits and work on
branches. We also learned how to review the history of a project and observe
changes in each step. These are the basics of Git. In
[Part 2]({{ site.baseurl }}{% link _posts/2024-05-15-eat-git-part-2.md %}),
we'll delve into the inner workings and explore Git data structures to
understand what you can find inside a *.git* directory.

[download-for-windows]: https://git-scm.com/download/win
[main-git-repo]: https://git.kernel.org/pub/scm/git/git.git/
[github-git-repo]: https://github.com/git/git
[pro-git-book]: https://git-scm.com/book/en/v2
[commit-message-guide]: https://cbea.ms/git-commit/
[git-lens]: https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens
[git-ignore-collection]: https://github.com/github/gitignore
[git-reference]: https://git-scm.com/docs
