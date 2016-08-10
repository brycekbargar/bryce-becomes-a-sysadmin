# Install CentOS Host

### Making an Installer ###

I looked at using CentOS 6 like the guide suggested, but [they seem very clear about which version to use][0]

> "Legacy versions of CentOS are no longer supported. For historical purposes, CentOS keeps an archive of older versions."

I downloaded the minimal ISO since I wanted to try to figure out what was needed, rather than just have things available via "black magic".

The first thing I tried was [UNetbootin][1] since it was the way I last installed linux (some 8 years ago!).

The [spanish install guide][2] (the first one I went to) actually recommends using UNetBootin as an alternative!

> "UNetbootin ofrece ejecutables para Windows y para Linux y permite crear medios USB booteables para varias distribuciones Linux, incluyendo CentOS."

But once I started troubleshooting the installing I looked at the [English install guide][3] which clearly state to use `dd`...

> "Moreover, the CentOS 7 installer image has a special partitioning which, as of July 2014, most Windows tools do NOT transfer correctly leading to undefined behaviour when booting from the USB key. Applications known (so far) to NOT work are unetbootin and "universal usb installler"."

When I ran `dd if=CentOS-6.5-x86_64-bin-DVD1.iso of=/dev/sdb bs=1M` I got an error on the block size so I ignored it. 

The `dd` operation was unreasonably slow, even for my ancient mac, so [I started troubleshooting][4].

> "The bytesize is 1M which transfers faster. On OS X you have to use `1m` (lowercase) instead of `1M`"

I hate computers...

While the copy operation was running I [learned I could use ctrl + t][5] to view its status

### Installation ###

The [computer][6] I'm using as a "server" for my home lab was originally purposed as a gaming/htpc/recording pc.

For storage it has a 120GB and 480GB ssd as well as two matching 2TB hdds.

The next part of my plan involves installing Windows 10 as a guest with GPU pass-through so I can still use it for it's intended purposes.

This means I want to give the Windows VM complete access to the 480gb ssd and one of the 2TB drives.

Most of the install was straightforward except for partitioning.

I ended up with

- On the 120gb ssd
  - 512mb `/boot`
  - 4096mb `swap`
  - 5120mb `/root`
  - 20gb `/home`
- On the remaining 2TB hdd
  - 2TB `/var`

I chose to do it this way because I don't plan on storing anything in `/home` that needs to survive reinstalls. I also learned that [kvm stores its images on `/var`][7] so I wanted to give that partition a lot of space.

Though in the future I will probably laugh about how naive this was, perfect is the enemy of the good...

A side note, CentOS kept trying to get me to use [LVM partitions][8], which seemed lovely in a homogeneous storage setup I would like my ssds to do all the heavy lifting for now...

### Post Install ###

I setup a root password and a single `bryce` user. I didn't make her an admin, since I thought I would still be able to `sudo` and just enter the root password.

Silly me...

If the option was "Make this user a sudoer" rather than "Make this user and administrator" I think it would have been more clear. Yay Learning!

I fixed this [by running][9]
```
%> su -
%> usermod -a -G wheel bryce
```

`-a` is for append to the following `-G` group `wheel` (admins).

A brief interlude while I updated [my dotfiles][10] to work with CentOS.

A less brief interlude while a hacked together ssh acccess. File permissions are...

To find the [ip address of a CentOS][11] machine use `sudo /sbin/ip addr`

```
bryce @ Bryces-MacBook-Pro vagrant ssh ~/vagrant/centos_7
%> ssh bryce@192.168.0.109
Last login: Mon Aug  8 04:30:53 2016
bryce @ kvm sudo /sbin/ip addr ~
%>
```

Success! 



[0]: https://www.centos.org/download/ 
[1]: https://unetbootin.github.io/
[2]: https://wiki.centos.org/es/HowTos/InstallFromUSBkey
[3]: https://wiki.centos.org/HowTos/InstallFromUSBkey
[4]: http://superuser.com/a/530121
[5]: http://askubuntu.com/a/538241
[6]: http://pcpartpicker.com/list/49RV7h
[7]: https://ask.fedoraproject.org/en/question/9584/while-using-libvirt-how-can-i-use-a-location-other-than-varliblibvirtimages-to-store-my-images/
[8]: https://wiki.archlinux.org/index.php/LVM_(Espa%C3%B1ol)
[9]: http://unix.stackexchange.com/a/210932
[10]: https://github.com/brycekbargar/dotfiles
[11]: https://www.centos.org/forums/viewtopic.php?t=44484


