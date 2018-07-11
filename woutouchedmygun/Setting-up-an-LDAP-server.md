---
author: Vassilis Vatikiotis
title: Setting up an LDAP server
tags:
  - linux
  - LDAP
---

# Setting up an LDAP server

Ubuntu 12.04

`# apt-get install nginx nginx-common nginx-light phpldapamdin slapd php5-suhosin`

**Logging**

By default slapd logs in `/var/log/syslog`. Let's change that to log in it's own file:

1.  `# touch /var/log/ldap.log`
2.  `# chown syslog.openldap /var/log/ldap.log`
3.  `# chmod g+rw /var/log/ldap.log`
4.  `# chmod u+rw /var/log/ldap.log`

Let's redirect slapd logs to the file we just created:

`vim /etc/rsyslog.d/50-default.conf` and add:

    *.*;auth,authpriv,local4.none -/var/log/syslog
    local4.* /var/log/ldap.log

Let's manage log rotation:

`vim /etc/logrotate.d/openldap` and add:

    /var/log/ldap.log {
      rotate 7
      daily
      missingok
      notifempty
      delaycompress
      compress
    }

Restart rsyslog service. Check [this](http://wiki.bluelightav.org/display/BLUE/LDAP+Authentication+-+Ubuntu+12.04#LDAPAuthentication-Ubuntu1204-Logging)

**phpLDAPAdmin and nginx**

`# apt-get install php5-cgi php5-fcgi`
`# rm /etc/nginx/sites-enabled/default`

`# vim /etc/nginx/site-available/phpldapadmin`

Inside the nginx conf file:

    server {
      server_name localhost;

      # document root
      root /usr/share/phpldapadmin/htdocs;
      index index.php index.html index.htm;

      # application: phpldapadmin
      location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        include fastcgi_params;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_pass 127.0.0.1:9000;
        #fastcgi_pass  unix:/var/run/php-fastcgi/php.sock;
      }

      # logging
      error_log /var/log/nginx/error.log;
      access_log /var/log/nginx/access.log;
    }

Note on `fastcgi_pass`: php is not configured to listen on the unix socket, only on the tcp socket.

_The above nginx host should be available on a private, administrative network interface and from the admin subnet ONLY! We don't want end users to be able to access phpLDALadmin!_

**Configuring LDAP: 0 step**

1.  Take a look at /etc/default/slapd. It controls how slapd exposes its services. If you need local ldap queries then use ldapi:///, otherwise if you want to provide ldap services in your network use ldap:///.
2.  /etc/ldap/ldap.conf controls and provides settings for various ldap utils. For instance, using `ldapsearch` with -ZZ (start TLS and fail if not successful) requires that a few TLS directives are set in this file.

**Configuring LDAP: 1st step, First steps**

I want to make it easy on myself and user phpLDAPadmin. I want to be able to see the DIT <em>and</em> the runtime configuration.

1.  Open phpLDAPadmin configuration file /etc/phpldapadmin/config.php and set the following up: `$servers->SetValue('server','base',array('cn=config', 'dc=my,dc=domain,dc=com'));`. Put it somewhere in the 'Define your LDAP servers in this section' section. This will enable you to see your DIT and the configuration as well.
2.  Once we set up a root password for the config database. Use `slappasswd` to create a password and then create an ldif file containing the following:

          dn: olcDatabase={0}config,cn=config
          add: olcRootPW
          olcRootPW: OutputOfSlappasswd

    The default rootDN for the config database is `cn=config`, so it's not necessary to define a `olcRootDN` attribute as well. By default, the config database is inaccessible (at least from phpLDAPadmin) _and_ there's no password for the rootDN, so by default (after installation) the config database is inaccesible to everyone _except the root user of the host slapd is running on (there's an ACL in the config db allowing this)_. This means that the config db can be accessed from the command line, using as root the ldap utilities, _via UNIX domain sockets i.e. ldapi_, but not from phpLDAPadmin. The rootDN for a database bypasses all ACLs.

3.  `# ldapmodify -Y EXTERNAL -H ldapi:/// -f ldif-file`. _As root_ we can access and manipulate the runtime configuration. So we insert a password, _via ldapi_ and now we can a) login as cn=config in phpLDAPAdmin *and* b) see the config db.

Accessing the runtime db via php  can be potentially dangerous. So I have restricted access to phpLDAPadmin down right to the network interface level and considering disabling UNIX domain sockets connections to slapd (within /etc/default/slapd).

**Configuring LDAP: 2nd step, Schema prep.**

