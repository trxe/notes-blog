---
layout: default
usemathjax: true
permalink: /network/ch9
---


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

Only for current neighbours (when computer receives frame from other coms)
IP addr, MAC addr, TTL

# Sending Frame in Same Subnet

Case 1: MAC adrress known
Case 2: Not known
- ARP Query packet
- B's IP address
- Dest MAC set to all 1
- all nodes will get this query packet, but only B will have a match of IP address and reply with MAC address
- A caches this mapping for TTL

# Sending Frame in Another Subnet

A should create link layer frame with 
- dest MAC: router interface MAC
- dest IP: B's IP  address

Router interface then receives and passes on to the next interface.

How do you know you are sending to a different subnet?
- If subnet doesn't have the same IP prefix