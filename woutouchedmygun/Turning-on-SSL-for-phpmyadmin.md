---
author: Vassilis Vatikiotis
title: Turning on SSL for phpmyadmin
tags:
  - linux
  - SSL
---

# Turning on SSL for phpmyadmin

**This is for Ubuntu 12.04**

Post [18](http://ubuntuforums.org/showthread.php?t=1099536&p=12094913#post12094913) was the right and less intrusive solution for me. One additional thing I had to do to get everything working was to enable the SSL module:
`a2enmod ssl`

After a restart everything worked.
