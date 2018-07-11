---
author: Vassilis Vatikiotis
title:  iSCSI and multipath setup on Ubuntu 12.04
tags:
  - linux
  - iSCSI
  - multiplath
---

# iSCSI and multipath setup on Ubuntu 12.04

<strong>iSCSI setup</strong>
Install the packages:

<code>$ sudo apt-get install open-iscsi open-iscsi-utils multipath-tools</code>

<code></code>Prepare the iSCSI interfaces in /etc/network/interfaces:

<pre># Do this for every interface
auto eth0
iface eth0 inet static
address 10.0.5.11
netmask 255.255.255.255 #beware
network 10.0.5.0
post-up sysctl -w net.ipv4.conf.eth0.rp_filter=0
post-up route add -host 10.0.5.2 dev eth0</pre>

The route is necessary cause, in our setup, we have 2 interfaces connecting to 2 iscsi targets and we want traffic to flow correctly. We could do away with setting up the routes, if we had setup our 2 iSCSI interfaces to be on different vlans. Before setting up the routes, we noticed that traffic destined for the second target was coming out from the wrong ethernet device.

Also note the netmask. If we'd put 255.255.255.0 we'd be creating an additional route for the 10.0.5.0 network. We don't want this.

Continuing with our network setup, and this could be an Ubuntu hiccup, we noticed that when setting up the routes for our iSCSI ifaces, 2 additional routes were setup, each one being a network wide route. We delete these 2 routes at the end of the interfaces file. (This 2 routes are created because we set an 255.255.255.0 netmask. Read the previous note).

