---
author: Vassilis Vatikiotis
title: Installing HP snmp agents on a Proliant server
tags:
  - linux
  - virtualisation
  - OpenNebula
---

# Installing HP snmp agents on a Proliant server

Note: This is on CentOS 6.4

- Download H SPP at <http://h18013.www1.hp.com/products/servers/management/spp/index.html>, get the latest.
- `# mount -o loop <SPPMDSLRH-filename> /mnt/SPP`
- `# cd /mnt/SPP/hp/swpackages`
- `yum localinstall` `the hp-health` and the `hp-snmp-agents` packages.
- `# /sbin/hpsnmpconfig`
  - `vi /etc/snmp/snmp.local.conf` and add the line `syscontact watchdog <watchdog@my.domain.com>`
- `# vi /opt/hp/hp-snmp-agents/cma.conf`.
  - Change the line containing `trapemail` at its end to reflect the correct email address.
- `# /etc/init.d/hp-snmp-agents start`
- `# chkconfig hp-snmp-agents on`

#### References

- <http://serverfault.com/questions/342383/how-do-i-get-my-hp-servers-to-email-me-when-a-drive-fails>
- <http://communitylinux.org/node/265>
