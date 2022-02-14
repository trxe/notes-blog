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

**Assumption**: Underlying channel may have bit errors

Checksum to detect errors. Then the receiving side (via network) informs sender either:

- ACK (acknowledgement): packet OKs
- NAK (negative acknowledgement): receiver explicitly tells sender packet had error

![rdt2.0](/notes-blog/assets/img/network/rdt2-0_time.png)

### rdt2.1

**Assumption**: ACK/NAK itself can be corrupted

- Sender retransmits upon receiving garbled ACK or NAK
  - Then duplicate packets?
  - Sequence number

NAK

