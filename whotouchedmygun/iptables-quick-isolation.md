---
author: Vassilis Vatikiotis
title: iptables quick isolation
tags:
  - linux
---

# iptables quick isolation

The following iptables for quick isolation of port 8080 on eth1. eth0 is open for all.

```bash
# iptables -A INPUT -s 127.0.0.1 -j ACCEPT
# iptables -A INPUT -i eth0 -j ACCEPT
# iptables -A INPUT -i eth1 -p tcp --dport 8080 -j ACCEPT
# iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
# iptables -A OUTPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
# iptables -P INPUT DROP
```
