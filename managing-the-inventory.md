---
layout: top-level
title: Managing The Inventory
prev: '<a href="multiple-operating-systems.html">Prev: Supporting Multiple Operating Systems</a>'
next: '<a href="miscellaneous-tips.html">Next: Miscellaneous Tips</a>'
---

# Managing The Inventory

## Naming Your Inventory Files

If `hostfile` in `ansible.cfg` points to a folder (or you use the `-i` switch to `ansible-playbook` to point to a folder), then Ansible will ignore any files that end in .ini in that folder.

## Setting Default Values For All Target Computers

Put these in group_vars/all.yml.

## Setting The Remote User For A Target Computer

## Setting The SSH Key For A Target Computer