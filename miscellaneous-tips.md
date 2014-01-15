---
layout: top-level
title: Miscellaneous Tips
prev: '<a href="multiple-target-computers.html">Prev: Supporting Multiple Target Computers</a>'
next: '<a href="feedback.html">Next: Feedback Most Welcome</a>'
---

# Miscellaneous Tips

To finish this book, here's a list of Ansible tips that don't really fit into anywhere else in the book, but which will help.

## Avoid Paramiko For Now

Throughout this book, we've been making Ansible connect to target computers via SSH; more specifically, by using a real SSH client.  This can be a little slow, especially as your playbooks grow in size and complexity.

Very early versions of Ansible got around this performance penalty by using a Python SSH library called Paramiko instead of a real SSH client.  Paramiko is definitely faster, but in my experience not reliable.  I've regularly had Ansible hang when using Paramiko - something that never happens with using a real SSH client.

## How To List All Of The Gathered Facts

I've no idea - it's time that I did.

## Uploading To The Remote User's Home Folder

If you want to upload a file into the remote user's home folder, use `~/` at the start of the __dest=__ path, and remember to add a __sudo: false__ statement to the task:

<pre>
---
- name: upload new bashrc file
  action: file src=bashrc dest=~/.bashrc mode=0644
  sudo: false
</pre>