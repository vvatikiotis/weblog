---
author: Vassilis Vatikiotis
title:  GRUB and gfx mode on Ubuntu 12.04 and CentOS
tags:
  - linux
  - GRUB
---

# GRUB and gfx mode on Ubuntu 12.04 and CentOS

Ubuntu:

For some reason (I suspect the gfxterm module is missing or grub scripts cannot locate it), I get a blank screen whenever an Ubuntu 12.04 finishes booting up. The machine is accessible via ssh but I don't have console access.

My solution:

<ol>
	<li>Open <em>/etc/default/grub</em></li>
	<li>Set <em>GRUB_CMDLINE_LINUX_DEFAULT="nomodeset"</em></li>
	<li>Add <em>GRUB_TERMINAL=console</em> and <em>GRUB_GFXPAYLOAD_LINUX=text</em></li>
</ol>
and after a update-grub no more blank screen after booting up.

CentOS
`grubby --update-kernel=ALL --remove-args="rhgb quiet"` for all and future kernel versions.

Add nomodeset as well in kernel stanza in /boot/grub/grub.conf
