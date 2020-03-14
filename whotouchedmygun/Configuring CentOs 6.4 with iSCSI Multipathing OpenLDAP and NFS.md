---
author: Vassilis Vatikiotis
title: Configuring CentOs 6.4 with iSCSI, Multipathing, OpenLDAP and NFS
tags:
  - linux
  - ldap
  - apache
---

# Configuring CentOs 6.4 with iSCSI, Multipathing, OpenLDAP and NFS

- `yum install wget mlocate ntp logwatch`
- epel and rpmforge repos installed. disabled by default. Default repos **always** take precedence over epel, which **always** takes precedence over rpmforge, <http://fedoraproject.org/wiki/EPEL/FAQ>

  - `wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm`
  - `rpm -Uvh epel-release-6-8.noarch.rpm`
  - <http://wiki.centos.org/AdditionalResources/Repositories/RPMForge#head-f0c3ecee3dbb407e4eed79a56ec0ae92d1398e01>
  - `wget http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.3-1.el6.rf.x86_64.rpm`

- `yum install yum-utils git subversion most rkhunter chkrootkit htop lsscsi ssmtp w3m jwhois screen crypto-utils phpldapadmin denyhosts` bind-utils

- `yum install sysstat`. Install ssmtp before removing postfix as sysstat requires an MTA package.

- `yum install xorg-x11-xauth` to fix the `X11 forwarding request failed on channel 0` error I was getting when sshing to the CentOS box, while having `X11Forwarding yes` in the sshd_config.
- `yum remove postfix, virt-manager`

- `chkconfig cups off`

- `chkconfig iptables off`. I'll enable it later.

- `setenforce 0` to change to SELinux permissive mode. YMMV. Also `vim /etc/selinux/config`

- ssh port 222, SElinux allows only 22
- need to use semanage to change it. `# yum install policycoreutils-python`
- `semanage port -l | grep ssh`
- `semanage port -a -t ssh_port_t -p tcp 222`

- /etc/hosts file

- /etc/mdadm.conf

- /etc/sysconfig/network and interfaces scripts

- /etc/sysctl.d/ rp_filter=0 setup for br1

- /etc/updatedb.conf prune /mnt

- /etc/sysctl.d/ kernel conf vars

- iscsi
- `yum install iscsi-initiator-utils device-mapper-multipath`
- /etc/sysctl.d/iscsi to set the rp_filter=0 for eth2 and eth3 for iscsi
- iscsi initiator name
- iscsi iface2 and iface3 under /var/lib/iscsi/ifaces. Remove the bnx2\* files as well. For example, for eth2:
  `iface.iscsi_ifacename = iface2 iface.net_ifacename = eth2 iface.transport_name = tcp iface.vlan_id = 0 iface.vlan_priority = 0 iface.iface_num = 0 iface.mtu = 0 iface.port = 0`
- <http://www.server-world.info/en/noteos=CentOS_6&p=iscsi&f=2>
- create/copy /etc/multipath.conf
- `mkdir /mnt/hitachi/Vol1`
- fstab entry

- ssmtp.conf

- ntp configuration

- libvirt, kvm
- the CentOS source image has to be located at /var/lib/libvirt/images otherwise virt-install fails with "permission denied"
- To make the VM autostart: `virsh autostart vmName`
- Setup VM's network

- generate ssl certificate. Place it under /etc/pki/tls. The key has to be only root readable

- httpd conf:
- redirect 10.0.4.2 to www.my.domain.com. In conf.d/phpldapadmin.
- accessible only by x.x.x.x/24
- https://10.0.4.2/ldapadmin is the only url that responds.
- `chkconfig httpd on`
- httpd uses SSL certificates located in /etc/pki/tls/{certs,private}

---

- phpLDAPadmin
- Uncomment the login auth_type session
- Set the array of base DNs: `$servers&gt;SetValue('server','base',array('cn=config', 'dc=my,dc=domain,dc=com'));`
- Comment: `$servers-&gt;setValue('login','attr','uid');`. Default is search by dn. Check config.php.

