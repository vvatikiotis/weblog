---
author: Vassilis Vatikiotis
title: Apache userdirs from LDAP
tags:
  - linux
  - ldap
  - apache
---

# Apache userdirs from LDAP

Debian Squeeze

Something like the following should be enough. I had no trouble at all.

    LDAPProtocolVersion   3
    LDAPUserDirUseTLS     off
    LDAPUserDirServer     ldap1.my.domain.com ldap2.my.domain.com
    LDAPUserDirDNInfo     cn=bind-user,dc=my,dc=domain,dc=com apassword
    LDAPUserDirBaseDN     ou=people,dc=my,dc=domain,dc=com
    LDAPUserDirFilter     "(&(uid=%u)(objectClass=posixAccount))"
    LDAPUserDir           public_html
    LDAPUserDirCacheTimeout 31104000

The mod_ldap_userdir homepage is in github, the directives page is what you're looking for, <https://github.com/jwm/mod_ldap_userdir/blob/master/DIRECTIVES>
