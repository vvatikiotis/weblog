---
author: Vassilis Vatikiotis
title:  Postfix, Dovecot, LDAP
tags:
  - linux
  - ldap
  - postfix
  - dovecot
---

# Postfix, Dovecot, LDAP

On a Debian squeeze box.

My postfix setup, in `main.cf`:

    alias_maps = hash:/etc/aliases ldap:/etc/postfix/ldap-aliases.cf
    local_recipient_maps = unix:passwd.byname $alias_maps

because I've set my mailer for local delivery.

The `ldap-aliases.cf` file contains:

    server_host = x.x.x.x x.x.x.x
    version = 3
    bind = no
    search_base= dc=my,dc=domain,dc=com
    query_filter= (mail=%s)
    result_attribute = uid

- No need to bind to check whether a user exists!
- email address is stored in `mail` attribute.
- inbox name is the user's `uid`

My dovecot setup, in `dovecot-ldap.conf`:

    hosts = x.x.x.x x.x.x.x
    auth_bind = yes
    auth_bind_userdn = uid=%u,ou=people,dc=my,dc=domain,dc=com
    base = dc=my,dc=domain,dc=com

- I use the authentication binds method and it's configured using the DN template method.
- Connection optimisation is setup as well. This is done in `dovecot.conf` and it's dead simple, <http://wiki.dovecot.org/AuthDatabase/LDAP/AuthBinds>
