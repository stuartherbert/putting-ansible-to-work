---
layout: top-level
title: Supporting Multiple Operating Systems
prev: '<a href="using-meta-roles.html">Prev: Using Meta-Roles</a>'
next: '<a href="managing-the-inventory.html">Next: Managing The Inventory</a>'
---

# Supporting Multiple Operating Systems

At first, it is very tempting to go with an extremely simple structure under the `plays/` folder.  This stores problems for the future.  Over time, you may need to add support for additional Linux distros (for example, you might start supporting CentOS as well as Ubuntu); you will almost certainly need to support multiple versions of the Linux distro that you and your team use daily (for example, Ubuntu 12.10 and Ubuntu 13.10).

## Support Change

Unless you're doing something very trivial in your provisioning, an upgrade from one version of an operating system to the next will break your provisioning scripts.