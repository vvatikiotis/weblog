---
author: Vassilis Vatikiotis
title:  LDAP client authentication on Debian squeeze
tags:
  - linux
  - ldap
  - postfix
---

# LDAP client authentication on Debian squeeze

I'm using the newer method, which involves `nslcd`

- `# apt-get install nslcd libnss-ldapd libpam-ldapd unscd`.
- `vim /etc/nslcd.conf`
  - Setup your `base`s, `uri` and so on.
  - `ldap_version 3`
  - Set `bind_timelimit` to 10.
  - There's no `bind_policy` option, nor `nss_timeout` options. According to [link](http://signalboxes.net/2012/01/reliable-ldap-timeout-with-nslcd/) this is problematic. The solution comes from the `pam_ldap` module, where the `ignore_authinfo_unavail` can be used so when there's no answer from an LDAP server the PAM stack ignores the `pam_ldap` module. _Very_ handy.
  - `nslcd.conf` file perms should be 640.
- Check `/usr/share/pam-configs/ldap` template, you may want to tweak something. I had to change the `minimum_uid` cause my ldap users start well above that. Additionally, the `ignore_authinfo_unavail` can be set there (although the issue seems to have already been dealt with).
- Set `rootpwmoddn` to a DN used for password modifications.
- Setup `binddn` and `bindpwd`. These are necessary if I want to retrieve full (multiple) group membership and account info.
- `# pam-auth-update`.

Simple.

**Massive question**: Do we actually have to set `binddn` and `bindpw` to get group information? Why? I can't see anything wrong with my ACLs.
**UPDATE**: The question remains. Even more, when I bind anonymously only half the user entries are returned! It's as if the other ones simply don't exist!
