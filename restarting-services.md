---
layout: top-level
title: Restarting Services
prev: '<a href="working-with-config-files.html">Prev: Working With Config Files</a>'
next: '<a href="making-roles-repeatable.html">Next: Making Roles Repeatable</a>'
---

# Restarting Services

## What Is A Service?

A service is an app such as Apache, MySQL or MongoDB.  It's a UNIX daemon that is normally started when the target computer is booted up.  Services run in the background, and you normally interact with them over network protocols.

In the old days, services were started and stopped using shell scripts found in the `/etc/init.d/` folder.  At the time of writing, Linux distros are transitioning to [systemd](http://www.freedesktop.org/wiki/Software/systemd/) for managing services.  OSX uses [launchd and launchctl](http://en.wikipedia.org/wiki/Launchd) to manage services.

## When Do I Need To Restart Services?

{% raw %}
You need to start or restart services:

* when you have installed a UNIX daemon or service
* when you have upgraded a UNIX daemon or service
* when you have installed a config file for the UNIX daemon or service

You start a service so that it is available for use.  You restart a service to pick up any changes that you have deployed since the service was last started or restarted.

## How Do I Restart Services?

Ansible has a mechanism for this:

* you create [handlers](key-concepts.html#handlers) - tasks that live in your role's `handlers/` folder - to start, stop and restart services
* you call the handlers using the `notify:` statement

Here's an example, based on the `tasks/install.yml` from [Building Software From Source](building-software-from-source.html):

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
  notify:
  - restart redis

- name: install Redis server startup script
  copy:
    src: redis.sh
    dest: /etc/init.d/redis
    owner: root
    group: root
    mode: 0644
  sudo: True
</pre>

<pre>
---
# file: roles/stuartherbert.redis-server/handlers/main.yml

- name: restart redis
  command: /etc/init.d/redis restart
  sudo: True
</pre>

Let's break that down and look at how it works.

## Creating Handlers For Services

Handlers are tasks that live in the role's `handlers/` folder.  Put each handler in its own YAML file, and include them in `handlers/main.yml`:

<pre>
---
# file: roles/stuartherbert.redis-server/handlers/start-redis.yml

- name: start redis
  command: /etc/init.d/redis start
  sudo: True
</pre>

<pre>
---
# file: roles/stuartherbert.redis-server/handlers/stop-redis.yml

- name: stop redis
  command: /etc/init.d/redis stop
  sudo: True
</pre>

<pre>
---
# file: roles/stuartherbert.redis-server/handlers/restart-redis.yml

- name: restart redis
  command: /etc/init.d/redis restart
  sudo: True
</pre>

<pre>
---
# file: roles/stuartherbert.redis-server/handlers/main.yml
- include: start-redis.yml
- include: stop-redis.yml
- include: restart-redis.yml
</pre>

If you put all of the handlers in handlers/main.yml, you cannot call a handler before a play finishes.  Having them in separate files also helps if you need to [add support for multiple operating systems](multiple-operating-systems.html).

<div class="callout info" markdown="1">
#### Handlers Can Do Only One Thing

Each handler is currently a __single task__.  You can't (to the best of my knowledge!) make a single handler execute multiple tasks.

If you need the handler to perform multiple tasks, you've got a few options:

1. Create multiple handlers, and add each of them to the `notify:` blocks in your install tasks.
1. Create your own Ansible module, and call that from the handler.

You _can_ use the [script module](http://docs.ansible.com/script_module.html) to upload and execute a shell script on the target computer too if required.
</div>

## Calling Handlers When A Service Needs Restarting

The `notify:` statement is used to tell Ansible that you want to call a handler:

<pre>
---
# file: roles/stuartherbert.redis-server/tasks/install.yml

- name: install Redis
  command: make install chdir=/tmp/redis-{{redis_server_version}}
  sudo: True
  notify:
  - restart redis
</pre>

`notify:` expects a list of handlers to call.  List each handler by its `name:`.

<div class="callout info" markdown="1">
#### Starting After A First-Time Install

When you install a new service using a package supplied by your Linux distro of choice, the package will normally start the service for you.  So you don't need to worry about starting it yourself, right?

It depends.

When the package starts the service for you, that service is running with a default config of some kind.  You'll probably want to install your own config for the service.  If you do, you'll need to restart the service after you have installed your config file.  I'll look at that in more detail in [Working With Config Files](working-with-config-files.html).
</div>

## Calling Handlers Before A Play Finishes

When tasks include a `notify:` statement, the handler is not called straight away.  Ansible adds each notified handler to an internal queue.  When the current block of tasks have finished (i.e. when all the roles have been applied), Ansible then calls all of the handlers that have been queued up.

So if we want a handler to be called sooner, how do we get around that?

It's really simple to do.  Handlers are tasks, and can be included like any other task.  Just include the handler's YAML file in your task, at the point where you want the handler to run, and the handler will run there and then.

For example, if we needed the Redis server up and running as soon as it was installed, we could do this:

<pre>
---
# file: roles/stuartherbert.redis-server/tasks/install.yml

- name: install Redis
  command: make install chdir=/tmp/redis-{{redis_server_version}}
  sudo: True

- include: ../handlers/restart-redis.yml
</pre>

## Calling All Queued Handlers Mid-way Through A Play

There is a second way to get Ansible to run all of the handlers that have been queued up, using the somewhat obscure `meta:` statement in a task:

<pre>
---
- name: flush all handlers
  meta: flush_handlers
</pre>

<div class="callout warning" markdown="1">
#### Are You Missing Any notify: Statements?

Get in the habit of adding `notify:` statements to every task that should have one, even if you believe that you're queueing up multiple restarts of a service.

If you don't, then any tasks that run after `meta: flush_handlers` may be forgetting to restart services on the target computer.
</div>