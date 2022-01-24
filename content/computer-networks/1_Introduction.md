---
layout: default
usemathjax: true
permalink: /network/ch1
---

* Table of Contents
{:toc}


# 1 - Introduction

## Internet components

- **Hosts/End system**: a device connected to the Internet
  - Run network apps
- **Communication links**: wired (fiber, copper) / wireless (radio, satellite)
  - Bandwidth: transmission rate (rate of putting bits on the links)
- **Packet switches**: forwarding groups of data (packets) over the network
  - Routers, switches

## Network edge

- End systems are **connected** to the network via access networks but are not part of it!

- **Network edge**: The area where the local network of a device interfaces with the internet
  - Hosts: clients, servers
- **Access network**: Connects end systems to edge routers.
  - Residential/Institutional/Mobile
  - Bandwidth, shared/dedicated

## Network core

- **Network core**: Interconnected mesh of routers
- **Packet-switching**: break application-level messages into packets and forward them on the links to other routers/hosts

### Packets

#### Sending

1. Break application message into $L$​ bit packets
2. Transmit packets at $R$​ bits/sec (aka link capacity/bandwidth)
3. **Store-and-forward**: Entire packet arrives at destination router before transmission to the next link.

$$
\text{transmission delay} = \text{ time to push 1 packet on link} = L / R
$$

Implications: 

1. $R$ is never shared. You can only transmit one packet at a time.
2. For each link the packet is transmitted on, incurs $\geq L/R$ delay.

**End-to-end delay**: delay incurred across first source to final destination

The router **can** send a packet to a destination **while** receiving another packet from source.

#### Queueing delay and loss:

![Packet queue](/notes-blog/assets/img/network/packet_queue.png)

If arrival rate $>$ transmission rate of a link:

- Buffer for packets to queue
- If buffer is full, incoming packets can be dropped
  - Lost packets may be retransmitted by previous node/source or not at all

Note: each outgoing link has a corresponding buffer

### Functions of network core

![Core functions](/notes-blog/assets/img/network/network_core_function.png)

**Routing**: Algorithms to determine source-destination route

- Creates a local forwarding table for that router to determine where to route different headers to
  - Headers: Metadata for routing e.g. IP **address**

**Forwarding**: Moving the packets from router's input to output

### Alternate core: Circuit switching

![Circuit switching](/notes-blog/assets/img/network/circuit_switching.png)

- Each link can accomodate several reservable "paths"/segments (dedicate resources to) one connection
- Pros (over packet switching): guaranteed performance
- Cons: Circuit segment idle if not used by call

## Internet structure

Network of networks (evolution is driven by economics and national policies)

- End systems connect to access ISPs
- Access ISPs interconnected to send packets to each other

### Evolution

| Options                                                      | Details                                                      |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![](/notes-blog/assets/img/network/internet_v1.png)<br />Complete graph, each ISP connected to every other | **Cons:** $O(N^2)$ connections: not scalable                 |
| ![](/notes-blog/assets/img/network/internet_v2.png)<br />Connect each access ISP to one global transit ISP | **Customer-provider**: Access network (customer) <br />pays global ISP (provider) to access other networks<br /><br />**Oligopoly**: Competition from other "global ISPs". <br />How to connect these? |
| ![](/notes-blog/assets/img/network/internet_v3.png)<br />- Bilateral connections (peering) to connect 2 ISPs<br />- 3rd-party Internet Exchange Points for multilateral connections | **Peering pros**: speed<br />**Peering cons**: price of laying cables<br />**IXP pros**: cheaper<br />**IXP cons**: congestion<br />**Overall cons**: even big IXPs insufficient coverage |
| ![](/notes-blog/assets/img/network/internet_v4.png)<br />- Regional ISPs cover a smaller geographical region<br />- Content providers (e.g. Google, Akamai) run their own networks | Content providers don't have to rely on other ISPs<br />to deliver services |

### Structure

![Internet structure](/notes-blog/assets/img/network/internet_tree.png)

- Siblings: peering
- Parent-Child: customer-provider
- At center: a small number of well connected networks

## Delay, loss, throughput

### Packet delay

![Packet delay](/notes-blog/assets/img/network/packet_delay.png)
$$
d_\text{nodal} = d_\text{proc} + d_\text{queue} + d_\text{trans} + d_\text{prop} 
$$

- $d_\text{proc}$: nodal **processing** ($< \micro$s)
  - check bit errors
  - determine output link
- $d_\text{queue}$: **queueing** delay
  - time waiting at output link for transmission
  - depends on level of congestion
- $d_\text{trans}$: **transmission** delay ($L/R$​)
- $d_\text{prop}$: **propagation** delay ($d/s$)
  - $d$ length of physical link
  - $s$ propagation speed (in fibre optic, $2 \times 10^8$m/s, close to speed of light)

### Throughput

![Throughput](/notes-blog/assets/img/network/throughput.png)

Rate (bits/s) at which bits are transferred **between a sender and receiver**.

Average throughput:
$$
\begin{aligned}
T_\text{ave} &= F/t & \text{where $t$ is the time to send F bits} \\
&= \frac{F}{F/R_s + F/R_c} \\
&= \frac{1}{1/R_s + 1/R_c}
\end{aligned}
$$
Instantaneous throughput: Depends on the bottleneck rate $\min(R_s, R_c)$​

Reality: in between (due to packet based atomic)

## Protocols

- Define format
- Order of messages sent and received amongst network entities
- Actions taken on message transmission
- Receipt

### Layered structure

- Each layer implements a service that
  - Performs a type of action in that layer
  - Relies on the layers below it
- Modularization makes maintenance and updating easier
  - change of implementation of layer's service

### Internet protocol stack

1. **Application** (FTP, SMTP, HTTP)
   - Support network applications
2. **Transport** (TCP, UDP)
   - Process-process data transfer
3. **Network** (IP, routing protocols)
   - Source-dest routing
4. **Link** (Ethernet, 802.111 - Wifi, PPP)
   - Data transfer between neighbour elements
5. **Physical** (Bits and wiring)

### Encapsulation

![Encapsulation](/notes-blog/assets/img/network/encapsulation.png)

At each stage, pass through multiple layers. The further lower down the stack, include more headers.

