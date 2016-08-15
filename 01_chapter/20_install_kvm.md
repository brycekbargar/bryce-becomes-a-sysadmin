# Install KVM #

[Yay for great docs!][kvm-faq]

Oh...

Those are actually just a FAQ and don't actually contain setup instructions...

Good thing there's [a whitepaper][whitepaper] with "Easy" in the name. This totally doesn't look impenatrable and enterprisey at all.

Ok. It's actually pretty straightforward so far.

#### 2.3 Required Packages ####

> "Verify the following kernel modules are loaded, and if not load manually: 

>kvm
  
>kvm_intel (only on Intel-based systems)"

I did this by [using `lsmod`][lsmod].

```
bryce @ kvm
%> lsmod | grep 'kvm'
kvm_intel 162153 0
kvm 525409 1 kvm_intel
```

Thankfully they are loaded, since I have no idea how to load them manually...

Ok. [In order to manually load the modules][arch-modprobe] run `modprobe kvm_intel && modprobe kvm`. Or something like that? Basically seems like magic...

Oh. Running `find /lib/modules/$(uname -r) -name '*kvm*'` shows they are indeed in the modules path and can just be magically `modprobe`'d. 

#### 2.4 OS Installation Source ####

In order to put an iso on the server I [used scp][scp].

```
bryce @ Bryces-MacBook-Pro
%> scp ~/Downloads/CentOS-7-x86_64-DVD-1511.iso bryce@192.168.0.109:/home/bryce/isos/

```

In hindsight, my partitioning scheme definitely should have accounted for ISO storage... Learning!

It took about 6 minutes for a Minimal CentOS iso to make it 5 feet across the room and onto my "server" over Ethernet. Is that slow? It feels slow...

#### 2.5 Disk Space ####

When I [setup the partitioning][partitioning] I put `/var` on it's own 2tb drive so disk space hopefully won't be an issue. It's definitely something I want to revisit, especially if there are performance issue having multiple VMs running off of a single HDD.

#### 2.6 Networking ####
/etc/sysconfig/network-scripts/ifcfg-em1) 

Trying to `echo 'BRIDGE=br0' >> /etc/sysconfig/network-scripts/ifcfg-em1` or edit the file directly fails on permissions (which makes sense...).

I ended up using `sudoedit /etc/sysconfig/network-scripts/ifcfg-em1` with my `$EDITOR` environment variable set to `nvim`. Using `sudoedit` instead of `sudo nvim` means that it will use my configurations for nvim rather than the configurations for root (yay pretty colors!).

Basically every command in this section needs `sudoedit` or `sudo`.

The command `sysctl -p /etc/sysctl.conf` can be shortened to `sysctl -p` because `man sysctl` states

> "-p[FILE], --load[FILE]

> Load in sysctl settings from the file specified or /etc/sysctl.conf if none given...

#### 3 Creating VMs ####

I figured I would screw up this command a couple times so I created a "script" to execute it instead. I made (_what I thought were_) sensible modifications to the install arguments. **Note:** you can't use the `--extra-args` flag with cdrom apparently (which is ok by me because I have no idea what a tty is...)

```
bryce @ kvm 
%> nvim create-dhcp-box

=== create-dhcp-box ===
#!/bin/sh
sudo virt-install \
 --network bridge:br0 \
 --name vm1 \
 --ram=1024 \
 --vcpus=1 \
 --disk size=10 \
 --graphics none \
 --cdrom ~/isos/CentOS-7-x86_64-DVD-1511.iso

===

bryce @ kvm 
%> chmod +x create-dhcp-box

bryce @ kvm 
%> ./create-dhcp-box
WARNING CDROM media does not print to the text console by default, so you likely will not see text install output. You might want to use --location. See the man page for examples of using --location with CDROM media

Starting install...Allocating 'dhcp-6.qcow2' | 10 GB 00:00:00ERROR internal error: process exited while connecting to monitor: 2016-08-14T19:21:24.717391Z qemu-kvm: -drive file=/home/bryce/isos/CentOS-7-x86_64-DVD-1511.iso,if=none,id=drive-ide0-0-0,readonly=on,format=raw: could not open disk image /home/bryce/isos/CentOS-7-x86_64-DVD-1511.iso: Could not open '/home/bryce/isos/CentOS-7-x86_64-DVD-1511.iso': Permission denied

Domain installation does not appear to have been successful.If it was, you can restart your domain by running: virsh --connect qemu:///system start dhcpotherwise, please restart your installation.

```

