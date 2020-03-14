---
author: Vassilis Vatikiotis
title: Notes on ocfs2
tags:
  - linux
  - ocfs2
---

# Notes on OCFS2

CentOS 6.5, `uname -r` => 2.6.39-400.211.3.el6uek.x86_64, ocfs2-tools version 1.8.0-10.el6

- **Important** Global heartbeat works with ocfs2-tools-1.8.0-11 and later.
- A nice post about global heartbeat <https://blogs.oracle.com/wim/entry/ocfs2_global_heartbeat>.
- An oracle article <http://docs.oracle.com/cd/E37670_01/E37355/html/ol_instcfg_ocfs2.html>. Notice the use of more than 3 shared devices in order to proper implement global heartbeat. We'll create 4 heartbeat devices.
- <http://richardpickett.com/ocfs2-over-iscsi-on-ubuntu-10-10-maverick-meerkat>
- <https://oss.oracle.com/pipermail/ocfs2-users/2010-March/004272.html>
- <http://www.querzone.de/wiki/Wiki.jsp?page=OCFS2>
- [The Official Oracle manual](https://docs.google.com/viewer?url=https%3A%2F%2Foss.oracle.com%2Fprojects%2Focfs2%2Fdist%2Fdocumentation%2Fv1.8%2Focfs2-1_8_2-manpages.pdf)
- Setting `error.panic` and `kernel.panic_on_oops` kernel variables (in 7.2.6).
- A `mkfs` sample: `mkfs.ocfs2 -b 4K -C 1M -L ocfs2volname -N 14 --fs-feature-level=max-features -T vmstore --cluster-name=ocfs2 --cluster-stack=o2cb --global-heartbeat /dev/sdX` to create a heartbeat device. Notice that we use the device, not a partition on it.
  - N is the number of nodes. For best performance set it to at least twice the actual number of nodes
  - C is the cluster size. For few large files set to 1M.
  - T specifies the type of usage for the file system. Not needed for heartbeat devices
  - Partition a volume and then use `mkfs.ocfs2` if it's going to be used as a datastore.
- To partition very large volumes use `gdisk`, it can be found in the EPEL repo.
  - `mkfs.ocfs2` is the next step. Remember to use the `-T vmstore` option.
- `mkfs.ocfs2` a partition only **once**, that means on one host.
- `vim /etc/sysconfig/o2cb`. Specify cluster name and tell the service to start on boot.

## Complete `mkfs` commands:

- To create a vmstore volume: `mkfs.ocfs2 -b 4K -C 1M -N 14 -L FSDS -T vmstore --cluster-name=ocfs2cluster --cluster-stack=o2cb --global-heartbeat /dev/mapper/fsdsp1`.
- To create a global heartbeat device: `mkfs.ocfs2 -b 4K -C 1M -N 14 -L HB1 --cluster-name=ocfs2cluster --cluster-stack=o2cb --global-heartbeat /dev/devX`, notice that we do _not_ partition the device.
- Chown to oneadmin before mounting (done once!)

## Timeouts

- O2CB_HEARTBEAT_THRESHOLD = 61, for multipath users (check <https://docs.google.com/viewer?url=https%3A%2F%2Foss.oracle.com%2Fprojects%2Focfs2%2Fdist%2Fdocumentation%2Fv1.8%2Focfs2-1_8_2-manpages.pdf>)

## Setup

- 4 heartbeat devices for global heartbeat.
- Heartbeat devices are not multipathed.
- O2CB_HEARTBEAT_THRESHOLD = 61 is 61 cause our storage volumes are multipathed.
- Heartbeat starts as soon as the volumes are mounted.
