---
layout: top-level
title: Miscellaneous Tips
prev: '<a href="ansibleworks-galaxy.html">Prev: Using Third-Party Roles From AnsibleWorks Galaxy</a>'
next: '<a href="feedback.html">Next: Feedback Most Welcome</a>'
---

# Miscellaneous Tips

To finish this book, here's a list of Ansible tips that don't really fit into anywhere else in the book, but which will help.

## Avoid Paramiko For Now

Throughout this book, we've been making Ansible connect to target computers via SSH; more specifically, by using a real SSH client.  This can be a little slow, especially as your playbooks grow in size and complexity.

Very early versions of Ansible got around this performance penalty by using a Python SSH library called Paramiko instead of a real SSH client.  Paramiko is definitely faster, but in my experience not reliable.  I've regularly had Ansible hang when using Paramiko - something that never happens with using a real SSH client.

## How To List All Of The Gathered Facts

<pre>
ansible &lt;hostname&gt; -m setup
</pre>

`<hostname>` must be a host listed in the Inventory.

## How To Gather Even More Facts

Ansible's built-in fact gathering generates a pretty comprehensive list.  If factor or ohai are installed on the target computer, Ansible will also use them to gather facts.

<pre>
# Ubuntu, Debian
sudo apt-get install facter ohai
</pre>

<div class="callout warning" markdown="1">
#### Will You Use The Facts?

Gathering facts using facter and ohai takes time, and adds to how long it takes your playbook to run against a target machine.  If you're not going to use the facts in your playbook, do you really need to gather them in the first place?
</div>

## Run A Single Play On A Single Computer

If you want to run just one play (for example, you want to make just one change to a computer),

<pre>
ansible-playbook -i inventory/&lt;hostfile&gt; -K plays/&lt;play.xml&gt;
</pre>

## Uploading To The Remote User's Home Folder

If you want to upload a file into the remote user's home folder, use `~/` at the start of the __dest=__ path, and remember to add a __sudo: false__ statement to the task:

<pre>
---
- name: upload new bashrc file
  action: file src=bashrc dest=~/.bashrc mode=0644
  sudo: false
</pre>