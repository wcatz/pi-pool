# wireguard notes

## Install Wireguard



```
sudo apt install wireguard
```

Become root

```text
sudo su
```

```text
cd /etc/wireguard
```

```text
wg genkey | tee C1-privkey | wg pubkey > C1-pubkey
```



