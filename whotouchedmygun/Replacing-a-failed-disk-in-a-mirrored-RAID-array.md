---
author: Vassilis Vatikiotis
title: Replacing a failed disk in a mirrored RAID array
tags:
  - linux
---

# Replacing a failed disk in a mirrored RAID array

Falko Timme's original [article](http://www.howtoforge.com/replacing_hard_disks_in_a_raid1_array) is more comprehensive, I'm just copying all the juicy stuff (YMMV) in order to preserve it and for my quick reference. Kudos Falko.

1.  Remove the disk:
    ```sh
    # mdadm --manage /dev/md0 --fail /dev/sdb1`
    # mdadm --manage /dev/md0 --remove /dev/sdb1
    ```
1.  Add the new disk:
    ```sh
    # mdadm --manage /dev/md0 --add /dev/sdb1
    ```
