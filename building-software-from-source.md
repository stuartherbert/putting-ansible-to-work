---
layout: top-level
title: Building Software From Source
prev: '<a href="installing-software-from-packages.html">Prev: Installing Software From Packages</a>'
next: '<a href="temporary-install-scripts.html">Next: Using Temporary Install Scripts</a>'
---

# Building Software From Source

## When Should You Build From Source?

There are four common scenarios where you'll want to build software from source.

* If there's no pre-compiled package available for the operating system, it's a good idea to build the software from source yourself.
* It's also one way to install software that you've customised in some way.  Most software apps can be compiled with a large array of optional features, and you may find that you need an optional feature that isn't enabled in a pre-built package.
* If you need to use the same software across multiple operating systems (such as Linux servers and Apple laptops), building the software yourself might be the only way to get the same versions of the software everywhere you use it.  Version mismatches between development and production environments can be a source of costly bugs.
* Your own software may only be available in source form.

<div class="callout danger" markdown="1">
#### Problems In Production Environments

Experienced system administrators are rightly very reluctant to build anything from source on a production server.

* Building from source takes time and eats CPU and memory.  Depending on the spec of the server, the server is unable to do its normal work whilst building is going on.
* The tools required to build from source can be useful to anyone who breaks into a server.  Leaving them off the box makes a hacker's job at least a little harder once they're in.

