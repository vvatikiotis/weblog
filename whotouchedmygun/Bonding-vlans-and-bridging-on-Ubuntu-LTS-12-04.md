---
author: Vassilis Vatikiotis
title:  Bonding, vlans and bridging on Ubuntu LTS 12.04
tags:
  - linux
  - vlans
  - bonding
  - networking
---

# Bonding, vlans and bridging on Ubuntu LTS 12.04

<pre># iSCSI interface
auto eth0
iface eth0 inet static
    address 10.0.5.11
    netmask 255.255.255.0
    post-up sysctl -w net.ipv4.conf.eth0.rp_filter=0
    post-up route add -host 10.0.5.2 dev eth0

# iSCSI interface
auto eth1
iface eth1 inet static
    address 10.0.5.12
    netmask 255.255.255.0
    post-up sysctl -w net.ipv4.conf.eth1.rp_filter=0
    post-up route add -host 10.0.5.4 dev eth1

###################################
#### BONDING, VLAN, BRIDGING ######
###################################
auto eth2
iface eth2 inet manual
    bond-master bond0
auto eth3
iface eth3 inet manual
    bond-master bond0

# bonding eth2 and eth3
auto bond0
iface bond0 inet manual
    bond-mode balance-xor
    bond-slaves none
    bond-miimon 100

#VLAN 3 - DMZ
auto bond0.3
iface bond0.3 inet manual
    vlan-raw-device bond0
auto br3-DMZ
iface br3-DMZ inet manual
    bridge_ports bond0.3
    bridge_maxwait 0
    bridge_fd 1
    bridge_stp off

# Bridge on Admin Net
# sits directly on bond0
# This block is last
# cause its IP will be the source
# IP of all outgoing packets
auto br1-ADMIN
iface br1-ADMIN inet static
    address 192.168.100.65
    netmask 255.255.255.0
    gateway 192.168.100.1
    dns-nameservers 143.233.226.3 143.233.226.2
    dns-search iit.demokritos.gr
    bridge_ports bond0
    bridge_maxwait 0
    bridge_fd 0
    bridge_stp off

post-up route del -net 10.0.5.0 netmask 255.255.255.0 dev eth0
post-up route del -net 10.0.5.0 netmask 255.255.255.0 dev eth1</pre>

This is the <em>/etc/network/interfaces</em> for a server connecting to an iSCSI SAN and using 2 additional eth devices, bonded, to provide network bridges on top of vlans.

The post-up statements on eth0 and eth1 are for iSCSI. As soon as each interface goes up, we change the rp_filter kernel setting (see iscsi documentation) and we add the necessary route. 10.0.5.2 and 4 are the iSCSI's IPs each on a separate controller.

The bonding, vlan, and bridging is quite straightforward. It's all in the ubuntu documentation on the internet. There are 2 very important points here:

<ul>
	<li>Don't assign an IP to a bonded interface if it's bound to a bridged interface. Following this, we create a bridge <em>br1-ADMIN</em> over <em>bond0</em>. The bridge gets the IP, not the bonded interface [1].</li>
	<li>I want outgoing packet to have a specific source address. To that end, I place the appropriate iface stanza as the last one, in our case <em>br1-ADMIN</em></li>
	<li>Bridge interface names MUST NOT contain ".", use "-" instead.</li>
	<li>Spanning tree (stp) is off. There are no network loops in the topology</li>
</ul>
One thing that bit me was the 1st bullet point. I had assigned an IP to the bonded interface, declaring it last and while everything seemed to be working I got bitten when I spun my first VM. Network behavior was erratic, to say the least, with loads of arp mayhem.

<em>References</em>

<ol>
	<li><a href="http://floatingatoll.posterous.com/ganeti-and-ubuntu-1004-lucid-configuring-brid">http://floatingatoll.posterous.com/ganeti-and-ubuntu-1004-lucid-configuring-brid</a></li>
	<li><a href="http://www.tolaris.com/2010/02/20/vlans-bridges-and-virtual-machines/">http://www.tolaris.com/2010/02/20/vlans-bridges-and-virtual-machines/</a></li>
	<li><a href="https://help.ubuntu.com/community/UbuntuBonding">https://help.ubuntu.com/community/UbuntuBonding</a></li>
	<li><a href="http://ubuntuforums.org/showthread.php?t=1876061">http://ubuntuforums.org/showthread.php?t=1876061</a></li>
	<li><a href="http://manpages.ubuntu.com/manpages/precise/man5/vlan-interfaces.5.html">http://manpages.ubuntu.com/manpages/precise/man5/vlan-interfaces.5.html</a></li>
	<li><a href="http://http://manpages.ubuntu.com/manpages/precise/man5/bridge-utils-interfaces.5.html">http://manpages.ubuntu.com/manpages/precise/man5/bridge-utils-interfaces.5.html</a></li>
</ol>
