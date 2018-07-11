---
author: Vassilis Vatikiotis
title: Coarse-grain authentication using PAM
tags:
  - linux
  - authentication
  - PAM
---

# Coarse-grain authentication using PAM

- I want to disallow a network user from logging or ssh'ing in a machine. The `pam_localuser` module can be and if I want to specify a password file other than `/etc/passwd`, the `file` parameter can be used, [check man page](http://linux.about.com/library/cmd/blcmdl8_pam_localuser.htm).
- Take care to use `pam_localuser` to the `account` pam management group, in addition to the `auth` group. If you don't then you'll be allowing SSH key-based authentication to _everyone_.
- Using the `pam_wheel` module I can specify which group I want to allow.

PS: this could be done in `sshd_config` using the `AllowUsers` directive. Public key authentication is controlled by the `PubKeyAuthentication` directive.
