---
layout: post
title: building SLES 16 vagrant-libvirt images using guestfs tools
---

SLES 16 has been released. In the past, SUSE offered ready built vagrant
images. Unfortunately that's not the case anymore, as with more recent
SLES15 releases the official images were gone.

In the past, it was possible to clone existing projects on the opensuse build
service to build the images by yourself, but i couldn't find any templates
for SLES 16.

Naturally, there are several ways to build images, and the tooling around
involves kiwi-ng, opensuse build service, or packer recipes etc.. (the latter
may not work anymore, as Yast has been replaced by a new installer, called
agma).

So my current take on creating a vagrant image for SLE16 has been the
following:

 * Spin up an QEMU virtual machine
 * Manually install the system, all in default except for one special setting:
   In the Network connection details, "Edit Binding settings" and set the
   Interface to Unbound, leave DHCP in place.
 * After installation has finished, shutdown.

Now, there are several guestfs-tools that can be used to work on the
created qcow2 image:

 * run virt-sysrpep on the image to wipe settings that might cause troubles:

{% highlight bash %}
 virt-sysprep -a sles16.qcow2
{% endhighlight %}

 * create a simple shellscript that setups all vagrant related settings:

{% highlight bash %}
#!/bin/bash
useradd vagrant
mkdir -p /home/vagrant/.ssh/
chmod 0700 /home/vagrant/.ssh/
echo "ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIF
o9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9W
hQ== vagrant insecure public key" > /home/vagrant/.ssh/authorized_keys
chmod 0600 /home/vagrant/.ssh/authorized_keys
chown -R vagrant:vagrant /home/vagrant/
# apply recommended ssh settings for vagrant boxes
SSHD_CONFIG=/etc/ssh/sshd_config.d/99-vagrant.conf
if [[ ! -d "$(dirname ${SSHD_CONFIG})" ]]; then
    SSHD_CONFIG=/etc/ssh/sshd_config
    # prepend the settings, so that they take precedence
    echo -e "UseDNS no\nGSSAPIAuthentication no\n$(cat ${SSHD_CONFIG})" > ${SSHD_CONFIG}
else
    echo -e "UseDNS no\nGSSAPIAuthentication no" > ${SSHD_CONFIG}
fi
SUDOERS_LINE="vagrant ALL=(ALL) NOPASSWD: ALL"
if [ -d /etc/sudoers.d ]; then
    echo "$SUDOERS_LINE" >| /etc/sudoers.d/vagrant
    visudo -cf /etc/sudoers.d/vagrant
    chmod 0440 /etc/sudoers.d/vagrant
else
    echo "$SUDOERS_LINE" >> /etc/sudoers
    visudo -cf /etc/sudoers
fi
 
mkdir -p /vagrant
chown -R vagrant:vagrant /vagrant
systemctl enable sshd
{% endhighlight %}

 * use virt-customize to upload the script into the qcow image:

{% highlight bash %}
 virt-customize -v -x -a sle16.qcow2 --upload vagrant.sh:/tmp/vagrant.sh
{% endhighlight %}

 * execute the script via:

{% highlight bash %}
 virt-customize -a sle16.qcow2 -v --run-command "/tmp/vagrant.sh"
{% endhighlight %}

After this, use the create-box.sh from the vagrant-libvirt project
to create an box image:

 https://github.com/vagrant-libvirt/vagrant-libvirt/blob/main/tools/create_box.sh

and add the image to your environment:

{% highlight bash %}
 create_box.sh sle16.qcow2 sle16.box
 vagrant box add --name my/sles16 test.box
{% endhighlight %}

and spin up your vagrant instances :-)
