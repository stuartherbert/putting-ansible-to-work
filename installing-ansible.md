---
layout: top-level
title: Installing Ansible
prev: '<a href="key-concepts.html">Prev: Key Concepts</a>'
next: '<a href="preparing-a-computer.html">Next: Preparing A Computer For Ansible</a>'
---

# Installing Ansible

## Where Do You Install Ansible?

Ansible gets installed onto your computer: your desktop and/or laptop.  You'll run it from there to provision your own computer, or other computers over your network via SSH.

## Do You Need A Server For Ansible?

You don't need to set aside a server to install Ansible onto.  Unlike alternatives, Ansible doesn't need a central server or central database at all.  Instead, Ansible reads all of its configuration from files stored in the playbook repo.

If there are several of you who share responsibility for managing a set of servers, each of you can run Ansible from your own computers, and share your Ansible playbook repo via source control.

## How Do You Install Ansible?

I recommend that you always install Ansible from [source](https://github.com/ansible/ansible).  Ansible is currently fast-moving, and grabbing the latest version will give you the latest bug fixes and features.  (As a rule of thumb, the official documentation always describes the very latest version).

First, install Python's YAML extension:

<pre>
# Ubuntu and Debian
sudo apt-get install python-yaml
</pre>

Then, install Python's Jinja2 templating library:

<pre>
# Ubuntu and Debian
sudo apt-get install python-jinja2
</pre>

Then, if you don't have it already, install Git:

<pre>
# Ubuntu and Debian
sudo apt-get install git
</pre>

Finally, install Ansible from Github:

<pre>
# Ubuntu and Debian
git clone https://github.com/ansible/ansible.git
cd ansible
sudo make install
</pre>

Ansible is now ready to run from your computer.

Before you can use Ansible, you need to prepare each computer that Ansible is going to provision.