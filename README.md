# rctl's DNS toolkit

This repository is for different DNS toolkits and servers I use for personal labs. This repo is great if you have Docker and want to play around with TinyDNS or Unbound. Checkout the tools folders for example configurations (used as default in Docker if no other is supplied).

## Unbound

To run Unbound with default config (do not expose to internet):

```
docker run -d -p 53:53/udp rctl/unbound:latest
```

To run Unbound with custom config:

```
docker run -d -p 53:53/udp -v /path/to/unbound.conf:/var/unbound/unbound.conf rctl/unbound:latest
```

To test:

```
dig google.com @127.0.0.1
```

## TinyDNS

To run TinyDNS with default config (not very useful):

```
docker run -d -p 53:53/udp rctl/tinydns:latest
```

To test (should return authoritative answer 10.0.0.1):

```
dig test.com @127.0.0.1
```

To run TinyDNS with custom config:

```
docker run -d -p 53:53/udp -v /path/to/tinydns.conf:/data  rctl/tinydns:latest
```

## Connect Unbound with TinyDNS

Create Docker network:
```
docker network create dnsnet --subnet 10.88.0.0/16
````

Save Unbound configuration to a file:
```
server:
        verbosity: 1
        interface: 0.0.0.0
        port: 53
        do-ip4: yes
        do-ip6: yes
        do-udp: yes
        do-tcp: yes
        do-daemonize: yes
        access-control: 0.0.0.0/0 allow
        chroot: "/var/unbound"
        username: "unbound"
        directory: "/var/unbound"
        root-hints: "/var/unbound/named.cache"
forward-zone:
        name: "test.com."
        forward-host: 10.88.0.5
```
Run Unbound with the saved configuration (you know what to replace):
```
docker run -it --name unbound -p 53:53/udp --net=dnsnet  -v /path/to/unbound.conf:/var/unbound/unbound.conf rctl/unbound:latest
```
Run TinyDNS with default config (will be authoritative for test.com):
```
docker run -it --name tinydns rctl/tinydns:latest
```
Connect TinyDNS to the network and assign a known IP address
```
docker network connect --ip 10.88.0.5 dnsnet tinydns
```
Test everything:
```
dig test.com @127.0.0.1

Expect:

; <<>> DiG 9.9.7-P3 <<>> test.com @127.0.0.1
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 55232
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;test.com.			IN	A

;; ANSWER SECTION:
test.com.		86399	IN	A	10.0.0.1

;; AUTHORITY SECTION:
test.com.		259199	IN	NS	ns1.test.com.

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: XXXXXXXXXXXXX
;; MSG SIZE  rcvd: 71
```
Cleanup:
```
docker stop tinydns unbound
docker rm tinydns unbound
docker network rm dnsnet
```