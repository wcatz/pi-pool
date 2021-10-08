---
description: >-
  DNS over TLS forwarding requests to cz.nic. Optionally serve DNS over TLS to a
  client on a Wireguard VPN.
---

# knot-resolver & kresd

## Getting Super Powers

{% embed url="https://knot-resolver.readthedocs.io/en/stable/index.html" %}

```bash
sudo lsof -i -P -n
networkctl status eth0
```

```bash
wget https://secure.nic.cz/files/knot-resolver/knot-resolver-release.deb
sudo dpkg -i knot-resolver-release.deb
sudo apt update
sudo apt install -y knot-resolver knot-dnsutils
```

{% hint style="warning" %}
 Super-powers are granted randomly so please submit an issue if you're not happy with yours.
{% endhint %}

```bash
sudo nano /etc/systemd/resolved.conf
# edit line 
# DNSStubListener=yes to DNSStubListener=no
sudo systemctl stop systemd-resolved
sudo systemctl status systemd-resolved
sudo rm /etc/resolv.conf
```

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

Configure netplan.

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

```bash
sudo netplan apply
```

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

```bash
sudo systemctl start systemd-resolved
```

```bash
sudo systemctl enable --now kresd@1.service
sudo systemctl start kresd@1.service
```

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

