---
layout: post
title: "Eat Git - Part 1"
categories: git
author:
- Mohammad Rahimi
---

## Preface

This tutorial on Git has five parts, and this is the first. The following shows
discussed material in each part:

1. Getting to Know Git
    - Creating Repositories
    - Working Tree and Index
    - Commits History
    - Search and Compare Tools
2. Behind the Scene
    - Git Internals
3. Local Workflows
    - Debugging with Git
    - Patch Files
    - Cherry Picking
    - Interactive Rebase
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

In the first part, we get to know Git and become comfortable using it. If you
are curious and want to know why everyone is talking about Git, this part should
be enough.

In the second part, I will explain how Git creates and stores the data
structures that enable it to do what it does. This part, in my opinion, is
necessary to become confident at using other tools introduced later.

In part 3, I will wrap up most of the things you need to know to use Git
effectively. Although Branches are useful even in a solo developer setting,
their introduction deferred to the next part.

In part 4, I will introduce Branches. They are essential if you are part of a
team working on a shared code-base. If you have been working with Git but want
to understand the utilization of what you know in a collaborative environment,
this is the part you need to read.

The last part is where I share my experience with you. In my previous two
positions, I introduced my colleagues to Git, and then we started using it in
our day-to-day development. This part sums up the mistakes that me and my
colleagues used to make and show how to fix things when something goes wrong.

### Why another Git tutorial?

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

## Installation

The best way to install Git is to head to
[Git Downloads](https://git-scm.com/downloads).

To install Git on Ubuntu:

```bash
sudo apt-get install git
```

To install on Mac:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
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

### Build from Source (Optional)

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
git clone --recurse-submodules --depth=1 https://git.kernel.org/pub/scm/git/git.git
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
can have a specific time-point, checked out.

Now, let's create a project and start working on it. In this initial example,
I'll create something meaningful.

```bash
```

[download-for-windows]: https://git-scm.com/download/win
[main-git-repo]: https://git.kernel.org/pub/scm/git/git.git/
[github-git-repo]: https://github.com/git/git
[pro-git-book]: https://git-scm.com/book/en/v2
