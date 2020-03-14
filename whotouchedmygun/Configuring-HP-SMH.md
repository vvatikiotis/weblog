---
author: Vassilis Vatikiotis
title: Configuring HP SMH
tags:
  - linux
  - HP
---

# Configuring HP SMH

On Ubuntu 12.04 for HP ProLiant DL385 G7.

First download the HP ProLiant Support Pack CD for dpkg-based distributions, I got it from <a href="http://h20565.www2.hp.com/portal/site/hpsc/template.PAGE/public/psi/swdDetails/?sp4ts.oid=4132957&amp;spf_p.tpst=swdMain&amp;spf_p.prp_swdMain=wsrp-navigationalState%3Didx%253D%257CswItem%253DMTX_c45bd2c25e6849d19f98427649%257CswEnvOID%253D4117%257CitemLocale%253D%257CswLang%253D%257Cmode%253D%257Caction%253DdriverDocument&amp;javax.portlet.begCacheTok=com.vignette.cachetoken&amp;javax.portlet.endCacheTok=com.vignette.cachetoken" target="_blank">here</a>.

Mount the iso `$ mount -o loop` and `dpkg -i` the .deb packages. Some additional packages have to be installed, they are all in the repository. The HP packages are installed, by default, under /opt.

To test if everything is ok we run `/opt/hp/hpsmh/sbin/smhconfig`. At first we
were greeted with

<blockquote>./smhconfig: error while loading shared libraries: libssl.so.0.9.8: cannot open shared object file: No such file or directory</blockquote>
It seems that the runtime linker can't find these libraries, which come as part of the HP iso. In order to fix this:

```sh
# echo "/opt/hp/hpsmh/lib" &gt; /etc/ld.so.conf.d/hpsmh.conf
# lddconfig
```

Fixed!

Enable the ACU (Array Configuration Utility) to be accessible via the SMH.

`# cpqacuxe --enable-remote`

You should last run:
`# /opt/hp/hpsmh/sbin/smhconfig -d adm`
where adm is the group name of the user able to login in SMH (SMH uses host's user authentication i.e. if your account belongs to the adm group then login in to SMH, otherwise add it).

Visit http://Your-Server-IP:2301/. This should automatically start the HP SMH server and redirect you https://Your-Server-IP:2381/.

<a href="http://www.ganesh.me/175-install-hp-system-management-homepage-with-proliant-support-pack-on-centos-5.html" target="_blank">This port</a> is my primary source.
Also check <a href="http://www.unrelatedshit.com/2011/05/30/proliant-management-tools-for-ubuntu-debian/" target="_blank">HP ProLiant management tools for Ubuntu/Debian based server</a> and <a href="http://www.datadisk.co.uk/html_docs/redhat/hpacucli.htm" target="_blank">how to use hpacucli</a>.

SNMP and HP Agents and HP Insight has also to be installed in order for SMH to provide meaningful info. This post to be continued...
