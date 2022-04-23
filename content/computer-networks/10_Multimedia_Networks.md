---
layout: default
usemathjax: true
permalink: /network/ch10
---

# Multimedia Networking

Audio/Video is predominant on the internet!

**OTT streaming**: Media service is offered directly to viewers 
without cable/broadcast/satellite TV platforms (e.g. through set-top box).

## Digitizing Audio

Analog audio will be sampled at constant rate (Hz: samples/sec)
- Phone: 8kHz sampling
- CD: 44.1kHz sampling

Sample then **quantized** in 2 dimensions:
- $2^8 = 256$ possible quantized values achievable with **8 bits**
- $2^{16} = 65536$ for a CD (16 bits)

Bit rate = samples per sec * quantum size

ADC: analog-to-digital converter

DAC: digital-to-analog converter

### Nyquist-Shannon sampling theorem

The sampling frequency must be at least double the analog's highest signal frequency (via fast fourier decomposition).

$$
f_s \ge 2d
$$

Lossy compression

## Digital video

Images at constant rate (frames/sec)
- each image is an array of pixels
- each pixel is a sequence of bits
- Encoding: use redundancy 
  - Spatial redunandcy (compression within the image)
  - Temporary redundancy (compression bby removing inbetween images)

CBR (Constant bit rate): Video encoding rate fixed
VBR (Variable bit rate): Encoding rate changes with amount of spatial/temporal coding.

**Spatial** coding:
- e.g. for region with many pixels of the same color: just have pixel color * N (number of repeated values)

**Temporal** coding:
- e.g. instead of sending complete frame at $i+1$, send only different frames from frame $i$.
- Motion compensation: was there any motion between 2 frames? then just shift the pixels by this amount, and only record those pixels that changed and not just moved.

### Multimedia transfer types

1. Streaming stored audio from server
2. Conversational (bidirectional live)
3. Streaming (unidirectional live)

![](/notes-blog/assets/img/network/video_streaming.png)

**Challenges**:
1. continuous playout constraints: 
   - the client playout begins, 
   - each frame of the playback must have the same timestamp as the origin
   - however **network delays jitter** (are variable) so we need a client-side buffer to match playout requirements.
   - video is buffered before passed to the decoder.
2. interactivity (pause, fast-forward, rewind, jump through)
3. lost/retransmitted video packets

![](/notes-blog/assets/img/network/real_video_streaming.png)

Hence in real world scenario, the player cannot immediately start 
playing upon receiving the first frame (need to wait for enough
frames in the buffer for continuous play first -- **client playout delay**).

### Client side buffering

1. Initial fill of buffer until playout starts at $t_p$
2. Playout begins at $t_p$
3. Buffer fill level varies wrt time  as fill rate $x(t)$ varies and $r$ (playout rate) is constant.

![](/notes-blog/assets/img/network/client_side_video_buffer.png)

- $x(t)$: instantaneous fill rate
- $\bar{x}$: average fill rate
- $r$: playout rate

Underflow: Buffer will run out as $\bar{x} < r$ 
(video freeze until buffer fills again)

Overflow: Buffer  won't empty, provided initial playout delay is large enough to absorb variability in $x(t)$.

**Initial playout delay tradeoff**: Buffer starvation less likely, 
but larger delay until player starts watching.

## Streaming multimedia

### UDP

**Push based streaming**: Server sends small packets of media at a rate such that 

$$
x_\text{send} = x_\text{encoding} = k
$$

Error recovery: application-level and time-permitting. 
Includes short playout delay to remove jitter.

**Transmission rate** is fixed by server, oblivious to congestion.

However end-to-end delay, playout delay can be very short.

Real Time Protocol (RTP): Multimedia payload types

UDP may not pass through firewalls.

### HTTP

**Pull based streaming**: Client pulls a multimedia file via HTTP GET.

Server prepared "pieces" of media (segments). 
Announces to Client to retrieve when needed.

Has 3 buffers: send buffer, receive buffer, application playout buffer.

Fill rate fluctuates due to TCP congestion control (send/receive buffer).

Needs larger playout delay as a result.

![](/notes-blog/assets/img/network/http_streaming.png)

## VoIP

Requirement: Conversational aspect:
- Higher delays noticeable
- $< 150$ ms good, $> 400$ ms bad

Talk spurts + Silent periods
- 64kbps sampling during talk spurt
- pkts only generated through talk spurts, silent periods ignore
- 20ms chunks, 64kbps $\Rightarrow$ each chunk is 160 **bytes**

