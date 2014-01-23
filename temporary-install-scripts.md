---
layout: top-level
title: Using Temporary Install Scripts
prev: '<a href="building-software-from-source.html">Prev: Building Software From Source</a>'
next: '<a href="working-with-config-files.html">Next: Working With Config Files</a>'
---

# Using Temporary Install Scripts

There'll be times when you get frustrated with Ansible and how it does things. Uploading and running a script to perform a task gets you past that frustration.

## What Is A Temporary Install Script?

A temporary install script is just a plain old UNIX shell script that you upload and run on the target computer.

## Why Use A Temporary Install Script?

I've noticed that new Ansible users will start writing temporary install scripts whenever they need to compile and install software from source code.  Learning how to do this entirely in Ansible takes time, and isn't obvious at first (although the chapter on [building software from source](building-software-from-source.html) in this book should help a lot!).

## Creating A Script

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

Yes, I've just shown you how to do it - now I'm telling you that you shouldn't.  Why?  It's a bad habit to get into.

* The more you resort to temporary install scripts, the less you're building up your Ansible skills. You've got to put the work in to get productive with Ansible.  Otherwise, what's the point of using Ansible at all?
* With Ansible, you can control what runs as `root`, and what runs as the unprivileged remote user.  With a temporary install script, you'll probably end up running all of it as `root`.  A buggy install script can do a lot of damage if it goes wrong.
* In your install script, how will you tell it which version of the software to install?  How will you manage installing all of the dependencies?
* Ansible's roles (used wisely!) give you a library that can quickly and easily be adapted and extended as your deployment needs change.  Temporary install scripts tend to end up being written without flexibility and modularity in mind.

If you use Ansible's built-in modules along with my [three-point plan for roles](planning-a-role.html), you'll have an automated deployment process that's reliable and repeatable.