---
description: cardano-node on Alpine Linux
---

# Alpine Notes

## Installing Alpine on Pi4

### Download Alpine

{% embed url="https://wiki.alpinelinux.org/wiki/Classic\_install\_or\_sys\_mode\_on\_Raspberry\_Pi" %}

{% embed url="https://dl-cdn.alpinelinux.org/alpine/v3.14/releases/aarch64/alpine-rpi-3.14.0-aarch64.tar.gz" %}

### Download headless boot overlay

{% embed url="https://wiki.alpinelinux.org/wiki/Raspberry\_Pi\_-\_Headless\_Installation" %}

### Boot into Alpine

```text
setup-ntp
setup-keymap
setup-hostname
setup-timezone
setup-apkrepos
```

#### Speed up boot time.

```text
 apk update 
 apk add haveged
 rc-update add haveged boot
 service haveged start
```

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



