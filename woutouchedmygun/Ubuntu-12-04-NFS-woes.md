---
author: Vassilis Vatikiotis
title: Ubuntu 12.04 - NFS woes
tags:
  - linux
  - ubuntu
  - NFS
---

# Ubuntu 12.04: NFS woes

This applies to other Ubuntu versions as well!

NFS slow mount. As a result, several services don't start, when they should.

Fix:

- specify the nfs version in mount options, like `nfsvers=3`,
- specify `bootwait` in the mount options in `/etc/fstab`, if you need your system to wait for NFS mount point. Useful for desktop installations.
- to watch `mountall` behaviour, specify `--verbose` in `/etc/init/mountall.conf`

#### Reference

[https://bugs.launchpad.net/ubuntu/+source/nfs-utils/+bug/891825](https://bugs.launchpad.net/ubuntu/+source/nfs-utils/+bug/891825)
