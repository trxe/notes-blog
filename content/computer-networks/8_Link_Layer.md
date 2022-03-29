---
layout: default
usemathjax: true
permalink: /network/ch8
---

(Role of link layer, Parity, CRC, shared medium access)

# Link Layer

Link layer sends datagram between adjacent nodes over **single link**.

Link-layer **frames** are used for transmitting IP datagrams as payload.

Each link layer protocols used for different types of links, and may have different extra services, but have a same basic set of services.

Examples of Link layer protcols:
- 4G/5G
- PPP
- Ethernet

![](/notes-blog/assets/img/network/routing.png)

## Possible Link Layer Services

- Framing [datagram -> header + datagram (payload) + trailer]
  - At network layer and above, datagrams are basically sent unchanged (except at NAT router)
  - Frames however are manipulated.
- Link access control
  - When many nodes share one link
  - Coordinate when each node is allowed to send.
- Reliable delivery
  - Less used on low bit-error links e.g. fibre
  - More used on high bit-error links e.g. wifi
- Error detection
- Error correction

## Implementation

Needs an adapter on each end of a link.

![](/notes-blog/assets/img/network/adapter_link.png)

Network interface controller (NIC) chip:
- Ethernet card
- 802.11 card

## Error Detection and Correction (EDC)

Schemes
- Checksum (TCP/UDP/IP)
- Parity Check
- CRC (Link)

Not 100% reliable. Larger EDC field yields better detection

### Parity Checking

Single bit: Append a single bit $x$ to the end.
- If number of 1s is even: $x = 0$
- Else $x = 1$
- Detects a **single** bit error

2D bit parity:
- For a $m \times n$ grid of bits
- Compute a parity bit for each row and each column.
  - Total $m + n + 1$ parity bits
- Detect and correct **single** bit error
- Detect **two** bit error

![](/notes-blog/assets/img/network/2d_parity.png)

### Cyclic redundancy check (CRC)

Powerful ED coding
- $D$: data bits (a single binary number)
- $G$: generator of $r+1$ bits, agreeded by sender/receiver a priori
- $R$: generates CRC of $r$ bits

1. Sender sends $(D, R)$
2. Receiver knows $G$, divides $(D, R)$ by $G$
  - bit-wise xor operation without carry/borrow.
3. If non-zero remainder: error detected.
  - Higher the $R$, lesser the chance.

![](/notes-blog/assets/img/network/crc.png)

## Multiple access protocols

Type 1: Point-to-point link (Sender and receiver connected by dedicated link)
- PPP
- Serial Line Internet Protocol (SLIP)

Type 2: Broadcast link (many nodes to one broadcast channel)
- 802.11 WiFi
- Satellite
- Ethernet with bus topology

How to prevent collisions in a broadcast channel?

Multiple Access Protocols are:
- distributed algos determininghow to share channel
- Coordination itself uses the channel (no out of band channel signals)

### Channel Partitioning

### Taking turns

### Random Access