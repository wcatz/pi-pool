# kresd & wireguard

## Getting Super Powers

Becoming a super hero is a fairly straight forward process:

```bash
vi /etc/systemd/resolved.conf     edit line 
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

{% hint style="info" %}
 Super-powers are granted randomly so please submit an issue if you're not happy with yours.
{% endhint %}

Once you're strong enough, save the world:

{% code title="hello.sh" %}
```bash
# Ain't no code for that yet, sorry
echo 'You got to trust me on this, I saved the world'
```
{% endcode %}



