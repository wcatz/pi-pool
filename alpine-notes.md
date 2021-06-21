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

Extract the contents into the mounted fat16 partition.



### Download headless boot overlay

{% embed url="https://wiki.alpinelinux.org/wiki/Raspberry\_Pi\_-\_Headless\_Installation" %}

{% embed url="http://www.sodface.com/repo/headless.apkovl.tar.gz" %}

Copy headless.apkovl.tar.gz onto the fat16 partition.

Create a file named usercfg.txt on the fat16 partition add the following.

```bash
## Pi Pool ##
over_voltage=6
arm_freq=2000
gpu_mem=32
disable-wifi
disable-bt
enable_uart=1
```

### Boot into Alpine

Set a password for root account.

```bash
passwd
```

{% embed url="https://wiki.alpinelinux.org/wiki/Alpine\_newbie\_apk\_packages" %}

Remove the local script service from the default run-level and delete the headless setup script.

```bash
rm -r /etc/local.d
rc-update del local default
```

Run Alpine's automatic system configuration suite individually.

```bash
setup-ntp #chrony
setup-keymap #none
setup-hostname #alpine
setup-timezone #UTC
setup-apkrepos #r
setup-lbu #none
setup-apkcache #none
```

Add user and add them to the wheel group

{% hint style="danger" %}
fix adduser to create home folder
{% endhint %}

```bash
adduser ada
apk add sudo nano htop
nano /etc/sudoers # uncomment %wheel ALL=(ALL) ALL
addgroup ada wheel
```

Edit /etc/ssh/sshd\_config

```bash
#    $OpenBSD: sshd_config,v 1.103 2018/04/09 20:41:22 tj Exp $

# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# This sshd was compiled with PATH=/usr/bin:/bin:/usr/sbin:/sbin

# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options override the
# default value.

#Include /etc/ssh/sshd_config.d/*.conf

#Port 22
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::

#HostKey /etc/ssh/ssh_host_rsa_key
#HostKey /etc/ssh/ssh_host_ecdsa_key
#HostKey /etc/ssh/ssh_host_ed25519_key

# Ciphers and keying
#RekeyLimit default none

# Logging
#SyslogFacility AUTH
#LogLevel INFO

# Authentication:

#LoginGraceTime 2m
PermitRootLogin no
#StrictModes yes
#MaxAuthTries 2
#MaxSessions 10

#PubkeyAuthentication yes

# Expect .ssh/authorized_keys2 to be disregarded by default in future.
#AuthorizedKeysFile    .ssh/authorized_keys .ssh/authorized_keys2

#AuthorizedPrincipalsFile none

#AuthorizedKeysCommand none
#AuthorizedKeysCommandUser nobody

# For this to work you will also need host keys in /etc/ssh/ssh_known_hosts
#HostbasedAuthentication no
# Change to yes if you don't trust ~/.ssh/known_hosts for
# HostbasedAuthentication
#IgnoreUserKnownHosts no
# Don't read the user's ~/.rhosts and ~/.shosts files
#IgnoreRhosts yes

# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication yes
#PermitEmptyPasswords no

# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)
ChallengeResponseAuthentication no

# Kerberos options
#KerberosAuthentication no
#KerberosOrLocalPasswd yes
#KerberosTicketCleanup yes
#KerberosGetAFSToken no

# GSSAPI options
#GSSAPIAuthentication no
#GSSAPICleanupCredentials yes
#GSSAPIStrictAcceptorCheck yes
#GSSAPIKeyExchange no

# Set this to 'yes' to enable PAM authentication, account processing,
# and session processing. If this is enabled, PAM authentication will
# be allowed through the ChallengeResponseAuthentication and
# PasswordAuthentication.  Depending on your PAM configuration,
# PAM authentication via ChallengeResponseAuthentication may bypass
# the setting of "PermitRootLogin without-password".
# If you just want the PAM account and session checks to run without
# PAM authentication, then enable this but set PasswordAuthentication
# and ChallengeResponseAuthentication to 'no'.
UsePAM yes

AllowAgentForwarding no
AllowTcpForwarding no
#GatewayPorts no
X11Forwarding no
#X11DisplayOffset 10
#X11UseLocalhost yes
#PermitTTY yes
#PrintMotd yes
#PrintLastLog yes
#TCPKeepAlive yes
#PermitUserEnvironment no
#Compression delayed
#ClientAliveInterval 0
#ClientAliveCountMax 3
#UseDNS no
#PidFile /var/run/sshd.pid
#MaxStartups 10:30:100
#PermitTunnel no
#ChrootDirectory none
#VersionAddendum none

# no default banner path
#Banner none

# Allow client to pass locale environment variables
AcceptEnv LANG LC_*

# override default of no subsystems
Subsystem sftp    /usr/lib/openssh/sftp-server

# Example of overriding settings on a per-user basis
#Match User anoncvs
#    X11Forwarding no
#    AllowTcpForwarding no
#    PermitTTY no
#    ForceCommand cvs server

```

```bash
setup-lbu #sda1
lbu commit -d
```

```bash
setup-disk
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

```bash
setup-lbu #none
setup-apkcache #none
```

```bash
lbu commit -d
reboot
```

{% embed url="https://wiki.alpinelinux.org/wiki/Newbie\_Alpine\_Ecosystem" %}

```bash
apk add htop sed attr dialog dialog-doc bash bash-doc bash-completion grep grep-doc
apk add util-linux util-linux-doc pciutils usbutils binutils findutils readline
apk add man man-pages lsof lsof-doc less less-doc nano nano-doc curl curl-doc
export PAGER=less
```

Enable additional repositories for apk.

#### Speed up boot time create entropy.

```bash
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

```bash
rc-update add swclock boot    # enable the software clock
rc-update del hwclock boot    # disable the hardware clock
```

### Zram-init

{% embed url="https://wiki.gentoo.org/wiki/Zram\#Initialization" %}

```bash
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

```bash
rc-config add zram-init boot
/etc/init.d/zram-init start
```

### s6

{% embed url="https://paulgorman.org/technical/linux-alpine.txt.html" %}

links

```bash
- http://www.skarnet.org/software/s6/index.html
- http://www.skarnet.org/software/s6/servicedir.html
- https://github.com/skarnet/s6-rc/tree/master/examples/source
- http://jaytaylor.com/notes/node/1484870107000.html
```

