---
description: cardano-node on Alpine Linux
---

# Alpine Notes

## Installing Alpine on Pi4

### Download Alpine

Download Alpine to local machine.

{% embed url="https://dl-cdn.alpinelinux.org/alpine/v3.14/releases/aarch64/alpine-rpi-3.14.0-aarch64.tar.gz" %}

Create a fat32 partition on your target drive and mount it.

Extract the contents into the mounted fat32 partition.

Create a file named **usercfg.txt** on the fat32 partition, add the following.

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

Plug in monitor and keyboard and boot RPi

```bash
passwd
```

{% embed url="https://wiki.alpinelinux.org/wiki/Alpine\_newbie\_apk\_packages" %}

## Setup Alpine 'sys' install

{% embed url="https://wiki.alpinelinux.org/wiki/Install\_to\_disk" %}

Run Alpine's automatic system configuration tool.

```bash
setup-alpine
```

> No disks available. Try boot media /media/usb? \(y/n\) \[n\] y

> WARNING: you are stopping a sysinit service 
>
> Unmounting /.modloop ...                                                                                                                                                                                         \[ ok \]

> Available disks are:
>
> sda    \(32.0 GB ASMT     2115            \)
>
> Which disk\(s\) would you like to use? \(or '?' for help or 'none'\) \[none\] sda
>
> The following disk is selected:
>
> sda    \(32.0 GB ASMT     2115            \)
>
> How would you like to use it? \('sys', 'data', 'lvm' or '?' for help\) \[?\] sys
>
> WARNING: The following disk\(s\) will be erased:
>
> sda    \(32.0 GB ASMT     2115            \)
>
> WARNING: Erase the above disk\(s\) and continue? \(y/n\) \[n\] y

## Create user

Add user ada and add it to the wheel group.

```bash
adduser ada
apk add sudo nano
nano /etc/sudoers # uncomment %wheel ALL=(ALL) ALL
addgroup ada wheel
```

Enable additional repositories for apk.

```bash
nano /etc/apk/repositories
```

```bash
 GNU nano 5.7                                                                                      /etc/apk/repositories                                                                                                 
#/media/usb/apks
http://dl-2.alpinelinux.org/alpine/v3.14/main
#http://dl-2.alpinelinux.org/alpine/v3.14/community
http://dl-2.alpinelinux.org/alpine/edge/main
http://dl-2.alpinelinux.org/alpine/edge/community
#http://dl-2.alpinelinux.org/alpine/edge/testing

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
apk add wget htop sed attr dialog dialog-doc bash bash-doc bash-completion grep grep-doc
apk add util-linux util-linux-doc pciutils usbutils binutils findutils readline
apk add man man-pages lsof lsof-doc less less-doc nano nano-doc curl curl-doc
export PAGER=less
```

Now when using ssh to enter the server use the -t switch for a bash shell. Otherwise you will login to Busybox's ash shell.

```bash
ssh ada@<private server ip> -t bash
```

{% hint style="info" %}
Slap Ubuntu's .bashrc file in the home directory
{% endhint %}

{% embed url="https://wiki.alpinelinux.org/wiki/Newbie\_Alpine\_Ecosystem" %}

#### Speed up boot time create entropy.

```bash
apk update
apk add haveged rng-tools
rc-update add haveged boot
rc-update add urandom boot
rc-update add rngd boot

```

### CPU frequency scaling

{% embed url="https://wiki.alpinelinux.org/wiki/CPU\_frequency\_scaling" %}

```bash
sudo nano /etc/local.d/cpufreq.start
```

```bash
#!/bin/sh

# Set the governor to ondemand for all processors
for cpu in /sys/devices/system/cpu/cpufreq/policy*; do
  echo ondemand > ${cpu}/scaling_governor              
done

# Reduce the boost threshold to 80%
echo 80 > /sys/devices/system/cpu/cpufreq/ondemand/up_threshold
```

Make it executable.

```bash
sudo chmod +x /etc/local.d/cpufreq.start
```

#### Clock-related error messages

During the booting time, you might notice errors related to the hardware clock. The Raspberry Pi does not have a hardware clock and therefore you need to disable the hwclock daemon and enable swclock:

