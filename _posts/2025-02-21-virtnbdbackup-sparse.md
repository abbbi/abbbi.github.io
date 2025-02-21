---
layout: post
title: virtnbdbackup 2.21
---

Yesterday i released a new version of
[virtnbdbackup](https://github.com/abbbi/virtnbdbackup) with a nice
improvement.

The new version can now detect zeroed regions in the bitmaps by comparing the
block regions against the state within the base bitmap during incremental
backup.

This is helpful if virtual machines run fstrim, as it results in less backup
footprint. Before the incremental backups could grow the same amount of size as
fstrimmed data regions.

I also managed to enhance the tests by using the arch linux cloud images. The
automated github CI tests now actually test backup and restores against a
virtual machine running an real OS.
