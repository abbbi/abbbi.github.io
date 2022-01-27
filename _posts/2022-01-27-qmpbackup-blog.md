---
layout: post
title: Qemu backup on Debian Bullseye
---

In my last [article](https://abbbi.github.io/debian/) i showed how to use the
new features included in Debian Bullseye to easily create backups of your
libvirt managed domains.

A few years ago as this topic came to my interest, i also implemented a rather
small utility (POC) to create full and incremental backups from standalone qemu
processes: [qmpbackup](https://github.com/abbbi/qmpbackup)

The workflow for this is a little bit different from the approach i have taken
with [virtnbdbackup](https://github.com/abbbi/virtnbdbackup).

While with libvirt managed virtual machines, the libvirt API provides all
necessary API calls to create backups, a running qemu process only provides the
QMP protocol socket to get things going.

Using the QMP protocol its possible to create bitmaps for the attached disks
and make Qemu push the contents of the bitmaps to a specified target directory.
As the bitmaps keep track of the changes on the attached block devices, you can
create incremental backups too.

The nice thing here is that the Qemu process actually does this all by itself
and you dont have to care about which blocks are dirty, like you would have
to do with the [Pull based](https://libvirt.org/kbase/live_full_disk_backup.html) approach.

**So how does it work**?

The utility requires to start your qemu process with an active QMP socket
attached, like so:

{% highlight bash %}
 qemu-system-<arch> <options> -qmp unix:/tmp/socket,server,nowait
{% endhighlight %}

Now you can easily make qemu push the latest data for a created bitmap
to a given target directory:

{% highlight bash %}
# qmpbackup --socket /tmp/socket backup --level full --target /tmp/backup/
[2022-01-27 19:41:33,819]    INFO  Version: 0.10
[2022-01-27 19:41:33,819]    INFO  Qemu version: [5.0.2] [Debian 1:5.2+dfsg-11+deb11u1]
[2022-01-27 19:41:33,825]    INFO  Guest Agent socket connected
[2022-01-27 19:41:33,825]    INFO  Trying to ping guest agent
[2022-01-27 19:41:38,827] WARNING  Unable to reach Guest Agent: cant freeze file systems.
[2022-01-27 19:41:38,828]    INFO  Backup target directory: /tmp/backup/
[2022-01-27 19:41:38,828]    INFO  FULL Backup operation: "/tmp/backup//ide0-hd0/FULL-1643308898"
[2022-01-27 19:41:38,836]    INFO  Wrote Offset: 0% (0 of 2147483648)
[2022-01-27 19:41:39,838]    INFO  Wrote Offset: 25% (541065216 of 2147483648)
[2022-01-27 19:41:40,840]    INFO  Wrote Offset: 33% (701890560 of 2147483648)
[2022-01-27 19:41:41,841]    INFO  Wrote Offset: 40% (867041280 of 2147483648)
[2022-01-27 19:41:42,844]    INFO  Wrote Offset: 50% (1073741824 of 2147483648)
[2022-01-27 19:41:43,846]    INFO  Wrote Offset: 59% (1269760000 of 2147483648)
[2022-01-27 19:41:44,847]    INFO  Wrote Offset: 75% (1610612736 of 2147483648)
[2022-01-27 19:41:45,848]    INFO  Saved disk: [ide0-hd0]
{% endhighlight %}

The resulting directory now contains a full backup image of the disk attached.

From this point on, its possible to create further incremental backups:

{% highlight bash %}
# qmpbackup  --socket /tmp/socket backup --level inc --target /tmp/backup/
[2022-01-27 19:42:03,930]    INFO  Version: 0.10
[2022-01-27 19:42:03,931]    INFO  Qemu version: [5.0.2] [Debian 1:5.2+dfsg-11+deb11u1]
[2022-01-27 19:42:03,933]    INFO  Guest Agent socket connected
[2022-01-27 19:42:03,933]    INFO  Trying to ping guest agent
[2022-01-27 19:42:08,938] WARNING  Unable to reach Guest Agent: cant freeze file systems.
[2022-01-27 19:42:08,939]    INFO  Backup target directory: /tmp/backup/
[2022-01-27 19:42:08,939]    INFO  INC Backup operation: "/tmp/backup//ide0-hd0/INC-1643308928"
[2022-01-27 19:42:08,953]    INFO  Wrote Offset: 0% (0 of 1835008)
[2022-01-27 19:42:09,957]    INFO  Saved disk: [ide0-hd0]
{% endhighlight %}

The target directory will now have multiple data backups:

{% highlight bash %}
/tmp/backup/ide0-hd0/
├── FULL-1643308898
└── INC-1643308928
{% endhighlight %}

**Restoring the image**

Using the `qmprebase` utility you can now rebase the images to the latest
state. The `--dry-run` option gives an good impression which command sequences
are required, if one wants only rebase to a specific incremental backup, thats
possible using the `--until` option.

{% highlight bash %}
# qmprebase rebase --dir /tmp/backup/ide0-hd0/ --dry-run
[2022-01-27 17:18:08,790]    INFO  Version: 0.10
[2022-01-27 17:18:08,790]    INFO  Dry run activated, not applying any changes
[2022-01-27 17:18:08,790]    INFO  qemu-img check /tmp/backup/ide0-hd0/INC-1643308928
[2022-01-27 17:18:08,791]    INFO  qemu-img rebase -b "/tmp/backup/ide0-hd0/FULL-1643308898" "/tmp/backup/ide0-hd0/INC-1643308928" -u
[2022-01-27 17:18:08,791]    INFO  qemu-img commit "/tmp/backup/ide0-hd0/INC-1643308928"
[2022-01-27 17:18:08,791]    INFO  Rollback of latest [FULL]<-[INC] chain complete, ignoring older chains
[2022-01-27 17:18:08,791]    INFO  Image files rollback successful.
{% endhighlight %}


**Filesystem consistency**

The backup utility also supports to freeze and thaw the virtual machines file
system in case qemu is started with a guest agent socket and the guest agent is
reachable during backup operation.

Check out the [README](https://github.com/abbbi/qmpbackup#readme) for the full
feature set.
