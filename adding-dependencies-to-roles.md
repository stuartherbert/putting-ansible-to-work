---
layout: top-level
title: Adding Dependencies To Roles
prev: '<a href="debugging-failing-roles.html">Prev: Debugging Failing Ansible Roles</a>'
next: '<a href="multiple-operating-systems.html">Next: Supporting Multiple Operating Systems</a>'
---

# Adding Dependencies To Roles

## What Is A Dependency?

In Ansible terms, a dependency is any role that needs to have run before the current role runs.

Dependencies are used to guarantee that the target computer is in a predictable condition:

* that any changes to the operating system's default configuration have been made;
* that any required compilers, dynamic languages, shared libraries, tools and other utilities have already been installed on the box;

and generally that the current role has everything it needs to execute successfully on the target machine.

<div class="callout info" markdown="1">
#### A Significant Design Constraint

Ansible's dependencies are extremely simple - little more than a list of roles to run first.  Unlike tasks, there's no `when:` clause to make them optional.  This simplicity makes them very straight-forward to use, and ensures that they heavily influence the design of roles and plays.

With the introduction of dependencies, plays become a simple list of roles to run, and it is down to each role in turn to ensure that it works correctly on each operating system that you want to manage using Ansible.  This greatly improves - indeed, practically enforces - the modularity and re-usability of roles, and has paved the way for [AnsibleWork's Galaxy](ansibleworks-galaxy.html), which I'll look at towards the end of this book.
</div>

## Why Do We Use Dependencies?

Adding a list of dependencies to roles makes the role much easier to re-use in multiple plays.

For example, let's imagine that we have the following plays in our playbook:

* marketing_website.yml
* user_website.yml
* admin_site.yml
* website_dev.yml

The first three plays install the marketing website, end-user website and the internal admin website into different environments, such as test, staging and production.  The final play installs everything onto a developer's desktop or laptop, so that they can work on adding new features and fixing any bugs that have been reported.

Which roles need to go into the `website_dev.yml` play?

__You don't have enough information to answer that question.__

I deliberately haven't told you anything about what languages the websites are written in, what other software the websites require, and so on.  To answer the question, you would need to study each play carefully to see which roles they use, and in what order.  You'd need to add the roles in the right order to the `website_dev.yml` play.  You'd need to know when each of the plays changed, so that you could incorporate those changes into the `website_dev.yml` play.

If you add dependencies to all of your roles, all you need to know is which roles install each of the websites. You can let Ansible worry about the rest.

## Where Do I Add The Dependency?

Dependencies go into the `meta/main.yml` file in your role:

<pre>
---
# file: roles/stuartherbert.php-composer/meta/main.yml
dependencies:
- { role: stuartherbert.php-cli }
- { role: stuartherbert.php-json }
- { role: stuartherbert.git }
</pre>

In this simple example, the _php-composer_ role depends on PHP's command-line tool, PHP's JSON extension and the Git command.  I can add the _php-composer_ role to any play, and be confident that it will work.

I can also add it as a dependency for any other role, and not worry about whether or not it is explicitly listed in the play at all.  Ansible will make sure that it has been executed before any role that depends on it.

## Adding Parameters To Dependencies

You can add parameters to dependencies.  This allows you to set the value of variables before a role is called:

<pre>
dependencies:
- { role: stuartherbert.redis-server, redis_server_instance: audit, redis_server_port: 6018 }
</pre>

It's very handy for when you run multiple copies of the same service on a target computer.  Each copy of the service typically needs its own config file and port number to listen on, and injecting variables into dependencies is one way to make that happen.