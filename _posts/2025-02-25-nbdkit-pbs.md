---
layout: post
title: proxmox backup nbdkit plugin
---

[nbdkit](https://libguestfs.org/nbdkit.1.html) is a really powerful NBD
toolkit.

Lately, i wanted to access VM backups from a Proxmox Backup Server via network
(not by using the proxmox-backup-client map function..)

For example, to test-boot a virtual machine snapshot directly from a backup.
NBD suits that usecase quite well, so i quickly put a nbdkit plugin together
that can be used to access virtual machine disk backups from proxmox backup
server via NBD.

The available [golang
bindings](https://github.com/elbandi/go-proxmox-backup-client) for the proxmox
backup client API, made that quite easy.

As nbdkit already comes with a neat COW plugin, its only been a few lines
of go code resulting in: [pbsnbd](https://github.com/abbbi/pbsnbd)
