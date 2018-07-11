---
author: Vassilis Vatikiotis
title: New Ubuntu Setup Up
tags:
  - linux
  - ubuntu
---

# New Ubuntu Setup Up

- Motd is disabled by commenting out the pam_motd lines under /etc/pam.d

- setup /etc/default/locale LANG, LANGUAGE, LC_ALL

- disable backport repositories. This is Ubnutu LTS, support for 5 yrs.

- /etc/default/bootlog set to Yes

- /etc/default/rkhnter setup

- chkrootkit setup. RUN_DAILY yes

- denyhosts setup.

- logwatch install

- git

- nginx

- open-iscsi, multipath-tool, nfs-kernel-server service start manually in rc.local and not in sysv/upstart scripts. Rearranged following script order
- open-iscsi, multipath-tools, nfs-kernel-server, umount-iscsi-volume
- in rc0.d rc2.d rc6.d and rcS.d

- open-iscsi, multipath, fsck and mounting the Hitach big Volume, nfs-kernel-server is done in rc.local

- mount -bind of nfs shares happens in rc.local

- KVM specific (https://help.ubuntu.com/community/KVM/Networking)
- set_cap to enable bridging
- cap_net_admin in /etc/security/capability.con
- to destroy default kvm network (name: default, it's NAT based) we do

1.  # virsh net-destroy default
2.  # virsh net-undefine default
3.  # service libvirtd restart
    The virbr0 iface is destroyed. dnsmasq isn't started.

- Setup a bridged iface. This will support
  a. infra mgmt iface
  b. VMs will connect to this bridge for net connectivity.

- console-log installed and configured

- we work with KVM virt-inst to setup VM (see KVM post)

- Network pkgs setup:
- uninstall dnsmasq pkg. We do not need it since we use bridging.
- uninstall isc-dhcp-\* pkgs. We do not need it since we use bridging.
- uninstall ebtables

References
http://www.thefanclub.co.za/how-to/how-secure-ubuntu-1204-lts-server-part-1-basics
