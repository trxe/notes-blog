---
layout: default
title: Computer Networks
permalink: /network
---

# Computer Networks

## Table of Contents

1. [Introduction](/notes-blog/network/ch1)
2. [Application Layer](/notes-blog/network/ch2)
3. [Socket Programming](/notes-blog/network/ch3)
4. [UDP Reliable Protocol](/notes-blog/network/ch4)
5. [TCP](/notes-blog/network/ch5)
6. [IP Addressing](/notes-blog/network/ch6)
7. [IP, Routing, ICMP](/notes-blog/network/ch7)
8. [Link Layer](/notes-blog/network/ch8)
9. [ARP, Ethernet and Switched LANs](/notes-blog/network/ch9)
10. [Multimedia Networking](/notes-blog/network/ch10)
11. [Network Security](/notes-blog/network/ch11)

## Web request summary

Visiting google.com:

1. [**DHCP request**, over **UDP port 68**] Startup PC. PC needs IP Address.
   1. DHCP process
     - PC broadcasts a DHCP discover message
     - Server broadcasts back a DHCP offer (with address under `yiaddr`)
     - PC broadcasts a DHCP request with attached `yiaddr`
     - Server broadcassts back a DHCP ack.
   2. DHCP **req** $\subset$ UDP **segment** $\subset$ IP **datagram** $\subset$ Ethernet **frame**
   3. Intermediate switches and first hop router and DHCP server learn of PC
     - `MAC address, Switch port` added to **switch table**
     - `IP address, MAC address, TTL` added to **ARP tables** in the router and server
2. [**DNS query**] DHCP server also returns the  IP of the **first hop router** and **local DNS server**
   1. Browser to request for IP of google.com from DNS server
     - [**ARP query**] To find MAC of local DNS server, PC broadcasts ARP query with blank MAC
     - Local DNS server replies to PC with its MAC
     - PC adds local DNS server to ARP table, and sends DNS query
   2. Local DNS server sends **iterated/recursive query** to reach authoritative DNS server.
3. [**HTTP request**, over **TCP port 80**] PC sends HTTP request to Google.
   1. [**TCP 3-way handshake**] 
      1. PC chooses init seq number $x$, SYNBit set to 1.
      2. Welcome socket at Google's Server accepts a TCP connection and creates a new connection socket for PC
      3. Server chooses init seq number $y$ and returns ack bit $x+1$, SYNbit anad ACKbit set to 1.
      4. PC sends back ack bit $y+1$ with ACKbit set to 1.
      5. Connection established.
   2. PC's Private IP is translated by NUS NAT router, added to **NAT table**
   3. IP datagram is routed with RIP etc.
4. At server neogtiation for secure connection
   1. Digital certificate of Facebook is verified.