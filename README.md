# rctl's DNS toolkit

This repository is for different DNS toolkits and servers I use for personal labs. This repo is great if you have Docker and want to play around with TinyDNS or Unbound. Checkout the tools folders for example configurations (used as default in Docker if no other is supplied).

## Unbound

To run Unbound with default config (do not expose to internet):

```
docker run -d -p 53:53/udp rctl/unbound
```

To run Unbound with custom config:

```
docker run -d -p 53:53/udp -v /path/to/unbound.conf:/var/unbound/unbound.conf rctl/unbound
```

To test:

```
dig google.com @127.0.0.1
```

## TinyDNS

To run TinyDNS with default config (not very useful):

```
docker run -d -p 53:53/udp rctl/tinydns
```

To test (should return authoritative answer 10.0.0.1):

```
dig test.lab @127.0.0.1
```

To run TinyDNS with custom config:

```
docker run -d -p 53:53/udp -v /path/to/tinydns.conf:/data  rctl/tinydns
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
        name: "test.lab."
        forward-host: 10.88.0.5
```
Run Unbound with the saved configuration (you know what to replace):
```
docker run -d --name unbound -p 53:53/udp --net=dnsnet  -v /path/to/unbound.conf:/var/unbound/unbound.conf rctl/unbound
```
Run TinyDNS with default config (will be authoritative for test.lab):
```
docker run -d --name tinydns rctl/tinydns
```
Connect TinyDNS to the network and assign a known IP address
```
docker network connect --ip 10.88.0.5 dnsnet tinydns
```
Test everything:
```
dig test.lab @127.0.0.1

Expect:

; <<>> DiG 9.9.7-P3 <<>> test.lab @127.0.0.1
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 55232
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;test.lab.			IN	A

;; ANSWER SECTION:
test.lab.		86399	IN	A	10.0.0.1

;; AUTHORITY SECTION:
test.lab.		259199	IN	NS	ns1.test.lab.

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: XXXXXXXXXXXXX
;; MSG SIZE  rcvd: 71
```
To see what is happening in your servers:
```
docker logs unbound
docker logs tinydns
```
Cleanup:
```
docker stop tinydns unbound
docker rm tinydns unbound
docker network rm dnsnet
```

## Running Local DNS with Unbound

Running a local DNS on your machine will decrease load times of DNS lookups, which may be handy if you run applications which do a lot of lookups. I use the following configuration to run a local DNS with unbound. If you want to do the same, run unbound with the **-p 127.0.0.1:53:53/udp** flag for setting up port. This flag will only allow traffic from your own machine (so that noone can use your machine for amplification attacks). Ideally, you can run your own DNS server on your local network instead of your own machine, but if you are always on the go like me then running it locally like this works fine as well. Here I have added priv.lab as an example of a localized domain that can be used for quick access (instead of using /etc/hosts file).

Unbound config:

```
server:
        verbosity: 4
        interface: 0.0.0.0
        port: 53
        do-ip4: yes
        do-ip6: yes
        do-udp: yes
        use-syslog: no
        access-control: 0.0.0.0/0 allow
        chroot: "/var/unbound"
        username: "unbound"
        directory: "/var/unbound"
        local-zone: "lab." static
        local-data: "priv.lab. IN A 10.16.0.1"
forward-zone:
    name: *
    forward-addr: 1.1.1.1
    forward-addr: 8.8.8.8
```