---
layout: default
usemathjax: true
permalink: /network/ch5
---

# 5 -- TCP

## Features

- Point-to-point
- Connection-oriented (handhskaing to initialize states of sender, receivers)
- Full duplex data
- Reliable byte stream (not packet stream)
- [Pipelined](https://trxe.github.io/notes-blog/network/ch4#pipeline-to-increase-utilization): Dynamic window size set by congestion/flow control

Needs to keep track of historical info (state), hence the requirement of sender/receiver buffers on **both client and server** (remember, TCP is bidirectional!).

- Packets sent but not yet acknowledged, 
- Packets received so far
- etc.

## Socket

A TCP socket is identified by 4-tuple: **(source IP, source port, dest IP, dest port)**

In TCP, the acknowledgement isn't acking segments, but a stream of bytes.

We can combine sending info and acknowledgement in the same segment!
 
## TCP Segment

![](/notes-blog/assets/img/network/tcp_segment.png)

How much app-layer data can a TCP segment carry?

- Maximum Segment Size: maximum amount of data that can be stored in a segment.
- TCP has 1460B maximum segment size (MSS).
  - Maximum amount of data that can be grabbed and placed in one segment
- limited by max transmission unit (MTU): 1500B for ethernet. This is to exclude 40B of packet headers
  - 20B TCP packet header
  - Not including app-layer packet header (this is within the payload).

### Sequence/Acknowledgement number

Two separate values, each counting by bytes.

- **Sequence** number
  - Index of the byte in the data stream (offset by a *random* initial number)
  - Sequence number may not start from 0.
- **Acknowledgement** number
  - Sequence number of the **next byte expected from** the other side.
    - i.e. Opposite end's last sequence number + received payload size
  - Has a ACK validity bit 

> Q: Why is the sequence number randomized? 
> 
> A: To improve security (prevent spoofing) and lessen the likelihood of swapping packets with the same seq# from various sessions between two hosts

![](/notes-blog/assets/img/network/tcp_ack_data.png)

Sequence number: Byte id that is a 32-bit integer that wraps around upon overflow.

Acknowledgement number: Last byte id received + 1.

### Receive Window

Used if application is consuming at a slower rate than TCP socket is receiving, in which case we need to implement **flow control**.

- Flow control: App-layer bottleneck. The receiver informs the sender that their buffer is overflowing soon and sender shoudl reduce the rate of sending.

Receive window specifies The number of bytes the receiver is willing to accept before the receive buffer overflows.

- Each TCP socket has its own buffer
- Receive buffer size set via socket options
- `rwnd` in TCP header is the receivers free buffer space (out of `RcvBuffer`)
- Sender must implement such that size of unacked data is within (0, `rwnd`) where `rwnd` is receiver's value.


## TCP 3-way handshake

RST, SYN, FIN

**SYNbit**: Indicator that this segment is for establishing TCP connection.

1. Client sends TCP SYN message with initial sequence number $x$.
2. Server sends TCP SYNACK message with initial seuqnce number $y$ and ACKnum $x+1$.
3. When Client receives the SYNACK, both Client and Server have established (ESTAB) the server is live.
   - The first segment Client sends now (ACKnum = $y+1$) can contain client-to-server data.

![](/notes-blog/assets/img/network/tcp_start.png)

ACKbit: Indicator that this segment's acknowledgement number is valid.

FINbit: One side informs the other that they want to close this connection (the other side will reply with an ACK). Means this side no longer sends out data, only acknowledgements.

1. Client sends FIN message with sequence number $u$. Now client can no longer send (but can receive) data.
2. Server sends ACK message with ACKnum $u+1$ and now Client waits for the server to close.
3. Server sends FIN message with sequence number $v$. Now server can no longer send data.
4. Client starts timer of (2 * Max Segment Lifetime) and sends ACK message with ACKnum $y+1$.
5. When Server receives ACK, closes. When Client ends its timer, closes.

![](/notes-blog/assets/img/network/tcp_close.png)

The FINbit packet might get lost. Hence we want to have this 2x bye. OTherwise the FINbit packet is resent.

## TCP reliable data transfer summary

rdt aspects: **pipelined** segments, **cumulative** acks, single **retransmission timer**

retransmission triggered by: **timeout events**, duplicate acks. Retransmission only resends the oldest unacked segment.

**cumulative** acks: if ACK=100 has been lost but ACK=120 is received, the former packet will not be retransmitted. 

![](/notes-blog/assets/img/network/tcp_fsm.png)

## TCP ACK generation details

The TCP receiver can wait up to 500ms (timeout value) for another segment if an in-order segment with expected sequence number and no pending ACK segments.

![](/notes-blog/assets/img/network/tcp_ack_gen.png)

TCP time out value must be minimally longer than RTT.

- Too short: unnecessary retransmits
- Too long: slow reaction to segment loss.
- Estimation of RTT: $(1-\alpha)\ \cdot \text{EstRTT} + \alpha \cdot \text{LastSampleRTT}$
  - Exponential weighted moving average, $\alpha \approx 1/8$.
- Estimation of Deviation from RTT (DevRTT): $(1-\beta) \cdot \text{EstRTT} + 4 \cdot \text{DevRTT}$
  - Typically $\beta \approx 1/4$
- Timeout = EstRTT + 4DevRTT

## Fast retransmit

If a segment is lost, there will be many duplicate ACKs from the pipeline (since last unACKed). 

Hence if 4 ACKs with same ACKnum, resend oldest unacked segment, so no need to wait for timeout.

## Out of order packets

![](/notes-blog/assets/img/network/out_of_order.png)

Chance of wraparound and old packet using a consecutive ACK is received is less with a bigger range of sequence numbers.