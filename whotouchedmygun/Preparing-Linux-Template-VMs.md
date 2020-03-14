---
author: Vassilis Vatikiotis
title: Preparing Linux Template VMs
tags:
  - linux
  - virtualisation
---

# Preparing Linux Template VMs

Tested on CentOS.

The original article is <a href="http://lonesysadmin.net/2013/03/26/preparing-linux-template-vms/">Preparing Linux Template VMs</a>, please read that first! This article is for preservation and my own personal use.

<h4>Step 1: Clean out yum and apt caches.</h4>

<p><code>/usr/bin/yum clean all</code>
<code>apt-get clean</code></p>

<h4>Step 2: Force the logs to rotate.</h4>

<p><code>/usr/sbin/logrotate –f /etc/logrotate.conf
/bin/rm –f /var/log/<em>-???????? /var/log/</em>.gz</code></p>

<h4>Step 3: Clear the audit log &amp; wtmp.</h4>

<p><code>/bin/cat /dev/null &gt; /var/log/audit/audit.log
/bin/cat /dev/null &gt; /var/log/wtmp</code></p>

This whole /dev/null business is also a trick that lets you clear a file without restarting the process associated with it, useful in many more situations than just template-building.

<h4>Step 4: Remove the udev persistent device rules.</h4>

<code>/bin/rm -f /etc/udev/rules.d/70\*</code>

<h4>Step 5: Remove the traces of the template MAC address and UUIDs.</h4>

<p><code>/bin/sed -i ‘/^&#40;HWADDR&#124;UUID&#41;=/d’ /etc/sysconfig/network-scripts/ifcfg-eth0</code>
Just removing unique identifiers from the template so the cloned VM gets its own.</p>

<h4>Step 6: Clean /tmp out.</h4>

<p><code>/bin/rm –rf /tmp/*
/bin/rm –rf /var/tmp/*</code></p>

Under normal, non-template circumstances you really don’t ever want to run rm on /tmp like this. Use tmpwatch or any manner of safer ways to do this, since there are attacks people can use by leaving symlinks and whatnot in /tmp that rm might traverse (“whoops, I don’t have an /etc/passwd anymore!”). Plus, users and processes might actually be using /tmp, and it’s impolite to delete their files. However, this is your template image, and if there are people attacking your template you should reconsider how you’re doing business. Really.

<h4>Step 7: Remove the SSH host keys.</h4>

<code>/bin/rm –f /etc/ssh/<em>key</em></code>

If you don’t do this all your VMs will have all the same keys, which has negative security implications.

<h4>Step 8: Remove the root user’s shell history</h4>

<p><code>/bin/rm -f ~root/.bash_history</code>
unset HISTFILE
No sense in keeping this history around, it’s irrelevant to the cloned VM.</p>
