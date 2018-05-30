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