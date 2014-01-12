---
layout: top-level
title: Building Roles
prev: '<a href="organising-your-ansible-files.html">Prev: Organising Your Ansible Files</a>'
next: '<a href="working-with-config-files.html">Next: Working With Config Files</a>'
---

# Building Roles

Roles are the LEGO brick of any Ansible playbook. You build your workstation or server from your roles.

## What Do Roles Contain?

Roles can contain any of:

* _files_ to be uploaded to the target computer
* _handlers_ to restart services after tasks have executed
* _metadata_ to list other roles that need to run before the current one
* _tasks_ to provision software and otherwise make changes to the target computer
* _templates_ to be expanded using Jinja2 and then uploaded to the target computer

## How Small Should Each Role Be?

Very small.  As small as you can make each one.  Let me explain.

Each role should do a single job, and no more.  It might install a command-line utility, or it might build a software library from source code.

Create a role for each piece of software that you install using Ansible.  Here are some examples from [my ansible-playbooks repo on GitHub](https://github.com/stuartherbert/ansible-playbooks):

* git
* hubflow
* nodejs
* redis2-server

Each of these roles installs a single software package.  In the case of the _git_ role, it's very trivial, and hardly seems worth the effort.  But here's the thing: small roles are the easiest to reuse.  Let me give you an example.

Let's say I have a play called _webserver.yml_, for a virtual machine that I'm using to host my software on.  It starts off using both the _nodejs_ and _redis2-server_ roles.  As demand for my software grows, a single virtual machine isn't going to be powerful enough any more: I'm going to need to split the software across several computers.  This is very easy to do, because I've created small reusable roles.  All I need to do is to create two plays: _frontend.yml_ will use the _nodejs_ role, and _backend.yml_ will use the _redis2-server_ role.  I probably don't have to change the roles at all, if I've already used templates to generate the config files for my software.

If I'd created a single role called _website.yml_ instead of separate _nodejs_ and _redis2-server_ roles at the start, then splitting the software across several VMs would be a lot more work.  If scaling the site was very urgent, the last thing I'd want would be extra work whilst under pressure, especially work that could have been completely avoided in the first place.

Whatever plays you're creating today, tomorrow you will have to create more plays.  If each role installs just one thing, then you can reuse your roles in tomorrow's new plays with the minimum of effort.

Just as your code should be modular, so should the roles in your playbook.

## The Three-Point Plan For Every Role

Follow this three-point plan when you are building a role, and you'll never go wrong:

1. __Inspect:__ Does the role need to act?
1. __Install:__ Add software to the computer.
1. __Enable:__ Update configuration, start services.

I put each step into a separate YAML file, and include them in `main.yml`.  This makes it really easy to skip the `install.yml` if the software is already installed.

<pre>
---
# file: roles/ubuntu-13.10/php-zmq/main.yml

- include: inspect.yml
- include: install.yml
  when: "php_zmq_installed != 'true'"
- include: enable.yml
</pre>

Most roles do not need all three steps.  For example, there's no configuration to update or services to restart after installing NodeJS itself.  In these cases, I just leave out the files I don't need:

<pre>
---
# file: roles/ubuntu-13.10/nodejs/main.yml
- include: inspect.yml
- include: install.yml
  when: "nodejs_installed != 'true'"
</pre>

## Step 1: Inspect

You are going to run your playbook against a target computer multiple times, especially when you're writing and testing a new role.  You're going to waste a lot of time waiting for Ansible if your software is constantly re-installing software that has already been installed.  The first thing each role should do is check to see if it needs to do anything at all.

Fortunately, most Ansible modules perform this check for you.  You'll only have to do it yourself when [building software from source](building-software-from-source.html) or installing software using a third-party package manager, such as PHP's `pecl`.  This is also covered in [Making Roles Repeatable](making-roles-repeatable.html) later in this book.

## Step 2: Install



## Step 3: Enable

Once the software is installed, the software needs to be configured and (if it is a service) restarted.  Put this in a separate `enable.yml` file so that it always runs even if the software is already installed.  That way, you can install configuration changes without having to re-install the software itself.

[Installing config files](working-with-config-files.html) and [restarting services](restarting-services.html) are covered in detail in their own chapters.

## Testing Your Role

