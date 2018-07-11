---
author: Vassilis Vatikiotis
title: Create a VM Image in OpenNebula
tags:
  - linux
  - virtualisation
  - OpenNebula
---

# Create a VM Image in OpenNebula

Tested on OpenNebula 4.4.1 on CentOS 6.5. It should work on recent OpenNebula versions.

This is almost copied from <a href="http://hpc.utp.edu.my/mediawiki/index.php/Tutorial:_Deploy_VM_Using_Image_Created_On_OpenNebula_Directly">Tutorial: Deploy VM Using Image Created On OpenNebula Directly</a>, for preservation and my own personal use. Check the original link first.

<h3>Step 1. Uploading the Installation CD Image</h3>

<p>a. Go to <code>Virtual Resources</code> -> <code>Images</code> -> <code>Create</code>
b. Set <code>Type</code> to CDROM.
c. Leave <code>Persistent</code> unchecked.
d. Use either <code>Provide a path</code> and set it to a URL pointing to an OS installation image or <code>Upload</code> and set it to an already downloaded OS installation image.</p>

We now have an installation cdrom image in our datastore

<h3>Step 2. Creating an empty hard disk (Datablock)</h3>

<p>a. Goto <code>Virtual Resources</code> -> <code>Images</code> -> <code>Create</code>
b. Set <code>Type</code> to Datablock
c. Tick <code>Persistent</code>. Every change made to this datablock needs to persist since this is going to be the hard drive for each newly deployed VM.
d. e. Set <code>Image Location</code> to Empty datablock, set <code>Size</code> and (optionally) <code>FS type</code> and `FS Driver` to qcow2.
e. Set <code>Driver prefix</code> to sd.
f. <em>Last time I wasn't setting `FS Driver` to qcow2 and ONE would barf</em></p>

This will create an empty datablock which will be used to install the OS.

<h3>Step 3. Creating an OS image installation template</h3>

<p>a. Goto <code>Virtual Resources</code> -> <code>Templates</code> -> <code>Create</code>
b. We are going to use the installation CD (Step 1) and the Datablock (Step 2). So, in <code>Storage</code> attach the installation CD and set <code>READONLY</code> to yes and attach the datablock as well.
c. Set the Network (select a nic)
d. Under <code>OS Booting</code>: In <code>Boot</code> set <code>Boot</code> to CDROM and in <code>Features</code> set <code>ACPI</code> to yes.
e. Under <code>Input/Output</code> set <code>VNC</code> and <code>Listen IP</code> to 0.0.0.0.
f. Under <code>Context</code> tick <code>Add Network contextualisation</code> and <code>Add OnGate token</code>.</p>

<h3>Step 4. Deploy a VM based on the OS image installation template (Step 3)</h3>

<p>a. Instantiate the template. This will create a VM, which will boot from the CDROM (Step 1) and install the OS in the datablock disk (Step2).
b. Shutdown the VM when installation is complete and delete the VM. Remember, our datablock is persistent so the OS is installed and all changes are persistent.
c. Goto <code>Virtual Resources</code> -> <code>Images</code> and change its <code>Type</code> from Datablock to OS.</p>

<h3>Step 5. Create a VM preparation template.</h3>

<p>a. Create a normal template. Under <code>Storage</code> attach only the disk we created on Step 4. The VM boots from this disk.
b. NB: don't do this is you want to have noVNC working!! Under `Other` pass the following in the `RAW` data section and set the `Type` to KVM. in order to enable serial console access, from KVM side. This requires to enable it from the VM's kernel side as well by tweaking the grub config file that specifies kernel options during boot.
`<devices><serial type="pty"><source path="/dev/pts/5"/><target port="0"/></serial><console type="pty" tty="/dev/pts/5"><source path="/dev/pts/5"/><target port="0"/></console></devices>`
c. All other settings just like Step 3.</p>

Instantiate the template and start a VM.

<h3>Step 6. Contextualisation</h3>

In our newly created VM:

1.  Go to <a href="http://docs.opennebula.org/stable/user/virtual_machine_setup/bcont.html">Basic Contextualisation</a>. The process is not straight forward so here's a rough guide.

- Open VM's VNC console.
- Setup the network for the VM.
- Setup ONE repository and install the contextualisation deb package. UPDATE: Download it manually at [http://dev.opennebula.org/attachments/download/750/one-context_4.4.0.deb](http://dev.opennebula.org/attachments/download/750/one-context_4.4.0.deb) since I can't find it in the ONE repo.

2.  NB: don't do this is you want to have noVNC working!! To enable serial console access from the VM side, open `/etc/default/grub` and set the following:

- `GRUB_TERMINAL=serial`
- `GRUB_CMDLINE_LINUX="console=tty0 console=ttyS0,115200n8"`
- `GRUB_SERIAL_COMMAND="serial --speed=115200 --unit=0 --word=8 --parity=no --stop=1"`
- Create `/etc/init/ttyS0.conf` with the following contents:

  ```sh
  # ttyS0 - getty
  #
  # This service maintains a getty on ttyS0 from the point the system is
  # started until it is shut down again.
  start on stopped rc or RUNLEVEL=[12345]
  stop on runlevel [!12345]
  respawn
  exec /sbin/getty -L -w 115200 ttyS0 vt102
  ```

- Full instructions at [https://help.ubuntu.com/community/SerialConsoleHowto](https://help.ubuntu.com/community/SerialConsoleHowto)

3.  Prepare the VM, i.e. clean and make it pristine, following <a href="http://www.whotouchedmygun.com/2014/03/17/preparing-linux-template-vms/">this guide</a>.
4.  Shutdown the VM and delete it.

Finally go to the OS image and set its <code>Type</code> to non persistent.

We can now use the OS image to instantiate new VMs.
