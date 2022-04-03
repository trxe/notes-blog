---
layout: default
usemathjax: true
permalink: /network/ch8
---

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

**Type 1**: Point-to-point link (Sender and receiver connected by dedicated link)
- Point-to-Point Protocol (PPP)
- Serial Line Internet Protocol (SLIP)

There is no need for multiple access control in a PtP link, each link is dedicated.

**Type 2**: Broadcast link (many nodes using same broadcast channel)
- 802.11 WiFi
- Satellite
- Ethernet with bus topology

How to prevent collisions in a broadcast channel?

Multiple Access Protocols are:
- distributed algos determininghow to share channel
- Coordination itself uses the channel (no out of band channel signals)

### Channel Partitioning

Divide a channel into smaller pieces, allocate each piece for exclusive use.

- Time Division Multiple Access (TDMA)
  - Rounds of fixed length slot
  - Unused slots go idle
  - Each node has a fixed slot. In use when active, empty when idle.
  - Gets the whole bandwidth for $1/n$th of the time.
  - Cons: Wasted resources, have to repartition when more added.
- Frequency division multiple access (FDMA)
  - Channel spectrum into frequency bands
  - Unused transmission time causes bands to go idle
  - Each node has a fixed range.
  - Gets $1/n$th bandwidth the whole time.
  - Cons: similar to TDMA except for the above.

### Taking turns

Nodes take turns to transmit.

**Polling**: Master node "invite" each slave nodes to transmit.

Cons:
- Overhead from polling every node
- If master fails, the whole system is down (single point of failure)

**Token passing**: Special frame/signal that is passed from node to node
sequentially. When it has the token, it is eligible for transmission.

The network is usually run as a ring.

Cons:
- Overhead from token passing
- If token lost, the whole system is down. Token must be regenerated.

### Random Access

No coordination between nodes. **Collision** handling instead.

- How to detect collisions
- How to recover from collisions

**Slotted ALOHA**:
- Assume that
  - frames are equal size
  - time dividedinto equal length slots
  - nodes transmit at beginning of slot
- Each node **listens** to channel while transmitting to detect collision.
- On collision: retransmit with probability $p$ in the next timeslot until success. Otherwise idle.
- $p$: proportional to collisions, inversely proportional to empty slots.

Has empty slots, collisions aside from success.

Problems: synchronization of clocks.

**Pure (unslotted) ALOHA**:
- No slot/synchronization
- Next transmission is immediate.
- Chance of collision is much higher due to **overlaps** between frames.
- Success is about half of slotted ALOHA's.

![Pure ALOHA](/notes-blog/assets/img/network/pure_aloha.png)

What if when detecting someone else is using the channel, don't use it?

**Carrier Sense Multiple Access (CSMA)**:
- Sense channel before transmission
  - If idle: transmit frame
  - If busy: defer transmission 
- Will collision ever happen?
  - YES. Propagation delay means two nodes may not immediately hear each other's transmission.
  - Even after a collision, it will continue to transmit the whole frame.

![](/notes-blog/assets/img/network/csma_collision.png)

**CSMA/CD (Collision detection)**:
- Same as CSMA but on collision detect, the sender will abort asap
  - Not instantaneous but fast.

![](/notes-blog/assets/img/network/csma_cd.png)

What if the frame size is too small? 
- The sender/receiver may not realise that their frame has collided.
- Hence there is a minimal frame size.

Early ethernet used CSMA/CD, but today ethernet uses star topology 
with point-to-point connections so no need for multiple access protocol. 

Early ethernet used  a **shared** coaxial cable, which computers connected to using **vampire taps**.

**CSMA/CA (Collision Avoidance)**:
- Collision detect is hard in wireless LAN compared to wired LAN.
  - Wired LAN can just sense the increase in signal voltage.
  - Wireless LAN has varying strength of signals.
- The receiver sends back to the sender a NAK/ACK.

