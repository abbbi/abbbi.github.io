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
[2022-01-27 17:12:15,068]    INFO  Version: 0.10
[2022-01-27 17:12:15,069]    INFO  Qemu version: [5.0.2] [Debian 1:5.2+dfsg-11+deb11u1]
[2022-01-27 17:12:20,073]    INFO  Backup target directory: /tmp/backup/
[2022-01-27 17:12:20,073]    INFO  FULL Backup operation: "/tmp/backup//ide0-hd0/FULL-1643299940"
[2022-01-27 17:12:20,081]    INFO  Wrote Offset: 0 of 2147483648
[2022-01-27 17:12:21,082]    INFO  Wrote Offset: 581566464 of 2147483648
[2022-01-27 17:12:22,084]    INFO  Wrote Offset: 765067264 of 2147483648
[2022-01-27 17:12:23,085]    INFO  Wrote Offset: 970129408 of 2147483648
[2022-01-27 17:12:26,465]    INFO  Saved disk: [ide0-hd0]
[2022-01-27 17:12:33,487]    INFO  Finished
{% endhighlight %}

The resulting directory now contains a full backup image of the disk attached.

From this point on, its possible to create further incremental backups:

{% highlight bash %}
# qmpbackup  --socket /tmp/socket backup --level inc --target /tmp/backup/
[2022-01-27 17:12:43,544]    INFO  Version: 0.10
[2022-01-27 17:12:43,545]    INFO  Qemu version: [5.0.2] [Debian 1:5.2+dfsg-11+deb11u1]
[2022-01-27 17:12:43,546]    INFO  Guest Agent socket connected
[2022-01-27 17:12:43,546]    INFO  Trying to ping guest agent
[2022-01-27 17:12:48,552] WARNING  Unable to reach Guest Agent: cant freeze file systems.
[2022-01-27 17:12:48,553]    INFO  Backup target directory: /tmp/backup/
[2022-01-27 17:12:48,553]    INFO  INC Backup operation: "/tmp/backup//ide0-hd0/INC-1643299968"
[2022-01-27 17:12:48,568]    INFO  Wrote Offset: 0 of 8257536
[2022-01-27 17:12:49,571]    INFO  Saved disk: [ide0-hd0]
{% endhighlight %}

The target directory will now have multiple data backups:

{% highlight bash %}
/tmp/backup/ide0-hd0/
├── FULL-1643299940
└── INC-1643299968
{% endhighlight %}

**Restoring the image**

Using the `qmprebase` utility you can now rebase the images to the latest
state, the `--dry-run` option gives an good impression which command sequences
are required, if one wants only rebase to a specific incremental backup:

{% highlight bash %}
# qmprebase rebase --dir /tmp/backup/ide0-hd0/ --dry-run
[2022-01-27 17:18:08,790]    INFO  Version: 0.10
[2022-01-27 17:18:08,790]    INFO  Dry run activated, not applying any changes
[2022-01-27 17:18:08,790]    INFO  qemu-img check /tmp/backup/ide0-hd0/INC-1643299968
[2022-01-27 17:18:08,791]    INFO  qemu-img rebase -b "/tmp/backup/ide0-hd0/FULL-1643299940" "/tmp/backup/ide0-hd0/INC-1643299968" -u
[2022-01-27 17:18:08,791]    INFO  qemu-img commit "/tmp/backup/ide0-hd0/INC-1643299968"
[2022-01-27 17:18:08,791]    INFO  Rollback of latest [FULL]<-[INC] chain complete, ignoring older chains
[2022-01-27 17:18:08,791]    INFO  Image files rollback successful.
{% endhighlight %}


**Filesystem consistency**

The backup utility also supports to freeze and thaw the virtual machines file
system in case qemu is started with a guest agent socket and the guest agent is
reachable during backup operation.

Check out the [README](https://github.com/abbbi/qmpbackup#readme) for the full
feature set.
