---
layout: default
usemathjax: true
permalink: /network/ch7
---

# IPv4 Header of Datagram

Notes: 
- IP checksum is computed with only the IP header, not the whole datagram.
- TTL refers to the **number of hops** to destination.
- The payload is a Transport layer TCP segment or UDP datagram.

![](/notes-blog/assets/img/network/ipv4.png)

## Why fragment?

Different link layers have different physically determined Maximum Transfer Units
(MTUs) -- the max amount a link level frame can carry.

![](/notes-blog/assets/img/network/ip_frag.png)

From 1 IP header + 1 payload $\rightarrow$ 3 $\times$ (IP header + $\frac{1}{3}$ 
payload).

- IP datagram = IP header + payload.
- Fragment offset: in units of **8-bytes** (since flags take up 3 bytes).
- Frag Flag: 1 if next fragment from same segment, 0 otherwise.

# Network Address Translation

**Public** IP addresses are *globally* unique and routable.

**Private** IP addresses are routable within an organization/home etc, but not globally.
- 10.0.0.0/8
- 172.16.0.0/16
- 192.168.0.0/16

Serving a packet from a Private IP address to the internet will fail (ISP will never).

All $n$ local area network (LAN) addresses $\rightarrow$ NAT $\rightarrow$ 1 public address 
to the wide area network (WAN).

![](/notes-blog/assets/img/network/nat.png)

- Outgoing datagram: **Map** (source ip, port num) to (NAT ip, new port num).
- Caching this **mapping** in NAT translation table
- Incoming datagram: **Map** (NAT ip, port num) to (source  ip, orig port num)

Problems:
- p2p applications will not work directly on two local connections.
  - The two NAT devices cannot find each other when they only know each other's local addresses.

### IPv6

Same as IPv4 in header structure, except **source, destination** addresses are now 128-bit.
Header is 40-byte.

# Routing