<pre>[post-up route del -net 10.0.5.0 netmask 255.255.255.0 dev eth0</pre>

<strong>Discovery process</strong>
Note that YMMV!

As per the iSCSI documentation and assuming that the packages are installed and the host is connected to the SAN box, the following are needed:

<ol>
	<li>Configure the iSCSI port net.ipv4.conf.ethX.rp_filter to be 0 or 2. This can be done either in /etc/sysctl.conf or using the post-up eth stanza hook at /etc/network/interfaces.</li>
	<li>Create a iface file under /etc/iscsi/ifaces. This can be done using the <em>iscsiadm</em> configuration utility. I did it manually so I have an iface0 file under the /etc/iscsi/ifaces which contains:
<code>
iface.iscsi_ifacename = iface0
iface.hwaddress = e4:11:5b:b0:6d:32
iface.transport_name = tcp
</code>
This is the absolute minimum for an iface file.</li>
	<li>Important! If you have dedicated more than 1 ethernet interface (in the host) for iSCSI, you might need to establish routes, <em>if you have both iSCSI interfaces on the same lan</em>. My route table looks like this:
<pre>10.0.5.3        0.0.0.0         255.255.255.255 UH    0      0        0 eth2
10.0.5.5        0.0.0.0         255.255.255.255 UH    0      0        0 eth3</pre>
10.0.5.3 and 10.0.5.5 are the iSCSI targets on the SAN box and I have dedicated eth2 and eth3 on the host. So I need traffic from eth2 to end up in 10.0.5.3 and traffic from eth3 to 10.0.5.5. I was bitten by the lack of routes cause all traffic was going out of eth2 so discovering a target in 10.0.5.5 was always failing.
There's no need for routes if each iSCSI interface is in each own vlan.</li>
	<li><code># iscsiadm -m discovery -t st -p ip:port -I ifaceX -P 1</code>, or in my case <code># iscsiadm -m discovery -t st -p 10.0.5.3 -I iface0 -P 1</code></li>
	<li>Prepare an iSCSI target in your SAN box. This step has to be taken after iSCSI discovery so your SAN box knows about the initiator. You may have to reissue the above iscsi discovery command.</li>
</ol>
The above steps should do the trick. As I said before, your mileage may vary.

To see all the discovered nodes:

<code># iscsiadm -m node -P [0 | 1]</code>

The -P flag indicates level of info.

To automatically login to a node edit /etc/iscsi/iscsid.conf:
<code> node.startup=automatic node.conn[0].startup = automatic </code>

and restart the iscsi daemon
<code># /etc/init.d/open-iscsi restart</code>
or
<code># iscsiadm -m node -T targetname -p ip:port --op update -n node.startup -v automatic</code>

Beware that if you have already discovered a node, the 1st method will not work. It will work, however, for nodes discovered afterwards. Also, if despite all these, sessions are not started automatically, check /etc/iscsi/nodes/.../.../iface file for node.startup and node.conn[0].startup values; they may still have the wrong value.

There is a bug in the open-iscsi init script, where the only automatic login happens on the default interface. check <a href="https://bugs.launchpad.net/ubuntu/+source/open-iscsi/+bug/1001535">https://bugs.launchpad.net/ubuntu/+source/open-iscsi/+bug/1001535</a>.

<strong>Multipath setup</strong>
Weird thing, there is no multipath configuration files under /etc. So we copy from the /usr/share/doc:

<code># cp /usr/share/doc/multipath-tools/examples/multipath.conf.synthetic /etc/multipath.conf</code>

<code></code>Without any specifics or optimizations the multipath configuration file has to contain:

<pre>defaults {
    user_friendly_names     yes
}
blacklist {
    devnode "^(ram|raw|loop|fd|md|dm-|sr|scd|st)[-1-9]*"
    devnode "^hd[a-z][[0-9]*]"
    wwid 3600508b1001cea3b91ee45c710e467a7 #sda
    devonode "sr0"
}
devices {
    device {
        vendor "HITACHI "
        product "DF600F.*"
        path_grouping_policy multibus
    }
}</pre>

We should blacklist anything that's not multipathed, e.g. cdrom, floppy, local drives. <code>/lib/udev/scsi_id -g /dev/sda</code> to find out their WWID. It's safer to reference devices by their WWID.
Use the friendly names property. It will give us a nice friendly name to work with the iscsi disk, instead of the ugly WWN (World Wide ID). the friendly name looks like <em>mpath0</em>.
The <em>vendor</em> and the <em>product</em> strings have to be specified, as per the example. The <em>vendor</em> string is <em>strictly</em> a 8 character string and the <em>product</em> is <em>strictly</em> a 16 character string (in iSCSI specification). Regular expressions work.

<strong>iSCSI, Multipath optimization</strong>
In order to minimize timeouts (read about it in the iSCSI spec), we set:
<code> replacement_timeout = 20 noop_out_timeout = 5 </code>
in iscsi.conf.
25 seconds in total before a failed iscsi path is replaced with a functional one.

Just as stated before, if you have created the nodes before setting these timeouts you can change them manually in the node files under /etc/iscsi/nodes, or using the iscsiadm utility with the <em>--op update</em> switch.

<strong>Testing</strong>

<pre># dd if=/dev/mapper/mpath0 of=/dev/null
# iostat -m 2</pre>

If the <em>mpath0</em> multipath device is mapped to <em>dm-0</em> and the iscsi devices are, say, <em>sdc</em> and <em>sdd</em>, then both iscsi devices have to be utilised to their max capabilities .i.e. 1gbps, and <em>dm-0</em> should report a cumulative speed of roughly 2gbps.

<strong>Finally</strong>
fdisk, lvm activities and add to fstab and we 're good to go.

<strong>UPDATE</strong>

<ul>
	<li><span style="font-size: 16px;">Fsck throws a few errors during boot when it tries to check the paths of a multipath device. Could this be a problem?</span></li>
	<li>_netdev has to be specified last in the options in /etc/fstab? While <code>defaults,auto,errors=panic,_netdev</code> works and a multipath device is added after network initialisation, it fails when _netdev isn't last!? Is this correct or irrelevant? <em>Answer: it works erratically, sometimes the multipath device is mounted, sometimes not. Seems it's a mountall bug.</em></li>
</ul>
<strong>References</strong>
<ul>
	<li><a href="http://www.captainstorage.info/?p=20">http://www.captainstorage.info/?p=20</a></li>
	<li><a href="http://www.open-iscsi.org/docs/README">http://www.open-iscsi.org/docs/README</a></li>
	<li><a href="http://www.howtoforge.com/using-iscsi-on-ubuntu-10.04-initiator-and-target">http://www.howtoforge.com/using-iscsi-on-ubuntu-10.04-initiator-and-target</a></li>
	<li><a href="http://manpages.ubuntu.com/manpages/precise/man5/multipath.conf.5.html">http://manpages.ubuntu.com/manpages/precise/man5/multipath.conf.5.html</a></li>
	<li><a href="https://help.ubuntu.com/12.04/serverguide/multipath-dm-multipath-config-file.html">https://help.ubuntu.com/12.04/serverguide/multipath-dm-multipath-config-file.html</a></li>
	<li><a href="https://access.redhat.com/knowledge/docs/en-US/Red_Hat_Enterprise_Linux/6/html-single/DM_Multipath/index.html">https://access.redhat.com/knowledge/docs/en-US/Red_Hat_Enterprise_Linux/6/html-single/DM_Multipath/index.html</a></li>
</ul>
