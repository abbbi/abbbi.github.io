---
layout: post
title: More work on virtnbdbackup
---

Had some time to add more features to my libvirt backup utility, now it
supports:

 * backing up virtual domains UEFI bios, initrd and kernel images if
   defined.
 * `virtnbdmap` now uses the nbdkit COW plugin to map the backups as regular
   NBD device. This allows users to replay complete backup chains
   (full+inc/diff) to recover single files. Also makes the mapped device
   writable, as such one can directly boot the virtual machine from the backup
   images.
   
Check out my [last article](https://abbbi.github.io/debian/) on that
topic or [watch it in action.](https://www.youtube.com/watch?v=dOE0iB-CEGM)

As a side note: still there's an [RFP](https://bugs.debian.org/1003167) open,
if one is interested in maintaining, as i find myself not having a valid
key in the keyring.. laziness.