Do you have a lot of servers to install software onto?  Consider building the software once in a development environment, and then turning it into a Dpkg or RPM package to install into your production environment.  The Fedora project has [a draft RPM manual](http://docs.fedoraproject.org/en-US/Fedora_Draft_Documentation/0.1/html/RPM_Guide/) to get you started with building RPMs (look at [Mock](https://fedoraproject.org/wiki/Projects/Mock) also!), and the Debian Handbook includes [a chapter on how to create your own Dpkgs](http://debian-handbook.info/browse/stable/debian-packaging.html).
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

* Compilers (C, C++), runtime engines (scripting languages), software development kit (Java and JVM languages)
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
# file: plays/redis-server.yml
- hosts: redis-servers
  roles:
  - stuartherbert.gcc
  - stuartherbert.make
  - stuartherbert.redis-server
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

A _tarball_ is a single file (in UNIX tar format) that you can download to the target computer.  It is often compressed using GZip, BZip2 or the newer LZMA/Xz utils.  It expands to one or more source files when untarred.  It's the equivalent of a Windows ZIP or RAR file.

Use the [get_url module](http://docs.ansible.com/get_url_module.html) to download the source code to the target computer's `/tmp` folder:

{% raw %}

<pre>
---
# file: roles/stuartherbert.redis-server/tasks/install.yml

- name: Download Redis source code
  get_url:
    url: "http://download.redis.io/releases/redis-{{redis_server_version}}.tar.gz"
    dest: "/tmp/redis-{{redis_server_version}}.tar.gz"

- name: Unpack the Redis source code
  command: tar -zxf /tmp/redis-{{redis_server_version}}.tar.gz creates=/tmp/redis-{{redis_server_version}} chdir=/tmp
</pre>

Don't forget to set default values for any variables that you use in your tasks:

<pre>
---
# file: roles/stuartherbert.redis-server/defaults/main.yml
redis_server_version: 2.8.4
</pre>

If the file has already been downloaded, Ansible's get_url module won't attempt to download it again.  This can be a problem if the download failed part-way through; you may need to log into the target computer and manually delete the file to force a second download attempt.

The same goes the command to unpack the code from the tarball.  I'm using the `creates` parameter to the [command module](http://docs.ansible.com/command_module.html) so that we only expand the tarball if the code hasn't yet been unpacked.  If the unpacking fails part-way through (for example, you filled up `/tmp` during the unpacking), you may need to log into the target computer and manually delete the unpacked code before running the playbook again.

Did you spot my use of the `chdir` parameter to the command module?  This tells Ansible to `cd` to the `/tmp` folder before running the `tar` command.  It's a very handy way of making sure that the unpacked code appears where I expect it to.

<div class="callout info" markdown="1">
#### Don't Hard Code Versions Into Your Tasks

Instead of hard-coding the version number of Redis that I want into the task, I've made a variable called `redis_server_version`.  This allows me to change the version of Redis on the target computer simply by setting a _host_var_ in the Inventory.

I can also have different versions of Redis on different target computers if I need to (for example, for testing out a newer release of Redis before I upgrade my production machines).
</div>

<div class="callout warning" markdown="1">
#### Why Didn't I Use The unarchive Module?

Ansible has a dedicated module for unpacking archives: the [unarchive module](http://docs.ansible.com/unarchive_module.html).  Normally, it's better to use an Ansible module instead of running shell commands on the target computer, because it makes the tasks clearer to read and we get the benefit of better error handling.  So why didn't I use it in this case?

The unarchive module uploads an archive from my computer (where Ansible is running) to the target computer.  For that to work, I need:

* a copy of the tarball on my local computer, and
* sufficient bandwidth to perform the upload

__I don't want a copy of the tarball in my playbook.__  If I add that to my Git repo, it's going to dramatically increase the size of my Git repo and make future Git clones too slow.  Now, I could avoid storing the tarball in my plabook by using the [local_action:](how-tasks-work.html#local_action:) statement to download the tarball to my computer when I run the play.  However, then I'd run into the other problem with using Ansible's unarchive module.

The second reason is that __it is much quicker to download the file than to upload it__.  In my case, my computer has 20 Mbit/sec (at home) or 100 Mbit/sec (at the office) in upload bandwidth, and it has to share that with other users.  A large upload is slower, even more so if I'm using a decent VPN to connect into our datacentre.  A large upload can also inconvenience other users of my internet connection. The target computers have anything from 80 Mbit/sec (at home) to several gigabits/sec (at our datacentre) in download bandwidth.  It's nearly always much quicker to download onto the target computer.

There is one downside that's worth mentioning.  If I was provisioning (say) 10 or more new computers at the same time, that's a lot of duplicate downloads putting avoidable strain onto the web sites that host the tarballs.  In that scenario, I'd [setup Ansible to use a HTTP proxy server](http://docs.ansible.com/playbooks_environment.html) for all of the downloads.
</div>

### Downloading From Source Control

Sometimes, there's no tarball available for the source code, and you have to checkout the code from source control.  Ansible supports [several source control systems](http://docs.ansible.com/list_of_source_control_modules.html), so you should have no trouble getting the code you need.

Use the [git module](http://docs.ansible.com/git_module.html) to clone the source code from GitHub:

<pre>
---
# file: roles/stuartherbert.redis-server/tasks/install.yml

- name: Download Redis source code
  git:
    repo: "https://github.com/antirez/redis.git"
    dest: "/tmp/redis-unstable"
    version: "origin/unstable"
</pre>

This tasks creates the folder `/tmp/redis-unstable` on the target computer, which contains the equivalent of:

<pre>
cd /tmp
git clone --recursive https://github.com/antirez/redis.git redis-unstable
cd redis-unstable
git checkout origin/unstable
</pre>

<div class="callout warning" markdown="1">
#### Updating An Existing Cloned Repo

If you run the example task a second time, Ansible's git module will not create a fresh clone of the upstream repo (because that could be a very slow operation).  Instead, it downloads any changes that have been pushed upstream.  This can cause problems if you are relying on those changes.

At the time of writing, the git module does the equivalent of:

<pre>
cd /tmp/redis-unstable
git fetch
git fetch --tags
</pre>

when updating an existing Git repo on the target computer.  The `git fetch` and `git fetch --tags` will pull down any changes, but without a `git merge` those changes will not appear in any branches that you have checked out.

The way to get around this is to always checkout branches starting `origin/`.  (This is a trick I've been using with Jenkins CI for a couple of years, for the same reason).

This only applies to building from branches. If you're building from a tag, there's nothing to worry about.
</div>

<div class="callout info" markdown="1">
#### Add The accept_hostkey: Param When Cloning From GitHub Using SSH

If you are cloning from GitHub using SSH (something you'd do when cloning a private repo), make sure that you add the `accept_hostkey:` parameter:

<pre>
---
# file: roles/stuartherbert.redis-server/tasks/install.yml

- name: Download Redis source code
  git:
    repo: "ssh:git@github.com:antirez/redis.git"
    dest: "/tmp/redis-unstable"
    version: "origin/unstable"
    accept_hostkey: "yes"
</pre>

This tells Ansible's git module to disable strict SSH hostkey checking.  Without it, if your target computer has never connected to GitHub's servers via SSH before, the target computer's SSH client will reject the connection and cause the task as a whole to fail.
</div>

## Step 3: Compiling Code

Compiling the code is the easy bit (at least from your role's point of view):

<pre>
---
# file: roles/stuartherbert.redis-server/tasks/install.yml

- name: compile Redis
  command: make -j{{ansible_processor_vcpus}} chdir=/tmp/redis-{{redis_server_version}}
</pre>

Don't forget to call `configure` first if the software you're building uses GNU Autotools.

<div class="callout info" markdown="1">
#### Dealing With Interactive Compilation

Sometimes, you'll come across a piece of software that prompts for information, normally during the `configure`.  After cursing at the software's author for their attempt to thwart automation, you'll need to automate your way around the problem.

1. In your role's `files/` folder, create a file called `input.txt`.  Fill it with the information that is prompted for, one answer per line of text.
2. Use Ansible's [copy module](http://docs.ansible.com/copy_module.html) to upload the `files/input.txt` file into the `/tmp` folder.
2. Use Ansible's [shell module](http://docs.ansible.com/shell_module.html) to compile your software, using the uploaded `input.txt` file as the input.

To show you what I mean, here's an example for compiling PHP's YAML extension:

<pre>


</pre>
The strange-looking empty file above actually contains two blank lines.  It is saved as `roles/stuartherbert.php-yaml/files/input.txt` on disk.

<pre>
---
# file: roles/stuartherbert.php-yaml/tasks/install.yml

- name: copy YAML input
  copy:
    src: input.txt
    dest: /tmp/php-yaml-input.txt
    mode: 0600

- name: install YAML pecl module
  shell: pecl install yaml &lt; /tmp/php-yaml-input.txt
  sudo: True
</pre>

This should work most of the time.  You'll only have problems if the software deliberately empties the input buffer before prompting.  If that happens, you'll have to consider patching the software's build process, or perhaps filing a bug with the software's author which explains how their approach prevents an automated install.  Be aware that many software authors have no experience of automation, and might need some help in understanding the benefits of the changes you're asking for.
</div>

<div class="callout info" markdown="1">
#### Clean Before You Build

Most software supports the `make clean` or `make distclean` commands.  They delete any files that you've previously compiled, so that any recompile is much more likely to pick up and compile any files that have changed since the last time you compiled the software on the target computer.  (This is better than trusting the author of the `Makefile` to have got his dependencies 100% right).

<pre>
- name: remove any previous Redis compile
  command: make clean chdir=/tmp/redis-{{redis_server_version}}
</pre>

</div>

<div class="callout warning" markdown="1">
#### Don't Make-bomb Your Target Computer

You'll understandably want the compile to be over and done with as quickly as possible.  However, don't make this mistake:

<pre>
- name: compile Redis
  command: make -j chdir=/tmp/redis-{{redis_server_version}}
</pre>

`make -j` tells the `make` tool to compile as many files in parallel as possible.  Depending on the code that you're compiling and the computer that you're doing the compiling on, this can spawn too many copies of the compiler at once and bring the target computer to a standstill.

Always use `make -j {{ansible_processor_vcpus}}` to tell `make` run a maximum of one compiler per CPU.  This will take into account Intel's hyperthreading and emulated CPUs inside virtual machines.
</div>

<div class="callout warning" markdown="1">
#### Do Not Build As 'root'

You may not have noticed, but none of the download and compile tasks contain any `sudo: true` statements.

It is good practice to never ever __ever__ compile software as the `root` user.  In fact, you should do as little as possible as the `root` user.  That way, if there are any surprises in the software's Makefiles or build scripts, the damage done is a lot less than it might otherwise be.
</div>

## Step 4: Installing Compiled Software

Most software comes with a `make install` command (or equivalent) that you can run:

<pre>
- name: install Redis
  command: make install chdir=/tmp/redis-{{redis_server_version}}
  sudo: True
</pre>

You'll have to run this stage using `sudo: true` so that the `make` command runs as `root`.

<div class="callout info" markdown="1">
#### Removing Old Versions Of Compiled Software

At the time of writing, Ansible and its major competitors cannot automatically remove old versions of compiled software for you.  When compiling software from source, new versions are installed over the top of any existing versions of software.  This can sometimes cause problems with dynamic libraries and with old C/C++ header files.

If you need to remove a piece of software for some reason, you've got two choices:

1. write tasks to remove the software file by file and folder by folder, or
2. nuke the target computer and re-install everything from scratch

Option 2 is the preferred way of doing things, and (because you've already automated everything) is much less effort than option 1.  Complete re-installs can, however, cause utter carnage if someone has forgotten to automate everything that's on the target computer.

If option 2 really isn't the way for you, then instead of using Ansible to compile software, you need to create your own pre-built packages for the software.  Then, you can use your operating system's package manager to remove old versions, something it will do automatically when you install newer versions of the packages that you've built.
</div>

## Step 5: Adding Startup Scripts

If you are building software that runs as a UNIX service / daemon, then you will need to install startup / shutdown scripts onto the target computer.  You will also need to enable them for the correct runlevels.

In the old days, these were simple shell scripts that lived in the `/etc/init.d/` folder, and symlinked into `/etc/rc.X` folders (where X is a number) to enable and disable them.  At the time of writing, things are no longer so simple.  There are currently almost as many different ways to do startup / shutdown scripts as there are Linux distributions, and some Linux distros currently support more than one way.  Other operating systems have their own approaches as well.

For each operating system that you're supporting in your Ansible playbook, you'll need to learn how startup scripts work on that operating system.  Very few software authors provide working startup scripts; you'll probably have to write your own.

When you've written the startup scripts, store them in your role's `files/` folder, and use the [copy module](http://docs.ansible.com/copy_module.html) to upload the startup scripts into the right place.

<pre>
---
# file: roles/stuartherbert.redis-server/tasks/install.yml

- name: install Redis server startup script
  copy:
    src: redis.sh
    dest: /etc/init.d/redis
    owner: root
    group: root
    mode: 0644
  sudo: True
</pre>

Always explicitly set the `owner:`, `group:` and `mode:` of the startup script.

## Recap: The Completed Role

Here's the list of files and folders for the completed role:

<pre>
roles/
  stuartherbert.redis-server/
    defaults/
      main.yml

    files/
      redis.sh

    tasks/
      install.yml
      main.yml
</pre>

... and here are the contents of those files:

<pre>
---
# file: roles/stuartherbert.redis-server/defaults/main.yml
redis_server_version: 2.8.4
</pre>

<pre>
---
# file: roles/stuartherbert.redis-server/tasks/main.yml
- include: install.yml
</pre>

<pre>
---
# file: roles/stuartherbert.redis-server/tasks/install.yml

- name: Download Redis source code
  get_url:
    url: "http://download.redis.io/releases/redis-{{redis_server_version}}.tar.gz"
    dest: "/tmp/redis-{{redis_server_version}}.tar.gz"

- name: Unpack the Redis source code
  command: tar -zxf /tmp/redis-{{redis_server_version}}.tar.gz creates=/tmp/redis-{{redis_server_version}} chdir=/tmp

- name: remove any previous Redis compile
  command: make clean chdir=/tmp/redis-{{redis_server_version}}

- name: compile Redis
  command: make -j{{ansible_processor_vcpus}} chdir=/tmp/redis-{{redis_server_version}}

- name: install Redis
  command: make install chdir=/tmp/redis-{{redis_server_version}}
  sudo: True

- name: install Redis server startup script
  copy:
    src: redis.sh
    dest: /etc/init.d/redis
    owner: root
    group: root
    mode: 0644
  sudo: True
</pre>

## What's Next

There are several ways that we can improve this role:

* We have to make sure that every play that includes this role also includes a complete list of all the tools that we need to compile this software from source.  That's an error-prone process.  It would be a lot better if this role included a list of tools that it needs.  I'll show you how to do that in [Adding Dependencies To Roles](adding-dependencies-to-roles.html).
* The role is going to download, compile and install the software every time the Ansible playbook runs.  It will save a lot of time if we only do the install when Redis is not successfully installed.  I'll show you how to do that in [Making Roles Repeatable](making-roles-repeatable.html).
* We haven't installed any config files for the software.  The software may refuse to start without one.  I'll show you how to do that in [Working With Config Files](working-with-config-files.html).
* If we're installing a new version of the software, or a new version of its config file, the software needs restarting to pick up those changes.  I'll show you how to do that in [Restarting Services](restarting-services.html).