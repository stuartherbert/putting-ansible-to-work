---
layout: top-level
title: Organising Your Ansible Files
prev: '<a href="how-tasks-work.html">Prev: How Tasks Work</a>'
next: '<a href="planning-a-role.html">Next: Planning A Role</a>'
---

# Organising Your Ansible Files

Now that you've created your first playbook, let's look at the structure of a playbook repo in detail.  This will help you see how the files we'll create in the next few chapters fit into the overall design of a playbook.

## Playbook Repo Structure

The playbook repo is structured like this:

<pre>
ansible.cfg

inventory/
    group_vars/
    	all.yml
    	&lt;group.yml&gt;
    host_vars/
    	&lt;hostname.yml&gt;
    hosts

plays/
    &lt;os&gt;/
        &lt;play.yml&gt;

roles/
    &lt;os&gt;/
        &lt;role&gt;/
            files/
            handlers/
                main.yml
            meta/
                main.yml
            tasks/
                main.yml
            templates/

site.yml
</pre>

and contains the following:

## ansible.cfg

`ansible.cfg` is a .ini-format config file that Ansible reads whenever we run it.  We use it to override defaults, and to avoid having to pass lots of parameters on the command-line all the time.

Ansible loads the first config file that it finds, and stops looking for other config files.  If you also have a `~/.ansible.cfg` or `/etc/ansible/ansible.cfg` file, this will override all the settings in those files; they will not be loaded at all.

Put the following in your `ansible.cfg` file:

<pre>
[defaults]
hostfile=inventory
roles_path=roles
</pre>

