---
layout: top-level
title: Key Concepts
prev: '<a href="why-ansible.html">Prev: Why Ansible?</a>'
next: '<a href="installing-ansible.html">Next: Installing Ansible</a>'
---

# Key Concepts

To get the most out of Ansible, and out of this book, there are a few terms that you need to become familiar with.

Don't worry if they don't mean much to you first time around.  I've put them at the front of the book so that you can easily refer back to them, and I'll repeat them when they crop up later on in the book.  By the end of the book, you should be able to come back to this chapter and find that each term makes a lot more sense :)

## Playbook Repo

A _playbook repo_ is a collection of files that, together, tell Ansible how to install software, and where to install it.  It's normally stored in a source control repository of some kind (hence the name _playbook repo_).

A playbook repo contains:

* One or more [playbooks](#playbooks) (required)
* One or more [roles](#roles) (required)
* The [Inventory](#inventory) (required)
* [Per-host variables](#host_vars) and [per-group variables](#group_vars) (optional)

I look at the playbook repo in detail in [Organising Your Ansible Files](organising-your-ansible-files.html).

You'll also find more information about playbook repos in the official Ansible docs:

* tbd

## Playbooks

A _playbook_ is a [YAML file](#yaml) that tells Ansible which [roles](#roles) to apply to a set of computers.

<pre>
---
- hosts: build-servers
  roles:
  - cmd-gcc
  - cmd-autotools
  - cmd-distcc
</pre>

This playbook tells Ansible to apply the roles _cmd-gcc, cmd-autotools, cmd-distcc_ to every host in the [Inventory](#inventory) that's in the _build-servers_ set.

I look at the playbook in detail in [Your First Playbook](your-first-playbook.html).

You'll also find more information about playbooks in the official Ansible docs:

* tbd

## YAML

_YAML_ is a syntax for storing structured data in plain text.  It's an alternative to JSON and XML, which you've probably seen and used before.  Like many things in web programming today, it was originally popularised by the Ruby On Rails framework.

Most of the files you'll write for Ansible will be YAML files.  There are a few exceptions; I'll point them out when they crop up.

This isn't a book about YAML, so rather than spend a lot of time talking about YAML's syntax, I've just gone ahead and included a lot of examples instead throughout the chapters.

## Roles

A _role_ is a single change that Ansible will make to a computer.  On disk, each role is a folder that contains:

* One or more [tasks](#tasks) (required)
* [Handlers](#handlers) to restart services when a task has run (optional)
* [Files](#files) to copy up to the computer you are provisioning (optional)
* [Templates](#templates) to expand and then copy up to the computer you are provisioning (optional)
* [Metadata](#metadata) to tell Ansible which roles this role depends on (optional)

Roles are said to be _applied to a computer_ when they have successfully run.

I've split the information about roles into several chapters of this book:

* [Building Roles](building-roles.html)
* [Restarting Services](restarting-services.html)
* [Adding Dependencies To Roles](adding-dependencies-to-roles.html)
* [Making Roles Repeatable](making-roles-repeatable.html)
* [Debugging Failing Roles](debugging-failing-roles.html)

You'll also find more information about roles in the official Ansible docs:

* tbd

## Tasks

A _task_ is a YAML file stored inside a [role](#roles)'s folder.  It contains instructions that tell Ansible how to apply a role to the target computer.  A role must contain at least one task file, and can contain more than one.

Tasks are such an integral part of roles that they're not discussed in their own chapter.  You'll find information about them in every chapter that discusses roles.

You'll also find more information about tasks in the official Ansible docs:

* tbd

## Modules

An Ansible _module_ is an Ansible code library that you can call from a [task](#tasks) or [handler](#handlers).  Ansible ships with a large (and always-growing!) set of modules.  You can also write your own modules - something that I do not cover in this book.

You'll see me talking about Ansible modules throughout the chapters that discuss [roles](#roles).

You'll also find more information about Ansible modules in the official Ansible docs:

* tbd

## Handlers

A _handler_ is a task that gets called when a `notify` event is triggered by one of your tasks.  They're originally intended to restart services and daemons after you've made a change (so that your change is picked up and becomes active), but you can use them to do whatever you need done.

I look at handlers in the chapter on [Restarting Services](restarting-services.html).

You'll also find more information about handlers in the official Ansible docs:

* tbd

## Files

A _file_ is part of a role; it is a file to copy from your [playbook repo](#playbook_repo) to the target computer.  It can be a plain-text file (such as a configuration file), a script that you want to run on the target computer, or even a binary of some kind.

A role can contain as many, or as few files as needed.

I look at files in these chapters:

* [Building Roles](building-roles.html)
* [Working With Config Files](working-with-config-files.html)
* [Building Software From Source](building-software-from-source.html)
* [Temporary Install Scripts](temporary-install-scripts.html)

You'll also find more information about files in the official Ansible docs:

* tbd

## Templates

A _template_ is part of a role; it is a file that is expanded using Jinja2 and then copied from your [playbook repo](#playbook_repo) to the target computer.  It's normally a plain-text configuration file.

A role can contain as many, or as few templates as needed.

I look at templates in these chapters:

* [Working With Config Files](working-with-config-files.html)

You'll also find more information about templates in the official Ansible docs:

* tbd

## Metadata

_Metadata_ is part of a role, and is currently used to tell Ansible which roles it needs to run before it runs the current role.  Or, in other words, which roles the current role depends upon.

I look at metadata in these chapters:

* [Adding Dependencies To Roles](adding-dependencies-to-roles.html)

You'll also find more information about metadata in the official Ansible docs:

* tbd

## Facts

A _fact_ is a constant that Ansible defines.  The first thing Ansible does when it runs is to example each computer listed in the [Inventory](#inventory), and define a set of facts that describe that computer's current state.

You can use facts and [variables](#variable) in your [tasks](#task), [handlers](#handler) and [templates](#templates).

I look at facts in these chapters:

* tbd

You'll also find more information about facts in the official Ansible docs:

* tbd

## Variables

A _variable_ is a piece of data that you define.  Variables can be set statically in the [Inventory](#inventory), [host_vars](#host_vars) or [group_vars](#group_vars), or you can set variables dynamically whilst executing [tasks](#tasks).

You can use [facts](#facts) and variables in your [tasks](#task), [handlers](#handler) and [templates](#templates).

I look at variables in these chapters:

* [Working With Config Files](working-with-config-files.html)
* [Making Roles Repeatable](making-roles-repeatable.html)
* [Managing The Inventory](managing-the-inventory.html)

You'll also find more information about variables in the official Ansible docs:

* tbd

## Inventory

The _Inventory_ is a list of hosts that you want Ansible to apply roles to, and a list of which roles apply to which hosts.  Hosts can also be grouped so that you can update groups separately, or apply different variables to different groups of hosts.  The Inventory is stored in a file on disk, and can be committed to source control.

I look at the Inventory in these chapters:

* [Managing The Inventory](managing-the-inventory.html)

You'll also find more information about templates in the official Ansible docs:

* tbd

## Host Vars

_Host vars_ are [variables](#variables) that you have defined for a single specific host.  They can be stored in the [Inventory](#inventory), and/or in their own per-host file on disk (which can be committed to source control).

You can use [facts](#facts) and variables in your [tasks](#task), [handlers](#handler) and [templates](#templates).

I look at host vars in these chapters:

* tbd

You'll also find more information about templates in the official Ansible docs:

* tbd

## Group Vars

_Group vars_ are [variables](#variables) that you have defined for every host in a group.  They are stored in their own per-group file on disk, which can be committed to source control.

I look at group vars in these chapters:

* tbd

You'll also find more information about templates in the official Ansible docs:

* tbd
