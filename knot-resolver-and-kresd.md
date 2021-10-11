---
description: >-
  DNS over TLS forwarding requests to cz.nic. Optionally serve DNS to a client
  on a Wireguard VPN.
---

# knot-resolver & kresd

## Install knot-resolver

{% embed url="https://knot-resolver.readthedocs.io/en/stable/index.html" %}

```bash
wget https://secure.nic.cz/files/knot-resolver/knot-resolver-release.deb
sudo dpkg -i knot-resolver-release.deb
sudo apt update
sudo apt install -y knot-resolver knot-dnsutils
```

{% hint style="warning" %}
 
{% endhint %}

Stop resolved, confirm it's stopped and remove the symbolic link to /etc/systemd/resolved.conf

```bash
sudo systemctl stop systemd-resolved
sudo systemctl status systemd-resolved
sudo rm /etc/resolv.conf
```

Point DNS to kresd, turn on DNSSEC and DNSOverTLS & disable the stub resolver.

```bash
sudo nano /etc/systemd/resolved.conf 
```

```bash
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.
#
# Entries in this file show the compile time defaults.
# You can change settings by editing this file.
# Defaults can be restored by simply deleting this file.
#
# See resolved.conf(5) for details

[Resolve]
# Some examples of DNS servers which may be used for DNS= and FallbackDNS=:
# Cloudflare: 1.1.1.1 1.0.0.1 2606:4700:4700::1111 2606:4700:4700::1001
# Google:     8.8.8.8 8.8.4.4 2001:4860:4860::8888 2001:4860:4860::8844
# Quad9:      9.9.9.9 2620:fe::fe
DNS=127.0.0.1 ::1
#FallbackDNS=
#Domains=
DNSSEC=yes
DNSOverTLS=yes
#MulticastDNS=no
#LLMNR=no
#Cache=no-negative
DNSStubListener=no
#DNSStubListenerExtra=
#ReadEtcHosts=yes
#ResolveUnicastSingleLabel=no
```

{% hint style="warning" %}
add ubuntu in /etc/hosts to make the below go away or set a FQDN.

sudo: unable to resolve host ubuntu: Name or service not known
{% endhint %}

Configure netplan to use kresd.

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

```yaml
# This file is generated from information provided by the datasource.  Changes
# to it will not persist across an instance reboot.  To disable cloud-init's
# network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
  ethernets:
    eth0:
      dhcp4: true
      dhcp4-overrides:
        use-dns: false
      dhcp6: true
      dhcp6-overrides:
        use-dns: false
      ipv6-privacy: true
      nameservers:
        addresses: [127.0.0.1]
      optional: true
  version: 2
```

Apply the changes.

```bash
sudo netplan apply
```

Open the kresd.conf file and replace the contents with the below.

```bash
sudo nano /etc/knot-resolver/kresd.conf
```

```lua
-- SPDX-License-Identifier: CC0-1.0
-- vim:syntax=lua:set ts=4 sw=4:
-- Refer to manual: https://knot-resolver.readthedocs.org/en/stable/

-- Network interface configuration
net.listen('127.0.0.1', 53, { kind = 'dns' })
net.listen('127.0.0.1', 853, { kind = 'tls' })
--net.listen('127.0.0.1', 443, { kind = 'doh2' })
net.listen('::1', 53, { kind = 'dns', freebind = true })
net.listen('::1', 853, { kind = 'tls', freebind = true })
--net.listen('::1', 443, { kind = 'doh2' })

-- Listen on Wireguard, interface must exist
--net.listen('10.220.0.1', 853, { kind = 'tls' })

-- Enable optional modules
modules = {
  'policy',
  'hints > iterate',  -- Load /etc/hosts and allow custom root hints
  'serve_stale < cache',
  'workarounds < iterate',
  'stats',
  'predict'
}

policy.add(policy.slice(
   policy.slice_randomize_psl(),
   policy.TLS_FORWARD({
      -- multiple servers can be specified for a single slice
      -- the one with lowest round-trip time will be used
      {'193.17.47.1', hostname='odvr.nic.cz'},
      {'185.43.135.1', hostname='odvr.nic.cz'},
   })
))

-- Cache size
cache.size = 100 * MB
```

Start systemd-resolved back up.

```bash
sudo systemctl start systemd-resolved
```

Enable the kresd systemd service and start it.

```bash
sudo systemctl enable --now kresd@1.service
sudo systemctl start kresd@1.service
```

Check for DNSSEC support.

```bash
resolvectl statistics
```

## Confirm

Install tshark.

```bash
sudo apt install tshark
```

```bash
sudo tshark dst port 853
```

Open another terminal and trigger a lookup.

```bash
kdig +dnssec armada-alliance.com
```

### Sword of Omens

```bash
sudo lsof -i -P -n
ss -a -t state established # -t for tcp
networkctl status eth0
resolvectl status
```

{% embed url="https://github.com/macvk/dnsleaktest" %}