Add the `misc` schema: Follow the directions in this [link](https://help.ubuntu.com/12.10/serverguide/openldap-server.html#openldap-configuration). The misc schema makes a few mail and nis related attributes available.

**Configuring LDAP: 2.5th step, TLS.**

Turning on TLS:

1.  Read the TLS section on <https://help.ubuntu.com/12.10/serverguide/openldap-server.html#openldap-configuration>. It uses gnutls instead of openssl to create certificates.
2.  The certificate key files have to be insecure, i.e. shouldn't be password protected. So file permissions are critical here. Use the group `ssl-cert` (from ssl-cert deb pkg) and make openldap a member of this group. The files have to be root:ssl-cert owned and their permissions 640, the directory containing them has to be root:ssl-cert owned and its permissions 710.
3.  Put the following definitions in a file:

         dn: cn=config
         changetype: modify
         add: olcTLSCertificateKeyFile
         olcTLSCertificateKeyFile: /etc/ssl/private/server.key.insecure
         -
         add: olcTLSCertificateFile
         olcTLSCertificateFile: /etc/ssl/certs/server.crt
         -
         add: olcTLSCACertificateFile
         olcTLSCACertificateFile: /etc/ssl/certs/ca.crt

    and execute: `# ldapmodify -Y EXTERNAL -H ldapi:/// -f ldif-file`

4.  We have to modify the slapd apparmor profile as well. Insert the following in /etc/apparmor.d/usr.sbin.slapd:

         /etc/ssl/private/ r,
         /etc/ssl/private/* r,
         /etc/ssl/certs/ r,
         /etc/ssl/certs/* r,

    and restart apparmor and slapd services.

5.  For convenience, add the TLS directives in /etc/ldap/ldap.conf. This file is used by various ldap utilities.
6.  (/etc/ldap/ldap.conf needs the proper TLS directives to be set) To test it use ldapsearch remotely, i.e. `dapsearch -x -ZZ -d 1 -H ldap://ldap.domain.com -D cn=admin,dc=domain,dc=com -W -b dc=domain,dc=com "(uid=your-uid)"`. `-d 1` is max debug level. There are issues with GnuTLS so this will tell you if the certificates are accepted by your ldap server.
7.  The certificate CN must match its hostname! Very important!

**Configuring LDAP: 3rd step, Monitor and Constraint overlay**

In order to load the monitor overlay put the following in a file:

    dn: cn=module{0},cn=config
    changetype: modify
    add: olcModuleLoad
    olcModuleLoad: back_monitor

and execute `# ldapmodify -Y EXTERNAL -H ldapi:/// -f ldif-file`

Now, to add the monitor database.

Put the following in a file:

    dn: olcDatabase=monitor,cn=config
    objectClass: olcDatabaseConfig
    olcDatabase: {2}monitor
    olcAccess: to * by ssf=128 dn.subtree=cn=Monitor by ssf=128 dn.exact=cn=admin,dc=your,dc=domain,dc=com write by users read by * none

and execute `# ldapadd -Y EXTERNAL -H ldapi:/// -f ldif-file`

In order to load the constraint module and create the constraint DN , put the following in a file:

    dn: cn=module{0},cn=config
    changetype: modify
    add: olcModuleLoad
    olcModuleLoad: constraint

    dn: olcOverlay=constraint,olcDatabase={1}hdb,cn=config
    changetype: add
    objectClass: olcOverlayConfig
    objectClass: olcConstraintConfig
    olcOverlay: constraint
    olcConstraintAttribute: mail regex ^[[:alnum:]]+@my.domain.com$</pre>

and execute `# ldapadd -Y EXTERNAL -H ldapi:/// -f ldif-file`

Restart slapd and logout-login if you are in phpldapadmin.

**Configuring LDAP: step 3.5**

I'm checking and documenting all the attributes found on my Ubuntu 10.04 LDAP installation, which was converted from slapd.conf to runtime configuration. See slapd.conf man page.

- Frontend database: For olcAddContentAcl, olcLastMod, olcSyncUseSubentry, use default values.
- Config database: olcAddContentAcl is true and _make sure ACLs are in place_.
- HBD database: olcDBNoSync to false for data integrity, oldAddContentAcl is true and _make sure ACLs are in place_.
- Configure the level of security for various tasks. The parameters here are global and inherited by all databases. olcLocalSSF is used for local LDAP sessions, e.g. ldapi://. Be sure to check slapd.conf man page.Similarly, put the following in a file:

        dn: cn=config
        changetype: modify
        replace: olcLocalSSF
        olcLocalSSF: 128
        #-
        #replace: olcSecurity
        #olcSecurity: update_ssf=128 simple_bind=128

  and execute `# ldapmodify -Y EXTERNAL -H ldapi:/// -f your-ldif-file.`

  The olcSecurity attribute is used to specify more fine grained security levels for various operations. Take care not to lock your self out. As long as olcLocalSSF is set to 128, access to ldapi:// can be used to restore access. This has to be explored!

- olcLogLevel attribute of cn=config should be `stats.`
- Create indexes, cache sizes for HDB database.

        dn: olcDatabase={1}hdb,cn=config
        changetype: modify
        add: olcDbIndex
        olcDbIndex: entryUUID eq
        -
        add: olcDbIndex
        olcDbIndex: entryCSN eq
        -
        add: olcDbIndex
        olcDbIndex: cn pres,eq,sub
        -
        add: olcDbIndex
        olcDbIndex: objectClass pres,eq,sub
        -
        add: olcDbIndex
        olcDbIndex: loginShell pres,eq,sub
        -
        add: olcDbIndex
        olcDbIndex: uidNumber pres,eq
        -
        add: olcDbIndex
        olcDbIndex: gidNumber pres,eq
        -
        add: olcDbIndex
        olcDbIndex: ou pres,eq,sub
        -
        add: olcDbIndex
        olcDbIndex: givenName pres,eq,sub
        -
        add: olcDbIndex
        olcDbIndex: memberUid pres,eq,sub
        -
        add: olcDbIndex
        olcDbIndex: uid pres,eq,sub
        -
        add: olcDbIndex
        olcDbIndex: sn pres,eq,sub
        -
        add: olcDbIndex
        olcDbIndex: mail pres,eq,sub
        -
        add: olcDbIndex
        olcDbIndex: uniqueMember pres,eq
        -
        add: olcDbCacheSize
        olcDbCacheSize: 1000
        -
        add: olcDbIDLCacheSize
        olcDbIDLCacheSize: 3000

  Execute `# ldapmodify -Y EXTERNAL -H ldapi:/// -f ldif-file`

- olcDbIDLCacheSize is the number of in-memory recent used index matches and for hdb it should be e times olcDbCacheSize.

**Configuring LDAP: Step 4, MirrorMode Replication**

Provider configuration:

    dn: cn=module{0},cn=config
    changetype: modify
    add: olcModuleLoad
    olcModuleLoad: syncprov

Execute: `# ldapmodify -Y EXTERNAL -H ldapi:/// -f ldif-file`. This will make the syncprov module to be loaded on slapd start.

    dn: olcOverlay=syncprov,olcDatabase={1}hdb,cn=config
    changetype: add
    objectClass: olcOverlayConfig
    objectClass: olcSyncProvConfig
    olcOverlay: syncprov

The above sets up the hdb database to be the provider of the replication process. Execute the <code>ldapmodify</code> command, as usual.

Consumer configuration:

    dn: olcDatabase={1}hdb,cn=config
    changetype: modify
    add: olcSyncRepl
    olcSyncRepl: rid=001
      provider=ldap://ldap1.your.domain.com
      starttls=critical
      tls_reqcert=allow
      binddn="cn=admin,dc=your,dc=domain,dc=com"
      bindmethod=simple
      credentials="WouldntYouLikeToKnow"
      searchbase="dc=your,dc=domain,dc=com"
      type=refreshAndPersist
      retry="60 10 300 12 7200 +"
      timeout=1

- `rid` identifies the current syncrepl directive withing the current consumer.
- if TLS request fails then session is aborted. Note that the syncrepl engine doesn't use the main TLS settings, instead the TLS parameters from ldap.conf(5) are used, or explicitly set in the syncrepl stanza. UPDATE: this ServerFault post <http://serverfault.com/questions/440968/openldap-startup-problems-after-upgrade> says that syncrepl uses its own TLS directives, and  the ones from /etc/ldap/ldap.conf. I follow the Ubuntu Server Guide which doesn't follow the ServerFault settings, instead it relies on the main TLS parameters.
- `tls_reqcert` is set to allow so if a client certificate check fails the session proceeds normally, i.e. a TLS session is initiated. The default is `demand` which on Ubuntu 12.04 fails due to GnuTLS bugged version (see [3])
- `provider` is the URL where the other master is. The URL must match the certificate's CN, which must match the server's FQDN!
- `binddn` is the dn of a user who can read every entry in the database.
- `searchbase` is the root of the replicated tree.
- Set olcLogLevel to <em>conns </em>to log and troubleshoot TLS session management.

Mirror mode:

Both servers have to be configured as consumers and providers. To that end, each server has to have a unique ServerID:

    dn: cn=config
    changetype: modify
    replace: olcServerID
    olcServerID: 1 ldap://ldap1.your.domain.com

    dn: olcDatabase={1}hdb,cn=config
    changetype: modify
    add: olcMirrorMode
    olcMirrorMode: TRUE

Execute the `ldapmodify` on each server.

**Configuring LDAP: Step 5, Various Tasks**

- I delete `cn=admin,dc=your,dc=domain,dc=com` and place the ldap admin someplace else in the DIT. YMMV.

**References**

1.  <https://help.ubuntu.com/12.10/serverguide/openldap-server.html#openldap-configuration>
2.  <http://www.math.ucla.edu/~jimc/documents/ldap/ldap-setup-1202.html>
3.  <http://serverfault.com/questions/398684/ubuntu-12-04-ldap-ssl-self-signed-cert-not-accepted>
4.  <https://help.ubuntu.com/community/GnuTLS>
