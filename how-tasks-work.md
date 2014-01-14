---
layout: top-level
title: How Tasks Work
prev: '<a href="first-playbook.html">Prev: Your First Playbook</a>'
next: '<a href="organising-your-ansible-files.html">Next: Organising Your Ansible Files</a>'
---

# How Tasks Work

You've just built and executed your first playbook, which gives you a working example that you can experiment with.  We're about to look at the overall structure of the playbook before diving into roles.  Before we do that though, I want to tell you a lot more about _tasks_, as they're fundamental to making roles actually do anything.

## What Is A Task?

A task is set of statements stored in a YAML file.  It tells Ansible what to do (the _action_), and what to tell the user it is doing (the _name_).

Here's the task from the Curl role you created when you built your first playbook:

<pre>
---
- name: install Curl
  action: apt pkg=curl state=latest
  sudo: true
</pre>

<div class="callout info" markdown="1">
#### What Does Each Line Say?

Each line is called a _statement_, and it tells Ansible something about the task.

* __name: install Curl__ tells Ansible to print _install Curl_ on the screen when this task runs (so that you know what Ansible is currently doing)
* __action: apt__ tells Ansible to call the [apt module](http://docs.ansible.com/apt_module.html).
* _pkg=curl_ and _state=latest_ are parameters for the apt module.
* __sudo: true__ tells Ansible that this task needs to be run by `root`

I've compiled a list of valid statements for you; you'll find it later in this chapter.
</div>

## Where Do Tasks Live?

When Ansible executes a role, it looks for the file `tasks/main.yml` inside the role's folder.  If `tasks/main.yml` exists, it starts executing each task listed in that file, from top to bottom.

You can put your tasks into additional files, and `include:` them from `tasks/main.yml`.

<div class="callout info" markdown="1">
#### Why Split Your Tasks Across Files?

There are two main reasons: readability, and to make it easy to skip tasks.

* `main.yml` is a very ambiguous name (and it is one that is reused in other places in Ansible for other things).  When someone looks at a `tasks/` folder, and only sees `main.yml`, they cannot immediately see the structure of your task.  Splitting up the tasks into `inspect.yml`, `install.yml` and `enable.yml` (which I'm going to show you how to do in [Planning A Role](planning-a-role.html)) makes it really obvious what your tasks are trying to achieve.

  This is very important when you're sharing your Ansible playbook with colleagues who don't edit the playbook very often.

* Ansible doesn't have an `if` statement as such.  It has a `when:` statement that can be added to both tasks and `include:` statements.  Adding a `when:` statement to each task in a file gets tedious very quickly, and creates a lot of duplication and clutter.  Adding a `when:` statement to an `include:` statement allows you to skip every task in the included file in one go.

  We're going to take advantage of that when I show you how to [make roles repeatable](making-roles-repeatable.html).
</div>

## What Does tasks/main.yml Contain?

`tasks/main.yml` contains one or more tasks, and / or one or more `include:` statements.

Use a hyphen at the beginning of the line to start a new task or `include:`.  This tells YAML how to convert the text in the file into a list of tasks for Ansible to understand.

<pre>
---
- name: install Curl
  action: apt pkg=curl state=latest
  sudo: true

- include: another-task.yml
- include: yet-another-task.yml
</pre>

## What Statements Can A Single Task Include?

At the time of writing, there isn't a handy list of these in the official Ansible docs that I can point you at.

Here's all of the prefixes that I know about, and a brief description of what they do.  There will be some that I've never come across, and by the time you read this, new statements may have been added to Ansible too.

* name:
* action:
* sudo:
* sudo_user:
* with_items:
* notify:
* remote_user:
* ignore_errors:
* register:
* when:
* meta:

Let's look at each of these in detail.

## name:

__name:__ has two purposes:

1. it tells Ansible what to print on the screen when the task executes, and
2. it is string you use in a __notify:__ statement when you want to trigger an event (such as [restarting services](restarting-services.html))

__name:__ is normally the first statement in every task, just because that makes the task is easier to read.

## action:

__action:__'s call modules.  Modules are (effectively) the code libraries that make changes on the target computer.

Ansible ships with a [large library of modules](http://docs.ansible.com/modules_by_category.html), and new modules are added to Ansible very frequently.

The format of an __action:__ is:

<pre>
action: &lt;module-name&gt; &lt;param&gt;=&lt;value&gt; ...
</pre>

where:

* `<module-name>` is the name of the Ansible module to call
* `<param>=<value>` is a named parameter to pass to the module you are calling to tell it what you want it to do.  Each module has its own list of parameters that you can pass.

<div class="callout warning" markdown="1">
#### A Task Cannot Have Multiple action: Statements

At the time of writing, each individual task in Ansible can only contain one __action:__ statement.  This is something that will catch you out until you get used to it.

If you need to perform two or more __action:__s, you will need to create a separate task for each __action:__.
</div>

<div class="callout warning" markdown="1">
#### Always Include The action: Prefix

When you read the official Ansible docs, you'll notice that most of the examples shown miss out the __action:__ at the start of the module line.  For example, instead of:

<pre>
- name: install Curl
  action: apt pkg=curl state=latest
  sudo: true
</pre>

the official docs will show:

<pre>
- name: install Curl
  apt: pkg=curl state=latest
  sudo: true
</pre>

Both are equally valid.

The __action:__ prefix is the original way to do it, and I recommend that you always include the __action:__ prefix in your tasks, for one simple reason.

As more and more modules are added to Ansible, the module names are going to start to clash with the prefixes for the other lines you can have in a task.  (This has already happened once, when the [user module](http://docs.ansible.com/user_module.html) was added to Ansible.)

Common sense dictates that something will have to give at some point in the future, especially if someone builds an npm / composer-like package ecosystem for Ansible.  Using the __action:__ prefix should ensure your tasks continue to work when that point is reached.
</div>

## sudo:
## sudo_user:
## with_items:
## notify:
## remote_user:
## ignore_errors:
## register:
## when:
## meta:


Now that you know how tasks work, let's take a look at the structure of a playbook repo.