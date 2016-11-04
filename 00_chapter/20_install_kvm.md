# Install KVM

[Yay for great docs!](http://www.linux-kvm.org/page/FAQ)

Oh...

Those are actually just a FAQ and don't actually contain setup instructions...

Good thing there's [a whitepaper](http://linux.dell.com/files/whitepapers/KVM_Virtualization_in_RHEL_7_Made_Easy.pdf) with "Easy" in the name. This totally doesn't look impenatrable and enterprisey at all.

Ok. It's actually pretty straightforward so far.

#### 2.3 Required Packages

> "Verify the following kernel modules are loaded, and if not load manually:
> 
> kvm
> 
> kvm\_intel \(only on Intel-based systems\)"

I did this by [using \`lsmod\`](http://www.cyberciti.biz/faq/linux-show-the-status-of-modules-driver/).

```sh
bryce @ kvm
%> lsmod | grep 'kvm'
kvm_intel 162153 0
kvm 525409 1 kvm_intel
```

Thankfully they are loaded, since I have no idea how to load them manually...

Ok. [In order to manually load the modules](https://wiki.archlinux.org/index.php/kernel_modules) run `modprobe kvm_intel && modprobe kvm`. Or something like that? Basically seems like magic...

Oh. Running `find /lib/modules/$(uname -r) -name '*kvm*'` shows they are indeed in the modules path and can just be magically `modprobe`'d.

#### 2.4 OS Installation Source

In order to put an iso on the server I [used scp](http://unix.stackexchange.com/a/106482).

```sh
bryce @ Bryces-MacBook-Pro
%> scp ~/Downloads/CentOS-7-x86_64-DVD-1511.iso bryce@192.168.0.109:/home/bryce/isos/

```

In hindsight, my partitioning scheme definitely should have accounted for ISO storage... Learning!

It took about 6 minutes for a Minimal CentOS iso to make it 5 feet across the room and onto my "server" over Ethernet. Is that slow? It feels slow...

#### 2.5 Disk Space

When I [setup the partitioning](./00_install_centos_host.md) I put `/var` on it's own 2tb drive so disk space hopefully won't be an issue. It's definitely something I want to revisit, especially if there are performance issue having multiple VMs running off of a single HDD.

#### 2.6 Networking

\/etc\/sysconfig\/network-scripts\/ifcfg-em1\)

Trying to `echo 'BRIDGE=br0' >> /etc/sysconfig/network-scripts/ifcfg-em1` or edit the file directly fails on permissions \(which makes sense...\).

I ended up using `sudoedit /etc/sysconfig/network-scripts/ifcfg-em1` with my `$EDITOR` environment variable set to `nvim`. Using `sudoedit` instead of `sudo nvim` means that it will use my configurations for nvim rather than the configurations for root \(yay pretty colors!\).

Basically every command in this section needs `sudoedit` or `sudo`.

The command `sysctl -p /etc/sysctl.conf` can be shortened to `sysctl -p` because `man sysctl` states

> "-p\[FILE\], --load\[FILE\]
> 
> Load in sysctl settings from the file specified or \/etc\/sysctl.conf if none given...

#### 3 Creating VMs

I figured I would screw up this command a couple times so I created a "script" to execute it instead. I made \(_what I thought were_\) sensible modifications to the install arguments. **Note:** you can't use the `--extra-args` flag with cdrom apparently \(which is ok by me because I have no idea what a tty is...\)

```sh
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

Starting install...
Allocating 'dhcp-6.qcow2' | 10 GB 00:00:00
ERROR internal error: process exited while connecting to monitor: 2016-08-14T19:21:24.717391Z qemu-kvm: -drive file=/home/bryce/isos/CentOS-7-x86_64-DVD-1511.iso,if=none,id=drive-ide0-0-0,readonly=on,format=raw: could not open disk image /home/bryce/isos/CentOS-7-x86_64-DVD-1511.iso: Could not open '/home/bryce/isos/CentOS-7-x86_64-DVD-1511.iso': Permission denied

Domain installation does not appear to have been successful.If it was, you can restart your domain by running: virsh --connect qemu:///system start dhcpotherwise, please restart your installation.

```

Awesome.

_An aside: I've now basically abandoned _[_the aforementioned whitepaper_](http://linux.dell.com/files/whitepapers/KVM_Virtualization_in_RHEL_7_Made_Easy.pdf)_ since I think I've gotten all I can out of it and am now striking out on my own into the wild_

### Authentication, libvirt, and you

It looks like a permissions issue? \(totally ignoring the cdrom\/gui issue for now...\) I found a [pretty helpful doc](http://wiki.libvirt.org/page/FAQ#My_VM_doesn.27t_show_up_with_.27virsh_list.27) and running

```sh
bryce @ kvm 
%> virsh uri
qemu:///session

```

plus [some follow-up reading](http://wiki.libvirt.org/page/FAQ#What_is_the_difference_between_qemu:.2F.2F.2Fsystem_and_qemu:.2F.2F.2Fsession.3F_Which_one_should_I_use.3F) led me to

> "All 'system' URIs \(be it qemu, lxc, uml, ...\) connect to the libvirtd daemon running as root which is launched at system startup. Virtual machines created and run using 'system' are usually launched as root, unless configured otherwise \(for example in \/etc\/libvirt\/qemu.conf\).
> 
> All 'session' URIs launch a libvirtd instance as your local user, and all VMs are run with local user permissions.
> 
> You will definitely want to use qemu:\/\/\/system if your VMs are acting as servers. VM autostart on host boot only works for 'system', and the root libvirtd instance has necessary permissions to use proper networkings via bridges or virtual networks. qemu:\/\/\/system is generally what tools like virt-manager default to."

Yay! Progress!

While I was looking for some answers I found that I had _accidentally_ created 100gb\(?\) worth of broken images that I needed to clear out.

```sh
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

I tried globbing `/var/lib/libvirt/images/dhcp-*.qcow2` and various other combinations but I think\(?\) there's a file permissions issue with zsh expansion? Does that even makes sense? I will have to check into it later when I'm on less of a "roll".

I added `--connect qemu:///system \` to my creation command and

```sh
bryce @ kvm
%> ./create-dhcp-box
WARNING CDROM media does not print to the text console by default, so you likely will not see text install output. You might want to use --location. See the man page for examples of using --location with CDROM media

Starting install...
Allocating 'dhcp.qcow2' | 10 GB 00:00:00
ERROR internal error: process exited while connecting to monitor: 2016-08-15T11:41:56.504281Z qemu-kvm: -drive file=/home/bryce/isos/CentOS-7-x86_64-DVD-1511.iso,if=none,id=drive-ide0-0-0,readonly=on,format=raw: could not open disk image /home/bryce/isos/CentOS-7-x86_64-DVD-1511.iso: Could not open '/home/bryce/isos/CentOS-7-x86_64-DVD-1511.iso': Permission denied

Domain installation does not appear to have been successful.If it was, you can restart your domain by running: virsh --connect qemu:///system start dhcpotherwise, please restart your installation.

```

lol progress...

[This question suggests](http://superuser.com/questions/298426/kvm-image-failed-to-start-with-virsh-permission-denied?answertab=active#tab-top) running qemu as root. Or at least I think that's what the implications of the following are.

```sh
=== /etc/libvirt/qemu.conf ===

...

User = "root"
group = "root"

...

===

```

This seems sketchy and bad though? Like, just making everything root feels wrong...

Further reading of that question led to

> "I also had to change the owner of one of my vms. It belonged to root which led to an access denied. sudo chown -R libvirt-qemu:kvm dbos\/ubuntu-kvm\/. You can run ls -l on\/your\/vm\/dir\/and\/its\/subdirs\/ to check permissions at each level. Ensure none of them belong to the root group and user"

I assume that because I'm running `sudo virt-install ...` it's creating the images as root?. Let's remove the sudo and try...

```sh
bryce @ kvm
%> ./create-dhcp-box
ERROR authentication failed: no agent is available to authenticate
```

Hmmm... I guess that makes sense. Why would I be able to access `qemu:///system` as a non-root user?

A quick confirmation.

```sh
bryce @ kvm 
%> virsh --connect qemu:///system
error: failed to connect to the hypervisor

bryce @ kvm 
%> sudo virsh --connect qemu:///system
Welcome to virsh, the virtualization interactive terminal.

Type: 'help' for help with commands
      'quit' to quit

virsh # quit
```

Progress! I think\(?\) I need to work on `libvirt` or `qemu` permissions? I noticed a couple of interesting groups and users

```sh
bryce @ kvm
%> less /etc/group
...
kvm:x:36:qemu
qemu:x:107:
libvirt:x:991:
...

bryce @ kvm 
%> less /etc/passwd
...
qemu:x:107:107:qemu user:/:/sbin/nologin
...

bryce @ kvm
%> ls -lA ~/isos
total 4228096
-rw-rw-rw-. 1 qemu qemu 4329570304 Aug 11 06:25 CentOS-7-x86_64-DVD-1511.iso

```

So. There are some `kvm` specific groups and a `qemu` user which happens to own the iso once I try running `virt-install`. I'm assuming that somehow I need to give `qemu` permission to `qemu:///system` and possibly some of the `kvm` groups as well? All the while _**I**_ don't want permissions to it because I should never need to connect directly. Really it's all speculation at this point...

_An aside: I reeeeeally don't like copy + pasting "solutions" without understanding exactly why they work, especially in my own projects where there isn't a time constraint, double-especially when the "solutions" are reeking with code smell like making an arbitrary thing root._

My suspicions are further confirmed after [finding someone who had success](https://libvirt.org/auth.html) disabling auth for libvirt \(though that is obviously wrong, even for a lab...\).

No idea why [this was hard to find](http://wiki.libvirt.org/page/Failed_to_connect_to_the_hypervisor)...

> You are trying to connect using unix socket. The connection to "qemu" without any hostname specified is by default using unix sockets. If there is no error running this command as root it's probably just misconfigured.
> 
> Solution
> 
> If you want to be able to connect as non-root user using unix sockets, configure following options in '\/etc\/libvirt\/libvirtd.conf' accordingly:
> 
> ```
> unix_sock_group = <group>
> unix_sock_ro_perms = <perms>
> unix_sock_rw_perms = <perms>
> ```
> 
> Yay! A starting point!

Ok. [Looks like the above is for](https://libvirt.org/auth.html) using unix sockets for permissions.

> "If libvirt does not contain support for PolicyKit, then access control for the UNIX domain socket is done using traditional file user\/group ownership and permissions. There are 2 sockets, one for full read-write access, the other for read-only access."

`unix_sock_rw_perms` must be for **r**ead\/**w**rite and `unix_sock_ro_perms` must be for **r**ead**o**nly permissions. \(also it took me a little to remember in unix everthing is a file including sockets...\).

[CentOS obviously supports PolicyKit](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Desktop_Migration_and_Administration_Guide/policykit.html) so I'm assuming the above docs for connecting as a non-root user don't apply. Time to learn about PolicyKit!

> "As far as the new features are concerned, authorization rules are now defined in JavaScript `.rules` files"

lol javascript.

> "The simplest way to ensure that your old rules are not overridden is to begin the name of all other .rules files with a number higher than 49."

I bet they would recommend using `!important` in CSS to ensure your old rules are not overridden...

After studying enough of `polkit` to [understand this rule](https://goldmann.pl/blog/2012/12/03/configuring-polkit-in-fedora-18-to-access-virt-manager/) and then adding the policy I still had no success connecting to `qemu:///system` as non-root. :\(

[Fortunately,](https://wiki.archlinux.org/index.php/Libvirt#Using_polkit)

> "As of libvirt 1.2.16, members of the `libvirt` group have passwordless access to the RW daemon socket by default."

```sh
bryce @ kvm 
%> sudo usermod -a -G libvirt bryce

bryce @ kvm
%> virsh --connect qemu:///system
Welcome to virsh, the virtualization interactive terminal.

Type: 'help' for help with commands
      'quit' to quit

virsh # quit

bryce @ kvm
%> sudo usermod -a -G libvirt qemu

bryce @ kvm
%> ./create-dhcp-box

WARNING CDROM media does not print to the text console by default, so you likely will not see text install output. You might want to use --location. See the man page for examples of using --location with CDROM media

Starting install...
Allocating 'dhcp.qcow2' | 10 GB 00:00:00
ERROR internal error: process exited while connecting to monitor: 2016-08-16T11:31:06.403938Z qemu-kvm: -drive file=/home/bryce/isos/CentOS-7-x86_64-DVD-1511.iso,if=none,id=drive-ide0-0-0,readonly=on,format=raw: could not open disk image /home/bryce/isos/CentOS-7-x86_64-DVD-1511.iso: Could not open '/home/bryce/isos/CentOS-7-x86_64-DVD-1511.iso': Permission denied

Domain installation does not appear to have been successful.
If it was, you can restart your domain by running:
   virsh --connect qemu:///system start dhcp
otherwise, please restart your installation.
```

I hate computers...

Well. I guess going down the `polkit`\/`libvirt` rabbit hole wasn't a complete waste of time since I learned stuff? Wait! I removed myself and qemu from the `libvirt` group since I want to keep the permissions at the minimum viable and `qemu` can't connect anymore! It wasn't a waste!

Having `qemu` in the `libvirt` group solves the `ERROR authentication failed: no agent is available to authenticate` issue when using `qemu:///system`!

Is it possible for one woman to be this dense? I think it is...

**The user running **`virt-install`** has to be able to connect to the libvirt daemon.**

Which basically I had setup from the beginning.

Basically all the error message is telling me is that I had the iso in a place that `qemu` couldn't access, which makes total sense since it was in my home directory...

```sh
bryce @ kvm
%> sudo gpasswd -d bryce libvirt

bryce @ kvm
%> sudo gpasswd -d qemu libvirt

bryce @ kvm
%> sudo mkdir -p /var/lib/libvirt/isos

# probably should have been a better copy command?
bryce @ kvm
%> sudo mv ~/isos/CentOS-7-x86_64-DVD-1511.iso /var/lib/libvirt/isos

bryce @ kvm 
%> sudo ls -lAd /var/lib/libvirt/* /var/lib/libvirt/isos/*drwx--x--x. 2 root root 6 Jun 23 09:26 /var/lib/libvirt/bootdrwxr-xr-x. 2 root root 4096 Aug 16 06:51 /var/lib/libvirt/dnsmasqdrwx--x--x. 2 root root 6 Jun 23 09:26 /var/lib/libvirt/filesystemsdrwx--x--x. 2 root root 61 Aug 16 06:38 /var/lib/libvirt/imagesdrwxr-xr-x. 2 root root 41 Aug 16 06:33 /var/lib/libvirt/isos-rw-rw-rw-. 1 qemu qemu 4329570304 Aug 11 06:25 /var/lib/libvirt/isos/CentOS-7-x86_64-DVD-1511.isodrwx------. 2 root root 6 Jun 23 09:26 /var/lib/libvirt/lxcdrwx------. 2 root root 6 Jun 23 09:26 /var/lib/libvirt/networkdrwxr-x--x. 7 qemu qemu 93 Aug 16 06:50 /var/lib/libvirt/qemu

bryce @ kvm
%> nvim create-dhcp-box

=== create-dhcp-box ===
#!/bin/sh
sudo virt-install \
 --connect qemu:///system
 --network bridge:br0 \
 --name vm1 \
 --ram=1024 \
 --vcpus=1 \
 --disk size=10 \
 --graphics none \
 --cdrom /var/lib/libvirt/isos/CentOS-7-x86_64-DVD-1511.iso

===

bryce @ kvm 
%> ./create-dhcp-box
WARNING CDROM media does not print to the text console by default, so you likely will not see text install output. You might want to use --location. See the man page for examples of using --location with CDROM media

Starting install...
Allocating 'dhcp-2.qcow2' | 10 GB 00:00:00
Creating domain... | 0 B 00:00:00
Connected to domain dhcp
Escape character is ^]

^]^]^]Domain installation still in progress. You can reconnect to
the console to complete the installation process.

```

Success! \(sort of...\)

```sh
bryce @ kvm 
%> sudo virsh list --all
  Id        Name        State
----------------------------------------------------
  -        dhcp        shut off

bryce @ kvm
%> sudo virsh undefine dhcp
Domain dhcp has been undefined

bryce @ kvm
%> sudo virsh list --all
  Id        Name        State
----------------------------------------------------

bryce @ kvm
%> sudo rm -f /var/lib/libvirt/images/dhcp.qcow2
```

### Headless Install of CentOS 7 Virtual Machine

After a ~~short~~ longer than expected break writing a node.js bot (super fun and highly recommended...) I'm back to becoming a SysAdmin! Having even just this meager documentation makes this much less of a chore...

I found a [wiki entry](https://wiki.centos.org/TipsAndTricks/VncHeadlessInstall) on installing CentOS 5/6ish over VNC headless, but honestly I'd like to try some less terrifying stuff first.

[This article](https://raymii.org/s/articles/virt-install_introduction_and_copy_paste_distro_install_commands.html#CentOS_7) has some interesting additional flags for `virt-install`.

```sh
bryce @ kvm 
%> less create-dhcp-box
#!/bin/sh
sudo virt-install \
 --connect qemu:///system \
 --network bridge:br0 \
 --name dhcp \
 --ram=1024 \
 --vcpus=1 \
 --disk size=10 \
 --graphics none \
 --console pty,target_type=serial \
 --os-type linux \
 --os-variant centos7
 --cdrom /var/lib/libvirt/isos/CentOS-7-x86_64-DVD-1511.iso \
 --extra-args 'console=tty50,115200n8 serial'

bryce @kvm
%> ./create-dhcp-box
ERROR Error validating install location: Distro 'centos7' does not exist in our dictionary

bryce @ kvm
%> virt-install --os-variant list
ERROR
--name is required
--memory amount in MiB is required
--disk storage must be specified (override with --disk none)
An install method must be specified
(--location URL, --cdrom CD/ISO, --pxe, --import, --boot hd|cdrom|...)
```

Ugggh. I hate computers. It appears CentOS7 can't be found, no worries though [I'll just list the available distros](http://thomasmullaly.com/2014/11/16/the-list-of-os-variants-in-kvm/).

jk, that command doesn't actually work?

`man virt-install` led me to a different command

```sh
bryce @ kvm
%> osinfo-query os | grep cent
 centos6.0 | CentOS 6.0 | 6.0 | http://centos.org/centos/6.0
 centos6.1 | CentOS 6.1 | 6.1 | http://centos.org/centos/6.1
 centos6.2 | CentOS 6.2 | 6.2 | http://centos.org/centos/6.2
 centos6.3 | CentOS 6.3 | 6.3 | http://centos.org/centos/6.3
 centos6.4 | CentOS 6.4 | 6.4 | http://centos.org/centos/6.4
 centos6.5 | CentOS 6.5 | 6.5 | http://centos.org/centos/6.5
 centos7.0 | CentOS 7.0 | 7.0 | http://centos.org/centos/7.0
```
Yay!

```sh
bryce @ kvm
%> ./create-dhcp-box
ERROR --extra-args only work if specified with --location. See the man page for examples of using --location with CDROM media
```

not yay... Some more digging in `man virt-install` and I think I'm starting to understand? The `--cdrom` option appears to be just something that is mounted when the new VM first boots. It could technically be [a Backstreet Boys CD](https://www.destructoid.com/games-time-forgot-monster-rancher-1-and-2-128720.phtml). Trying to pass `--extra-args` to a random .iso file will of course be problematic. 

From the `--location` option documentation:

> "Distribution tree installation source. virt-install can recognize certain distribution trees and fetches a bootable kernel/initrd pair to launch the install."

So using `--extra-args` is obviously expecting some sort of kernel to be passed to and the only way to specify that kernel is with the `--location` arg. It all makes sense now!

```sh
bryce @ kvm 
%> sudo mkdir /var/lib/libvirt/mount

bryce @ kvm
%> sudo mount -o loop /var/lib/libvirt/isos/CentOS-7-x86_64-DVD-1511.iso /var/lib/libvirt/mount
mount: /dev/loop0 is write-protected, mounting read-only

bryce @ kvm%> nvim create-dhcp-box

=== create-dhcp-box ===
#!/bin/sh
sudo virt-install \
 --connect qemu:///system \
 --network bridge:br0 \
 --name dhcp \
 --ram=1024 \
 --vcpus=1 \
 --disk size=10 \
 --graphics none \
 --console pty,target_type=serial \
 --os-type linux \
 --os-variant centos7.0 \
 --location /var/lib/libvirt/mount \
 --extra-args 'console=tty50,115200n8 serial'

===
bryce @ kvm nvim
%> ./create-dhcp-box

Starting install...
Retrieving file vmlinuz... | 9.8 MB 00:00:00
Retrieving file initrd.img... | 73 MB 00:00:00
Allocating 'dhcp.qcow2' | 10 GB 00:00:00
Creating domain... | 0 B 00:00:00
Connected to domain dhcp
Escape character is ^]

^]^]^]^]^]^]Domain installation still in progress. You can reconnect tothe console to complete the installation process.

bryce @ kvm 
%> sudo virsh list --all
  Id        Name        State
----------------------------------------------------
  -         dhcp        running

bryce @ kvm
%> sudo virsh undefine dhcp
Domain dhcp has been undefined

bryce @ kvm
%> sudo rm /var/lib/libvirt/images/dhcp.qcow2
```

So close! It's actually running and not complaining about the CDROM! Hmmm... It's probably time to follow the [terrifying wiki entry](https://wiki.centos.org/TipsAndTricks/VncHeadlessInstall) from earlier to make an ISO that can be booted/installed headless.