App layer header on each chunk.
Chunk + header encapsulated in UDP/TCP segment.

App sends segment into socket per 20ms.

 ## Two types of packet loss

1. Network loss: datagram lost due to network congestion
2. Delay loss: datagram arrives after scheduled playout time
  - Due to processing/queueing/end-system delays
  - Max tolerable delay: 400ms

Loss can be tolerated to some extent depending on encoding and concealment.

### Playout delay

**Fixed playout delay** $q$: 
- Play chunk with timestamp $t$ at time $t+q$.
- Large $q$: Less packet loss, less interactivity
- Small $q$: More interactivity, higher loss rate

**Adaptive playout delay**: To achieve low playout delay AND low late loss rate.
- estimate network delay
- adjust playout delay at start of each talk spurt
- **silent periods** compressed/elongated
- talk spurt chunks still played per 20ms

Exponentially weighted moving average (EWMA): used to estimate required delay.
- $\alpha$: **adjustable weight constant**
- $d_i$: delay estimate after $i$th packet
- $d_{i-1}$: delay estimate history
- $r_i - t_i$: time received - time sent(timestamp)

Delay component 1 (average delay):

$$
d_i = (1 - \alpha)d_{i-1} + \alpha(r_i - t_i)
$$

EWMA to compute standard deviation as well with weight $\beta$.

Delay component 2 (standard deviation):

$$
v_i = (1 - \beta)v_{i-1} + \beta |r_i - t_i - d_i|
$$

When should I playout? Dependson $\%$ threshold of packets you must send successfully. 
If I want confidence of 99%, then i set my delay to 3 or 4 SD $v_i$
above average.

playout time = $t_i + d_i + Kv_i$

![](/notes-blog/assets/img/network/playout_delay.png)

### Recovery from packet loss

1. ACK/NAK (each takes one RTT)
2. Forward error correction (FEC): sending extra bits to recover without retransmission
   1. Simple FEC
   2. Piggybacking FEC
   3. Interleaving FEC

**Simple FEC**:
- For each group of $n$ chunks, create one redundant chunk by XORing n original chunks
- Send $n+1$ chunks, increase bandwidth by $1/n$.
- Reconstruct original $n$ chunks if at most one chunk is lost.
  - XOR of other chunks with this one.

**Piggybacking FEC**:
-  Send lower resolution audio stream as redundant information
-  Nominal stream PCM at 64kbps, **redundant stream** at 13kbps

**Interleaving FEC**:
- Scramble at server
- Send to client
- Unscramble at client
- Any packet loss is averaged out.

![](/notes-blog/assets/img/network/interleaving.png)

# Real Time Protocol (RTP)

Runs over UDP (in UDP segments). Each packet contains
- payload type identification
- packet sequence numbering
- timestamp

Does **not** provide QoS guarantees:
- encapsulation ony seen at end systems
- best-effort service

Example:
- sending 64kbps PCM-encoded voice over RTP
- Every 20ms, 160 bytes of encoded data is collected as chunk
- audio chunk + RTP header = RTP packet $\subset$ UDP seg
  - RTP header contains info of audio encoding, sequence numbers, timestamps

Suite:
- RTP -- UDP, Sender to receiver
- RTCP (Control) -- UDP, Bidirectional
- RTSP (Streaming) -- TCP (port 554), Receiver to sender
  - defines control sequences useful in controlling multimedia playback.

Above transport layer, below application layer, considered transport layer.

Advantage:
- short end-to-end latency (100 to 500ms)

Challenges:
- Requires special-purpose server for media, e.g. fine-grained packet scheduling, keep state
- firewall issues (TCP/UDP)
- No web-caching

## DASH

Divide media into streamlets, which will be
transcoded into different quality.

.mpd file describes available videos and qualities.

Client executes ABR (adaptive bitrate algorithm) to determine which segment to next download.

Advantage:
- simple server (no state, scalable)
- no firewall issues
- image web-caching usable for video

Disadvantage:
- Based on media segment transmissions (long, 2-10s)
- No low-latency due to buffering on both sides

Transport layer protocol: **TCP**
- No heavy real time requirements (not bidirectional communication)
- Allows for retransmission of lost packets to prevent corrupted video data from being played.

## WebRTC

Transport layer protocol: **UDP**
- Real time bidirectional communication
- Too much delay significantly would significantly impact call quality
  - Retransmission worses delay
- Accpetability of packet loss
