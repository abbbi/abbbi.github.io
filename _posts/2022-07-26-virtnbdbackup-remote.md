---
layout: post
title: Added remote capability to virtnbdbackup
---

Latest [virtnbdbackup](https://github.com/abbbi/virtnbdbackup) version now
supports backing up remote libvirt hosts, too. No installation on the
hypervisor required anymore:

{% highlight bash %}
virtnbdbackup -U qemu+ssh://usr@hypervisor/system -d vm1 -o /backup/vm1
{% endhighlight %}

Same applies for restore operations, other enhancements are:

 * New backup mode [auto](https://github.com/abbbi/virtnbdbackup#rotating-backups) which allows 
   easy backup rotation.
 * Option to freeze only specific filesystems within backed up domain.
 * Remote backup via dedicated network: use `--nbd-ip` to bind the remote
   NDB service to an specific interface.
 * If virtual machine requires additional files like specific UEFI/Kernel
   image, these are saved via SFTP from the remote host, too.
 * Restore operation can now
   [adjust](https://github.com/abbbi/virtnbdbackup#restoring-with-modified-virtual-machine-config)
   domain config accordingly (and redefine it if desired).

Next up: [add TLS support](https://github.com/abbbi/virtnbdbackup/issues/66) for remote NBD connections.
