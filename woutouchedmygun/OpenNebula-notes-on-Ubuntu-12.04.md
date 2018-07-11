---
author: Vassilis Vatikiotis
title: OpenNebula notes on Ubuntu 12.04
tags:
  - linux
  - ubuntu
---

# OpenNebula notes on Ubuntu 12.04

## Install HOSTS

ntp pm-utils vlan bridge-utils ocfs2-tools lsscsi htop virt-top ruby libpam-cracklib rkhunter console-log

## Install FRONTEND

open-iscsi ntp pm-utils vlan ocfs2-tools sqliteman libpam-cracklib rkhunter console-log (sqliteman for managing the sqlite storage backend of ONE)

## Install on HP ProLiant DL380 G7 Server for SMH

xsltproc libc6-i386 lib32gcc1 lib32stdc++6

- All HP deb packages except the hp-snmp-agents and hp-smh-templates
  Follow the post to configure HP SMH

## Remove HOSTS

ntpdate ntfs-3g

## Remove FRONTEND

ntpdate ntfs-3g

## network stuff in HOST

To enable bridged networking
set cap stuff https://help.ubuntu.com/community/KVM/Networking

## ERRARA?

/var/lib/one/config has DATASTORE with 2 /
http://lists.opennebula.org/pipermail/users-opennebula.org/2012-June/019427.html
KVM: apparmor.d/usr.sbin.libvirtd, at the end,
/var/lib/one/\*\* lrwk,
All changes should happen under the /etc/apparmor.d/local dir

## user configuration in HOST

oneadmin:cloud 1001:1001
Insert one-frontend oneadmin's id_rsa

- Take care to replace all instance of group 'oneadmin' in /etc/groups
- Change user and group vars in /etc/libvirt/qemu.conf to 'oneadmin' and 'cloud'
- `# cat /etc/udev/rules.d/60-qemu-kvm.rules`
- KERNEL=="kvm", GROUP="cloud", MODE="0660"
- THIS USED TO WORK vim /etc/libvirt/libvirtd.conf unix_sock_group = "cloud"

## SSL conf in ALL

Add openldap user to ssl-cert group

```sh
# mkdir /etc/ssl/site/{certs,private}
# chown root:ssl-cert /etc/ssl/site/private
# chmod 750 /etc/ssl/site/private
```

Copy the certificates and key in the directories.
In private: # chmod 640 all-files and # chown root:ssl-cert all-files

## motd

Disabling it: find where the pam_motd is used, under /etc/pam.d and comment it out.
dpkg-reconfigure landscape-common Do not display system stats.

## ntp configuration in ALL

All nodes have to be synchronized
Install ntpd In /etc/default/ntp: NTPD_OPTS='-g -I eth1'
eth1 is the mgmt interface

In /etc/ntpd.conf there has to be a line like this:
server time.ntpserver.org iburst
The iburst argument causes ntpd to sync very quickly with this server after starting up.

## /etc/hosts configuration in ALL

```
10.0.4.2 infra1.xxx.xxx infra1
10.0.4.3 infra2.xxx.xxx infra2
10.0.4.4 infra3.xxx.xxx infra3
10.0.4.5 infra4.xxx.xxx infra4
10.0.4.101 one-frontend.xxx.xxx one-frontend one
```

## rkhunter

See /etc/default/rkhunter in infra4

## console-log

See /etc/console-log.conf in infra4

## cciss-vol-status on HP machines

man cciss_vol_status
For infra4 the raid device is /dev/sg0

## OCFS2 in ALL

cfdisk /dev/mpath2 mkfs.ocfs2 -b 4K -C 256K -L "datastore_name" /dev/mapper/mpath2-part1 - we use the default N max num of nodes which is 8

- the volume label has to be unique (used in mount-by-label) http://richardpickett.com/ocfs2-over-iscsi-on-ubuntu-10-10-maverick-meerkat Beware of OCFS2 and iscsi timeouts.
  https://oss.oracle.com/pipermail/ocfs2-users/2010-March/004272.html Same as above http://www.querzone.de/wiki/Wiki.jsp?page=OCFS2

Chown to oneadmin before mounting (done once!)
Mount ocfs2 device in /etc/rc.local. It doesn't work on fstab cause it hangs cause ocfs2 tries to mount before multipathd prepares the mapper devices in /dev/mapper /etc/rc.local

`# mount ocfs2 volumes here, instead of fstab mount -o _netdev,errors=panic /dev/mapper/mpath0-part1 /var/lib/one/datastores/0 mount -o _netdev,errors=panic /dev/mapper/mpath2-part1 /var/lib/one/datastores/102`

