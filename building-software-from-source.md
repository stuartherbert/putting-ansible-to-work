---
layout: top-level
title: Building Software From Source
prev: '<a href="installing-software-from-packages.html">Prev: Installing Software From Packages</a>'
next: '<a href="temporary-install-scripts.html">Next: Using Temporary Install Scripts</a>'
---

# Building Software From Source

## When Should You Build From Source?

There are four common scenarios where you'll need to build software from source.

* If there's no package available for the operating system, it's a good idea to build the software from source yourself.
* It's also the best way to install software that you've customised in some way.  Most software apps can be compiled with a large array of optional features, and you may find that you need an optional feature that isn't enabled in a pre-built package.
* If you need to use the same software across multiple operating systems (such as Linux servers and Apple laptops), building the software yourself might be the only way to get the same versions of the software everywhere you use it.  Version mismatches between development and production environments can be a source of costly bugs.
* Your own software may only be available in source form.

<div class="callout danger" markdown="1">
#### Problems In Production Environments

Experienced system administrators are rightly very reluctant to build anything from source on a production server.

* Building from source takes time and eats CPU and memory.  Depending on the spec of the server, the server is unable to do its normal work whilst building is going on.
* The tools required to build from source can be useful to anyone who breaks into a server.  Leaving them off the box makes a hacker's job at least a little harder once they're in.

Do you have a lot of servers to install software onto?  Consider building the software once in a development environment, and then turning it into a Dpkg or RPM package to install into your production environment.  The Fedora project has [a draft RPM manual](http://docs.fedoraproject.org/en-US/Fedora_Draft_Documentation/0.1/html/RPM_Guide/) to get you started with RPM, and the Debian Handbook includes [a chapter on how to create your own Dpkgs](http://debian-handbook.info/browse/stable/debian-packaging.html).
</div>

## The Build Process

You can build the vast majority of software from source using these steps:

1. Install the tools needed.
1. Download the source code for the software, and unpack it if required.
1. Compile the code (if required).
1. Install the software.
1. Install startup scripts.

## Step 1: Installing Tools

You're going to need the right tools to build software from source.  These include:

* Compilers (C, C++), runtime engines (scripting languages), development kit (Java and JVM languages)
* Build tools such as `autotools` and `make` (C, C++) or `maven` (Java et al)
* Software libraries that the software you're building links against (C, C++ again)
* Extensions and modules that the software you're building uses when it runs (dynamic languages)

Some of these, in turn, will have their own dependencies that you need to install too.

<div class="callout info" markdown="1">
#### Which Tools Do You Need?

It's beyond the scope of this book to tell you which tools you need to build which software - there's simply too much software out there for me to cover here.  Most software includes a _README_ or _INSTALL_ file which tells you what you need (or they've published the information on their website).  Most software authors are also willing to answer questions too.

When I get really stuck, I go and take a look at [packages.gentoo.org](http://packages.gentoo.org).  Gentoo Linux is a Linux distro where everything is built from source - which means that their source control repo is full of instructions on how to compile software.  Gentoo Linux tends to be a lot more up to date than the leading Linux distros, and the chances are high that they have build instructions for the latest version of whatever software you want to build from source.

[Arch Linux's packages](https://www.archlinux.org/packages/) are another good source of working build instructions.
</div>

Create Ansible roles for each of the tools that you need.  Add the roles to your plays, so that they are executed before the role that needs them:

<pre>
---
# file: plays/nodejs-server.yml
- hosts: nodejs-servers
  roles:
  - stuartherbert.gcc
  - stuartherbert.make
  - stuartherbert.libzmq
  - stuartherbert.nodejs
</pre>

### Installing Common Tools In One Go

Both Debian-based (such as Ubuntu) and RedHat-based (such as CentOS and Fedora) Linux distros have a way of installing common compilers and build tools in a single go:

<pre>
---
# file: roles/stuartherbert.build-tools/tasks/redhat_install.yml
- name: "RPMs: install common build tools"
  shell: yum -y groupinstall "Development Tools"
  sudo: True
</pre>

<pre>
# file: roles/stuartherbert.build-tools/tasks/ubuntu_install.yml
- name: "DPKGs: install common build tools"
  apt:
    pkg: build-essential
    state: latest
  sudo: True
</pre>

## Step 2: Downloading Source Code

Source code tends to come in two flavours - a downloadable zip/tarball, or a downloadable source-control repository.

### Downloading A Tarball

A _tarball_ is a single file (in UNIX tar format) that you can download to the target computer.  It is often compressed using GZip, BZip2 or the newer Xz utils.  It expands to one or more source files when untarred.  It's the equivalent of a Windows ZIP or RAR file.

<pre>
---
# file: roles/stuartherbert.nodejs/tasks/ubuntu_install.yml
</pre>

### Downloading From Source Control

## Step 3: Compiling Code

## Step 4: Installing Compiled Software

## Step 5: Adding Startup Scripts

## Dealing With Downloadable Installers

No chapter on building software from source would be complete without looking at a recent trend - downloadable installers.  This is where you download an installer from the author's website, and run it