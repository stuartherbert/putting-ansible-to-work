---
layout: top-level
title: Key Concepts
prev: '<a href="why-ansible.html">Prev: Why Ansible?</a>'
next: '<a href="installing-ansible.html">Next: Installing Ansible</a>'
---

# Key Concepts

To get the most out of Ansible, and out of this book, there are a few terms that you need to become familiar with.

Don't worry if they don't mean much to you first time around.  I've put them at the front of the book so that you can easily refer back to them, and I'll repeat them when they crop up later on in the book.  By the end of the book, you should be able to come back to this chapter and find that each term makes a lot more sense :)

## How Does Ansible Work?

When you run Ansible, on the command-line you tell it:

* the computer or list of computers that you want to provision ([the Inventory](#inventory)), and
* the provisioning instructions to follow ([the playbook](#playbook)).

Ansible logs into each computer via SSH, and [inspects the current state](#gathering_facts).  It turns this state into [facts](#facts) - constants that your provisioning instructions can use to decide what to do and what not to do.  It's important to note that, at this point, it isn't looking to see what you have already installed onto the computer - it is looking at things like what operating system you have installed.

Ansible then runs through the provisioning instructions, executing instructions known as [tasks](#tasks).  You will see each task scroll up the screen in turn, along with what happened:

* _changed_: the task ran, and changed something on the computer or changed the value of an Ansible [variable](#variables)
* _ok_: the task ran, but nothing changed as a result
* _skipped_: no attempt was made to run the task
* _failed_: the task ran but reported an error

Errors in Ansible are normally fatal: execution stops there and then, and you need to fix the problem before you run Ansible again.  This is a good safety measure, and it avoids compounding problems.

At the end of a successful run, Ansible will print out a summary of how many tasks it has run, how many tasks failed, and how many computers it was unable to contact at all.

## Ansible Versioning

At the time of writing, the current version of Ansible is v1.4.  Ansible is currently not [semantically versioned](http://semver.org).

* Bug fixes aren't backported to old, stable releases.  You will need to upgrade to get bug fixes.
* Currently there isn't an ecosystem around distributing third-party modules separately from Ansible (there's no [npm](https://npmjs.org/) or [composer](http://getcomposer.org)-like community yet); new modules get added to upcoming releases of Ansible.  There's an opportunity there for someone to create that community and really accelerate the adoption of Ansible.
* The [official Ansible docs](http://docs.ansibleworks.com) normally track [the latest code in GitHub](https://github.com/ansible/ansible).
* New releases can (and will) break your existing playbooks.  Upgrade early, upgrade often, and set time aside to test your playbooks after every upgrade.

You might be feeling uncomfortable about managing crucial infrastructure, and be concerned about whether Ansible is stable enough to adopt.  Don't worry.  This is normal with new cutting-edge software, should settle down in time, and in my opinion is worth it at this point in Ansible's development.

I will show you [how to install the latest code from GitHub](installing-ansible.html) later in this book.

## Playbook Repo

A _playbook repo_ is a collection of files that, together, tell Ansible how to install software, and where to install it.  It's normally stored in a source control repository of some kind (hence the name _playbook repo_).

A playbook repo contains:

* One or more [plays](#plays) (required)
* One or more [roles](#roles) (required)
* The [Inventory](#inventory) (required)
* [Per-host variables](#host_vars) and [per-group variables](#group_vars) (optional)

Historically, Ansible encouraged installing all of these things centrally, and this is still reflected in both the official Ansible docs and in the behaviour of Ansible.  As developers though, we're much more used to having our projects checked out of source control into folders under our home folder - and that's the way I'm going to show you in this book.  You can keep everything up to date simply by sharing changes to your playbook repo via source control, and you can even setup more than one playbook repo if you have to (for example, you might keep one playbook repo per customer or contract).

I explain the playbook repo in detail in [Organising Your Ansible Files](organising-your-ansible-files.html).

You'll also find more information about playbook repos in the official Ansible docs, where they'll often simply be referred to as _example playbooks_:

* [Playbook Best Practices: Content Organisation](http://docs.ansible.com/playbooks_best_practices.html#content-organization)

## Playbook

The _playbook_ is the contents of your _playbook repo_.  It's a sporting term: the playbook contains all of the different [plays](#plays) that are available to perform - i.e. all of the different provisioning instructions that Ansible can carry out.

You'll often see the terms _playbook_ and _play_ used interchangeably in the official Ansible documentation, and on the command-line, when Ansible expects a _playbook_, you can pass it a _play_ instead if you want.  This can be confusing at first.  I'll do my best to keep the two terms separate in this book.

I cover the playbook in these chapters:

* [Your First Playbook](first-playbook.html)
* [Organising Your Ansible Files](organising-your-ansible-files.html)

## Play

A _play_ is a [YAML file](#yaml) that tells Ansible which [roles](#roles) to apply to a set of computers.

<pre>
---
- hosts: build-servers
  roles:
  - cmd-gcc
  - cmd-autotools
  - cmd-distcc
</pre>

This play tells Ansible to apply the roles _cmd-gcc, cmd-autotools, cmd-distcc_ to every host in the [Inventory](#inventory) that's in the _build-servers_ set.

You'll often see the terms _playbook_ and _play_ used interchangeably in the official Ansible documentation, and on the command-line, when Ansible expects a _playbook_, you can pass it a _play_ instead if you want.  I'll do my best to keep the two terms separate in this book.

I explain the play in detail in [Your First Playbook](your-first-playbook.html).

You'll also find more information about plays in the official Ansible docs:

* [Playbooks](http://docs.ansible.com/playbooks.html)
* [Playbooks: Special Topics](http://docs.ansible.com/playbooks_special_topics.html)

## YAML

_YAML_ is a syntax for storing structured data in plain text.  It's an alternative to [JSON](http://www.json.org/) and [XML](http://www.w3.org/XML/), which you've probably seen and used before.  Like many things in web programming today, it was originally popularised by the Ruby On Rails framework.

Most of the files you'll write for Ansible will be YAML files.  There are a few exceptions; I'll point them out when they crop up.

This isn't a book about YAML, so rather than spend a lot of time talking about YAML's syntax, I've just gone ahead and included a lot of examples instead throughout the chapters.  There are times when Ansible pushes its use of YAML a bit close to the edge, which can cause syntax errors; I'll cover these as and when you're likely to run into them.

<div class="callout info" markdown="1">
#### All YAML Files Start With ---

As you work your way through the book, you'll notice that every example YAML file in the book starts with three hyphens on the first line.  This is part of the YAML spec.

If you leave out the three hyphens, your YAML file won't load.
</div>

If you really want to read more about YAML, there's [an official website for YAML](http://www.yaml.org/), and AnsibleWorks have posted [their own YAML syntax walkthrough](http://docs.ansible.com/YAMLSyntax.html).

## Roles

A _role_ is a single change that Ansible will make to a computer.  On disk, each role is a folder that contains:

* One or more [tasks](#tasks)
* [Handlers](#handlers) to restart services when a task has run
* [Files](#files) to copy up to the computer you are provisioning
* [Templates](#templates) to expand and then copy up to the computer you are provisioning
* [Metadata](#metadata) to tell Ansible which roles this role depends on

A role must contain at least a task, metadata or a handler.

Roles are said to be _applied to a computer_ when they have successfully run.

Most of the book is about roles - they are fundamental to how to work with Ansible.  I've split the information across these chapters:

* [Building Roles](building-roles.html)
* [Working With Config Files](working-with-config-files.html)
* [Restarting Services](restarting-services.html)
* [Adding Dependencies To Roles](adding-dependencies-to-roles.html)
* [Making Roles Repeatable](making-roles-repeatable.html)
* [Debugging Failing Roles](debugging-failing-roles.html)
* [Building Software From Source](building-software-from-source.html)
* [Temporary Install Scripts](temporary-install-scripts.html)
* [Using Meta-Roles](using-meta-roles.html)

You'll also find more information about roles in the official Ansible docs:

* [Playbook Roles And Include Statements](http://docs.ansible.com/playbooks_roles.html)

## Tasks

A _task_ is a YAML file stored inside a [role](#roles)'s folder.  It contains instructions that tell Ansible how to apply a role to the target computer.  A role must contain at least one task file, and can contain more than one.

Tasks are so important that I've dedicated a chapter to explaining [how tasks work](how-tasks-work.html).  You'll also find information about them in every chapter that discusses roles.

The same applies to the official Ansible docs.

## Modules

An Ansible _module_ is an Ansible code library that you can call from a [task](#tasks) or [handler](#handlers).  Ansible ships with a large (and always-growing!) set of modules.  You can also write your own modules - something that I do not cover in this book.

I mention Ansible modules throughout the chapters that discuss [roles](#roles).

You'll also find more information about Ansible modules in the official Ansible docs:

* [About Modules](http://docs.ansible.com/modules.html)
* [Module Index](http://docs.ansible.com/modules_by_category.html) - whenever I'm working with Ansible, I have this open in a browser tab!

## Handlers

A _handler_ is a task that gets called when a `notify` event is triggered by one of your tasks.  They're originally intended to restart services and daemons after you've made a change (so that your change is picked up and becomes active), but you can use them to do whatever you need done.

I explain handlers in the chapter on [Restarting Services](restarting-services.html).

You'll also find more information about handlers in the official Ansible docs:

* [Handlers: Running Operations On Change](http://docs.ansible.com/playbooks_intro.html#handlers-running-operations-on-change)

## Files

A _file_ is part of a role; it is a file to copy from your [playbook repo](#playbook_repo) to the target computer.  It can be a plain-text file (such as a configuration file), a script that you want to run on the target computer, or even a binary of some kind.

A role can contain as many, or as few files as needed.

I look at files in these chapters:

* [Working With Config Files](working-with-config-files.html)
* [Building Software From Source](building-software-from-source.html)
* [Temporary Install Scripts](temporary-install-scripts.html)

You'll also find more information about files in the official Ansible docs:

* [File Module](http://docs.ansible.com/file_module.html)

## Templates

A _template_ is part of a role; it is a file that is expanded using [Jinja2](http://jinja.pocoo.org/docs/) and then copied from your [playbook repo](#playbook_repo) to the target computer.  Templates are normally used for plain-text configuration files.

A role can contain as many, or as few templates as needed.

I explain templates in detail in [Working With Config Files](working-with-config-files.html)

You'll also find more information about templates in the official Ansible docs:

* [Template Module](http://docs.ansible.com/template_module.html)

## Metadata

_Metadata_ is part of a role, and is currently used to tell Ansible which roles it needs to run before it runs the current role.  Or, in other words, which roles the current role depends upon.

I look at metadata in these chapters:

* [Adding Dependencies To Roles](adding-dependencies-to-roles.html)

You'll also find more information about metadata in the official Ansible docs:

* [Role Dependencies](http://docs.ansible.com/playbooks_roles.html#role-dependencies)

## Facts

A _fact_ is a constant that Ansible defines.  The first thing Ansible does when it runs is to example each computer listed in the [Inventory](#inventory), and define a set of facts that describe that computer's current state (known as [gathering facts](#gathering_facts)).

You can use facts and [variables](#variable) in your [tasks](#task), [handlers](#handler) and [templates](#templates).

I look at facts in these chapters:

* tbd

You'll also find more information about facts in the official Ansible docs:

* [Information Gathered From Systems: Facts](http://docs.ansible.com/playbooks_variables.html#information-discovered-from-systems-facts)

## Gathering Facts

_Gathering facts_ is what Ansible does before it executes a playbook.  Ansible logs into the target computer (normally via SSH) and defines a set of [facts](#facts) that describes the computer's current state.  These facts are then available to use in your [tasks](#task), [handlers](#handler) and [templates](#templates).

## Variables

A _variable_ is a piece of data that you define.  Variables can be set statically in the [Inventory](#inventory), [host_vars](#host_vars) or [group_vars](#group_vars), or you can set variables dynamically whilst executing [tasks](#tasks).

You can use [facts](#facts) and variables in your [tasks](#task), [handlers](#handler) and [templates](#templates).

I look at uses of variables in these chapters:

* [Working With Config Files](working-with-config-files.html)
* [Making Roles Repeatable](making-roles-repeatable.html)
* [Supporting Multiple Operating Systems](multiple-operating-systems.html)
* [Supporting Multiple Target Computers](multiple-target-computers.html)

You'll also find more information about variables in the official Ansible docs:

* [Variables](http://docs.ansible.com/playbooks_variables.html)

## Inventory

The _Inventory_ is a list of hosts that you want Ansible to apply roles to, and a list of which roles apply to which hosts.  Hosts can also be grouped so that you can update groups separately, or apply different variables to different groups of hosts.  The Inventory is stored in a file on disk, and can be committed to source control.

The _Inventory_ can also be dynamic: generated by code, or otherwise retrieved from another software system of some kind.  You can even mix and match the two approaches, having some of the Inventory in static files, and some of it dynamic.  It's an advanced topic that's beyond the scope of this book, and covered in the official Ansible docs.

I look at the Inventory in these chapters:

* [Organising Your Ansible Files](organising-your-ansible-files.html)
* [Supporting Multiple Target Computers](multiple-target-computers.html)

You'll also find more information about the Inventory in the official Ansible docs:

* [Inventory](http://docs.ansible.com/intro_inventory.html)
* [Dynamic Inventory](http://docs.ansible.com/intro_dynamic_inventory.html)

## Host Vars

_Host vars_ are [variables](#variables) that you have defined for a single specific host.  They can be stored in the [Inventory](#inventory), and/or in their own per-host file on disk (which can be committed to source control).

You can use [facts](#facts) and variables in your [tasks](#task), [handlers](#handler) and [templates](#templates).

I look at host vars in these chapters:

* tbd

You'll also find more information about host vars in the official Ansible docs:

* [Inventory: Host Variables](http://docs.ansible.com/intro_inventory.html#host-variables)

<div class="callout info" markdown="1">
#### Not Just Key: Values

Although we're only going to store simple key/value pairs in the host vars in this book, _value_ doesn't have to be a string.  You can create nested arrays and objects too.
</div>

## Group Vars

_Group vars_ are [variables](#variables) that you have defined for every host in a group.  They are stored in their own per-group file on disk, which can be committed to source control.

I look at group vars in these chapters:

* tbd

You'll also find more information about group vars in the official Ansible docs:

* [Inventory: Group Variables](http://docs.ansible.com/intro_inventory.html#group-variables)

<div class="callout info" markdown="1">
#### Not Just Key: Values

Although we're only going to store simple key/value pairs in the host vars in this book, _value_ doesn't have to be a string.  You can create nested arrays and objects too.
</div>

## Target Computer

A _target computer_ is a computer that is provisioned via Ansible.  The list of target computers and the plays to apply to them is stored in the [Inventory](#inventory).  Target computers can include your own desktop / laptop, a virtual machine running locally or in the cloud, or a physical server sat in a datacenter.

I look at the target computer in detail in [Preparing A Computer For Ansible](preparing-a-computer.html).

In the official Ansible docs, you'll often seen target computers referred to as _hosts_ or _the host_.  I've found that _host_ just ends up confusing people, as they're used to the term _host_ meaning where software is running.

## Remote User

The _remote user_ is the UNIX user account that Ansible should log into on the [target computer](#target_computer).  Ansible will normally log into this account via SSH to perform [tasks](#tasks).

I look at remote users in detail in [Preparing A Computer For Ansible](preparing-a-computer.html).

## Other Terms

The official Ansible docs provide a [glossary](http://docs.ansible.com/glossary.html) of all the terms and concepts that crop up throughout the Ansible documentation.  It's a good place to start when you come across something that's completely new to you in Ansible.
