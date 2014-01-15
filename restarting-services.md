---
layout: top-level
title: Restarting Services
prev: '<a href="working-with-config-files.html">Prev: Working With Config Files</a>'
next: '<a href="making-roles-repeatable.html">Next: Making Roles Repeatable</a>'
---

# Restarting Services

## When Do I Need To Restart Services?

## Creating Handlers For Services

Put each handler in its own YAML file, and include them in main.yml.

If you put all of the handlers in handlers/main.yml, you cannot call a handler before a play finishes.

## Calling Handlers When A Service Needs Restarting

## Calling Handlers Before A Play Finishes

When tasks include a __notify:__ statement, each notified handler is added to a queue inside Ansible, and not called straight away.  When the current block of tasks have finished (i.e. when all the roles have been applied), Ansible then calls all of the handlers that have been queued up.

So if we want a handler to be called sooner, how do we get around that?

It's really simple to do.  Handlers are tasks, and can be included like any other task.  Just include the handler's YAML file in your task, at the point where you want the handler to run, and the handler will run there and then.

## Calling All Queued Handlers Mid-way Through A Play

There is a second way to get Ansible to run all of the handlers that have been queued up, using the somewhat obscure __meta:__ statement in a task:

<pre>
---
- name: flush all handlers
  meta: flush_handlers
</pre>

<div class="callout warning" markdown="1">
#### Are You Missing Any notify: Statements?

Get in the habit of adding __notify:__ statements to every task that should have one, even if you believe that you're queueing up multiple restarts of a service.

If you don't, then any tasks that run after __meta: flush_handlers__ may be forgetting to restart services on the target computer.
</div>