- OpenLDAP
- `yum install openldap openldap-clients openldap-servers.`
- `yum remove nss-pam-ldapd`
- `chkconfig slapd on`

- setup logging
- `touch /etc/rsyslog.d/slapd.conf`
- `echo "local4.* /var/log/ldap.log" /etc/rsyslog.d/slapd.conf`
- `service rsyslog restart`

- log rotation:
- `vim /etc/logrotate.d/ldap`

  ```
  /var/log/ldap.log {
  rotate 7
  daily
  missingok
  notifempty
  delaycompress
  compress
  }
  ```

- `cp /usr/share/openldap servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG`
- `chown -R ldap:ldap /var/lib/ldap`
- `service slapd restart`
- `slappasswd`
- `vim /etc/openldap/db.lidf`

```
dn: olcDatabase={0}config,cn=config
changetype: modify
replace: olcRootPW
olcRootPW: {SSHA}9cuHU5PuQqtBKZvYMfUVNXgGFT9Kq7CU
```

- `ldapadd -Y EXTERNAL -H ldapi:/// -f db.ldif`

- Access phpldapadmin with: `cn=config` and your password.

- Create an `olcModuleList` entry to hold the modules we need:
- `ldapadd -Y EXTERNAL -H ldapi:/// -f file .ldif` the following:

  ```
  dn: cn=module,cn=config
  cn: module
  objectclass: olcModuleList
  olcmoduleload: back_hdb
  olcmoduleload: back_monitor
  olcmoduleload: syncprov
  olcmoduleload: constraint
  olcmodulepath: /usr/lib64/openldap
  ```

- Time to insert an hdb database:
- `ldapadd -Y EXTERNAL -H ldapi:/// -f file .ldif` the following:
  ```
  dn: olcDatabase=hdb,cn=config
  objectclass: olcDatabaseConfig
  objectclass: olcHdbConfig
  olcdatabase: hdb
  olcdbdirectory: /var/lib/ldap
  olcdbindex: objectClass pres,eq
  olcrootdn: cn=ldapadmin,ou=adm,dc=my,dc=domain,dc=com
  olcsuffix: dc=my,dc=domain,dc=com
  ```

There are a lot more I used to setup the configuration paramaters of the database, but I think these are the minumum.

- Let's get rid of the bdb database.
- `service slapd stop`
- `rm /etc/openldap/slapd.d/cn\=config/olcDatabase\=\{2\}bdb.ldif` and rename the olcDatabase={x}dbb file under slapd.d dir. The `x` has to reflect the new number of the hdb config entry.
- Also edit slapd.d/cn=config/olcDatabase={new_number_x}hdb.ldif and change the `dn` and the `oldDatabase` attributes to reflect that new `x`.
- `service slapd start`
- Cheeky and dangerous. _Don't do this if your slapd server is running!_

- Let's add the `constraint` module in hdb.
- `ldapadd -Y EXTERNAL -H ldapi:/// -f file .ldif` the following:

  ```
  dn: olcOverlay=constraint,olcDatabase={2}hdb,cn=config
  changetype: add
  objectClass: olcOverlayConfig
  objectClass: olcConstraintConfig
  olcOverlay: constraint
  olcConstraintAttribute: mail regex ^[[:alnum:]]+@my.domain.com$
  ```

- Let's add the `syncprov` module in hdb.
- `ldapadd -Y EXTERNAL -H ldapi:/// -f file .ldif` the following:

  ```
  dn: olcOverlay=syncprov,olcDatabase={2}hdb,cn=config
  changetype: add
  objectClass: olcOverlayConfig
  objectClass: olcSyncProvConfig
  olcOverlay: syncprov
  ```

- Let's prepare for replication
- Add an `olcServerID: 1 ldap://host.fqdn`. The fqdn has to be the server's fqdn hostname and has to agree with the TLS certificate CN. The ID (number) has to be unique among the replicated servers.
- `ldapmodify -Y EXTERNAL -H ldapi:/// -f file .ldif` the following:

  ```
  olcSyncrepl: rid=001 provider=ldap://master.fqdn
  starttls=critical
  tls_reqcert=allow
  schemachecking=on
  binddn="cn=ldapadmin,dc=my,dc=domain,dc=com"
  bindmethod=simple
  credentials="bluhpass"
  searchbase="dc=my,dc=domain,dc=com"
  type=refreshAndPersist
  retry="60 10 300 12 7200 +" timeout=1
  ```

