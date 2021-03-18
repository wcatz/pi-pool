---
description: 'optimize hardware, optimize Ubuntu'
---

# Server Setup

## Configure Hardware

Lets save some power, raise the governor on the cpu a bit and set gpu ram as low as we can.

{% hint style="warning" %}
Here are some links for overclocking and testing your drive speeds. If you have a fan and heat sinks you can safely go to 2000. It's up to you. Just pay attention to over volt recommendations to go with your chosen clock speed.

* [https://www.raspberrypi.org/documentation/configuration/config-txt/overclocking.md](https://www.raspberrypi.org/documentation/configuration/config-txt/overclocking.md)
* [https://www.seeedstudio.com/blog/2020/02/12/how-to-safely-overclock-your-raspberry-pi-4-to-2-147ghz/](https://www.seeedstudio.com/blog/2020/02/12/how-to-safely-overclock-your-raspberry-pi-4-to-2-147ghz/)
* [https://dopedesi.com/2020/11/24/upgrade-your-raspberry-pi-4-with-a-nvme-boot-drive-by-alex-ellis-nov-2020/](https://dopedesi.com/2020/11/24/upgrade-your-raspberry-pi-4-with-a-nvme-boot-drive-by-alex-ellis-nov-2020/)

Take note that Ubuntu stores config.txt in a different location than Raspbian.
{% endhint %}

### Test drive speed

#### Write speed

```text
sudo dd if=/dev/zero of=/tmp/output conv=fdatasync bs=384k count=1k; sudo rm -f /tmp/output
```

#### Read speed

```text
sudo hdparm -Tt /dev/sda
```

### Overclock, memory & radios

Edit  /boot/firmware/config.txt. Just paste Pi Pool additions in at the bottom.

```bash
sudo nano /boot/firmware/config.txt
```

```
[pi4]
max_framebuffers=2

[all]
arm_64bit=1
kernel=vmlinuz
cmdline=cmdline.txt
initramfs initrd.img followkernel

# Enable the audio output, I2C and SPI interfaces on the GPIO header
dtparam=audio=on
dtparam=i2c_arm=on
dtparam=spi=on

# Enable the serial pins
enable_uart=1

# Comment out the following line if the edges of the desktop appear outside
# the edges of your display
disable_overscan=1

# If you have issues with audio, you may try uncommenting the following line
# which forces the HDMI output into HDMI mode instead of DVI (which doesn't
# support audio output)
#hdmi_drive=2

# If you have a CM4, uncomment the following line to enable the USB2 outputs
# on the IO board (assuming your CM4 is plugged into such a board)
#dtoverlay=dwc2,dr_mode=host

## Pi Pool ##
over_voltage=2
arm_freq=1750
gpu_mem=16
disable-wifi
disable-bt
```

```text
sudo reboot
```

## Configure Ubuntu

### Disable the root user

```text
sudo passwd -l root
```

### Secure shared memory

Open /etc/fstab.

```text
sudo nano /etc/fstab
```

Add this line at the bottom, save & exit.

```text
tmpfs	/run/shm	tmpfs	ro,noexec,nosuid	0 0
```

### Increase open file limit

Open /etc/security/limits.conf.

```text
sudo nano /etc/security/limits.conf
```

Add the following to the bottom, save & exit.

```text
ada soft nofile 800000
ada hard nofile 1048576
```

### Optimize performance & security

Add the following to the bottom of /etc/sysctl.conf. Save and exit.

{% hint style="info" %}
[https://gist.github.com/lokhman/cc716d2e2d373dd696b2d9264c0287a3](https://gist.github.com/lokhman/cc716d2e2d373dd696b2d9264c0287a3)
{% endhint %}

{% hint style="warning" %}
I am disabling IPv6 and IPv4 forwarding. You may want these. I have seen claims that IPv6 is slower and get's in the way. &lt;find this later&gt;
{% endhint %}

```text
sudo nano /etc/sysctl.conf
```

```text

## Pi Pool ##

fs.file-max = 10000000
fs.nr_open = 10000000

# enable forwarding if using wireguard
net.ipv4.ip_forward=0

# ignore ICMP redirects
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0

net.ipv4.icmp_ignore_bogus_error_responses = 1

# disable IPv6
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1

# block SYN attacks
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 2048
net.ipv4.tcp_synack_retries = 3
net.ipv4.netfilter.ip_conntrack_tcp_timeout_syn_recv=45

# in progress tasks
net.ipv4.tcp_keepalive_time = 240
net.ipv4.tcp_keepalive_intvl = 4
net.ipv4.tcp_keepalive_probes = 5

# reboot if we run out of memory
vm.panic_on_oom = 1
kernel.panic = 10

# Use Google's congestion control algorithm
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
```

#### Load our changes after boot

Create a new file. Paste, save & close.

```text
sudo nano /etc/rc.local
```

```text
#!/bin/bash

# Give CPU startup routines time to settle.
sleep 120

sysctl -p /etc/sysctl.conf

exit 0
```

### Disable IRQ balance

{% hint style="info" %}
[**http://bookofzeus.com/harden-ubuntu/server-setup/disable-irqbalance/**](http://bookofzeus.com/harden-ubuntu/server-setup/disable-irqbalance/)
{% endhint %}

You should turn off IRQ Balance to make sure you do not get hardware interrupts in your threads. Turning off IRQ Balance, will optimize the balance between power savings and performance through distribution of hardware interrupts across multiple processors.

Open /etc/default/irqbalance and add to the bottom. Save, exit and reboot.

```text
sudo nano /etc/default/irqbalance
```

```text
ENABLED="0"
```

```text
sudo reboot
```

### Swapping to disk

These images do not have swap. The following Reddit post sums up how I feel about it. It is our responsibility as pool operators to know when hardware upgrades are required and have a solution in place if needed. If a relay runs out of memory it will be an abnormal event. If something like that is happening I don't want Ubuntu to swap the problem to disk. I want a kernel panic and a reboot. Then I want my monitoring server to alert me. This is one reason why two or more relays are recommended. This is not an email server....

You do not have to agree with me and are welcome to add a swap partition. Ubuntu starts swapping after 40% of memory is used by default. Which is a vm.swappiness setting of 60. If you must use swap be sure to set it to 10, swap at 90% used.

> [I don't use swap. I have the following reasons:](https://www.reddit.com/r/Ubuntu/comments/fd19mz/do_i_need_swap/fjg9tqm?utm_source=share&utm_medium=web2x&context=3)

> * Linux does swapping during disk I/O based on the swappiness factor. It needs to be set to basically 0, as I feel that any value is too high. The default 60 is so high that you can end up reading and writing several dozen MB/s to swap continuously when reading data from a modern NVMe drive, which can do 1 GB/s or more. Even if it's only few % of that which gets turned to swapping, that sustained over few dozens of seconds still in practice swaps out your terminal emulator, your window manager, your browser and everything else. Eventually it devolves to the point where swapin and swapout rates become the same, i.e. the kernel has squeezed everything it possibly can to swap, and is still looking for more pages, but everything user does now causes pages to get swapped back in.
> * swapping in addition to already doing heavy disk i/o makes Linux run like a dog, probably due to disk scheduling related reasons. In practice, the machine will spend literal seconds switching browser tabs, and stutters so hard that not even the mouse cursor will move sometimes. It feels like your Linux system has become completely overpowered by the simple act of reading some random file in the background.
> * even zram swap \(which swaps just to memory using deflate compression\) with swappiness tuned to 0 or 1 seems to cause some amount of visible stutter that is not present when Linux has no swap. In this case, there is no additional disk I/O, and the compression should only take mere microseconds per page, so I'm not quite sure why that happens.
>
> Conclusion: as I've found no configuration where use of swap wouldn't have a visible detrimental effect, I run all my Linux systems without any swap, even systems that have just 4 GB of memory \(they are single purpose machines\) up to the ones that have 96 GB \(can run a dozen VMs & crap\). In my experience, the memory extension concept of swap isn't that great due to the low performance of SSDs relative to main memory, and even something like zram swap works much better to extend your system's capability without seemingly completely halting it when swapping gets heavy. However, I've had a few machines with zram enabled crash in a way that implicated I/O related to zram, so I'm not using even that anymore.
>
> Besides hibernation memory storage, the only actual use case of swap is discovering anonymous memory pages that the application left laying around and which it doesn't need anymore, but didn't free to the system. The kernel has no other means to discover them and free them for some other, more useful purpose. Ideally, swapped out memory should never need to be paged in again, not even at application's exit. This may be a sizable fraction of used memory, perhaps up to about 20 % based on some measurements I have made with both server and desktop workloads. I'd rather see people study what this memory is for, e.g. if GTK+ or some other library allocates a lot of stuff during init that it never really needs, and whether libraries could be fixed to free this kind of useless memory themselves.
>
> As you have observed, swap is less of a problem these days for SSDs. Wear leveling takes care of the problem of swap constantly exercising the same sectors of the drive. Bear in mind, however, that consumer SSDs can break easily, particularly if they are based on the currently cheapest 4-bit QLC cells, as they have dramatically lower endurance than server-grade 2-bit MLC, which are also found on the roughly twice as expensive prosumer type drives, e.g. Samsung Pro line. While I don't expect swap to be the thing that kills your drive, a sustained write load can still break the cheapest drives in some weeks or months, so their endurance in practice can run out. And as I said, Linux's swappiness settings are such that they turn even pure disk reading into disk writing due to I/O pressure encouraging the kernel to find more free RAM, and the bad ramifications from swapping are in my experience quite serious, even when the SSD itself can take the additional unnecessary write load.

