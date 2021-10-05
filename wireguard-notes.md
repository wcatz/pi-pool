# wireguard notes

From the [WireGuard](https://www.wireguard.com/) project homepage:

WireGuard is an extremely simple yet fast and modern VPN that utilizes state-of-the-art cryptography. It aims to be faster, simpler, leaner, and more useful than IPsec, while avoiding the massive headache. It intends to be considerably more performant than OpenVPN. WireGuard is designed as a general purpose VPN for running on embedded interfaces and super computers alike, fit for many different circumstances. Initially released for the Linux kernel, it is now cross-platform \(Windows, macOS, BSD, iOS, Android\) and widely deployable.

```bash
sudo apt install wireguard
```

Become root

```bash
sudo su
```

Enter the Wireguard folder and restrict access to root for any new files created.

```bash
cd /etc/wireguard
umask 077
```

Generate key pair for the core node.

```bash
wg genkey | tee C1-privkey | wg pubkey > C1-pubkey
```

Create a Wireguard configuration file.

```bash
nano /etc/wireguard/wg0.conf
```

Edit key values and endpoint. You will need to generate a keypair on the remote node and copy it's public key here.

{% hint style="info" %}
As long as one endpoint is reachable Wireguard will connect. This is handy for punching through NAT firewalls.
{% endhint %}

```bash
[Interface]
Address = 10.0.0.1/32
SaveConfig = true
ListenPort = 51820
PrivateKey = <qKGaBCnUQq2G821v1l2jm2xJc6IC9izOG2G92kyoEH8=>

[Peer]
PublicKey = <FnXP9t17JXTCf3kyuTBh/z83NeJsE8Ar2HtOCy2VPyw=>
AllowedIPs = 10.0.0.2/32
Endpoint = <Remote nodes public ip or dnsname>:51820
PersistentKeepalive = 21

```

wg-quick does all the work for us setting up the interface.

```bash
wg-quick up wg0
```

Enable the Wireguard service to automatically restart on boot.

```bash
sudo systemctl enable wg-quick@wg0
```

```bash
sudo wg show
```

