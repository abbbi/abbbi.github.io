---
layout: post
title: qmpbackup and proxmox 9
---

The latest Proxmox release introduces a new Qemu machine version that seems to
behave differently for how it addresses the virtual disk configuration.

Also, the regular "query-block" qmp command doesn't list the created bitmaps
like usual.

If the virtual machine version is set to "9.2+pve", everything seems to work
out of the box.

I've released [Version
0.50](https://github.com/abbbi/qmpbackup/releases/tag/v0.50) with some small
changes so its compatible with the newer machine versions.
