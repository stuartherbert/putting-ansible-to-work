---
layout: top-level
title: Preface
prev: '<a href="index.html">Prev: Putting Ansible To Work</a>'
next: '<a href="introduction.html">Next: Introduction</a>'
---

# Preface

Back in November 2012, I had a difficult problem to solve.

I'd recently created [Storyplayer](http://datasift.github.io/storyplayer), a new test automation tool, which I was using at work to move from manual ad-hoc testing to repeatable automated testing.  Repeatable tests need repeatable test environments, and back then I simply didn't have any to use in our testing.

Although we'd been using Chef at work for years to automate deployments, it was part of the problem, not part of the solution.  We had a single Chef server controlling all of our environments, and quite rightly our Operations team couldn't give out access to it to our engineers.  It would have been too easy for a simple mistake to trash our production environment, something no startup can afford.  Resources were extremely tight, and there was neither the time nor the spare hardware to setup and maintain a separate Chef server just for testing.

I needed something that would run on a reasonable-spec dev workstation, that wouldn't need a dedicated server that I didn't have, and that could be shared with all our engineers without risk.  Ansible fit the bill perfectly.

Fourteen months later, and not only does Ansible build all of our test environments, but we're starting to use it to build workstations for new starters when they arrive, so that they get up and running as quickly as possible.  We've baked support for Ansible into Storyplayer, which is now available as an open-source tool.

I've learned a lot about Ansible in those fourteen months, and through this book I hope to share that experience with you.