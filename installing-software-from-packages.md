---
layout: top-level
title: Installing Software From Packages
prev: '<a href="building-a-role.html">Prev: Building A Role</a>'
next: '<a href="building-software-from-source.html">Next: Building Software From Source</a>'
---

# Installing Software From Packages

When you build a role for Ansible, you'll go through these steps:

1. Make the role install software from packages (this chapter), or [build software from source](building-software-from-source.html) perhaps by [using temporary install scripts](temporary-install-scripts.html).  You may need to [install config files](working-with-config-files.html) and [restart services](restarting-services.html) too.
1. [Make the role repeatable](making-roles-repeatable.html) so that the role is safe (and speedy) to run more than once on the same target computer.  You may need to [debug the role](debugging-failing-roles.html) if you run into problems.
1. [Add dependencies to the role](adding-dependencies-to-roles.html) to make sure that the role has everything it needs in order to succeed.
1. Add [support for multiple operating systems](multiple-operating-systems.html) as and when required.