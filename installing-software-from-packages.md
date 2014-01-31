---
layout: top-level
title: Installing Software From Packages
prev: '<a href="building-a-role.html">Prev: Building A Role</a>'
next: '<a href="building-software-from-source.html">Next: Building Software From Source</a>'
---

# Installing Software From Packages

The quickest way to turn a target computer into a machine that's ready to be used is to install pre-packaged software provided by your operating system or third parties such as [NPM](http://npmjs.org).

Ansible supports most of the major packaging managers for Linux, and plenty of third-party packaging managers too.  You'll find [a full list of packaging modules](http://docs.ansible.com/list_of_packaging_modules.html) in the official Ansible docs.

<div class="callout danger" markdown="1">
#### Beware Third-Party Packages

If your operating system doesn't provide a package for the software you need, and neither does the software's author, you should always [build the software from source](building-software-from-source.html).

__Downloading a package made by a third party who you do not know is dangerous.__

* You do not know that the package is safe to install on your target computer.
* You do not know that the package contains software that is safe to run on your target computer.

Hacked servers can damage your reputation, cost your business money, and (if you hold personal data) lead to further legal and reputational problems.  Making sure your servers only contain software you trust is one part of the puzzle of keeping your servers and data secure.
</div>

## How To Install The Latest Version Of A Package

Most of the time, you'll want to install the latest version of a package.  This is really easy to do:

<pre>
---
# file: roles/stuartherbert.git/tasks/ubuntu_install.yml

- name: "Ubuntu: install the 'git' command"
  apt:
    pkg: git
    state: latest
  sudo: True
</pre>

## How To Install A Specific Version Of A Package

Sometimes, you'll want to install a specific version of a package.

{% raw %}

<pre>
---
# file: roles/stuartherbert.git/tasks/ubuntu_install.yml

- name: "Ubuntu: install the 'git' command"
  apt:
    pkg: "git={{git_version}}"
    state: present
  sudo: True
</pre>

{% endraw %}

<pre>
---
# file: roles/stuartherbert.git/defaults/main.yml
git_version: 1:1.8.3.2-1
</pre>

You can then override the `git_version` variable using the Inventory's _host\_vars_ or _group\_vars_ as required.

<div class="callout warning" markdown="1">
#### Version Numbers May Not Be Portable

Unfortunately, there are a number of problems with trying to install specific versions of packages across different operating systems.

* Different operating systems ship with different versions of the same software.  For example, on Ubuntu 13.10, the version of Git is (at the time of writing) __1:1.8.3.2-1__, whilst on CentOS 6 it is __1.7.1__.
* The version numbers for pre-built packages can be specific to the operating system you're running on the target computer.  The Git package is a great example of this, as the Ubuntu version (at the time of writing) is __1:1.8.3.2-1__, compared to __1.7.1__ on CentOS 6.
* Even if the version numbers are the same, it's very common for Linux distributions to include additional changes to a piece of software, such as backporting bug fixes and security patches.  This can lead to the same version of the software having subtle (and not-so-subtle) differences on different operating systems.

The pragmatic approach is to only require target computers running the same operating system to have the same version of the software installed - and that's normally done by just using the latest package available.  If you really do need total control over software versions, you're better off [building the software from source](building-software-from-source.html) instead.
</div>

## How To Install Pending Software Updates

I find it very handy to upgrade all the packages on a target computer to the latest version (but see the huge caveat below!)  This is easily done with the following tasks:

<pre>
---
# file: roles/stuartherbert.software-updates/tasks/ubuntu_install.yml

- name: "Ubuntu: get latest software updates"
  command: apt-get update
  sudo: True

- name: "Ubuntu: install software updates"
  command: apt-get upgrade -y
  sudo: True
</pre>

This _get / install_ approach is very easy to adapt for other Linux distros.

It's worth looking at the Ansible docs for your package manager of choice, as there can be handy time-saving features built-in which help with software updates.  For example, on Ubuntu, we can use the [apt module](http://docs.ansible.com/apt_module.html) to only update the list of software updates if it has been at least an hour since our last attempt:

<pre>
# file: roles/stuartherbert.software-updates/tasks/ubuntu_install.yml

- name: "Ubuntu: install Aptitude"
  apt:
    pkg: aptitude
    state: latest
  sudo: True

- name: "Ubuntu: get latest software updates"
  apt:
    update_cache: yes
    cache_valid_time: 3600
  sudo: True

- name: "Ubuntu: install latest software updates"
  apt:
    update: safe
  sudo: True
</pre>

I use the first method - running the actual commands I'd use in a Terminal - because I understand exactly what it will do.  I get nervous around package updates, as I don't want a broken target computer to fix, and prefer to be cautious and stick with what I know!

<div class="callout warning" markdown="1">
#### Do You Want The Latest Versions Of Everything?

Sometimes, software upgrades are not backwards-compatible, and upgrading _everything_ will leave you with a broken target computer that you have to fix by hand.  You might be better off with roles that install specific versions of packages instead.

Try both approaches, and see which works for you.
</div>

<div class="callout info" markdown="1">
#### How To Upgrade The Operating System

You _can_ upgrade the operating system using Ansible (for example, move from Ubuntu 12.04 to Ubuntu 13.10, or CentOS 5.x to CentOS 6.latest), but __I don't recommend it__.  You'll find a full discussion of this in [Managing Your Base Operating System](managing-your-base-operating-system.html) later in this book.
</div>

## How To Remove A Package

If you ever need to remove a package, it's very easy to do:

<pre>
---
- name: uninstall 'Git'
  apt:
    pkg: git
    state: absent
</pre>

<div class="callout warning" markdown="1">
#### Removing Packages Is Often A 'Smell'

Just as you get _code smells_ when writing code, you also get them when automating software deployment.  Whenever you need to remove a package, it's important to ask yourself why.

Sometimes, you might be better off re-installing the operating system from scratch and then re-running your playbook.
</div>