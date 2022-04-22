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

**Flag**: 1 if there's a next segment, otherwise 0

**Lenght**: length of entire IP fragment (header + payload)

**Offset**: offset from start of original datagram in **8-bytes** (sets of 8 bytes)

**MTU** (Max Transfer Unit): The maximum amount of data (bytes) a **link-level frame** can carry.

**Checksum**: same checksum as in transport layer

![](/notes-blog/assets/img/network/internet_checksum.png)

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

Routing between Autonomous Systems (ASs) in the internet (network of networks) is done hierarchically.

AS: Mini internet under **one organization's** control.

**Intra-AS** 
- Single admin
- No policy decisions
- Performance priority
- Ignore hosts; view only routers and the links between them.
- **Least cost** path between source and destination routers.
  - Shortest path? Congestion level? Money?

**Inter-AS** routing.
- Multiple admins
- Policy priority

## Routing algos

**Link state** algorithms: all routers know the whole network topology (graph) and link cost (edge weight). 
- Can use Dijkstra's Algorithm.
- Routers periodically broadcast links to each other 
- A lot of broadcasting global information between each other, so not used cos too costly.

**Distance vector** algorithms: 
- routers know **immediate neighbours** only. 
- Exchange of "local views" and update its own "local views".
- Iterative
  - Swap local view with neighbours
  - Update local view
  - Repeat until no change in local view.

### Bellman-Ford

Let $d_x(y)$ be the least cost path from $x$ to $y$.

Let $c(x,y)$ be the cost of the direct link between nodes $x$ and $y$.
$c(x,y) = \infty$ if $x, y$ are not neighbours.

$$
d_x(y) = \min_{v \in \text{neighbours}(x)} {c(x,v) + d_v(y)}
$$

```python
n = V.length
for i in range(0, n):
  for edge in graph:
    relax(edge)
```

Key property: If P is the shortest path from S to D, and node X is on P, 
then P must be the shortest path from S to X $\cup$ shortest path from X to D.

Distributed: Each node receive info from neighbours, recompute and distributes again

Iterative: Continues until two iterations are the same (no more information)

Asynchronous: Nodes do not have to compute in a certain order.

## Routing Information Protocol (RIP)

- **Hop count** as metric, using the distance-vector (DV) protocol with BF algorithm.
- Entries in routing table: subnet masks
- Exchange routing table updates every 30s
- If no update <3min, assume neighbour failed

# Internet Control Message Protocol (ICMP)

ICMP are carriedin IP datagrams.

Contained after IP header (IP header - ICMP header - data).

**Header**: Type + Code (for specific status) + Checksum

Examples
- Echo request reply (`ping`).
- Use of `ping` and TTL to create `traceroute`: