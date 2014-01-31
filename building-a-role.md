---
layout: top-level
title: Building A Role
prev: '<a href="planning-a-role.html">Prev: Planning A Role</a>'
next: '<a href="installing-software-from-packages.html">Next: Installing Software From Packages</a>'
---

# Building A Role

## The WorkFlow For Building A Role

Follow these steps when you're adding new roles to your Ansible playbook:

1. Make the role [install software from packages](installing-software-from-packages.html), or [build software from source](building-software-from-source.html) perhaps by [using temporary install scripts](temporary-install-scripts.html).  You may need to [install config files](working-with-config-files.html) and [restart services](restarting-services.html) too.
1. [Make the role repeatable](making-roles-repeatable.html) so that the role is safe (and speedy) to run more than once on the same target computer.
1. You may need to [debug the role](debugging-failing-roles.html) if you run into problems.
1. [Add dependencies to the role](adding-dependencies-to-roles.html) to make sure that the role has everything it needs in order to succeed.  In larger playbooks, you'll find it really helps if you [create groups of roles](using-meta-roles.html) to simplify your dependencies.
1. Add [support for multiple operating systems](multiple-operating-systems.html) as and when required.

<div class="callout info" markdown="1">
#### Bookmark This Page

If you're reading this book in a browser, bookmark this page now, so that you can keep coming back to it as you build up your confidence and experience with role-building in Ansible.
</div>

Let's get started by looking at how to [install software from packages](installing-software-from-packages.html).