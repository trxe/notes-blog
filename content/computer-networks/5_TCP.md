---
layout: default
usemathjax: true
permalink: /network/ch5
---

# TCP

Needs to keep track of historical info (state)

- Packets sent but not yet acknoowledged, 
- Packets received so far
- etc.

So need send/receive buffers

How much app-layer data can a TCP segment carry?

- 1460B maximum segment size.
- limited by max transmission unit (MTU): 1500B for ethernet. This is to exclude 40B of packet headers
  - 20B TCP packet header
  - Not including app-layer packet header.

Sequence number may not start from 0

## Socket

Id'd  by 4-tuple: (source IP, source port, dest IP, dest port)	

In TCP, the acknowledgement is`n't acking segments, but a stream of bytes

We can combine sending info and acknowledgement in the same segment!

![](/notes-blog/assets/img/network/tcp_ack_data.png)

Sequence number: Byte id that is a 32-bit integer that wraps around upon overflow.

Acknowledgement number: Last byte id received + 1.

- Receive Window: The number of bytes the receiver is willing ot acccept
  - Flow control: App-layer bottleneck. The receiver informs the sender that their buffer is overflowing soon and sender shoudl reduce the rate of sending.
  - Each TCP socket has its own buffer
  - `rwnd` in TCP header is the recievers free buffer space (out of `RcvBuffer`)
  - Sender must implement such tthat unacked data is within (0, `rwnd`)

## TCP 3-way handshakes

RST, SYN, FIN

SYNbit: Indicator that this segment is for establishing TCP connection.

ACKbit: Indicator that this segment's acknowledgement number is valid.

FINbit: One side informs the other that they want to close this connection (the other side will reply with an ACK). Means this side no longer sends out data, only acknowledgements.

![](/notes-blog/assets/img/network/tcp_start.png)

![](/notes-blog/assets/img/network/tcp_close.png)

The FINbit packet might get lost. Hence we want to have this 2x bye. OTherwise the FINbit packet is resent.