If you forget to create the `ansible.cfg` file, then Ansible will be unable to find where we have stored the Inventory or the roles!  (Ansible expects the Inventory to be in the top-level folder, and Ansible expects to find roles in the same folder as the `<play.yml>` files.  We're moving both to make life easier down the road).

Ansible's GitHub repo contains [a full list of valid ansible.cfg settings](https://raw.github.com/ansible/ansible/devel/examples/ansible.cfg).

## inventory/

`inventory/` is the folder where we store our list of target computers, and the Ansible variables for each target computer and group of computers.

The official Ansible documentation recommends that [the Inventory goes in your top-level folder](http://docs.ansible.com/playbooks_best_practices.html#directory-layout), along with your `<play.yml>` files.  In my experience, the top-level folder quickly gets very cluttered, and the less that goes there the better for long-term management.  Put everything in the `inventory/` sub-folder from the very start to keep your top-level folder manageable.

## inventory/group_vars/

`inventory/group_vars/` is the folder where we store any Ansible variables that we want to set for each group of target computers.

The `group_vars/` folder has to be in the same folder that your inventory files are in, otherwise Ansible cannot find the group vars.  (Ansible will also look for `group_vars/` in the same folder that your playbook is in.  The `group_vars/` are really part of the Inventory; put them in the `inventory/` folder.)

## inventory/group_vars/all.yml

`inventory/group_vars/all.yml` is a special case of the regular `<group.yml>` file.  It contains Ansible variables and values that apply to all groups (and, therefore, all target computers in the Inventory).

Put all the variables that need default values in this file.

Ansible looks for and loads __all__ of these files in order:

* `inventory/group_vars/all.yml`
* `inventory/group_vars/<group.yml>`
* `inventory/host_vars/<hostname.yml>`

The contents of the files are merged together.  This allows the `<group.yml>` files to override the `all.yml` file, and for the `<hostname.yml>` file to override both of them.

__Note__ that there is no equivalent `inventory/host_vars/all.yml` file.

## inventory/group_vars/&lt;group.yml&gt;

`inventory/group_vars/<group.yml>` is a YAML file that contains Ansible variables and their values.  These variables are set when Ansible is running plays for any target computer in that group.

See [inventory/group_vars/all.yml](#inventory/group_vars/all.yml) for details on the order in which Ansible loads variables from various places.

## inventory/host_vars/

`inventory/host_vars/` is the folder where we store any Ansible variables that we want to set on a host-by-host basis.

The `host_vars/` folder has to be in the same folder that your inventory files are in, otherwise Ansible cannot find the host vars.  (Ansible will also look for `host_vars/` in the same folder that your playbook is in.  The `host_vars/` are really part of the Inventory; put them in the `inventory/` folder.)

## inventory/host_vars/&lt;hostname.yml&gt;

`inventory/host_vars/<hostname.yml>` is a YAML file that contains Ansible variables and their values.  These variables are set when Ansible is running plays against the target computer called &lt;hostname&gt;.

&lt;hostname&gt; can be:

* a fully-qualified domain name (e.g. www.example.com)
* a hostname that your computer can resolve (e.g. just 'www' instead of 'www.example.com')
* an IP address (e.g. 127.0.0.1)

See [inventory/group_vars/all.yml](#inventory/group_vars/all.yml) for details on the order in which Ansible loads variables from various places.

## inventory/hosts

`inventory/hosts` is an inventory file, in .ini format.  It contains a list of target computers, and the groups that they are organised into.

A simple example would be:

<pre>
[web_servers]
www1.example.com
www2.example.com

[database_servers]
db1.example.com
</pre>

These [static inventory files are very powerful](http://docs.ansible.com/intro_inventory.html), and can do a lot more than I cover in this book.

One of the things you can do is put host vars inside the static inventory file.  This can quickly clutter up your static inventory file.  Put the host vars inside a `<host.yml>` file inside the `inventory/host_vars` folder instead to keep your static inventory file readable.

The official Ansible documentation recommends that [you should create one inventory file per environment](http://docs.ansible.com/playbooks_best_practices.html#how-to-arrange-inventory-stage-vs-production) to make it easier to see which environment each target computer belongs to.  For example, you would create `inventory/production` for all of your production servers, and `inventory/staging` for all of your staging servers.  This is good advice to follow as your Inventory grows.

Ansible will automatically load every file in the `inventory/` folder as a static inventory file, __provided__ that each file has no file extension.  It's to do with the powerful [dynamic inventory support](http://docs.ansible.com/intro_dynamic_inventory.html), an advanced topic that I don't cover in detail in this book.

## plays/

`plays/` is the folder where all of the `<play.yml>` files go, such as `web-dev.yml` from [our example first playbook repo](first-playbook.yml).

The official Ansible documentation recommends that [your plays go in your top-level folder](http://docs.ansible.com/playbooks_best_practices.html#directory-layout), along with your Inventory.  In my experience, the top-level folder quickly gets very cluttered, and the less that goes there the better for long-term management.  Put your plans in a sub-folder from the very start to keep your top-level folder manageable.

## plays/&lt;os&gt;/

This is a folder that stores all plays that are valid for `<os>`, where `<os>` is the name and (optionally) the version of an operating system.  This approach makes it much easier to [support multiple operating systems](multiple-operating-systems.html) in your playbook repo.

For example,

<pre>
plays/
    ubuntu_13_04/
    ubuntu_13_10/
</pre>

## plays/&lt;os&gt;/&lt;play.yml&gt;

`plays/<os>/<play.yml>` is a YAML file that stores the list of roles to apply to every target computer that is in a given group.

<pre>
---
- hosts: &lt;group-name&gt;
  roles:
  - &lt;os&gt;/&lt;role1&gt;
  - &lt;os&gt;/&lt;role1&gt;
</pre>

Create a new `<play.yml>` file for every `<group-name>` that you want to use in the Inventory.  I always create one `<play.yml>` file for each group, and name them the same, so that it's always easy to find which files to read and edit when I want to remind myself what the group is for.

Plays are very powerful, and have been part of Ansible for much longer than roles.  We're going to keep them very simple in this book, and push all the hard work into the roles and tasks instead.

## roles/

`roles/` is the folder where we store all of the detailed instructions for how to provision using Ansible.

Ansible expects to find your `roles/` folder where your `<play.yml>` files are stored.  You must create the `ansible.cfg` file [listed above](#ansiblecfg) to make sure that Ansible looks in the `roles/` folder.

## roles/&lt;os&gt;/

`roles/<os>/` is a folder that stores roles that work on a specific operating system.  `<os>` is the name of the operating system, such as:

* ubuntu
* ubuntu_12_04
* ubuntu_13_10

This approach makes it much easier to [support multiple operating systems](multiple-operating-systems.html) in your playbook repo.

## roles/&lt;os&gt;/&lt;role&gt;/

`roles/<os>/<role>/` is a folder that contains a single role.  Ansible uses the folder name as the name of the role.

Add roles to `<play.xml>` files to build up the instructions for how to provision a group of computers.  Roles that are not in a `<play.xml>` file will never get executed.

Create a new `<role>/` folder for every new thing that you want to install using Ansible.  It is best to keep each role as small and focused as possible.  That way, you can re-use roles in different `<play.xml>` files, and it makes it easier to test and update a role without breaking any other roles.

A `<role>/` folder must contain one of:

* `handlers/`
* `meta/`
* `tasks/`

otherwise it does nothing.

## roles/&lt;os&gt;/&lt;role&gt;/files/

`roles/<os>/<role>/files/` is a folder that holds any files that the `<role>` copies up to the target computer.  Use the [copy module](http://docs.ansible.com/copy_module.html) to perform the upload.

## roles/&lt;os&gt;/&lt;role&gt;/handlers/

`roles/<os>/<role>/handlers/` is a folder that contains any tasks that can be triggered by other modules.  There must be a `main.yml` file in this folder which contains the tasks.  `main.yml` can include other task files if required.

For example:

<pre>
---
# file roles/ubuntu-13.10/apache2/handlers/main.yml

- name: restart Apache2
  action: command /etc/init.d/apache2 restart
  sudo: True

- name: stop Apache2
  action: command /etc/init.d/apache2 stop
  sudo: True
</pre>

Most roles will not need to define any handlers, and won't need to have a `handlers/` folder.

Handlers can be called by any role, not just the role that they are defined in.  Ansible cannot find a handler if the role that defines it hasn't been included in a `<play.xml>` file.

I cover handlers in detail in [Restarting Services](restarting-services.html) later in the book.

## roles/&lt;os&gt;/&lt;role&gt;/meta/

`roles/<os>/<role>/meta/` is a folder that contains a list of any roles that this role depends on.  There must be a `main.yml` file in this folder which contains the list.

For example:

<pre>
---
# file roles/ubuntu-13.10/mod_php/main.yml

dependencies:
- { role: ubuntu-13.10/apache2 }
</pre>

Most roles will not need to define any dependencies, and won't need to have a `meta/` folder.

I cover dependencies in detail in [Adding Dependencies To Roles](adding-dependencies-to-roles.html) later in the book.

## roles/&lt;os&gt;/&lt;role&gt;/tasks/

`roles/<os>/<role>/tasks/` is a folder that contains a list of tasks to execute when the role is applied to a target computer.  There must be a `main.yml` file in this folder which contains the list of tasks.  `main.yml` can include other task files if required.

For example:

<pre>
---
# file roles/ubuntu-13.10/curl/main.yml

- name: install curl
  action: apt pkg=curl state=latest
</pre>

I cover tasks in detail in [Building Roles](building-roles.html) later in the book.

## roles/&lt;os&gt;/&lt;role&gt;/templates/

`roles/<os>/<role>/templates/` is a folder that holds any files that the `<role>` expands using Jinja2 and then copies up to the target computer.  Use the [template module](http://docs.ansible.com/template_module.html) to perform the upload.

I cover templates in detail in [Working With Config Files](working-with-config-files.html) later in this book.

## site.yml

`site.yml` is a YAML file that acts as the playbook.  It contains a list of all of the `<play.xml>` files that Ansible should use to see which target computers need which roles applying.

For example:

<pre>
---
# file site.yml

- include: plays/centos-6/webserver.yml
- include: plays/ubuntu-13.10/webserver.yml
- include: plays/ubuntu-13.10/dbserver.yml
</pre>

When you run Ansible, this is the file that you pass to it on the command line:

<pre>
ansible-playbook site.yml
</pre>

It is convention that this file is called `site.yml`.  Don't be surprised if you ever come across playbook repos where the authors have given this file a different name.

## Building Your Playbook

As you can see, there's quite a lot to an Ansible playbook.  Don't worry, with a bit of practice it's really a very straight-forward structure to work in.  Over the next few chapters, I'll show you how to go about adding entries to your Ansible playbook to provision most of the software that you're likely to deal with.