--- 
layout: post
title: Failover hosts using Xen, DRBB and Heartbeat
created: 1239221978
---
After quite a lot of reading and a morning playing I managed to get failover Xen hosts working.

The idea was to have 2 physical servers to run 2 (or more) Xen hosts between them. If one server was to die or needed some work doing
on it then the domU would automatically move to the other node.

<div style="text-align:center">
<img src="/images/failover.png" alt="Failover Diagram" width="499" height="169" />
</div>

I've done some testing and all appears to work fine. However let me stress that this is not live migration so you would suffer about a minute or so outage
(not really a big deal in the grand scheme of things).

Click the "Read More" button for full details on the setup.
<!--break-->
<br />

<h3>Requirements</h3>
<ul>
  <li>Debian Etch</li>
  <li>Xen Installed</li>
</ul>

<br />

<h3>Installing DRBD</h3>
<em>To be done on both Xen nodes</em>

DRBD is only at version 7 in Debian Etch, so we pull in version 8 from <a href="http://www.backports.org">backports.org</a>

Version 8 comes with some helper scripts for Xen which make the failover a lot easier to configure.

These are the packages we need:
<ul>
  <li><a href="http://packages.debian.org/etch-backports/drbd8-utils">drbd8-utils</a></li>
  <li><a href="http://packages.debian.org/etch-backports/drbd8-source">drbd8-source</a></li>
</ul>

If you are unsure on how to install stuff from backports then please read the documentation located <a href="http://backports.org/dokuwiki/doku.php?id=instructions">here</a>

After installing the packages we need to build the DRBD kernel module. The easiest way to achieve this is to do:
<pre>module-assistant auto-install drbd8</pre>

Then load the module by doing
<pre>modprobe drbd</pre>

<h3>Installing Heartbeat</h3>
<em>To be done on both Xen nodes</em>

This can be installed straight from the Debian repos:
<pre>aptitude install heartbeat-2</pre>

Next there is some basic configuration to do. First we need to create <em>/etc/heartbeat/ha.cf</em>
with the following contents:
<pre>
  # Enable new cluster manager
  crm on
  # Specify interface to send bcast packets out on
  bcast eth0
  # Specify nodes in cluster, these must correspond with "uname -n"
  nodes xen-1 xen-2
</pre>

Then we create <em>/etc/heartbeat/authkeys</em> with the following contents:
<pre>
  auth 1
  1 sha1 SomeLongStringWithRandomChars
</pre>
Note that <em>SomeLongStringWithRandomChars</em> should be randomly generated and the same on both nodes.


<h3>Setting up DRBD resources</h3>
<em>To be done on both Xen nodes</em>

I generally use LVM for my Xen hosts, so first off I create all the paritions required for my 2 hosts:
<pre>
  lvcreate -L10G -n host1-root xenhostfs
  lvcreate -L1G  -n host1-swap xenhostfs
  lvcreate -L10G -n host2-root xenhostfs
  lvcreate -L1G  -n host2-swap xenhostfs
</pre>


Next we need to create <em>/etc/drbd.conf</em> and configure our resources (DRBD devices):

<pre>
#
# Global Parameters
#
global {
        # Participate in http://usage.drbd.org
        usage-count yes;
}

#
# Settings common to all resources
#
common {
        # Set sync rate
        syncer { rate 10M; }

        # Protocol C : Both nodes have to commit before write
        # is considered successful
        protocol C;
        net {
                # Xen tests that it can write to block device
                # before starting up. Not allowing this causes
                # migration to fail.
                allow-two-primaries;
                
                # Split-brain recovery parameters
                after-sb-0pri discard-zero-changes;
                after-sb-1pri discard-secondary;
        }
}

#
# Resource Definitions
#
resource "host1_root" {

        on xen1 {
                # The block device it will appear as
                device    /dev/drbd0;

                # The device we are mirroring
                disk      /dev/xenhostfs/host1-root;

                # Store DRBD meta data the above disk
                meta-disk internal;

                # Address of *this* host and port to replicate over
                # You must use a different port for each resource
                address   10.0.0.1:7790;
        }

        on xen2 {
                device    /dev/drbd0;
                disk      /dev/xenhostfs/host1-root;
                meta-disk internal;
                address   10.0.0.2:7790;
        }

}

