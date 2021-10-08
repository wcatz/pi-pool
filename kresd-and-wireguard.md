# kresd & wireguard

## Getting Super Powers

{% embed url="https://knot-resolver.readthedocs.io/en/stable/index.html" %}



{% hint style="warning" %}
 Super-powers are granted randomly so please submit an issue if you're not happy with yours.
{% endhint %}

```bash
sudo nano /etc/systemd/resolved.conf
edit line 
#DNSStubListener=yes to be DNSStubListener=no
sudo systemctl stop systemd-resolved
sudo systemctl status systemd-resolved
sudo rm /etc/resolv.conf
```

```
wget https://secure.nic.cz/files/knot-resolver/knot-resolver-release.deb
sudo dpkg -i knot-resolver-release.deb
sudo apt update
sudo apt install -y knot-resolver knot-dnsutils
```

Once you're strong enough, save the world:

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

```bash
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

```bash
-- listen to local connections
net.listen(net.lo, 53)
net.listen(net.eth0, 853, { kind = 'tls' })

-- Enable optional modules
modules = {
  'policy',
  'hints',
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

```

```bash
sudo systemctl enable kresd@1.service
sudo systemctl start kresd@1.service
```

