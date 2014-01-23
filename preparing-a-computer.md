---
layout: top-level
title: Preparing A Computer For Ansible
prev: '<a href="installing-ansible.html">Prev: Installing Ansible</a>'
next: '<a href="your-first-playbook.html">Next: Your First Playbook</a>'
---

# Preparing A Computer For Ansible

Before you can use Ansible to provision a computer, it needs to be prepared first, by following the steps in this chapter.

## What Type Of Computer Can Ansible Provision?

Ansible can be used to provision practically any UNIX-like computer that supports Python and SSH.  This can include:

* your own desktop or laptop
* a virtual machine either locally or in the cloud
* a physical server running in a datacenter

Throughout this book, I'm going to call the computer that is being provisioned the [target computer](key-concepts.html#target_computer).

You can use Ansible to provision multiple computers at a time.

## Install SSH

You need to make sure that an SSH daemon is installed and running on the target computer.  When you run Ansible, it is going to log into the target computer via SSH.

<pre>
# Ubuntu, Debian
sudo apt-get install openssh-server
sudo /etc/init.d/ssh start
</pre>

## Create A User Account

You need to create an account for Ansible to log into on the target computer.  This can be your own account, or it can be a dedicated account just for Ansible to use.

Most of the time, you should create an account for you (e.g. for me, the account would be called `stuart`), and use that when you run Ansible (I'll show you how in the next chapter).  That way, the target computer's logs will keep a record of who made changes to the computer.  This is very handy if there are several of you who share responsibility for maintaining your servers.

All virtual machines built by [vagrant](http://vagrantup.com) come with a `vagrant` user account.  You can reuse the `vagrant` account (and its existing SSH key) with Ansible instead.  You can also adopt the same idea and have a standard account on all of your machines for automated provisioning.  This is a good approach when building scratch dev and test environments on your side of your firewall.

Whichever way you go, I'm going to call this user the [remote user](key-concepts.html#remote_user) throughout the rest of this book.

You can have different remote users on each target computer.  In the next chapter, I'll show you how to tell Ansible which remote user to use when you run your first playbook, and in [Supporting Multiple Target Computers](multiple-target-computers.html) I'll show you how to configure Ansible so that it remembers which remote user to use for which target computer.

## Setup Sudo

You should never ever let anyone log into your computers remotely as `root`; you should always require them to log in as a normal user account first, and then use the `sudo` command to perform privileged operations.  It's better for the security of your server, and your logs will contain entries telling you who has used `sudo` to make changes to your servers.

Many of the operations that you'll want Ansible to do have to be done as `root`.  The remote user must be able to use the `sudo` command to run commands as `root`.  This is normally done by adding the remote user to the `admin` or `wheel` group on the target computer.  Check your operating system's `/etc/sudoers` config file for details.

<div class="callout warning" markdown="1">
#### What About Password-less Sudo?

Normally, the `sudo` command will prompt you for your password when you run it.  This is to make sure that it is you who is running the command, rather than someone who has somehow gotten into your account.  It's one way of limiting the damage if you do get hacked (or someone sits down at your computer whilst you're away for a few minutes).

It's there for a very good reason, and you should be very reluctant to disable `sudo` asking for your password.  Ansible can prompt you for your password before it logs into the target computer, so that it can still use `sudo`.

The only problem comes if you're using Ansible to manage multiple computers and you're not using the same password on every computer.  (This can happen when you're using different remote users on different target computers, for example).  As far as I know, Ansible doesn't ask you for the password for each remote user, and the only way to make this work is to reconfigure `sudo` to stop prompting the remote users for a password.

(You can use the Inventory and playbooks to tell Ansible which password to use for each remote user, but this is even more insecure than adopting password-less `sudo` imho).
</div>

## Install An SSH Key

Each remote user, on each target computer, needs an SSH keys installing so that Ansible can log into the computer to make changes.  You can use the same key everywhere (for example, use your own SSH key if Ansible is going to log in as you on each target computer).

If you don't want to type in your SSH key passphrase all the time, use `ssh-add` on your computer to setup an SSH agent.  (If you've never done this before, you might find Daniel Robbin's [Keychain](http://www.funtoo.org/Keychain) utility to be a great help).

Using more than one SSH key can cause problems - not with Ansible, but with SSH itself.  When you get to over six SSH keys, you might start seeing SSH failing to authenticate against the target computer.  That's because the OpenSSH client will happily try every SSH key you've told it about using `ssh-add`, and the OpenSSH server will ban you when six keys have failed.

The way around this is to make sure Ansible knows which key to use for which target computer.  I'll cover how to do this in [Supporting Multiple Target Computers](multiple-target-computers.html) towards the end of this book.

(Ansible does support logging into target computers via SSH using passwords, but no computer accessible from the Internet should ever allow remote access via passwords.  All it takes is one weak password, and the box can be cracked.  It's just not worth it.)

<div class="callout warning" markdown="1">
#### What About Passphrase-less SSH Keys?

It's not unusual for people to create SSH keys that have no passphrase to use with automation tasks.  They're essential for automation, because you're normally not there to type in a passphrase when automation software runs.

They are, however, a security risk, especially if the private key is installed onto your servers at all.  If someone breaks into one server and finds that private key, then they can log into any of your other servers where you're using your automation key.  That's a terrible situation to end up in.

Which way is right depends on how you're going to run Ansible:

* If you're always going to be running Ansible manually from your terminal window, then you never need to use a passphrase-less SSH key.  You're there to type in your passphrase, or run `ssh-add`.
* If Ansible is going to run automatically - from `cron`, or triggered by a software build script for example - then you'll need to use a passphrase-less SSH key.
</div>

## Install Python's SELinux Library

If the target computer is running SELinux (i.e. it is running RedHat or CentOS), then you need to install an extra Python library onto that computer:

<pre>
# RedHat, CentOS
sudo yum install libselinux-python
</pre>

## Automating The Prep Work

If you're tearing down and building servers / virtual machines on a regular basis, you'll want to automate as much of this prep work as possible.  You don't need anything sophisticated; a short `bash` script will do the job just fine.  You can store the script in your playbook repo alongside your playbooks, so that everything is kept in one place.

## Ready To Provision

When you get to here, you've finished preparing the target computer.  Ansible can now log into the target computer as the remote user using SSH, and can execute privileged commands using `sudo`.

It's time to write your first Ansible playbook.