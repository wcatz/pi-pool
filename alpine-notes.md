---
description: cardano-node on Alpine Linux
---

# Alpine Notes

## Installing Alpine on Pi4

### Download Alpine

{% embed url="https://wiki.alpinelinux.org/wiki/Classic\_install\_or\_sys\_mode\_on\_Raspberry\_Pi" %}

Create 2 partitions with gparted. fat16 = 250MB and ext4 rest of the drive space.

Mount the fat16 partition on your local machine.

Download Alpine to local machine.

{% embed url="https://dl-cdn.alpinelinux.org/alpine/v3.14/releases/aarch64/alpine-rpi-3.14.0-aarch64.tar.gz" %}

Extract the contents into the mounted fat16 partition. Do this as root



### Download headless boot overlay

{% embed url="https://wiki.alpinelinux.org/wiki/Raspberry\_Pi\_-\_Headless\_Installation" %}

{% embed url="http://www.sodface.com/repo/headless.apkovl.tar.gz" %}

Copy headless.apkovl.tar.gz onto the fat16 partition.

Create a file named usercfg.txt on the fat16 partition add the following.

```text
## Pi Pool ##
over_voltage=6
arm_freq=2000
gpu_mem=32
disable-wifi
disable-bt
enable_uart=1
```

### Boot into Alpine

Set a password for root account\(lovelace\).

```text
passwd
```

Remove the local script service from the default run-level and delete the headless setup script.

```text
rm /etc/local.d/headless.start
rc-update del local default
```

{% embed url="https://wiki.alpinelinux.org/wiki/Newbie\_Alpine\_Ecosystem" %}

```text
: > .ash_history
```

Run Alpine's automatic system configuration suite.

```text
setup-ntp
setup-keymap
setup-hostname
setup-timezone
setup-apkrepos
setup-lbu
setup-apkcache
setup-disks
```

> No disks available. Try boot media /media/usb? \(y/n\) \[n\] y

> * WARNING: you are stopping a sysinit service
> * Unmounting /.modloop ...                                                                                                                                                                                         \[ ok \]
>
>   Available disks are:
>
>   sda    \(32.0 GB ASMT     2115            \)
>
>   Which disk\(s\) would you like to use? \(or '?' for help or 'none'\) \[none\] sda
>
>   The following disk is selected:
>
>   sda    \(32.0 GB ASMT     2115            \)
>
>   How would you like to use it? \('sys', 'data', 'lvm' or '?' for help\) \[?\] sys
>
>   WARNING: The following disk\(s\) will be erased:
>
>   sda    \(32.0 GB ASMT     2115            \)
>
>   WARNING: Erase the above disk\(s\) and continue? \(y/n\) \[n\] y

```text
apk add htop sed attr dialog dialog-doc bash bash-doc bash-completion grep grep-doc
apk add util-linux util-linux-doc pciutils usbutils binutils findutils readline
apk add man man-pages lsof lsof-doc less less-doc nano nano-doc curl curl-doc
export PAGER=less
```

Enable additional repositories for apk.

#### Speed up boot time create entropy.

```text
apk update
apk add haveged rng-tools
rc-update add haveged boot
rc-update add urandom boot
rc-update add rngd boot

```

{% hint style="warning" %}
CPU scaling is handled by Sayshar\[SRN} repo to come later. Use as reference only.
{% endhint %}

{% embed url="https://wiki.alpinelinux.org/wiki/CPU\_frequency\_scaling" %}

{% embed url="https://wiki.alpinelinux.org/wiki/Change\_default\_shell" %}

{% embed url="https://wiki.alpinelinux.org/wiki/Writing\_Init\_Scripts" %}

#### Clock-related error messages

During the booting time, you might notice errors related to the hardware clock. The Raspberry Pi does not have a hardware clock and therefore you need to disable the hwclock daemon and enable swclock:

rc-update add swclock boot \# enable the software clock.

```text
rc-update add swclock boot    # enable the software clock
rc-update del hwclock boot    # disable the hardware clock
```

### Zram-init

{% embed url="https://wiki.gentoo.org/wiki/Zram\#Initialization" %}

```text
load_on_start="yes"

unload_on_stop="yes"
 
num_devices="2"

type0="swap"
flag0=
size0="2048"
maxs0=2
algo0=lz4

type1="/tmp"
flag1="ext4"
size1="2048"
```

```text
rc-config add zram-init boot
/etc/init.d/zram-init start
```

### s6

{% embed url="https://paulgorman.org/technical/linux-alpine.txt.html" %}

links

```text
- http://www.skarnet.org/software/s6/index.html
- http://www.skarnet.org/software/s6/servicedir.html
- https://github.com/skarnet/s6-rc/tree/master/examples/source
- http://jaytaylor.com/notes/node/1484870107000.html
```

