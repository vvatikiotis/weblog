---
Author: Vassilis Vatikiotis
Tags: Linux KVM
Date: 21 August 2019
Title: KVM serial console
---

# KVM serial console

With KVM, to access the virtual machine’s console under X Window, type:

`# virt-manager`
If you aren’t under X Window, there is another way to access a virtual machine’s console: you can go through a serial console.

On the virtual machine, add ‘console=ttyS0‘ at the end of the kernel lines in the /boot/grub2/grub.cfg file:

`# grubby --update-kernel=ALL --args="console=ttyS0"`
Note: Alternatively, you can edit the /etc/default/grub file, add ‘console=ttyS0‘ to the GRUB_CMDLINE_LINUX variable and execute ‘# grub2-mkconfig -o /boot/grub2/grub.cfg‘.

Now, reboot the virtual machine:

`# reboot`
With KVM, connect to the virtual machine’s console (here vm.example.com):

`# virsh console vm.example.com`
Connected to domain vm.example.com
Escape character is ^]

Red Hat Enterprise Linux Server 7.0 (Maipo)
Kernel 3.10.0-121.el7.x86_64 on an x86_64

vm login:
Emergency procedure
Sometimes you have lost all links to your virtual machine (error in the /etc/fstab file, ssh configuration, etc) and, as you didn’t set up any virtual console, you are in real trouble. There is still a solution!
Connect to the physical host and shut down your virtual machine (here called vm.example.com):

`# virsh destroy vm.example.com`
Define where the virtual machine image file is located (by default in the /var/lib/libvirt/images directory with a name like vm.example.com.img):

`# virsh dumpxml | grep "source file="`
      <source file='/var/lib/libvirt/images/vm.example.com.img'/>
Map your virtual machine image file into the host environment (-a for add and -v for verbose):

`# kpartx -av /var/lib/libvirt/images/vm.example.com.img`
add map loop0p1 (253:2): 0 1024000 linear /dev/loop0 2048
add map loop0p2 (253:3): 0 10240000 linear /dev/loop0 1026048
From the previous display, you know that you’ve got two partitions (in fact /boot and /, distinguishable by their respective size).
You need to mount the /boot partition to be able to change the grub configuration:

`# mount /dev/mapper/loop0p1 /mnt`
Then, edit the /mnt/grub2/grub.cfg file and add ‘console=ttyS0‘ at the end of every line containing /vmlinuz (the linux kernel).
Unmount the partition:

`# umount /mnt`
Unmap the virtual machine image file (-d for delete and -v for verbose):

`# kpartx -dv /var/lib/libvirt/images/vm.example.com.img`
del devmap : loop0p2
del devmap : loop0p1
loop deleted : /dev/loop0
Restart your virtual machine:

`# virsh start vm.example.com`
Domain vm.example.com started
Connect to your virtual machine console:

`# virsh console vm.example.com`
Connected to domain vm.example.com
Escape character is ^]

CentOS Linux 7 (Core)
Kernel 3.10.0-123.el7.x86_64 on an x86_64

vm login:
This procedure works for RHEL 6/CentOS 6 and RHEL 7/CentOS 7.