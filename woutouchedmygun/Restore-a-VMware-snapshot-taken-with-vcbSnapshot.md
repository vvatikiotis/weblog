---
author: Vassilis Vatikiotis
title: Restore a VMware snapshot taken with vcbSnapshot
tags:
  - linux
  - virtualisation
---

# Restore a VMware snapshot taken with vcbSnapshot

Some good references:

- <http://dsumsky.blogspot.gr/2008/10/vcb-basic-usage-vm-restore-with.html>
- <http://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=1006844>

In a nutshell:

1.  Go to the snapshot directory.
1.  Backup the original `catalog` file.
1.  Edit it and change the storage spec and the hostname spec.
1.  Use the `vcbRestore` cmd tool:

    `vcbRestore -s /snapshot-dir -h localhost -u`

Again, this is for my own reference, If this is of any help I'm happy for you, otherwise Google is your friend.
