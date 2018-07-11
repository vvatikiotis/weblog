---
author: Vassilis Vatikiotis
title:  Notes on setting up an NFS server
tags:
  - linux
  - NFS
  - ubnutu
---

# Notes on setting up an NFS server

For Ubuntu 12.04

<a href="https://help.ubuntu.com/community/NFSv4Howto" target="_blank">This</a> and <a href="https://help.ubuntu.com/community/SettingUpNFSHowTo" target="_blank">this</a> are very good source of information.

Some point of importance:

<ul>
	<li>Ubuntu 12.04 has replaced portmap with rpcbind.</li>
	<li>Related to the above point: portmapper is no longer needed and one port is used for all communication, port 2049.</li>
	<li>This is optional. If you choose to setup RPC lockdown follow the instructions of the 2nd link. Keep in mind that localhost and your machine's IP has to be included in the list of allowed hosts. Restart rpcbind service whenever hosts.allow and hosts.deny are changed. <code>showmount -e</code> is your friend. <code>rpcinfo</code> is another one.</li>
	<li>From the 1st link, 4th note of the NFSv4 Client section: "NFSv4 has a global root directory and all exported directories are children to it".</li>
	<li>This is NFSv4 specific: When setting up <code>/etc/exports</code> take care if, when and where to set the fsid option. Initially I set it up as fsid=0 and as soon as I tried to mount an NFS export point I would get "mount.nfs4: access denied by server while mounting nfs1:/export/users". I removed it completely from my exports file. UPDATE: fsid=0 has to be used in the root entry <em>only, which is the 1st exported directory i.e. /export</em>.</li>
	<li>From the 1st link, 3rd note of the NFSv4 Client section: " With a type of nfs4 this option is ignored, but can be used with mount -O _netdev in scripts later. Currently Ubuntu Server does not come with the scripts needed to auto-mount nfs4 entries in /etc/fstab after the network is up."!</li>
	<li>I don't trust Ubuntu (for now) to mount network filesystems, as the very useful /etc/init.d/netfs script is missing. To that end: /etc/default/rcS DELAYLOGIN var is set to yes.</li>
	<li>NFSv4 export configurations are not always backwards compatible to NFSv3.</li>
	<li>Take care to 1st create /export, b) create directories under /export and c) mount --bind these directories (created in step b), and <em>not</em> the /export dir.</li>
	<li>idmapd service is only needed for NFSv4. We can safely ignore it if we deploy NFSv3 style.</li>
	<li>For Ubuntu 12.04 the /etc/idmapd.conf default settings for:
<ul>
	<li>Domain, is <code>$ hostname --fqdn</code>, so setup the fqdn hostname in /etc/hosts</li>
	<li>Method in Translation section, is nsswitch, which means LDAP will be queried first, which suits me fine since I have already set up my nfs server as an ldap client for user authentication.</li>
</ul>
</li>
</ul>
<strong>References</strong>
<ul>
	<li><span style="font-size: 16px;">Apart from the 2 links provided at the beginning of the post, </span><a style="font-size: 16px;" href="http://nfsv4.bullopensource.org/doc/NFS3_NFS4_migration.pdf">NFSv3 to NFSv4 migration guide</a><span style="font-size: 16px;"> is very useful.</span></li>
	<li><a href="http://www.centos.org/docs/5/html/Deployment_Guide-en-US/s1-nfs-server-config-exports.html" target="_blank">This</a> CentOS link makes what the NFSv4 root pseudo-file system is and how it can be used.</li>
	<li>There is a very interesting post, a bit disturbing though, at <a href="http://serverfault.com/questions/415860/nfs4-permission-denied-when-userid-does-not-match-even-though-idmap-is-working" target="_blank">ServerFault</a>. The comments and the links in them are important!</li>
</ul>
