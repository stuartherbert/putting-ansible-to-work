---
layout: top-level
title: Your First Playbook
prev: '<a href="preparing-a-computer.html">Prev: Preparing A Computer For Ansible</a>'
next: '<a href="organising-your-ansible-files.html">Next: Organising Your Ansible Files</a>'
---

# Your First Playbook

For your first playbook, we're going to automate installing [Curl](http://curl.haxx.se/) onto your computer.  This is a trivial example, which we'll build on in later chapters.

Rather than download a ready-made example repo, I want you to create your first playbook by hand, by following the instructions below.  Doing so will help you learn what goes where.  (It's fine to cut and paste to avoid typing errors, btw.)

## Create A Skeleton Playbook Repo

Throughout these examples, I'm assuming that you're storing your playbook repo in `~/ansible-playbooks`.  If you decide to create it somewhere else, you'll need to adjust the examples accordingly.

### Create A Structure

The first thing you need to do is to create a folder structure to store your playbook in.

<pre>
mkdir ~/ansible-playbooks
cd ~/ansible-playbooks
mkdir -p inventory/group_vars
mkdir -p inventory/host_vars
mkdir plays
</pre>

Save the following into the file `~/ansible-playbooks/ansible.cfg`:

<pre>
[defaults]
hostfile=inventory/
</pre>

### Create The Curl Role

Create the folder where the [tasks](key-concepts.html#tasks) for the 'curl' role will live:

<pre>
mkdir -p ~/ansible-playbooks/plays/curl/tasks
</pre>

Save the following into the file `~/ansible-playbooks/plays/curl/tasks/main.yml`:

<pre>
---
- include: curl-install.yml
</pre>

Save the following into the file `~/ansible-playbooks/plays/curl/tasks/curl-install.yml`:

<pre>
---
- name: install Curl
  action: apt pkg=curl state=latest
  sudo: true
</pre>

### Create The Web-Dev Play

Save the following into the file `~/ansible-playbooks/plays/web-dev.yml`

<pre>
---
- hosts: web-dev
  roles:
  - curl
</pre>

### Add The Play To The Playbook

Save the following into the file `~/ansible-playbooks/site.yml`

<pre>
---
- include: plays/web-dev.yml
</pre>

### Add Your Computer To The Inventory

Save the following into the file `~/ansible-playbooks/inventory/hosts`

<pre>
[web-dev]
localhost
</pre>

Save the following into the file `~/ansible-playbooks/inventory/host_vars/localhost.yml`

<pre>
---
ansible_connection: local
</pre>

Congratulations.  You've just created your first playbook.

## Running Your First Playbook

Run your first Ansible playbook:

<pre>
cd ~/ansible-playbooks
ansible-playbook -K site.yml
</pre>

Ansible will prompt you for your sudo password.  Type it in, and press ENTER to continue.

If all has gone well, you will see this on the screen:

<pre>
sudo password:

PLAY [web-dev] ****************************************************************

GATHERING FACTS ***************************************************************
ok: [localhost]

TASK: [curl | install Curl] ***************************************************
changed: [localhost]

PLAY RECAP ********************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0
</pre>

Congratulations.  You've just run your first playbook.

## What Did We Just Do?

We've just done the following:

1. Created a folder structure to hold all of our files for Ansible.

   This is something that you normally only do once, when you create a new playbook repo.

1. Created an Ansible role called 'curl'

   A role is the building block of Ansible.  You create roles to install software, and then pull them into plays to combine them into provisioning instructions.

   In this case, the 'curl' role uses the [apt module](http://docs.ansible.com/apt_module.html) to install the latest version of the package called 'curl' onto the target computer.  If the target computer is RedHat-based, you'd use the [yum module](http://docs.ansible.com/yum_module.html) instead to achieve the same thing.

1. Created the 'web-dev' play

   A play is a list of roles to apply to any target computer listed in the Inventory that matches the `hosts` string at the top of the file.  You create a new play for every different purpose a target computer can have.  In the Inventory, you tell Ansible which plays to run against a target computer; you can run more than one play against the same computer.

   In this case, the 'web-dev' play says to apply the 'curl' role to every member of the 'web-dev' group in the Inventory.

1. Added the 'web-dev' play to the playbook

   By convention, the playbook file in Ansible is called `site.yml`, and lives in the top-level folder of your playbook repo.  It contains a list of all of the plays that are available, and you use the Inventory to say which plays should run against which target computers.

1. Added 'localhost' to the Inventory

   The file `inventory/hosts` is a .ini file that contains a list of which hosts to run which plays against, and the file `inventory/host_vars/localhost.yml` contains a list of variables to use when provisioning the target computer called 'localhost'.

   In this case, we've added 'localhost' to the 'web-dev' group.  Every computer listed in the 'web-dev' group will have the 'web-dev' play run against it.  We've told Ansible to treat 'localhost' as the current computer, so that it doesn't login via SSH to act.

1. Executed the playbook

   Finally, we've run the playbook, by telling Ansible where our playbook is.  This has executed the 'web-dev' play, which has applied the 'curl' role to the target computer 'localhost'.

Although this is a trivial example, these are the same actions that you'd take no matter how complex your playbook or inventory.  Working with Ansible is a cycle of creating roles, combining them into plays, updating the Inventory with new plays and target computers, and running Ansible to execute your changes.