Awesome.

It looks like a permissions issue? (totally ignoring the cdrom/gui issue for now...) I found a [pretty helpful doc][libvirt-list] and running
```
bryce @ kvm 
%> virsh uri
qemu:///session

```

plus [some follow-up reading][libvirt-qemu] led me to

> "All 'system' URIs (be it qemu, lxc, uml, ...) connect to the libvirtd daemon running as root which is launched at system startup. Virtual machines created and run using 'system' are usually launched as root, unless configured otherwise (for example in /etc/libvirt/qemu.conf).

> All 'session' URIs launch a libvirtd instance as your local user, and all VMs are run with local user permissions.

> You will definitely want to use qemu:///system if your VMs are acting as servers. VM autostart on host boot only works for 'system', and the root libvirtd instance has necessary permissions to use proper networkings via bridges or virtual networks. qemu:///system is generally what tools like virt-manager default to."


Yay! Progress!

While I was looking for some answers I found that I had _accidentally_ created 100gb(?) worth of broken images that I needed to clear out.

```
bryce @ kvm 
%> sudo ls -lA /var/lib/libvirt/images
total 22512
-rw-r--r--. 1 root root 10739318784 Aug 14 12:58 dhcp-1.qcow2
-rw-r--r--. 1 root root 10739318784 Aug 14 13:12 dhcp-2.qcow2
-rw-r--r--. 1 root root 10739318784 Aug 14 13:23 dhcp-3.qcow2
-rw-------. 1 root root 10739318784 Aug 14 13:47 dhcp-4.qcow2
-rw-r--r--. 1 root root 10739318784 Aug 14 14:15 dhcp-5.qcow2
-rw-r--r--. 1 root root 10739318784 Aug 14 14:21 dhcp-6.qcow2
-rw-r--r--. 1 root root 10739318784 Aug 14 14:43 dhcp-7.qcow2
-rw-------. 1 root root 10739318784 Aug 14 14:44 dhcp-8.qcow2
-rw-r--r--. 1 root root 10739318784 Aug 14 14:45 dhcp-9.qcow2
-rw-r--r--. 1 root root 10739318784 Aug 14 12:56 dhcp.qcow2

...

bryce @ kvm 
%> sudo rm -f /var/lib/libvirt/images/dhcp-5.qcow2

```

I tried globbing `/var/lib/libvirt/images/dhcp-*.qcow2` and various other combinations but I think(?) there's a file permissions issue with zsh expansion? Does that even makes sense? I will have to check into it later when I'm on less of a "roll".

I added `--connect quemu:///system \` to my creation command and

```
bryce @ kvm 
%> ./create-dhcp-box
ERROR no connection driver available for quemu:///system
```

Yay? Progress?

[libvirt-qemu]: http://wiki.libvirt.org/page/FAQ#What_is_the_difference_between_qemu:.2F.2F.2Fsystem_and_qemu:.2F.2F.2Fsession.3F_Which_one_should_I_use.3F
[libvirt-list]: http://wiki.libvirt.org/page/FAQ#My_VM_doesn.27t_show_up_with_.27virsh_list.27
[partitioning]: ./00_install_centos_host.md
[scp]: http://unix.stackexchange.com/a/106482
[arch-modprobe]: https://wiki.archlinux.org/index.php/kernel_modules
[lsmod]: http://www.cyberciti.biz/faq/linux-show-the-status-of-modules-driver/
[whitepaper]: http://linux.dell.com/files/whitepapers/KVM_Virtualization_in_RHEL_7_Made_Easy.pdf
[kvm-faq]: http://www.linux-kvm.org/page/FAQ