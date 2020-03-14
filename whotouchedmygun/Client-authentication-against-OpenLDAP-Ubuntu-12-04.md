---
author: Vassilis Vatikiotis
title: Client authentication against OpenLDAP, for Ubuntu 12.04
tags:
  - linux
  - HP
---

# Client authentication against OpenLDAP, for Ubuntu 12.04

This [link](https://help.ubuntu.com/community/LDAPClientAuthentication) contains all the necessary information. All changes to `/etc/ldap.conf`, meaning the old method is used (no nslcd, no \*ldapd packages). Some points that I had to fiddle with:

- When using multiple ldap servers (for redundancy) I had to tweak `bind_timelimit` to a lower number. I don't want to wait 30 secs (default value) when my first uri entry (first ldap server) is dead in order to try the second ldap uri.
- For the same reason I changed `bind_policy` to soft so reconnect will fail immediately and try the second uri entry.
- This can be found [here](https://help.ubuntu.com/community/LDAPClientAuthentication#nss_base_.3Cmap.3E_.28recommended.29). In summary, the `nss_base_*` directives have to be set otherwise logging in using an ldap account is not possible.
- I set the `binddn` and `bindpw` directives as I've setup a bind user in ldap server. I do not allow anonymous binds. To hide the `bindpw` password I removed read permissions on others.
- I also set `rootbinddn` to an ldap user who's authorised to change attributes in ldap entries. For instance, I need the local root user to be able to change a user's password.
- I set the `pam_password` directive to `exop` to make `passwd` use the ldap server password policy (i.e. value of `olcPasswordHash`). To have this working properly, I needed to add the following in the ldap server configuration:

  - olcPasswordCryptSaltFormat: $6$%.86s
  - olcPasswordHash: {CRYPT}

  Ok, in plain English now. When `olcPasswordHash` is set to {CRYPT}, then crypt(3) is used. crypt(3) is a cryptographic function which accommodates several encryption algorithms. {CRYPT} does NOT imply "use the crypt algorithm". When {CRYPT} is used, `olcPasswordCryptSaltFormat` specifies which encryption algorithm crypt(3) uses. $6$ is SHA512 and %.86s is 86 characters. Read "man 3 crypt" and "man 5 slapd.conf". Also take a look at [this](http://www.shermann.name/2010/08/openldap-passwd-and-crypt-passwords.html).
  Added bonus for `exop` and using crypt(3). Passwords are UNIX compatible and can be transferred to a Unix password file.

- **Watch this:** when I change the password using phpLDAPadmin the new password entry doesn't follow the `olcPasswordCryptSaltFormat` format, instead the new password is hashed using plain crypt. Since `pam_password` is exop and the 2 `olcPassword*` directives are set in ldap server (see previous bullet), both hashed passwords work. Use case: I change my password using the `passwd` shell command. The password stored in ldap follows `olcPasswordCryptSaltFormat`. Logging in works. Then I change my password within phpLDAPadmin. The password stored in ldap is a simple crypt encoded password. Logging still works. To ensure that the ldap passwords are stored in their strong form, the last password change should happen via the shell passwd command. There should be a fix for this! (or I'm missing something).
  **Answer:** Setting `pam_password` to `exop` and thus using the server's password policy means that crypt(3) handles the password generation. Since phpLDAPadmin is a web interface, there should be a php cryptographic function that behaves and produces similar output to crypt(3). Is there such a function? For the time being I do not use phpLDAPadmin to set passwords. The shell is fine.

Some additional staff:

- Authentication involves PAM so the ldap pam profile has to be activated, under `/usr/share/pam-configs`. I set minimum uid to 1000, which is the default.
