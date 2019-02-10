---
layout: post
title: "Things I wish I knew"
date:	2010-04-21
category: tech
tags: vmware virtualization
---

Following are a few things that I wish I knew before I started my VMWare deployment.  These are things that nobody seems to tell you, no matter where you look.  I will probably be updating this list from time to time.

* You can only create 2 TB datastores in VMFS version 3.  I know that all the guides and information say that the largest files size is 2TB.  That is true as well.
* Don’t use VLAN 1.  Ever.  Seriously.
* Don’t use App Pools.  They don’t work like you think they do.  Do some research before you decide to use them.
* If you have to use snapshots, do not let them stay around for very long!  Delete old snapshots before too much time passes.  Snapshots create sparse volumes that keep changes written to the disk since the snapshot was created.  Each snapshot can create a new disk up to the size of the virtual disk.  If you have a 20GB vmdk with 2 snapshots, that could take up 60GB!  I’ve seen instances where snapshots fill up the datastore, then make it impossible to collapse the snapshots (since a certain amount of free disk space is needed).  Each snapshot you create also impacts disk performance.
