---
layout: top-level
title: Planning A Role
prev: '<a href="organising-your-ansible-files.html">Prev: Organising Your Ansible Files</a>'
next: '<a href="installing-software.html">Next: Installing Software Using Ansible</a>'
---

# Planning A Role

A role tells Ansible how to make a change to the target computer.  Make every role follow my simple three-point plan to create roles that are robust and repeatable.

## What Does A Role Contain?

A role can contain any of:

* _files_ to be uploaded to the target computer
* _handlers_ to restart services after tasks have executed
* _metadata_ to list other roles that need to run before the current one
* _tasks_ to provision software and otherwise make changes to the target computer
* _templates_ to be expanded using Jinja2 and then uploaded to the target computer

You only need

* _files_ and _templates_ when [working with config files](working-with-config-files.html)
* _handlers_ when [restarting services](restarting-services.html)
* _metadata_ when [your role needs other roles to have run first](adding-dependencies-to-roles.html)
* _tasks_ when [installing software](installing-software.html)

## How Small Should Each Role Be?

Very small.  As small as you can make each one.  Let me explain.

Each role should do a single job, and no more.  It might install a command-line utility, or it might build a software library from source code.  Create a separate role for each piece of software that you install using Ansible.

Here are some examples from [my ansible-playbooks repo on GitHub](https://github.com/stuartherbert/ansible-playbooks):

* git
* hubflow
* nodejs
* redis2-server

Each of these roles installs a single software package.  In the case of the _git_ role, it's very trivial, and hardly seems worth the effort.  But here's the thing: small roles are the easiest to reuse.

<div class="callout info" markdown="1">
#### Why Reuse Matters

Let's say I have a play called _webserver.yml_, for a virtual machine that I'm using to host my software on.  It starts off using both the _nodejs_ and _redis2-server_ roles.  As demand for my software grows, a single virtual machine isn't going to be powerful enough any more: I'm going to need to split the software across several computers.  This is very easy to do, because I've created small reusable roles.  All I need to do is to create two plays: _frontend.yml_ will use the _nodejs_ role, and _backend.yml_ will use the _redis2-server_ role.  I probably don't have to change the roles at all, if I've already used templates to generate the config files for my software.

If I'd created a single role called _website.yml_ instead of separate _nodejs_ and _redis2-server_ roles at the start, then splitting the software across several VMs would be a lot more work.  If scaling the site was very urgent, the last thing I'd want would be extra work whilst under pressure, especially work that could have been completely avoided in the first place.
</div>

Whatever plays you're creating today, tomorrow you will have to create more plays.  If each role installs just one thing, then you can reuse your roles in tomorrow's new plays with the minimum of effort.

Just as your code should be modular, so should the roles in your playbook.

## The Three-Point Plan For Every Role

Follow this three-point plan when you are building a role, and you'll never go wrong:

1. __Inspect:__ Does the role need to act?
1. __Install:__ Add software to the computer.
1. __Enable:__ Update configuration, start services.

When Ansible executes a role, it starts by loading the file `tasks/main.yml` inside the role's folder, and running the instructions it finds in there.  Put the instructions for each step into a separate YAML file, and include these files in `tasks/main.yml`.  This is the easiest way to skip the `tasks/install.yml` if the software is already installed.

Here is an example for a _libzmq4_ role, which installs [ZeroMQ](http://zeromq.org) by following the three-point plan:

{% raw %}

<pre>
---
# file: roles/linux/libzmq4/tasks/main.yml

- include: inspect.yml
- include: install.yml
  when: "libzmq4_installed.stdout != 'true'"
- include: enable.yml
</pre>

<pre>
---
# file: roles/linux/libzmq4/tasks/inspect.yml

- name: is libzmq already installed?
  action: shell executable=/bin/bash if [[ "`pkg-config --modversion libzmq`" == "{{zeromq_version}}" ]] ; then echo 'true' ; else echo 'false' ; fi
  sudo: False
  register: libzmq_installed
</pre>

<pre>
---
# file: roles/linux/libzmq4/tasks/install.yml

- name: download libzmq source code
  action: get_url url=http://download.zeromq.org/zeromq-{{zeromq_version}}.tar.gz dest=/tmp/zeromq-{{zeromq_version}}.tar.gz

- name: unpack libzmq source code
  action: command tar -C /tmp -zxf '/tmp/zeromq-{{zeromq_version}}.tar.gz' creates='/tmp/zeromq-{{zeromq_version}}'

- name: configure libzmq source code
  action: command ./configure chdir='/tmp/zeromq-{{zeromq_version}}'

- name: compile libzmq source code
  action: command make -j chdir='/tmp/zeromq-{{zeromq_version}}'

- name: install libzmq
  action: command make install chdir='/tmp/zeromq-{{zeromq_version}}'
  sudo: True
</pre>
{% endraw %}

## Step 1: Inspect

Before you install any software onto the target computer, take a look at the target computer and see whether the software is already there or not.

Most Ansible modules perform this check for you.  You'll only have to do it yourself when [building software from source](building-software-from-source.html) or installing software using a third-party package manager, such as PHP's `pecl`.

I'll cover how to perform this step in [Making Roles Repeatable](making-roles-repeatable.html) later in this book.

<div class="callout info" markdown="1">
#### Why Do We Need To Inspect?

You are going to run your playbook against a target computer multiple times, especially when you're writing and testing a new role.

You're going to waste a lot of time waiting for Ansible if your software is constantly re-installing software that has already been installed.  Even if you're willing to put up with this, chances are your colleagues won't.
</div>

## Step 2: Install

Use [Ansible's extensive list of modules](http://docs.ansible.com/modules_by_category.html) to install software onto the target computer.

## Step 3: Enable

Once the software is installed, the software needs to be configured (if you have any configuration files to install) and (if it is a service) restarted.

Put this in a separate `enable.yml` file so that it always runs even if the software is already installed.  That way, you can install configuration changes without having to re-install the software itself.

[Installing config files](working-with-config-files.html) and [restarting services](restarting-services.html) are covered in detail in their own chapters.

## Testing Your Role

