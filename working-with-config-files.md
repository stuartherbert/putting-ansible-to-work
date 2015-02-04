---
layout: top-level
title: Working With Config Files
prev: '<a href="temporary-install-scripts.html">Prev: Using Temporary Install Scripts</a>'
next: '<a href="restarting-services.html">Next: Restarting Services</a>'
---

# Working With Config Files

Once you've installed a piece of software on the target computer (either [from a package](installing-software-from-packages.html) or [built from source](building-software-from-source.html)), you'll probably want to replace the default config file with a config file of your own.

## Types Of Config File

Config files come in two flavours: static config files, and templated config files.

* __Static config files__ are text (or binary) config files that can be uploaded to the target computer as-is.
* __Templated config files__ contain variables that need to be replaced when Ansible uploads the file to the target computer.

## Installing A Static Config File

Follow these steps:

1. Prepare the static config file.
1. Create the folder where the config file will live.
1. Upload the config file to the target computer.
1. Restart the service that the config file is for.

### Prepare The Config File

You need to create a config file for Ansible to load.  The normal approach is to take an existing config file for the app, edit it to suit, and then add it to the role's `files/` folder.

For example, the config file for the [Redis server](http://redis.io) would go in this folder in my Ansible playbook:

{% raw %}
<pre>
roles/
  stuartherbert.redis-server/
    files/
      redis.conf
</pre>

### Creating The Config Folder

On Linux, global config files normally live somewhere under `/etc`.  Exactly where will depend on the app that you're installing.

Use the [file module](http://docs.ansible.com/file_module.html) to create the folder where the config will live.

<pre>
---
# file: roles/stuartherbert.redis-server/tasks/enable.yml

- name: create the config folder for Redis
  file:
    path: /etc/redis
    state: directory
    owner: root
    group: root
    mode: 0755
  sudo: True
</pre>

<div class="callout warning" markdown="1">
#### Always Create The Config Folder

If you try to upload a file into a folder that doesn't exist, Ansible won't attempt to create that folder for you. It will halt with a fatal error.

Whenever you are uploading a file to a target computer, always create the folder first.
</div>

### Uploading The Config File

Use the [copy module](http://docs.ansible.com/copy_module.html) to upload your config file to the target computer.

<pre>
---
# file: roles/stuartherbert.redis-server/tasks/enable.yml

- name: upload the config file
  copy:
    src: redis.conf
    dest: /etc/redis/redis.conf
    owner: root
    group: root
    mode: 0644
  sudo: True
</pre>

<div class="callout info" markdown="1">
#### Setting Ownership And Permissions

Whenever you upload a file to a target computer, always set the `owner:`, `group:` and `mode:` of the uploaded file.  By being explicit, you ensure that each file is installed with the correct security permissions each and every time.
</div>

### Restarting The Service

Once you have uploaded the config file, restart the service so that the app picks up the new config.  This is done by adding a `notify:` statement to the task:

<pre>
---
# file: roles/stuartherbert.redis-server/tasks/enable.yml

- name: upload config file
  copy:
    src: redis.conf
    dest: /etc/redis/redis.conf
    owner: root
    group: root
    mode: 0644
  sudo: True
  notify:
  - restart redis
</pre>

The `notify:` statement tells Ansible to call a _handler_ called _restart redis_. I look at how to write handlers in [Restarting Services](restarting-services.html).

## Installing A Templated Config File

The steps for installing a templated config file are very similar to [installing a static config file](#installing_a_static_config_file).

1. Define the variables.
1. Prepare the templated config file.
1. Create the folder where the config file will live.
1. Upload the config file to the target computer.
1. Restart the service that the config file is for.

### Define The Variables

Take a look at the config file, and work out which settings in the config file need to turned into variables.

<div class="callout info" markdown="1">
#### Which Settings Should Become Variables?

Work through the config file, and look for the following:

* Any setting that might be different on different target computers.
* Any setting that needs hostnames / IP addresses of other target computers.

For example: if we look at the config file for the Redis server, we could take the port number that the Redis server listens on and turn that into a variable.  If we were setting up a Redis cluster, we'd also have variables for the host and port of the master server.
</div>

It's a good idea to set default values for every variable that can have a sensible default.  Put these in your role's `defaults/main.yml` file:

<pre>
roles/
  stuartherbert.redis-server/
    defaults/
      main.yml
</pre>

<pre>
---
# file: roles/stuartherbert.redis-server/defaults/main.yml

stuartherbert_redis_server_port: 6018
</pre>

<div class="callout info" markdown="1">
#### Prefix Your Variable Names

In Ansible, it's best to think of all variables as being global variables.  The variables that you set in a role have to share the same scope as the variables in every other role in your playbook, including any third-party roles that you download and use.

Prefix your variable names with the name of your role.  You can't use periods and hyphens in variable names; turn these into underscores.  For example, the role _stuartherbert.redis-server_ becomes the prefix *stuartherbert_redis_server*.  This does result in variable names that are a bit of a mouthful :(
</div>

### Prepare The Config File

Take the config file for the app, replace the settings with your variables, and then add it to the role's `files/` folder.

For example, the config file for the [Redis server](http://redis.io) would go in this folder in my Ansible playbook:

<pre>
roles/
  stuartherbert.redis-server/
    templates/
      redis.conf
</pre>

and the variables would look like this:

<pre>
# Redis configuration file example

################################ GENERAL  #####################################

# Accept connections on the specified port, default is 6379.
# If port 0 is specified Redis will not listen on a TCP socket.
port {{ stuartherbert_redis_server_port }}
</pre>

<div class="callout info" markdown="1">
#### Learning How To Use Jinja2

Ansible uses the [Jinja2 templating engine](http://jinja.pocoo.org) to expand your variables in your templated config files.  Ansible also turns the inventory into variables that you can use in your config files if needed.

You can find more information here:

* [Jinja2 docs](http://jinja.pocoo.org/docs/)
* [Ansible: variables](http://docs.ansible.com/playbooks_variables.html)
</div>

### Creating The Config Folder

On Linux, global config files normally live somewhere under `/etc`.  Exactly where will depend on the app that you're installing.

Use the [file module](http://docs.ansible.com/file_module.html) to create the folder where the config will live.

<pre>
---
# file: roles/stuartherbert.redis-server/tasks/enable.yml

- name: create the config folder for Redis
  file:
    path: /etc/redis
    state: directory
    owner: root
    group: root
    mode: 0755
  sudo: True
</pre>

<div class="callout warning" markdown="1">
#### Always Create The Config Folder

If you try to upload a file into a folder that doesn't exist, Ansible won't attempt to create that folder for you. It will halt with a fatal error.

Whenever you are uploading a file to a target computer, always create the folder first.
</div>

### Uploading The Config File

Use the [copy module](http://docs.ansible.com/copy_module.html) to upload your config file to the target computer.

<pre>
---
# file: roles/stuartherbert.redis-server/tasks/enable.yml

- name: upload the config file
  copy:
    src: redis.conf
    dest: /etc/redis/redis.conf
    owner: root
    group: root
    mode: 0644
  sudo: True
</pre>

<div class="callout info" markdown="1">
#### Setting Ownership And Permissions

Whenever you upload a file to a target computer, always set the `owner:`, `group:` and `mode:` of the uploaded file.  By being explicit, you ensure that each file is installed with the correct security permissions each and every time.
</div>

### Restarting The Service

Once you have uploaded the config file, restart the service so that the app picks up the new config.  This is done by adding a `notify:` statement to the task:

<pre>
---
# file: roles/stuartherbert.redis-server/tasks/enable.yml

- name: upload config file
  copy:
    src: redis.conf
    dest: /etc/redis/redis.conf
    owner: root
    group: root
    mode: 0644
  sudo: True
  notify:
  - restart redis
</pre>

The `notify:` statement tells Ansible to call a _handler_ called _restart redis_. I look at how to write handlers in [Restarting Services](restarting-services.html).