/etc/ocfs2/cluster.conf

- the node name has to be the **local** hostname of each machine.
- cluster node count is 8, just as the default when we mkfs the ocfs2 partitions
- ocfs2console: It spits out an error msg on startup. Resolution here https://bugs.launchpad.net/ubuntu/+source/ocfs2-tools/+bug/923754

## iSCSI configuration in ALL

Change initiator name

## Datastores

0 System datastore LABEL SYSDS (UBUNTU: in cluster NONE so it can be shared!)
102 iitds (500GB) (in clusrter iit) 103 infrads (300GB) (in cluster infrastructure)
For live migration all hosts need to be able to ssh via pubkey to each other. Transfer all pubkeys

## Elegant Guest Shutdown

To enable elegant shutdown of domains, ensure they respond to ACPI power button presses. On Linux, install acpid in the guest OS.

## ONE Templates

It seems that you are setting the port manually in the graphics section ans is trying to bind to port 1. Just set the LISTEN and TYPE parameters. The port will be automatically generated for you:
GRAPHICS=[ TYPE="vnc", LISTEN="0.0.0.0" ]
Otherwise the VM template will not be instantiated

Sample template

```
CONTEXT=[
SSH_PUBLIC_KEY="ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAQDhGBp3Ih5/1iXNbH6ulYcmsgeHzNc8CtEOCNK79AtE/00XNOO9aD87m4XabNiOLDBf1/TwBbmWyJSWBLmg4TdriVkQfqLOz+g/I9gwp7aM4ZRLpX4EMOkx5NNasdx4GI5KE4NZF05B+6bsxOBP6WEI+do+OEBWCIqEY6nwH3c/w7M8xA1Ei9wm6raN1uKaXL6Ba29isu+Yy/O63blvJaJR09brrgr0Pwy8OfO9WqmU32yDLa1jhVwGj3d/5nUTQE7mCBeqGezTWk5CbkrNxNGyF6TmGTSBEaFyh/5EwWvbNbUhobSJYOtVId5N5Jyy2eER52YueUJEDhZTVwW8tAYL0w1 username rsa" ]
CPU="1"
DISK=[ IMAGE="dns", IMAGE_UNAME="oneadmin" ]
GRAPHICS=[ LISTEN="0.0.0.0", TYPE="vnc" ]
INPUT=[ BUS="usb", TYPE="mouse" ]
MEMORY="1024"
NAME="test"
NIC=[ IP="xxx.xxx.xxx.xxx", NETWORK="DMZ", NETWORK_UNAME="oneadmin" ]
OS=[ ARCH="x86_64", BOOT="hd" ]
RAW=[ TYPE="kvm" ]
TEMPLATE_ID="0"
```

## Upgrading ONE

- It seems that vnc settings are gone when updating. Running $ sudo /usr/share/opennebula/install_novnc.sh fixes the problem as it (also) writes /etc/sunstone-server.conf

## Lighttpd configuration

- Default fqdn/ui path points to OCCI. The only thing changed here is the port in the proxy.server stanza. OCCI port is 4567 - fqsn:8443 points to Sunstone.

The following addition to the config file accomplisehd that.
$SERVER["socket"] == ":8443" {
ssl.engine = "enable" ssl.pemfile = "/etc/lighttpd/server.pem" proxy.server = ( "" =&gt; ( "" =&gt; ( "host" =&gt; "127.0.0.1", "port" =&gt; 9869 ) ) )
}

## LDAP in FRONTEND

- change :auth to :opennebula in the sunstone and occi conf files
- create a bind user in LDAP
- chown oneadmin:cloud ldap_auth.conf and chmod o-r ldap_auth.conf this is necessary cause the onebind ldap password is in this file so we need to protect it from prying eyes.
- in /usr/share/opennebula/occi/ui/public/js/login.js comment out the line password = CryptoJS.SHA1(password)
- In LDAP server we have to set

1.  add under global dn cn=config group: olcPasswordCryptSaltFormat: "$6$%.86s"
2.  add under DN: olcDatabase={-1}frontend,cn=config olcPasswordHash: {CRYPT}
    In ldap clients, ldap.conf (haven't done this step) search for the "pam_password" entry and change it to "exop" This point relates to the overal unixldap password hash issue.

---

SETTING UP HIGH AVAILABILITY IN OPENNEBULA WITH LVM

- http://blog.opennebula.org/?p=1523
- Create a Windows-XP VM http://cloudblab.files.wordpress.com/2012/08/opennebula-3-6-0-in-ubuntu-12-04-windowsxp-iscsi-datastore.pdf