- `ldapmodifly -Y EXTERNAL -H ldapi:/// -f file .ldif` the following:

  ```
  dn: olcDatabase={2}hdb,cn=config
  changetype: modify
  add: olcMirrorMode
  olcMirrorMode: TRUE
  ```

- In order to see if replication works:
- Set the starttls to no in the olcSyncrepl
- Beware of any constraints, they may break replication!

- cn=config attributes values:
- olcGentleHUP: true
- olcLogLevel: stats stats
- olcPasswordCryptSaltFormat: "$6$%.86s"
- cn=frontend,cn=config values:
- olcPasswordHash: {CRYPT}

---

- TLS setup (obsolete)
- Create CA: `certutil -S -s "C=GR, ST=Attiki, L=Agia Paraskevi, O=NCSR Demokritos, OU=Institute of Informatics and Telecommunications, CN=IIT NCSR CA" -n IITCA -x -t "CTu,u,u" -1 -2 -5 -f /etc/openldap/certs/password -v 120 -d /etc/openldap/certs`, it's an SSL CA and it's used for certificate signing.
- Create signing request :`certutil -R -s "C=GR,ST=Attiki,L=Agia Paraskevi,O=NCSR Demokritos,OU=Institute of Informatics and Telecommunications,CN=IIT NCSR" -o slapd.req -d ./certs`
- Create the certificate: `certutil -C -i slapd.req -o slapd.crt -d certs -c IITCA -v 240 -f ./certs/password`
- Add it to the Mozilla nss DB: `certutil -A -n "IIT slapd" -i slapd.crt -t "CT,," -d /etc/openldap/certs`
- It seems there's no interoperability between OpenSSL and Mozilla NSS. TLS handshaking between a CentOS 6 using Mozilla NSS and an Ubuntu 10.04 using OpenSSL breaks. I could be very wrong here!
- Ubuntu 10.04 come with gnutls support. No mozilla nss!

---

- NFS - server
- `yum install nfs-utils nfs-utils-lib nfs4-acl-tools` and `chkconfig nfs on`
- `cat /etc/exports`:
  `/export/users xxx.xxx.xxx.xxx(rw,no_subtree_check,mp=/mnt/hitachi/Vol1,sync)`
- `exportfs -ra`
- NFS - client

- `vim /etc/certmonger.conf`.
- Change `notification_method` to `mail`.
- Change `notification_destination` to an email address.
- `service certmonger restart`.

- enable iptables.

---

### References

- OpenLDAP installation
- [Installing and configuring openLDAP in CentOS 6.3](http://linuxserverathome.com/articles/installing-and-configuring-openldap-2423-centos-63)
- &lt;http://www.whotouchedmygun.com/wp-admin/post.php?post=621&amp;action=edit&amp;message=10&gt;
- &lt;http://wiki.centos.org/AdrianHall/CentralizedLDAPAuth?highlight=%28ldap%29#head-c44e0ab6188ce37ff04eb13dbb34e3f61d08ddab&gt;

- TLS (one deadly serious bitch!)
- &lt;http://gurkulindia.com/main/2013/04/rhel-6-3-ldap-series-part-4-troubleshooting/&gt;
- &lt;http://www.mozilla.org/projects/security/pki/nss/tools/certutil.html&gt;
- &lt;http://www.bradchen.com/blog/2012/08/openldap-tls-issue&gt;
- &lt;http://www.openldap.org/lists/openldap-technical/201202/msg00460.html&gt;

- SSSD stuf
- &lt;http://www.couyon.net/1/post/2012/04/enabling-ldap-usergroup-support-and-authentication-in-centos-6.html&gt;
- &lt;http://www.whotouchedmygun.com/wp-admin/post.php?post=621&amp;action=edit&amp;message=10&gt;
