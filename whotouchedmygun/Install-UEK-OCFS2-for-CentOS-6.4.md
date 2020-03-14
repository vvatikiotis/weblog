---
author: Vassilis Vatikiotis
title: Install UEK OCFS2 for CentOS 6.4
tags:
  - linux
  - ubuntu
---

# Install UEK OCFS2 for CentOS 6.4

- Follow instructions [here](http://public-yum.oracle.com/), for Oracle Linux 6
  - `# cd /etc/yum.repos.d`
  - `# wget --no-check-certificate https://public-yum.oracle.com/public-yum-ol6.repo`
- Edit the repo file and enable the [ol6_UEK_latest] repository only.
- Import the Oracle GPG key, `rpm --import http://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol6`
- `# yum install kernel-uek ocfs2-tools`
- Edit `/boot/grub/menu.lst` and set the `default` option to the UEK kernel (NB, counting starts from 0).