resource "host1_swap" {

        on xen1 {
                device    /dev/drbd1;
                disk      /dev/xenhostfs/host1-swap;
                meta-disk internal;
                address   10.0.0.1:7791;
        }

        on xen2 {
                device    /dev/drbd1;
                disk      /dev/xenhostfs/host1-swap;
                meta-disk internal;
                address   10.0.0.2:7791;
        }
}

resource "host2_root" {

        on xen1 {
                device    /dev/drbd2;
                disk      /dev/xenhostfs/host2-root;
                meta-disk internal;
                address   10.0.0.1:7792;
        }

        on xen2 {
                device    /dev/drbd2;
                disk      /dev/xenhostfs/host2-root;
                meta-disk internal;
                address   10.0.0.2:7792;
        }
}

resource "host2_swap" {

        on xen1 {
                device    /dev/drbd3;
                disk      /dev/xenhostfs/host2-swap;
                meta-disk internal;
                address   10.0.0.1:7793;
        }

        on xen2 {
                device    /dev/drbd3;
                disk      /dev/xenhostfs/host2-swap;
                meta-disk internal;
                address   10.0.0.2:7793;
        }
}
</pre>

Now we have to initialise the metadata on the DRBD resources. This takes some time
so I advise a coffee or lunch at this point ;)
<pre>
 drbdadm create-md host1_root
 drbdadm create-md host1_swap
 drbdadm create-md host2_root
 drbdadm create-md host2_swap
</pre>

Now that we have done that we can start up drbd:
<pre>
/etc/init.d/drbd restart
</pre>

Right now all that is set up we now need to select one node as the primary, (say xen1).

<em>On xen1 only</em> we now do the following which does 2 things:
<ol>
<li>Sets xen1 as the primary for the DRBD devices</li>
<li>Syncs all the data from xen1 to xen2</li>
</ol>
<pre>
drbdadm -- --overwrite-data-of-peer primary host1_root
drbdadm -- --overwrite-data-of-peer primary host1_swap
drbdadm -- --overwrite-data-of-peer primary host2_root
drbdadm -- --overwrite-data-of-peer primary host2_swap
</pre>

You can check the process of his by doing
<pre>
cat /proc/drbd
</pre>

which will give you output something like
<pre>
version: 8.0.14 (api:86/proto:86)
GIT-hash: bb447522fc9a87d0069b7e14f0234911ebdab0f7 build by phil@fat-tyre, 2008-11-12 16:40:33
 0: cs:Connected st:Primary/Secondary ds:UpToDate/UpToDate C r---
    ns:52120 nr:1016 dw:503652 dr:203481 al:121 bm:872 lo:0 pe:0 ua:0 ap:0
        [>...................] sync'ed:  1.0% (10380903/10485760)K
        finish: 0:07:43 speed: 10,836 (10,836) K/sec
        resync: used:0/61 hits:2296 misses:10 starving:0 dirty:0 changed:10
        act_log: used:0/127 hits:120616 misses:128 starving:0 dirty:7 changed:121
 1: cs:Connected st:Primary/Secondary ds:UpToDate/UpToDate C r---
    ns:0 nr:0 dw:4 dr:1348 al:1 bm:12 lo:0 pe:0 ua:0 ap:0
...
</pre>

Note that you do not have to wait for this to finish, you can carry on using your /dev/drbdN devices
but preformance will be reduced until it has completed the sync.


<h3>Creating Xen DomU</h3>
I'm not going to go through the complete details of this as its a reasonable assumption that if you're reading
this then you have used Xen before, or at least are capable of reading around.

<em>This is done only on the primary host</em>

However here are the edited highlights. Firstly create the filesystems:
<pre>
mkfs.ext3 /dev/drbd0
mkswap    /dev/drbd1
mkfs.ext3 /dev/drbd2
mkswap    /dev/drbd3
</pre>

Now mount and use debootstrap to create the base installs:
<pre>
mkdir /mnt/host1 /mnt/host2
mount /dev/drbd0 /mnt/host1
mount /dev/drbd2 /mnt/host2
debootstrap --arch i386 etch /mnt/host1 http://ftp.uk.debian.org/debian/
debootstrap --arch i386 etch /mnt/host2 http://ftp.uk.debian.org/debian/
</pre>

