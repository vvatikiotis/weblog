---
author: Vassilis Vatikiotis
title: OpenNebula tweaks for Centos
tags:
  - linux
  - centos
---

# OpenNebula tweaks for Centos

For Centos 6.4

s### Prepare system

- Install the UEK kernel (oracle) to get OCFS2 support.
- Install, if necessary, iscsi and multipath support.

### Networking

- **NB**: it makes sense to use the same network driver for both declaring hosts and making the relevant network templates.
- `yum install vconfig`
- create a `/sbin/brctl` symlink to `/usr/sbin/brctl`
- We use the 802.1q, which means that ONE will create the bridge and eth vlan interfaces (on the hosts).

  - That means that the `oneadmin` user has to be able to execute, via `sudo`, without aquirring a tty and with no password, the `vconfig`, `brctl` and `ip` linux commands. To that end, `/etc/sudoers.d/opennebula` is as follows:

            Cmnd_Alias ONENETWORKING = /sbin/vconfig, /usr/sbin/brctl, /sbin/brctl, /sbin/ip
            Defaults:oneadmin !requiretty
            oneadmin   ALL=(ALL) NOPASSWD: ONENETWORKING

  - Here are the network scripts for the bridge and eth vlan interfaces, just in case we want to use the dummy network driver, which means that the admin has to create these interfaces in advance, as part of the base system installation. ONE will do nothing in terms of bridge and eth vlan interfaces management:

            # cat /etc/sysconfig/network-scripts/ifcfg-eth0.4
            DEVICE=eth0.4
            ONBOOT=yes
            NM_CONTROLLED=no
            BOOTPROTO=none
            PEERDNS=yes
            NOZEROCONF=no
            USERCTL=no
            VLAN=yes
            BRIDGE=brZOO

            # cat /etc/sysconfig/network-scripts/ifcfg-brZOO
            DEVICE=brZOO
            HWADDR=xx:xx:xx:xx:xx:xx
            TYPE=Bridge
            IPV6INIT=no
            IPV6_AUTOCONF=no
            DELAY=0
            STP=yes
            ONBOOT=yes
            NM_CONTROLLED=no
            PEERDNS=yes
            NOZEROCONF=no
            USERCTL=no
            BOOTPROTO=none
            NETMASK=255.255.255.128
            BROADCAST=xx.xx.xx.255
            DNS1=xx.xx.xx.3
            DNS2=xx.xx.xx.2
