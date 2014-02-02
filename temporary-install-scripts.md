---
layout: top-level
title: Using Temporary Install Scripts
prev: '<a href="building-software-from-source.html">Prev: Building Software From Source</a>'
next: '<a href="working-with-config-files.html">Next: Working With Config Files</a>'
---

# Using Temporary Install Scripts

There'll be times when you get frustrated with Ansible and how it does things. Uploading and running a script to perform a task gets you past that frustration.

## What Is A Temporary Install Script?

A temporary install script is just a plain old UNIX shell script that you upload and run on the target computer.  It contains all of the instructions required to download, compile, and install a piece of software.

## Why Use A Temporary Install Script?

The main advantage of temporary install scripts is that they use skills you're already comfortable with.

* You already know how to write shell scripts.
* You already know how to install software from the command-line.

It's very quick and easy to combine this skills to knock up a script that gets the job done.  Ansible also makes it easy to upload and run these scripts on the target computer.

## Creating A Script

Creating a temporary install script is easy.  Install the software by hand from the command-line, then use the `history` command to see what you did.

If we take the role from the [Building Software From Source](building-software-from-source.html) chapter and replace it with a temporary install script, we get something like this:

<pre>
#!/bin/bash
#
# file: roles/stuartherbert/redis-server-by-script/files/install.sh

REDIS_VERSION=${1:-2.8.4}
cd /tmp
wget http://download.redis.io/releases/redis-$REDIS_VERSION.tar.gz
tar -zxf redis-$REDIS_VERSION.tar.gz
cd redis-$REDIS_VERSION
make -j `nproc`
sudo make install
</pre>

The script works (tested on Ubuntu 13.10), and is certainly a lot shorter (and a lot less effort) than the equivalent tasks in Ansible.

## Running The Script Through Ansible

Use Ansible's [script module](http://docs.ansible.com/script_module.html) to upload and execute the script in a single statement:

<pre>
---
# file: tasks/stuartherbert.nodejs-by-script/tasks/main.yml

- name: run NodeJS install script
  script: install-nodejs.sh
  sudo: True
</pre>

## Don't Do This!

Yes, I've just shown you how to do it - now I'm telling you that you shouldn't.  Why?  Let's start by looking at the example script once more:

<pre>
#!/bin/bash
#
# file: roles/stuartherbert/redis-server-by-script/files/install.sh

REDIS_VERSION=${1:-2.8.4}
cd /tmp
wget http://download.redis.io/releases/redis-$REDIS_VERSION.tar.gz
tar -zxf redis-$REDIS_VERSION.tar.gz
cd redis-$REDIS_VERSION
make -j `nproc`
sudo make install
</pre>

There are several problems with this script that you don't run into when you use Ansible tasks to accomplish the same thing.

1. _Error handling:_ the example has absolutely no error checking in it at all.  The script just goes from one command to the next, assuming that everything is working just fine.  What happens if the download fails?  What happens if the tarball cannot be unpacked?  What happens if the compilation fails?

   With Ansible, we get error checking at each step of the way.  If a step fails, Ansible stops.  There's no risk of an error cascading, of unexpected damage being done to the target computer.  That will save you a lot of trouble one day.

1. _sudo handling:_ the example uses the `sudo` command inside the script.  You're forced to use password-less `sudo` everywhere that you want this role to work.  You're forced to compromise security for convenience.

   If you still want to use the script, the alternative is to tell Ansible to run the script as `root`.  This greatly increases the amount of work being done with full `root` privileges.  Any bugs, mistakes or other surprises have the potential to be much more costly.

   With Ansible, you can control what runs as `root` and what runs as the unprivileged remote user.  We can keep to a minimum what we do with full sysadmin privileges.  Ansible handles the `sudo` password prompt for us, so that we don't have to resort to using password-less `sudo` everywhere.

1. _you still have to use Ansible to do everything else:_ things like [uploading config files](working-with-config-files.html), [restarting services](restarting-services.html), [making roles repeatable](making-roles-repeatable.html), [adding role dependences](adding-role-dependencies.html) and adding [support for multiple operating systems](multiple-operating-systems.html) all have to be done using Ansible anyway.  You've still got to learn most of Ansible in order to deliver working systems automation.

I've noticed that new Ansible users find it very tempting to write temporary install scripts whenever they need to compile and install software from source code.  Learning how to do this entirely in Ansible takes time, and isn't obvious at first (although the chapter on [building software from source](building-software-from-source.html) in this book should help a lot!).

If you use Ansible's built-in modules along with my [three-point plan for roles](planning-a-role.html), you'll have an automated deployment process that's reliable and repeatable.