Create <em>/mnt/host1/etc/fstab</em> and <em>/mnt/host2/etc/fstab</em> with the following content:
<pre>
proc          /proc        proc    defaults                     0   0
/dev/xvda     /            ext3    defaults,errors=remount-ro   0   1
/dev/xvdb     none         swap    sw                           0   0
</pre>

Then unmount <em>/mnt/host1</em> &amp; <em>/mnt/host2</em> and we are ready to configure Xen.

<h3>Xen Domain Config</h3>
<em>To be done on both Xen nodes</em>

Create <em>/etc/xen/host1.dom</em> with the following contents:
<pre>
#  -*- mode: python; -*-
#----------------------------------------------------------------------------
# Kernel image file.
kernel = "/boot/vmlinuz-2.6.18-6-xen-686"
ramdisk = "/boot/initrd.img-2.6.18-6-xen-686"

# Initial memory allocation (in megabytes) for the new domain.
memory = 1024

# A name for your domain. All domains must have different names.
name = "host1"

# Filesystems
disk = [ 'drbd:host1_root,xvda,w', 'drbd:host1_swap,xvdb,w']

# Network
vif = ['bridge=xenbr0']

# Set root device.
root = "/dev/xvda ro"

# Sets runlevel 4.
extra = "4"
</pre>
and <em>/etc/xen/host2.dom</em> 
<pre>
#  -*- mode: python; -*-
#----------------------------------------------------------------------------
# Kernel image file.
kernel = "/boot/vmlinuz-2.6.18-6-xen-686"
ramdisk = "/boot/initrd.img-2.6.18-6-xen-686"

# Initial memory allocation (in megabytes) for the new domain.
memory = 1024

# A name for your domain. All domains must have different names.
name = "host2"

# Filesystems
disk = [ 'drbd:host2_root,xvda,w', 'drbd:host2_swap,xvdb,w']

# Network
vif = ['bridge=xenbr0']

# Set root device.
root = "/dev/xvda ro"

# Sets runlevel 4.
extra = "4"
</pre>

Now on ONE of the nodes ONLY you can start the xen instances by doing:
<pre>
xm create /etc/xen/host1.dom
xm create /etc/xen/host2.dom
</pre>

<h3>Configuring Heartbeat</h3>
We need to create 3 XML files:
<ul>
  <li><strong>bootstrap.xml</strong>: This sets up the defaults for heartbeat Cluster Resource Manager (CRM)</li>
  <li><strong>host1.xml</strong>: Heartbeat config for host1</li>
  <li><strong>host2.xml</strong>: Heartbeat config for host2</li>
</ul>

Now the documentation for these XML files is somewhat thin on the ground. The only real thing to go on is the DTD
which, on Debian, lives here:<em>/usr/lib/heartbeat/crm.dtd</em>

You can also refer to the online version <a href="http://hg.clusterlabs.org/pacemaker/dev/file/tip/xml/crm-1.0.dtd">here</a> which
is always the latest version.


Anyway here are the contents of the 3 files (just save them to your home directory for now).

<em>bootstrap.xml</em>
<em>To be done on one Xen node only</em>
<pre>
&lt;cluster_property_set id="bootstrap"&gt;
   &lt;attributes&gt;
      &lt;nvpair id="bootstrap01" name="transition_idle_timeout" value="60"/&gt;
      &lt;nvpair id="bootstrap02" name="default_resource_stickiness" value="0"/&gt;
      &lt;nvpair id="bootstrap03" name="default_resource_failure_stickiness" 
       value="-500"/&gt;
      &lt;nvpair id="bootstrap04" name="stonith_enabled" value="false"/&gt;
      &lt;nvpair id="bootstrap05" name="stonith_action" value="reboot"/&gt;
      &lt;nvpair id="bootstrap06" name="symmetric_cluster" value="true"/&gt;
      &lt;nvpair id="bootstrap07" name="no_quorum_policy" value="stop"/&gt;
      &lt;nvpair id="bootstrap08" name="stop_orphan_resources" value="true"/&gt;
      &lt;nvpair id="bootstrap09" name="stop_orphan_actions" value="true"/&gt;
      &lt;nvpair id="bootstrap10" name="is_managed_default" value="true"/&gt;
   &lt;/attributes&gt;