### Zram-init

{% embed url="https://wiki.gentoo.org/wiki/Zram\#Initialization" %}

```bash
apk add zram-init
```

```bash
mv /etc/conf.d/zram-init /etc/conf.d/zram-init.bak
```

```bash
nano /etc/conf.d/zram-init
```

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
sudo rc-update add zram-init boot
/etc/init.d/zram-init start
```

### Cardano-node init file

{% embed url="https://wiki.alpinelinux.org/wiki/Writing\_Init\_Scripts" %}

```bash
sudo nano /etc/init.d/cardano-node
```

Configure init to start it after startup.

```bash
#!/sbin/openrc-run

name=$RC_SVCNAME
description="Cardano node service"

depend() {
        after network-online
}

#source /home/ada/cnode_env
start() {

        ebegin "Starting $RC_SVCNAME"
        start-stop-daemon --background --start --exec /home/ada/.local/bin/cardano-node run \
        --make-pidfile --pidfile /var/run/cardano-node.pid \
        -- --topology /home/ada/pi-pool/files/mainnet-topology.json \
           --database-path /home/ada/pi-pool/db \
           --socket-path /home/ada/pi-pool/db/socket \
           --host-addr 0.0.0.0 \
           --port 3003 \
           --config /home/ada/pi-pool/files/mainnet-config.json
        eend $?
}

start_post() {
rcstatus=blank
        while [ -f /var/run/cardano-node.pid ]
        do
                sleep 10
                rcstatus=`rc-service $RC_SVCNAME status | awk '{print $NF}'`
                if [ -z $rcstatus ]; then
                        rc-service -c $RC_SVCNAME restart
                        break
                fi
        done &
}

stop() {
        ebegin "Stopping $RC_SVCNAME"
        start-stop-daemon --stop --exec /home/ada/.local/bin/cardano-node run \
        --pidfile /var/run/cardano-node.pid \
        -s 2
        eend $?
}
```

```bash
sudo chmod +x /etc/init.d/cardano-node
```

Add to boot & start command.

```bash
sudo rc-update add cardano-node
sudo rc-service cardano-node start # or stop
rc-status cardano-node
```

### Install/configure Prometheus

```bash
wget -O ~/prometheus.tar.gz https://github.com/prometheus/prometheus/releases/download/v2.27.1/prometheus-2.27.1.linux-arm64.tar.gz
tar -xzvf prometheus.tar.gz
mv prometheus-* /etc/prometheus
```

Configuration file.

```bash
sudo nano /etc/prometheus/prometheus.yml
```

```bash
global:
  scrape_interval:     15s # By default, scrape targets every 15 seconds.

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
    monitor: 'codelab-monitor'

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label job=<job_name> to any timeseries scraped from this config.
  - job_name: 'Prometheus' # To scrape data from the cardano node
    scrape_interval: 5s
    static_configs:
#      - targets: ['<CORE PRIVATE IP>:12798']
#        labels:
#          alias: 'C1'
#          type:  'cardano-node'
#      - targets: ['<RELAY PRIVATE IP>:12798']
#        labels:
#          alias: 'R1'
#          type:  'cardano-node'
      - targets: ['localhost:12798']
        labels:
          alias: 'N1'
          type:  'cardano-node'

#      - targets: ['<CORE PRIVATE IP>:9100']
#        labels:
#          alias: 'C1'
#          type:  'node'
#      - targets: ['<RELAY PRIVATE IP>:9100']
#        labels:
#          alias: 'R1'
#          type:  'node'
      - targets: ['localhost:9100']
        labels:
          alias: 'N1'
          type:  'node'
```

#### Init file.

```bash
sudo nano /etc/init.d/prometheus
```

```bash
#!/sbin/openrc-run

name=$RC_SVCNAME
description="Prometheus service"

depend() {
	after network-online
}

start() {

#source /home/ada/cnode_env

        ebegin "Starting $RC_SVCNAME"
        start-stop-daemon --background --start --exec /etc/prometheus/prometheus \
        --make-pidfile --pidfile /var/run/prometheus.pid \
        -- --config.file="/etc/prometheus/prometheus.yml" \
        --web.listen-address="0.0.0.0:9090"
        eend $?

}

