---
layout: top-level
title: Why Ansible?
prev: '<a href="introduction.html">Prev: Introduction</a>'
next: '<a href="key-concepts.html">Next: Key Concepts</a>'
---
# Why Ansible?

_Ansible has become my go-to tool whenever I need to work around a package manager or am tempted to write a setup.sh script._

## What Problem Does Ansible Solve?

* Have you ever needed to build a new dev machine, perhaps for a new starter at work who isn't going to be familiar with all the software you use?
* Ever needed to build and rebuild test or staging environments reliably and repeatedly?
* Ever needed to add a machine to your production environment, and need to be confident that it will work first time?

These are the problems that Ansible solves.

## What Is Ansible?

[Ansible](http://ansibleworks.com) is a _provisioning tool_: you use it to install software onto computers in an automated way.

If we count the venerable [CFengine](http://cfengine.com) as a 1st generation provisioning tool, and both [Chef](http://getchef.com) & [Puppet](http://puppetlabs.com) as rival (and very successful) 2nd generation provisioning tools, then Ansible would count as one of the 3rd generation provisioning tools.  All of these tools borrow ideas from each other, and are worth considering before making a final choice.

Thanks to its simplicity and no-infrastructure demands, Ansible has already found a sweet spot amongst developers who are looking for an automated provisioning tool.  That's the constituency that I belong to, and who this book is for.

## Why Ansible?

There are many other provisioning tools out there - far too many to list here.  They are capable, well supported, and very popular.  So why should you give Ansible a look?

Here's why I picked Ansible:

* It uses Python, which already comes with your computer.

  There's a few Python modules to install (Jinja2, YAML, and SELinux support), and that's it.  No need to go through the pain of Ruby / RubyGems hell.  That is a huge amount of hassle completely avoided.

* Things happen in the order that you want.

  When you run Ansible, it goes through your [playbooks](key-concepts.html#playbooks) and executes them in the order that they are written in.  Need X to happen before Y?  Simply put X in your playbooks before Y.  Sounds obvious, but when Ansible started, other tools didn't work like this.

* Everything you want it to do is stored in a file.

  Your list of servers, their variables, your list of what software to install and how to install it - everything that describes what you want Ansible to do lives in a file.  That makes it incredibly easy to version control, and to share between people.

* Everything runs from your computer.

  You don't need to setup a server to run Ansible from.  You don't need to setup a database anywhere.  Just install Ansible on your computer, checkout your playbooks from source control, and then you're ready to run Ansible from the command-line.  Anyone can be up and running in a matter of minutes, and there's no central infrastructure to worry about maintaining.

Ansible isn't the only tool that does these things, but it's the best one I've found so far that does all of these things.

## What Does Ansible Run On?

Ansible runs best on Linux distributions and on other UNIX-like operating systems such as Apple's OS X.  It may run on Windows, but it has been many years since I used Windows, so I can't help you with that at all.

Let's take a look at how Ansible works, and the key concepts that you need to learn to get the most out of Ansible, before showing you how to install Ansible onto your computer.