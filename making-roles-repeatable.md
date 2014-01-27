---
layout: top-level
title: Making Roles Repeatable
prev: '<a href="restarting-services.html">Prev: Restarting Services</a>'
next: '<a href="debugging-failing-roles.html">Next: Debugging Failing Ansible Roles</a>'
---

# Making Roles Repeatable

## What Do You Mean Repeatable?

## Why Does A Role Need To Be Repeatable?

## Adding An Inspect Step To Your Roles

## Detecting Installed Version Numbers

Many pieces of software follow the 'X.Y.Z' numbering scheme.  You can't use Jinja2's built-in filters to convert these to numbers that you can compare against, because of the multiple decimal points.

Options:

1. use awk and cut to only look at the first part of the version number
1. a custom Jinja2 filter to compare version numbers
1. a custom Ansible module to compare version numbers

I need to investigate these.

## Testing Your Repeatable Roles