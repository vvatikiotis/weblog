---
author: Vassilis Vatikiotis
title:  Managing KVM virtual machines on Ubuntu 12.04
tags:
  - linux
  - KVM
---

# Managing KVM virtual machines on Ubuntu 12.04

<strong>First steps</strong>

<ol>
<ol>
	<li>Install virt-manager in a local desktop. I wouldn't install this on the kvm host. The package name is virt-manager [1]. <code>#&gt; sudo apt-get install virt-manager</code></li>
</ol>
</ol>
netcf package needed?

delete default kvm virtual net used in NAT.

MAC addresses have to end in an even number.

No need for ip_forward=1 cause the host doesn't route anything(so far, in our setup).

<ol>
	<li>In kvm host, make the user responsible for kvm operations a member of the libvirt group. If we don't then any connection from the local desktop (where we installed virt-manager) to the kvm host using virt-manager will fail. `$ usermod -a -G libvirtd tony`</li>
	<li>Within virt-manager we have to configure our store and our network.</li>
	<li>In kvm host ip_forward=1 ?</li>
</ol>
The simplest scenario would be to keep the libvirt defaults. So our VM would be stored as files in a local (or attached) disk of the kvm host. Similarly, our VMs would be NAT'd so no inbound connections to our VMs would be possible. apt-get install pm-utils power managment capabilities <strong>Moving away from the libvirt defaults</strong> virt-manager will allow us to configure the libvirt settings and manage our VMs. libvirt settings are stored in <em>/etc/libvirt</em> as xml files. As we tweak the libvirt configuration, existing files will be modified and new files will be created. It's a good idea to check these files in order a) to get a feeling of what's happening while we configure and b) check the xml format and the available options. The alternative would be to check <em>virsh</em> man pages; for the brave and patient. It's not too difficult but it has a gazillion options. Better to let virt-manager help us and walk our way backwards to understand what's happening behind the scene. You should use kvm but this requires a cpu with the virtualization extensions. The qemu option uses only emulation so guest will run much slower in this mode. Yes, KVM *only* works on cpus with virtualization extensions. If your cpu doesn't have those features it will fallback to qemu so you don't have a choice in that case. You can also use kqemu if you have that option but kqemu isn't being maintained very much since most of the focus is now on kvm. <strong>References</strong> 1. <a href="http://www.terminal-skillness.com/2011/12/using-virt-anager-on-os-x-without-vms-flawlessly/">http://www.terminal-skillness.com/2011/12/using-virt-anager-on-os-x-without-vms-flawlessly/</a>
