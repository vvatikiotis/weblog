---
author: Vassilis Vatikiotis
title:  KVM guest VM console access
tags:
  - linux
  - KVM
---

# KVM guest VM console access

**Ubuntu 12.04 Host and Ubuntu 12.04 guest VM.**

The following information is almost copied from <a title="http://ubuntuforums.org/showpost.php?p=9610421&amp;postcount=7" href="http://ubuntuforums.org/showpost.php?p=9610421&amp;postcount=7" target="_blank">http://ubuntuforums.org/showpost.php?p=9610421&amp;postcount=7</a>

You want to access a guest VM from the terminal i.e.

<ul>
	<li>You want a console terminal where you can login and work on a shell</li>
	<li>Additionally, you want to see all console messages, even the ones during boot time.</li>
</ul>
For the second bullet, put the following in /etc/default/grub
<code>
GRUB_CMDLINE_LINUX="console=ttyS0,38400n8 console=tty0"
</code>
and then do a <em>update-grub</em>.

For the first bullet a couple of things have to be in place. First, the VM's xml configuration file, located under /etc/libvirt/qemu, has to contain an entry such as:

[xml]

[/xml]

Additionally, in order to have shell access, connect to your VM using ssh or vnc and create the following upstart script.

/etc/init/ttyS0.conf

```
# ttyS0 - getty
#
# This service maintains a getty on ttyS0 from the point the system is
# started until it is shut down again.</code>

start on stopped rc RUNLEVEL=[2345]
stop on runlevel [!2345]

respawn
exec /sbin/getty -L 38400 ttyS0 xterm
```

You can reboot the guest VM or you can do a `service ttyS0 start` to start the ttyS0 service.

In case you changed the guest VM's xml configuration file while the VM is running, you need to do the following at the host.

```
# reload libvirt-bin
# virsh shutdown
# virsh start
```

UPDATE: You do not need to reload libvirt-bin to activate the VM's xml changes. Just shutdown the guest and issue:

`# virsh define`

And you are ready to access the guest VM:

`# virsh console`

UPDATE:
`$ virt-install -n hitrack --ram 512 --vcpus=1 --os-type linux --os-variant=rhel6 --disk path=/var/lib/libvirt/images/hitrack.img,bus=virtio,size=8 -l ./CentOS-6.2-x86_64-bin-DVD1.iso --graphics none --accelerate --network bridge:br1,model=virtio -v -x "console=ttyS0"`

or, for vnc,

`$ virt-install -n hitrack --ram 512 --vcpus=1 --os-type linux --os-variant=rhel6 --disk path=/var/lib/libvirt/images/hitrack.img,bus=virtio,size=8 -l ./CentOS-6.2-x86_64-bin-DVD1.iso --graphics vnc,listen=0.0.0.0 --accelerate --network bridge:br1,model=virtio -v -x "console=ttyS0"`

Take note of the `--network=bridge:br1`. This is how it should when we use bridged networking.

Something like the above will create the VM, proceed with the installation in text mode and take care of all the above!

Another example:

`# virt-install -n parser --description "Vieras parser" --ram 2048 --vcpus=2 --cpu=host --os-type linux --os-variant=rhel6 --disk path=/var/lib/libvirt/images/vieras-parser.img,bus=virtio,format=qcow2,size=32 -l /var/lib/libvirt/boot/CentOS-6.5-x86_64-bin-DVD1.iso --graphics none --accelerate --network bridge:br101,model=virtio -v -x "console=ttyS0"`

Check virt-install man pages to use the 'location' option
