---
layout: top-level
title: How Tasks Work
prev: '<a href="organising-your-ansible-files.html">Prev: Organising Your Ansible Files</a>'
next: '<a href="planning-a-role.html">Next: Planning A Role</a>'
---

{% raw %}
# How Tasks Work

We're about to dive into roles and the common problems you're going to use roles to solve.  Before we do that though, we need to look at _tasks_ - the instructions that live inside roles.

## What Is A Task?

A task is set of statements stored in a YAML file.  It tells Ansible what to do (the _action_), and what to tell the user it is doing (the _name_).

Here's the task from the Curl role you created when you built your first playbook:

<pre>
---
- name: "Ubuntu: install Curl"
  apt:
    pkg: curl
    state: latest
  sudo: true
</pre>

<div class="callout info" markdown="1">
#### What Does Each Line Say?

Each line is called a _statement_, and it tells Ansible something about the task.

* __name: "Ubuntu: install Curl"__ tells Ansible to print _Ubuntu: install Curl_ on the screen when this task runs (so that you know what Ansible is currently doing)
* __apt:__ tells Ansible to call the [apt module](http://docs.ansible.com/apt_module.html).  This line is known as the _action statement_.
* _pkg: curl_ and _state: latest_ are parameters for the apt module.
* __sudo: true__ tells Ansible that this task needs to be run by `root`

I've compiled a list of valid statements for you and what they do; you'll find it later in this chapter.
</div>

## Where Do Tasks Live?

When Ansible executes a role, it looks for the file `tasks/main.yml` inside the role's folder.  If `tasks/main.yml` exists, it starts executing each task listed in that file, from top to bottom.

You can put your tasks into additional files, and `include:` them from `tasks/main.yml`.

<div class="callout info" markdown="1">
#### Why Split Your Tasks Across Files?

There are two main reasons: to make it easy to skip tasks, and for [multiple operating system support](multiple-operating-systems.html):

* Ansible doesn't have an `if` statement as such.  It has a `when:` statement that can be added to both tasks and `include:` statements.  Adding a `when:` statement to each task in a file gets tedious very quickly, and creates a lot of duplication and clutter.  Adding a `when:` statement to an `include:` statement allows you to skip every task in the included file in one go.

  We're going to take advantage of that when I show you how to [make roles repeatable](making-roles-repeatable.html).

* Instead of creating different roles for different operating systems, the Ansible way is to have each role support every operating system that you need it to work on.  Putting the instructions for each operating system into seperate files keeps things well-organised.  It also means you don't have to put `when:` statements to skip other operating systems on each task, just on each `include:` statement in `tasks/main.yml`.
</div>

## What Does tasks/main.yml Contain?

`tasks/main.yml` contains one or more tasks, and / or one or more `include:` statements.

Use a hyphen at the beginning of the line to start a new task or `include:`.  This tells YAML how to convert the text in the file into a list of tasks for Ansible to understand.

<pre>
---
# an example task
- name: install Curl
  apt:
    pkg: curl
    state: latest
  sudo: true

# example include statements
- include: another-task.yml
- include: yet-another-task.yml
</pre>

## What Statements Can A Single Task Include?

At the time of writing, there isn't a handy list of these in the official Ansible docs that I can point you at.

Here's all of the prefixes that I know about, and a brief description of what they do.  There will be some that I've never come across, and by the time you read this, new statements may have been added to Ansible too.

* name:
* action: or &lt;module&gt;:
* sudo:
* sudo_user:
* remote_user:
* notify:
* ignore_errors:
* register:
* when:
* meta:
* with_items:
* with_fileglob:
* with_together:
* with_subelements:
* with_sequence:
* with_random_choice:
* with_first_found:
* with_lines:
* with_indexed_items:
* with_flattened:

Let's look at each of these in detail.

## name:

__name:__ has two purposes:

1. it tells Ansible what to print on the screen when the task executes, and
2. it is string you use in a __notify:__ statement when you want to trigger an event (such as [restarting services](restarting-services.html))

__name:__ is normally the first statement in every task, just because that makes the task is easier to read.

## action: and &lt;module&gt;:

__action:__'s call modules.  Modules are (effectively) the code libraries that make changes on the target computer.

Ansible ships with a [large library of modules](http://docs.ansible.com/modules_by_category.html), and new modules are added to Ansible very frequently.

The format of an __action:__ is:

<pre>
action: &lt;module-name&gt; &lt;param&gt;=&lt;value&gt; ...
</pre>

where:

* `<module-name>` is the name of the Ansible module to call
* `<param>=<value>` is a named parameter to pass to the module you are calling to tell it what you want it to do.  Each module has its own list of parameters that you can pass.

<div class="callout info" markdown="1">
#### RTFM!

You'll find it handy to keep the [official Ansible docs](http://docs.ansible.com/modules_by_category.html) open in a browser tab or two so that you can quickly lookup the parameters for every module that you want to use.
</div>

<div class="callout warning" markdown="1">
#### A Task Cannot Have Multiple action: Statements

At the time of writing, each individual task in Ansible can only contain one __action:__ statement.  This is something that will catch you out until you get used to it.

If you need to perform two or more __action:__s, you will need to create a separate task for each __action:__.
</div>

<div class="callout info" markdown="1">
#### action: is Optional

When you read the official Ansible docs, you'll notice that most of the examples shown miss out the __action:__ at the start of the module line.  For example, instead of:

<pre>
- name: install Curl
  action: apt pkg=curl state=latest
  sudo: true
</pre>

the official docs will show:

<pre>
- name: install Curl
  apt:
    pkg: curl
    state: latest
  sudo: true
</pre>

Both are equally valid.

The __action:__ prefix is the original way to do it, and using the module's name directly is the current recommended way to do it.
</div>

## sudo:

__sudo:__ tells Ansible whether the task should run as `root` or not.

* __sudo: true__ means run as `root`
* __sudo: false__ means run as the remote user

If omitted, the default is __sudo: false__.

<div class="callout warning" markdown="1">
#### Safety First

In each play, you can add a __sudo: true__ statement to change the default behaviour, so that you don't have to type __sudo: true__ in all of your tasks.  Whilst it might be tempting to do this, _don't_.

Experienced sysadmins don't login as `root` and do everything from there.  For safety's sake, they login as themselves, and use `sudo` only when they intend to run commands as `root`.  It's a habit that reduces the amount of box-nuking mistakes they make.

Adopt the same principle in your playbook.  Leave the default to be __sudo: false__, and explicitly add __sudo: true__ only if the task must be run as `root`.
</div>

## sudo_user:

__sudo_user:__ tells Ansible to run the command as a different user, instead of running it as `root` or as the remote user.  The format is:

<pre>
sudo: true
sudo_user: &lt;username&gt;
</pre>

<div class="callout info" markdown="1">
#### Using Other Users

Why would you need to run commands as a different, non-`root` user?

* Provisioning a box isn't limited to installing software.  Setting up and maintaining user accounts, and their associated dotfiles, is also part of the provisioning process.
* On UNIX systems, each service runs as a user, and it has become common practice to give each service its own user.  You can use __sudo_user: true__ when you need to run a command as one of these users.
</div>

## remote_user:

__remote_user:__ tells Ansible to log into the target computer as a different user.  The format is:

<pre>
remote_user: &lt;username&gt;
</pre>

This user will need your SSH key in their `authorized_keys` file, otherwise Ansible will not be able to login.

I can't think of a good reason for a task to ever use __remote_user:__.  I would always use __sudo_user:__ in a task, and set __remote_user:__ for each target computer.  I'll cover that in more detail in [Supporting Multiple Target Computers](multiple-target-computers.html) later in this book.

## notify:

__notify:__ tells Ansible to call a handler after the task has completed.

<pre>
---
- name: install Apache
  apt:
    pkg: apache2
    state: latest
  sudo: true
  notify:
  - restart Apache2
  - restart PHP-FPM
</pre>

<div class="callout info" markdown="1">
#### What Is A Handler?

A handler is a task that is defined in the `handlers/` folder instead of the `tasks/` folder.  Handlers only get executed when called from a task's __notify:__ statement.

In theory, handlers can also use __notify:__, but I've never tried it myself.
</div>

<div class="callout warning" markdown="1">
#### When Do Handlers Run?

A handler is executed at the end of a play: i.e. only when all of the roles listed in a play file have been applied.

This has implications for how we design and build handlers, which I'll cover in [Restarting Services](restarting-services.html).
</div>

## ignore_errors:

If a task fails, Ansible's default behaviour is to treat it as a fatal error, and stop provisioning the target computer.  __ignore_errors:__ tells Ansible to continue if the current task fails.

<pre>
- name: restart Apache2
  command: /etc/init.d/apache2 restart
  sudo: true
  ignore_errors: true
</pre>

## register:

__register:__ tells Ansible to capture the output of a command and store it as a variable.  Variables can be used in templates, and in [conditional statements](http://docs.ansible.com/playbooks_conditionals.html).

<pre>
---
- name: how many CPUs are available for compiling?
  command: nproc
  register: env_cpus_found
</pre>

<div class="callout warning" markdown="1">
#### Registered Variables Are Objects, Not Strings

This looks reasonable, but isn't valid:

<pre>
- name: how many CPUs are available for compiling?
  command: nproc
  register: env_cpus_found

# this will not work
- name: compile libfoo
  command: make -j {{env_cpus_found}}
</pre>

`env_cpus_found` is an object, not a string.  Ansible has to create an object, so that it can store the command's stdout and stderr output, in case you need to tell them apart.

<pre>
# this will work
- name: compile libfoo
  command: make -j {{env_cpus_found.stdout}}
</pre>

It's confusing because variables loaded from the Inventory and facts gathered from each target computer are nominally strings rather than objects.  This is going to catch you out the first few times.
</div>

## when:

__when:__ is the closest that Ansible has to an `if` statement.  Use __when:__ if a task should run some of the time.

<pre>
# this runs every time
- name: is /tmp full?
  shell: df -k /tmp | tail -n 1 | awk '{ print $5 }' | tr -d '%'
  register: tmp_percentage

# this is skipped if /tmp is less than 95% full
- name: empty /tmp
  shell: find /tmp -atime 7 -print0 | xargs rm -rf
  when: tmp_percentage.stdout|int >=95
</pre>

The __when:__ statement is evaluated using the Jinja2 templating engine, which makes it very powerful.  You'll find more details in [the official Ansible docs](http://docs.ansible.com/playbooks_conditionals.html#the-when-statement).

<div class="callout warning" markdown="1">
#### Break The when: Habit

If you're not careful, you'll find yourself using duplicate __when:__ statements throughout your tasks.  In my experience, any duplicate __when:__ statements end up being a maintenance nightmare: it's just too easy to edit some of them and miss out others, which leads to a lot of wasted time debugging failing Ansible plays.

Follow my [three point plan for roles](planning-a-role.html) and my [overlay strategy for supporting multiple operating systems](multiple-operating-systems.html) and you will avoid problems from over-using __when:__.
</div>

## meta:

__meta:__ is currently used to tell Ansible to run any queued-up handlers.  (Normally, handlers do not run until the end of the current block of tasks - i.e. when the current list of roles have all been applied).

<pre>
---
- name: flush handlers
  meta: flush_handlers
</pre>

I'll look at handlers in detail in [Restarting Services](restarting-services.html) later in this book.

## with_items:

__with_items:__ allows you to run the same task in a loop, iterating over a list of items.

Looping is out of scope for this book.  You can read the [official Ansible docs on looping](http://docs.ansible.com/playbooks_loops.html) to learn more.

## with_fileglob:

__with_fileglob:__ allows you to run the same task in a loop, iterating over a list of files on the target computer.

Looping is out of scope for this book.  You can read the [official Ansible docs on looping](http://docs.ansible.com/playbooks_loops.html) to learn more.

## with_together:

__with_together:__ allows you to run the same task in a loop, iterating over two or more arrays at the same time.

Looping is out of scope for this book.  You can read the [official Ansible docs on looping](http://docs.ansible.com/playbooks_loops.html) to learn more.

## with_subelements:

__with_subelements:__ allows you to run the same task in a loop, iterating over the properties inside a variable.

Looping is out of scope for this book.  You can read the [official Ansible docs on looping](http://docs.ansible.com/playbooks_loops.html) to learn more.

## with_sequence:

__with_sequence:__ allows you to run the same task in a loop, iterating over a generated sequence of numbers.

Looping is out of scope for this book.  You can read the [official Ansible docs on looping](http://docs.ansible.com/playbooks_loops.html) to learn more.

## with_random_choice:

__with_random_choice:__ allows you to set the `item` variable to one entry from a list of choices.

Looping is out of scope for this book.  You can read the [official Ansible docs on looping](http://docs.ansible.com/playbooks_loops.html) to learn more.

## with_first_found:

__with_first_found:__ allows you to run a task against the first file found from several search paths.

Looping is out of scope for this book.  You can read the [official Ansible docs on looping](http://docs.ansible.com/playbooks_loops.html) to learn more.

## with_lines:

__with_lines:__ allows you to run a task against each line output by a command.

Looping is out of scope for this book.  You can read the [official Ansible docs on looping](http://docs.ansible.com/playbooks_loops.html) to learn more.

## with_indexed_items:

__with_indexed_items:__ allows you to run a task in a loop against one or more variables that each contain an array.

Looping is out of scope for this book.  You can read the [official Ansible docs on looping](http://docs.ansible.com/playbooks_loops.html) to learn more.

## with_flattened:

__with_flattened:__ allows you to run a task in a loop against one or more variables that each contain a nested set of arrays.

Looping is out of scope for this book.  You can read the [official Ansible docs on looping](http://docs.ansible.com/playbooks_loops.html) to learn more.

## until:, retries: and delay:

__until:__, __retries:__ and __delay:__ allow you to retry the same task multiple times until a condition is met.

Looping is out of scope for this book.  You can read the [official Ansible docs on looping](http://docs.ansible.com/playbooks_loops.html) to learn more.

{% endraw %}