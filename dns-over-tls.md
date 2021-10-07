---
description: >-
  Use Stubby to encrypt DNS queries and Dnsmasq to cache the results for
  subsequent requests
---

# DNS over TLS

## Install Stubby

```
sudo apt install stubby
```

Start the Stubby service, enable at boot and confirm it is running.

```bash
sudo systemctl start stubby
sudo systemctl enable stubby
sudo systemctl status stubby
```

Confirm Stubby is listening on loopback\(127.0.0.1:53\).

```bash
dig @127.0.0.1 armada-alliance.com
```

Should return something like this. Take note of the query time. In this example it took 231ms.

```bash
; <<>> DiG 9.16.8-Ubuntu <<>> @127.0.0.1 armada-alliance.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 58565
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;armada-alliance.com.		IN	A

;; ANSWER SECTION:
armada-alliance.com.	722	IN	A	185.199.109.153
armada-alliance.com.	722	IN	A	185.199.108.153

;; Query time: 231 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Thu Oct 07 09:39:28 UTC 2021
;; MSG SIZE  rcvd: 80
```

Open the Stubby configuration file.

```bash
sudo nano /etc/stubby/stubby.yml
```

Change the listen\_addresses entry to port 5300.

```bash
listen_addresses:
  - 127.0.0.1@53000
  - 0::1@53000
```

Save the file and restart Stubby.

```bash
sudo systemctl restart stubby.service
```

Confirm Stubby is no longer listening on 127.0.0.1:53. Request should eventually timeout.

```bash
dig @127.0.0.1 armada-alliance.com
```

Confirm Stubby is listening on port 5300.

```bash
dig @127.0.0.1 -p 53000 armada-alliance.com
```

```bash
; <<>> DiG 9.16.8-Ubuntu <<>> @127.0.0.1 -p 53000 armada-alliance.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 9907
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 03616d1624cf753801000000615ec782f6ad59a19ac3ad44 (good)
;; QUESTION SECTION:
;armada-alliance.com.		IN	A

;; ANSWER SECTION:
armada-alliance.com.	3600	IN	A	185.199.108.153
armada-alliance.com.	3600	IN	A	185.199.109.153

;; Query time: 567 msec
;; SERVER: 127.0.0.1#53000(127.0.0.1)
;; WHEN: Thu Oct 07 10:10:10 UTC 2021
;; MSG SIZE  rcvd: 146
```

That one took 567ms. We can use Dnsmasq to cache lookups.

## Install Dnsmasq

```bash
sudo apt install dnsmasq
```

Move the default config file and create a new one.

```bash
sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.bak
sudo nano /etc/dnsmasq.conf
```

Configure Dnsmasq to use Stubby and bind to the loopback interface\(lo\)

```bash
server=127.0.0.1#53000
listen-address=127.0.0.1
interface=lo
bind-interfaces
```

Save the file, restart Dnsmasq and enable it to start on boot.

```bash
sudo systemctl restart dnsmasq
sudo systemctl enable dnsmasq
```

Confirm everything is working.

```bash
dig @127.0.0.1 armada-alliance.com
```

```bash
; <<>> DiG 9.16.8-Ubuntu <<>> @127.0.0.1 armada-alliance.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 61523
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;armada-alliance.com.		IN	A

;; ANSWER SECTION:
armada-alliance.com.	533	IN	A	185.199.108.153
armada-alliance.com.	533	IN	A	185.199.109.153

;; Query time: 399 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Thu Oct 07 10:30:48 UTC 2021
;; MSG SIZE  rcvd: 80
```

399ms. Run the same command.

```bash
dig @127.0.0.1 armada-alliance.com
```

```bash
; <<>> DiG 9.16.8-Ubuntu <<>> @127.0.0.1 armada-alliance.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 63210
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;armada-alliance.com.		IN	A

;; ANSWER SECTION:
armada-alliance.com.	524	IN	A	185.199.109.153
armada-alliance.com.	524	IN	A	185.199.108.153

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Thu Oct 07 10:30:57 UTC 2021
;; MSG SIZE  rcvd: 80
```

This one should be pretty quick. Congrats DNS caching of DNS over TLS. Now we have to configure Ubuntu to use it and prevent DNS leaks.