&lt;/cluster_property_set&gt;
</pre>

<em>host1.xml</em>
<pre>
&lt;resources&gt;
 &lt;primitive id="host1" class="ocf" type="Xen" provider="heartbeat"&gt;

  &lt;operations&gt;
   &lt;op id="host1-op01" name="monitor" interval="10s" timeout="60s" prereq="nothing"/&gt;
   &lt;op id="host1-op02" name="start" timeout="60s" start_delay="0"/&gt;
   &lt;op id="host1-op03" name="stop" timeout="300s"/&gt;
 &lt;/operations&gt;

 &lt;instance_attributes id="host1"&gt;
  &lt;attributes&gt;
   &lt;nvpair id="host1-attr01" name="xmfile" value="/etc/xen/host1.dom"/&gt;
   &lt;nvpair id="host1-attr02" name="target_role" value="started"/&gt;
  &lt;/attributes&gt;
 &lt;/instance_attributes&gt;

 &lt;meta_attributes id="host1-meta01"&gt;
  &lt;attributes&gt;
   &lt;nvpair id="host1-meta-attr01" name="allow_migrate" value="true"/&gt;
  &lt;/attributes&gt;
 &lt;/meta_attributes&gt;

 &lt;/primitive&gt;
&lt;/resources&gt;
</pre>

<em>host2.xml</em>
<pre>
&lt;resources&gt;
 &lt;primitive id="host2" class="ocf" type="Xen" provider="heartbeat"&gt;

  &lt;operations&gt;
   &lt;op id="host2-op01" name="monitor" interval="10s" timeout="60s" prereq="nothing"/&gt;
   &lt;op id="host2-op02" name="start" timeout="60s" start_delay="0"/&gt;
   &lt;op id="host2-op03" name="stop" timeout="300s"/&gt;
 &lt;/operations&gt;

 &lt;instance_attributes id="host2"&gt;
  &lt;attributes&gt;
   &lt;nvpair id="host2-attr01" name="xmfile" value="/etc/xen/host2.dom"/&gt;
   &lt;nvpair id="host2-attr02" name="target_role" value="started"/&gt;
  &lt;/attributes&gt;
 &lt;/instance_attributes&gt;

 &lt;meta_attributes id="host2-meta01"&gt;
  &lt;attributes&gt;
   &lt;nvpair id="host2-meta-attr01" name="allow_migrate" value="true"/&gt;
  &lt;/attributes&gt;
 &lt;/meta_attributes&gt;

 &lt;/primitive&gt;
&lt;/resources&gt;
</pre>

Then we need to load them:
<pre>
cibadmin -C -o crm_config -x bootstrap.xml
cibadmin -C -o resources -x host1.xml
cibadmin -C -o resources -x host2.xml
</pre>

<h3>Monitoring</h3>
Basic monitoing is done with <em>crm_mon</em>. Issuing this command will result in the following
which is updated every 15 seconds:
<pre>
============
Last updated: Wed Apr  8 21:08:56 2009
Current DC: xen2 (1b3dbdb3-f9ca-4d16-bf7d-8a57167b85ed)
2 Nodes configured.
2 Resources configured.
============

Node: xen2 (1b3dbdb3-f9ca-4d16-bf7d-8a57167b85ed): online
Node: xen1 (ea043f8c-5afc-4aee-9ffc-d17cb3cfed06): online

host1     (heartbeat::ocf:Xen):   Started xen1
host2     (heartbeat::ocf:Xen):   Started xen1
</pre>


<h3>Basic Operations</h3>
<strong>Migrating</strong>
To move host2 from xen1 to xen2 you do:
<pre>
crm_resource --migrate --resource host2 --host-uname xen2
</pre>

<strong>Stopping/Starting</strong>
To stop host2:
<pre>
crm_resource --resource host2 --set-parameter target_role \
             --property-value stopped
</pre>
...and to start it again:
<pre>
crm_resource --resource host2 --set-parameter target_role \
             --property-value started
</pre>

<h3>Credits</h3>
Thanks to <a href="http://www.netdotnet.net/">Doug</a> for pointing out all my lame mistakes.
