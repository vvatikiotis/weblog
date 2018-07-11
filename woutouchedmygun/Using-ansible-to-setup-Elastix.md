---
author: Vassilis Vatikiotis
title: Using ansible to setup Elastix
tags:
  - linux
---

# Using ansible to setup Elastix

#### MacOS 10.9 Mavericks and Homebrew installation

- Mavericks comes with Python 2.7.5
- No need to install python using brew
- `brew install ansible`

#### Create VM

`virt-install -n elastix-2.4 --ram 2048 --vcpus 2 --os-type linux --os-variant=rhel6 --disk path=/var/lib/libvirt/images/elastix-2.4.img,bus=virtio,size=20 -l /var/lib/libvirt/images/Elastix-2.4.0-Stable-x86_64-bin-04feb2013.iso --graphics none --network bridge:br1,model=virtio -v -x "console=ttyS0"`

- No additional packages installed!

#### To activate Elastix

- Turn off SELinux
- `setenforce 0`
- `vim /etc/selinux/config` and make it `permissive`
- `service iptables stop &amp;&amp; chkconfig iptables off`
- Use ssh port 222 instead of 22
- and enable `PubkeyAuthentication`
- and `service sshd restart`
- reboot

#### Pre ansible tasks

- Generate ssh key: `ssh-keygen -t rsa -C"email@domain ansible" and save it as`id_rsa_ansible` - **don't overwrite any existing key pairs**.
- `cat ~/.ssh/id_dsa.pub | ssh you@remote 'cat - &gt;&gt; ~/.ssh/authorized_keys'` to place the ansible publi key in /root/.ssh/authorized_keys in the newVM.
- _Take care of the dir and file permissions of the .ssh directory in the VM_
- `ssh-add ansible-private-key` locally to add the ansible key to the keyring. `ssh-add -l` to list the keys in keyring.
- Create a `hosts` file somewhere containing the following:
  &gt; `elastix ansible_ssh_host= ansible_ssh_port=222 ansible_ssh_user=root`

- _For CentOS &lt; 6_ `ansible elastix -i ./hosts -m raw -a"yum install -y python-simplejson"` to install the json module needed for elastix to work.
  See
- To test it: `ansible all -i ./hosts -m ping`. Success if you get

```
elastix | success >> {
    "changed": false,
    "ping": "pong"
}
```
