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
* [Legendary Technology: New Raspberry Pi 4 Bootloader USB](https://jamesachambers.com/new-raspberry-pi-4-bootloader-usb-network-boot-guide/)

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

vm.swappiness=10
vm.vfs_cache_pressure=50

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

### Zram swap

{% embed url="https://haydenjames.io/raspberry-pi-performance-add-zram-kernel-parameters/" %}

```text
sudo apt install zram-config
sudo reboot
```

