---
layout: default
usemathjax: true
permalink: /network/ch4
---

# UDP, Reliable Protocol

Transport layer and Network layer's interfacing

Transport layer (UDP, TCP): logical communication between processes

- Multiplexing / Demultiplexing (which message is for which process?)

Network layer (IP): logical communication between hosts

- Is unreliable "best effort"
- for data integrity etc., need to implement on top of the network layer (in transport/application layer)

## UDP [User Datagram Protocol]

Only adds on top of IP:

1. Connectionless muxing/demuxing
2. Checksum

Is unreliable, and to achieve reliable UDP transmission, **application** implements error detection/recovery.

### Demultiplexing

**Multiplexing**: method via which analog/digital signals combine into one signal over shared medium.

**Demultiplexing**: opposite process that separates signals.

For the server, a single port serves **every client**. UDP's segment includes:

- source port number (16-bit)
- destination port number (16-bit)
- length of entire UDP segment (64-bit, 8 bytes + application payload)
- checksum
- application payload

### Checksum

To detect errors in the segment.

Computation:

1. Let UDP segment be a sequence of 16-bit integers
2. Apply binary addition on every 16-bit integer, carry over (wraparound) the MSB to the LSB if MSB = 1
3. Continue until the last integer
4. Compute 1's complement to get UDP checksum

Note: even if the checksum + sum = 111...1, there could still be some error in the packet.

## Reliable data transfer

**Application** layer assumes the underlying **transport layer** is reliable.

The transport layer's service is to provide reliable data transfer (rdt) via the underlying **unreliable network layer**.

### Interface

- `rdt_send()`: Application $\rightarrow$ Transport layer (**Sender**)
- `udt_send()`: Transport $\leftrightarrow$​​ Network layer (**Sender**)
  - Called by `rdt_send` to send to network
- `rdt_rcv()`: Transport $\leftrightarrow$ Network layer (**Receive**)
- `deliver_data()`: Application $\leftarrow$​​ Transport (**Receive**)
  - Called by `rdt_rcv` to deliver to app

### rdt1.0

**Assumption**: Underlying channel is perfectly reliable

![rdt1.0](/notes-blog/assets/img/network/rdt1-0.png)

### rdt2.0

**New Limitation**: Underlying channel may have bit errors

Checksum to detect errors. Then the receiving side (via network) informs sender either:

- ACK (acknowledgement): packet OKs
- NAK (negative acknowledgement): receiver explicitly tells sender packet had error

![rdt2.0 Timeline](/notes-blog/assets/img/network/rdt2-0_time.png)

![rdt2.0](/notes-blog/assets/img/network/rdt2-0.png)

### rdt2.1

**New Limitation**: ACK/NAK itself can be corrupted

- Sender retransmits upon receiving garbled ACK or NAK
  - Possibility of **duplicate packet** (when an ACK is corrupted)
- Sequence number
  - Put in the packet header
  - In basic (stop and wait protocol) version, only need ${0, 1}$
  - If two consecutive packets have the same sequence number, receiving end should only deliver the second

**Sender state diagram**:

![rdt2.1](/notes-blog/assets/img/network/rdt2-1-sender.png)

**Receiver state diagram**:

![rdt2.1](/notes-blog/assets/img/network/rdt2-1-rcv.png)

### rdt2.2

**New Limitation**: Eliminate need for NAK

Solution: Instead of NAK, receiver **explicitly includes sequence number** of last ACKed packet.

Duplicate ACK at sender will result in retransmission of current packet.

![rdt2.2](/notes-blog/assets/img/network/rdt2-2-sender.png)

![rdt2.2](/notes-blog/assets/img/network/rdt2-2-rcv.png)

### rdt3.0

**New Limitation**: Underlying channel also can lose whole packets (data, ACKs)

Approach: Sender has a timeout for ACK. Retransmit if no ACK received within this time.

What if ACK is delayed but not lost? Duplicate ACK handling handled by sequence numbers.

![Packet Loss](/notes-blog/assets/img/network/rdt3-0-packet-loss.png)

![And Timeouts](/notes-blog/assets/img/network/rdt3-0-timeouts.png)

![rdt3.0](/notes-blog/assets/img/network/rdt3-0-sender.png)

Note that the timeout + seuqence number handles multiple cases without differentiating them

## Performance of rdt3.0 (Stop and Wait)

Very bad because of poor utilization.

$D_\text{trans} = L/R = 1KB / (10^9 \text{ bits}/s) = 8 \text{ms}$

$U_\text{sender} = \frac{D_\text{trans}}{RTT + D_\text{trans}}$ is very small when $RTT >> D_\text{trans}$.

### Pipeline to increase utilization

![rdt3.0](/notes-blog/assets/img/network/rdt-pipeline.png)

Sender sends multiple "in-flight" packets at once, so range of sequence numbers must be increased. 

### Go-Back-N

We have a **sliding window** for the range of sequence numbers for the inflight packets.

When a new ACK is received, move forward the sliding window.

On duplicate ACKs (NAK), sender ignores the ACK, and receiver discards all packets with sequence numbers $\neq$​​​ latest ACK sequence number + 1. After a timeout sender sends again from latest ACK + 1. [Goes back to the last N]

Receiver maintains a cumulative ACK.

No need for a buffer as **all out of order packets are discarded**

**Pros**: Simple implementation of receiver

**Cons**: Inefficient due to discarding packets (if pipeline bigger, worse efficiency as likely to cause more discarded packets)

### Selective repeat

Receiver individually acknowledges all received packets. Out of order packets are discarded.

The sender now has to maintain a timer for each unACKed packet.

**Pros**: Less wasted transmissions

**Cons**: Much more complicated receiver
