---
layout: top-level
title: Supporting Multiple Target Computers
prev: '<a href="multiple-operating-systems.html">Prev: Supporting Multiple Operating Systems</a>'
next: '<a href="miscellaneous-tips.html">Next: Miscellaneous Tips</a>'
---

# Supporting Multiple Target Computers

## Naming Your Inventory Files

If `hostfile` in `ansible.cfg` points to a folder (or you use the `-i` switch to `ansible-playbook` to point to a folder), then Ansible will ignore any files that end in .ini in that folder.

## Setting Default Values For All Target Computers

Put these in group_vars/all.yml.

## Setting The Remote User For A Target Computer

## Setting The SSH Key For A Target Computer

## Having Different Remote Users On Different Target Computers

{% raw %}

<pre>
---
# file: site.yml
- hosts: all
  remote_user: "{{ remote_user }}"

- include: play1.yml
- include: play2.yml
</pre>

<pre>
---
# file: inventory/host_vars/host1.yml
remote_user: ec2-user
</pre>

<pre>
---
# file: inventory/host_vars/host2.yml
remote_user: vagrant
</pre>

{% endraw %}