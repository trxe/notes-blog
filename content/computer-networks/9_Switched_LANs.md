---
layout: default
usemathjax: true
permalink: /network/ch9
---

# Hardware and Ethernet

Communication from interface to interface
- within subnet
- across subnets

Requires HWID of individual interfaces.

## MAC Address

Media Access Control (MAC address) on each NIC (aka physical/LAN addr)
- Send/Receive link frames
- Manufacturer assigned in NIC ROM
- 48 bits (12 hex digits in groups of 2)
- Must be unique

MAC address allocation is administered by IEEE. 

First 3 bytes is the vender of the adapter (see [https://macvendors.com]()).

|   | IP Address | MAC Address |
| - | ---------- | ----------- |
| bits | 32  | 48 |
| move | datagrams from src to dest  | frames over every link |
| assigned | dynamically | permanently |

## Address Resolution Protocol

How do we find the MAC address of a receiving host, given its IP address?

### ARP Table

Contains **IP addr, MAC addr, TTL**. 
- TTL refers to this ARP table entry's TTL.
- This forms a list of neighbours/edges between this interface and others.
- When a frame is received  from another interface, it is added to this table.

### ARP Query

A sender **broadcasts** a query packet, with dest MAC address `FF-FF-FF-FF-FF-FF`.

All other nodes will check if the **IP address** in the query packet matches its own.

On a match, the node will reply with a frame containing it's MAC address.

The sender then caches the IP address and MAC address pair in its ARP table.

### Sending Frame in Same Subnet

Suppose A wants to send to B (A and B are in the same subnet).

Case 1: B's MAC address known
- Create a frame with B's MAC address and send
- Only B will process it
- Other nodes process **but ignore** this frame.

Case 2: B's MAC not known
- A sends an ARP Query packet
- On receiving B's MAC address, goto Case 1.

### Sending Frame in Another Subnet

Q: How do you know you are sending to a different subnet?

A: If subnet doesn't have the same IP prefix.

A should create link layer frame with 
- dest MAC: **gateway (router) interface (of the subnet) MAC**
  - Hence router will receive a match for this frame.
  - Router accept and passes the datagram to the network layer
  - Network layer **checks IP address**, use routing protocol to send it to B's gateway router.
- dest IP: **B's IP address**
  - B's router receives the datagram
  - Sends from source (MAC is that of B's router) to destination (B's MAC)
  - If B's MAC not known, ARP query is performed.

Router interface then receives and passes on to the next interface via RIP or other routing protocol.

Once the destination IP is in the same subnet as the src IP, the link-layer frame with B's MAC address
is made.

# LAN technologies

LAN is any computer network interconnecting computers within a geographical area.

Some LAN technologies:
- IBM Token Ring (802.5)
- Ethernet (802.3)
- WiFi (802.11)
- etc.

Physical topologies:
- bus: all nodes can collide with each other
- star: switch in center, nodes don't collide

## Ethernet

Ethernet standards have:
- Different speeds
- Different physical layer media (twisted pair/fibre physical layer)
- MAC protocol and frame format

Twisted pair copper:
- RJ45 connector
- CAT 5/6/7 cable
- Max speed 10Gbps
- Max length 100m

Optical Fibre:
- Left SC/PC, right SC/PC connectors
- Single mode fibre

Ethernet frame: NIC adapter encapsulates IP datagram

Payload : minsize 46 bytes, maxsize  MT
- dest addr + src addr + type + payload + crc = 64 byte window frame soze;
- Cell synchronization

MAin usage of the  ethernet **preamble**: square wave pattern
- width of a single bit
- clock rate of the sender

**Src/Dest address**: MAC addresses
- If NIC receive matching destination address, pass payload to network layer protocol
**Type**: Network layer protocol
**CRC**: Cheksum. Corrupted frame will be dropped

Delivery service features:
- **Connectionless**: no handshaking between send/receive NIC
- **Unreliable**: receiving NIC doesn't send ACK or NAK to sender NIC.
  - Recovery only when higher level protocol detects missing frame.

### Bus topology

![](/notes-blog/assets/img/network/bus_ethernet.png)

If A sends frame at time $t$, propagation time between A and D is $d$,
and D sends at $t + d - 1$ => COLLISION!

Ethernet CSMA/CD:
1. Create frame with Network layer datagram.
2. If NIC senses the channel is idle, begin transmit.
3. If NIC transmits entire frame without interrupt: done
4. If another transmission is detected: abort + **send jam signal** to alert of collision.

Post abortion of signal: Binary/**Exponential back-off**
- After $m$th collision, chose $K \in {0, 1, \dots, 2^m - 1}$
- NIC waits for $512K$ bit times
- Retry transmission.

Maximum $m$ imposed: 16.

More collisions = heavier load $\Rightarrow$ 
Usually means longer back-off interval with more collision.

## Star topology

Each end host is connected to a switch, which provides a dedicated channel per host.

### Ethernet switches

- Store and forward frames
- Examine incoming frame's MAC, forwards frame to one or more outgoing links.

A pure ethernet switch has no IP address, and is transparent/invisible to hosts.

- On receiving a frame from port A, a dumb switch can send the frame to all the other ports.
- It attempts to learn which port is connected to which end host to improve performance [Switch table]
- Buffers frames, is full duplex.
- Ethernet protocol: no collisions

#### Switch table

Records of **[MAC addr, Interface (port), TTL]**:
- which endhost through which interface?
- On receiving a frame from A, cache in the table
- If B is found in the table, forward the frame via its port
- Otherwise send to all ports

A hub does not keep a switch table, but otherwise does the same thing.

![](/notes-blog/assets/img/network/switch_v_router.png)

## Other Userful References

https://www.practicalnetworking.net/series/packet-traveling/host-to-host-through-a-switch/