start_post() {
rcstatus=blank
        while [ -f /var/run/prometheus.pid ]
        do
                sleep 10
                rcstatus=`rc-service $RC_SVCNAME status | awk '{print $NF}'`
                if [ -z $rcstatus ]; then
                        rc-service -c $RC_SVCNAME restart
                        break
                fi
        done &
}

stop() {
	ebegin "Stopping $RC_SVCNAME"
        start-stop-daemon --stop --exec /etc/prometheus/prometheus \
        --pidfile /var/run/prometheus.pid \
        -s 2
        eend $?
}
```

```bash
sudo chmod +x /etc/init.d/prometheus
```

```bash
sudo rc-update add prometheus
sudo rc-service prometheus start
rc-status prometheus
```

### Install/configure Node Exporter

```bash
wget -O ~/node_exporter.tar.gz https://github.com/prometheus/node_exporter/releases/download/v1.1.2/node_exporter-1.1.2.linux-arm64.tar.gz
```

```bash
tar -xzvf node_exporter.tar.gz
sudo mv node_exporter-*.linux-arm64 /etc/node_exporter
```

#### Init file.

```bash
sudo nano /etc/init.d/node-exporter
```

```bash
#!/sbin/openrc-run

name=$RC_SVCNAME
description="Node exporter service"

depend() {
	after network-online
}

start() {

#source /home/ada/cnode_env

        ebegin "Starting $RC_SVCNAME"
        start-stop-daemon --background --start --exec /etc/node_exporter/node_exporter \
        --make-pidfile --pidfile /var/run/node-exporter.pid \
        -- --web.listen-address="0.0.0.0:9100"
        eend $?

}

start_post() {
rcstatus=blank
	while [ -f /var/run/node-exporter.pid ]
	do
		sleep 10
		rcstatus=`rc-service $RC_SVCNAME status | awk '{print $NF}'`
		if [ -z $rcstatus ]; then
	                rc-service -c $RC_SVCNAME restart
			break
		fi
	done &
}

stop() {
	ebegin "Stopping $RC_SVCNAME"
        start-stop-daemon --stop --exec /etc/node_exporter/node_exporter \
        --pidfile /var/run/node-exporter.pid \
        -s 2
        eend $?
}
```

```bash
sudo chmod +x /etc/init.d/node-exporter
```

```bash
sudo rc-update add node-exporter
sudo rc-service node-exporter start
rc-status node-exporter
```

### Install/configure Grafana

```bash
sudo apk add grafana
```

```bash
sudo nano /etc/grafana.ini
```

> Change http port from 3000 to 5000 so it doesn't clash with cardano-node and remove the ;

```bash
sudo nano /etc/init.d/grafana
```

```bash
#!/sbin/openrc-run

name=$RC_SVCNAME
description="Grafana service"

depend() {
        after network-online
}

start() {

        ebegin "Starting $RC_SVCNAME"
        start-stop-daemon --background --start --exec /usr/sbin/grafana-server \
        --make-pidfile --pidfile /var/run/grafana.pid \
        --  --homepath=/usr/share/grafana \
            --config=/etc/grafana.ini
        eend $?

}

start_post() {
rcstatus=blank
        while [ -f /var/run/grafana.pid ]
        do
                sleep 10
                rcstatus=`rc-service $RC_SVCNAME status | awk '{print $NF}'`
                if [ -z $rcstatus ]; then
                        rc-service -c $RC_SVCNAME restart
                        break
                fi
        done &
}

stop() {
        ebegin "Stopping $RC_SVCNAME"
        start-stop-daemon --stop --exec /usr/sbin/grafana-server \
        --pidfile /var/run/grafana.pid \
        -s 2
        eend $?
}
```

```bash
sudo chmod +x /etc/init.d/grafana
```

```bash
sudo rc-update add grafana
sudo rc-service grafana start
rc-status grafana
```

### Download db/ folder

```bash
wget -r -np -nH -R "index.html*" -e robots=off https://db.adamantium.online/db/
```

