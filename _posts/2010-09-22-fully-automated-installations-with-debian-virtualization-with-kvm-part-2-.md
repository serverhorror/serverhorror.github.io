---
layout: post
title: Fully Automated Installations with Debian - (Virtualization with KVM - Part
  2)
categories: []
tags: []
status: pending
type: post
published: false
meta: {}
author: 
---
<p>Continued from the <a href="http://blog.serverhorror.com/2010/05/20/virtualization-with-kvm-part-1/">last time</a>, let's see where we are:</p>
<ul>
<li><del datetime="2010-05-20T11:11:42+00:00"><a href="http://blog.serverhorror.com/2010/05/20/virtualization-with-kvm-part-1/">Get Started! Do the basic setup, verify we have KVM working</a></del></li>
<li><strong>Automated Guest Installations from the direct physical host</strong></li>
<li>Offer IPv4 connectivity over public IPs to  guest machines</li>
<li>Offer IPv6 and IPv4 connectivity to guest machines</li>
<li>Offer IPv6 as the only connectivity to guest machines</li>
<li>Automagically create new guests with a single command</li>
</ul>
<hr />This really is just a quick copy and paste write up how to get a <a href="http://www.informatik.uni-koeln.de/fai/">Fully Automatic Installation (FAI)</a> Server up and running. It does not deal with further customizations.</p>
<p>First things first. In our <a href="http://blog.serverhorror.com/2010/05/20/virtualization-with-kvm-part-1/">last post I showed a very basic setup for a bridge</a> to get started with. Now is the time to use it.</p>
<p>Configure your DHCP Server (ISC DHCP3 in this case)</p>
<pre>ddns-update-style none;
authoritative;
log-facility local7;
subnet 172.16.0.0 netmask 255.255.0.0 {
    range 172.16.1.10 172.16.1.254;
    option routers 172.16.0.1;
    next-server 172.16.0.1;
    option domain-name "serverhorror.com";
    option domain-name-servers 8.8.8.8;
    option broadcast-address 172.16.255.255;
    default-lease-time 60;
    max-lease-time 300;
    ### FAI START ###
    filename "/fai/pxelinux.0"; # needed for FAI to find it"s boot environment

    ### FAI Hosts ###
    host demohost {
      # find that by either tcpdumping, using 'info network' in the qemu monitor
      # or just trust me that it is the default MAC-address kvm uses
      hardware ethernet 52:54:00:12:34:56;
      fixed-address 172.16.2.1
    }

    ### FAI END ###
}</pre>
<p>Enable Network Address Translation (NAT)</p>
<pre>sudo iptables -t nat -A POSTROUTING -o br0 -j MASQUERADE</pre>
<pre>sysctl -w net.ipv4.conf.br0.forwarding=1</pre>
<p>To make that stuff permanent add it as a <tt>pre-up</tt> to your <tt>/etc/network/interfaces</tt> file under the appropriate stanza
Get the Fully Automatic Installer (FAI):</p>
<pre>sudo apt-get install fai-quickstart</pre>
<p>This will install quite some stuff, it's OK. You'll appreciate all of it once you dive into FAI. We need to make sure a few things got set up properly. The defaults are pretty much OK to play around with. Check the following locations:</p>
<ul>
<li><tt>/etc/fai/fai.conf</tt> -</li>
<li><tt>/etc/fai/make-fai-nfsroot.conf</tt></li>
</ul>
<p>I usually keep with the defaults until I have the need to do more stuff. According to the FHS the /srv/ location is perfectly suited (As I don't take the notes from a fresh server you might want to note that i had <tt>/srv/tftpd</tt> already populated with some stuff. Still <tt>/srv/tftp/fai</tt> looks good to me as it kind of introduces a new namespace and doesn't clash with anything I had in there before.)
<strong>This will take time</strong>:</p>
<pre>$ sudo fai-setup
Warning: The home dir /var/log/fai you specified already exists.
.
.
.
FAI setup finished.
Log file written to /var/log/fai/fai-setup.log</pre>
<p>It populates <tt>/srv/fai/nfs</tt> and <tt>/srv/fai/config</tt> with some basic stuff which you can use to install clients right away (well nearly, we need to finish some steps to actually be able to install...)
Also check after running <tt>sudo fai-setup</tt> that the following is in your <tt>/etc/exports</tt>:</p>
<pre># the FAI NFS root (Surprise!) used by clients during installation
/srv/fai/nfsroot (async,ro,no_subtree_check,no_root_squash)
# Mounted by clients during install so that they know what to do
/srv/fai/config (async,ro,no_subtree_check,no_root_squash)</pre>
<p>OK - Installation time. Just don't do it now, read the next 3 pieces of code before continueing:</p>
<pre>sudo /usr/bin/kvm -k de \
    -name test \
    -boot order=cn,once=n \
    -drive file=test.img,if=virtio,format=raw,media=disk,boot=on \
    -usbdevice tablet \
    -watchdog i6300esb \
    -m 1024 -vnc :1 -balloon virtio \
    -pidfile /var/run/kvm.test \
    -net nic,vlan=0 -net tap,vlan=0,name=eth0,ifname=tap0,script=no,downscript=no \
    -monitor stdio</pre>
<p>To follow everything that happens immediately stop the machine:</p>
<pre>QEMU 0.12.4 monitor - type 'help' for more information
(qemu) stop
(qemu)</pre>
<p>Fire up your favorite VNC client software and connect to your host on display one (port 5901) - <strong>there's no authentication whatsoever. Everything is plaintext - you've been warned!</strong></p>
<pre>$ sudo netstat -tulpen|grep kvm
tcp        0      0 0.0.0.0:5901            0.0.0.0:*               LISTEN      0          3560102     28553/kvm</pre>
<p>Should look something like this:
<a href="http://blog.serverhorror.com/wp-content/uploads/2010/06/KVM_FAI_Startup.png"><img class="alignnone size-full wp-image-831" title="KVM FAI Startup" src="http://blog.serverhorror.com/wp-content/uploads/2010/06/KVM_FAI_Startup.png" alt="KVM FAI Startup" width="760" height="462" /></a></p>
<hr />OK - Installation time. For real this time:</p>
<pre>(qemu) c</pre>
<p>And watch it installing. The installation will prompt you in this default showcase to hit return once it's finished. If you set up kvm properly it should then just reboot into the real operating system.</p>
<p>So where are we now?</p>
<ul>
<li><del datetime="2010-05-20T11:11:42+00:00"><a href="http://blog.serverhorror.com/2010/05/20/virtualization-with-kvm-part-1/">Get Started! Do the basic setup, verify we have KVM working</a></del></li>
<li><del datetime="2010-06-06T19:41:49+00:00"><strong>Automated Guest Installations from the direct physical host</strong></del></li>
<li>Offer IPv4 connectivity over public IPs to  guest machines</li>
<li>Offer IPv6 and IPv4 connectivity to guest machines</li>
<li>Offer IPv6 as the only connectivity to guest machines</li>
<li>Automagically create new guests with a single command</li>
</ul>
