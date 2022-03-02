---
layout: default
usemathjax: true
permalink: /network/ch2
---

# 2 -- Application Layer

Network applications

- run on end systems/hosts
- communicate over network
- no need to write software for network-core devices

Network core doesn't run user applications.

## App architectures

### Client-server architecture

- Server:
  - Always-on host
  - Permanent IP address
  - Scalability: Must have data center (with thousands of servers)
- Client:
  - Dynamic IP address (different access networks)
  - Communicate with server to get service
  - Possibly intermittent connection

### P2P architecture

- No always-on server
- Arbitrary end systems directly communicate
- Scalability: New peers bring service capacity (seeds), but also more service demands (leech)
  - Self-scalability
- Peers are intermittedly connected and often change IP addresses (who?) 
  - complex management

## Process communication

**Process**: program running in host. 

- **Server** process: Process that **waits** to be contacted
- **Client** process: Process that **initiates** the communication

(P2P architectures must have both in each peer!)

Inter-process communication (IPC) across different hosts by **exchanging messages** (data packets @ application layer)

### Sockets 

A local process sends/receive messages to/from its socket (the **interface** between application and the transport layers, i.e., between localhost and entire internet)

*Note: Application  layer (upper layer) utilizes the transport layer (below) for its transport service.*

Analogy: the door between your localhost and "outdoors" outside of your localhost. It is an **abstraction** (like an API).

### Addressing processes

Motivation:

- To receive messages, process must have **identifier**
- The IP address is not precise enough to idenitify individual processes in the hosts of this access network.

The identifier includes **IP address** and **port numbers**.

- HTTP server: 80
- Email: 25

Miscellaneous:

- Port number advantage over PID
  - Consistent port numbers for well known services (HTTP, email) whereas PID is always varying
- Socket $\neq$ Port: befoire u use a socket you need to bind it with a port number
- Can 2 processes listen on the same port number? (in other words, multiple applications binding to a port)
  - In practice yes, but is quite complicated and implementation dependent (to ensure consistency etc).

## Transport service requirements

Different services have different requirement dimensions:

- data integrity
  - limited loss toleration
  - some 100% reliable data transfer
- timing (delay/latency)
  - Low delay to be effective
- Throughput
  - minimum amount to be effective (e.g. real time gaming)
  - others can make do with whatever amt they get
- security
  - encryption, data integrity...

## Transport layer services

Only 2 types of services provided by transport layer.

### TCP

- Reliable (whatever source sends, destination **will** receive)
- Flow control 
- Congestion control 
- Nothing else except data integrity
- Connection 

### UDP

- Unreliable data transfer
- Does not provide anything!
- Cheaper (overhead, complexity etc. hence shorter delay)
  - Multimedia

### App-layer protocol

- Open protocols
  - defined in RFC (Request for Comment) open documents
  - Allow interoperability
  - If you don't like browser A, you can write browser B because share same protocol
  - e.g. HTTP, SMTP
- Proprietary
  - e.g. Skype

### Web and HTTP

HTTP: HyperText Transfer Protocol uses **TCP**

- Web page contains objets
  - HTML File,
  - JPEG image,
  - etc.
- https://www.versluis.com/2020/10/spawn-characters-ue4/
  - Host name: https://www.versluis.com
  - Path name: <u>/2020/10/spawn-characters-ue4/</u>

**Process**:

1. Client initiates TCP to server (creating a socket on port 80)
2. Server accepts TCP connection from client
3. HTTP messages exchanged between client and server
4. Close TCP connection

Features of HTTP:

- HTTP is stateless
  - Retains no info about past client request (e.g. IP address and message contents)
  - Protocols that are stateful are complex (TCP)
    - Complexity: if server/client crashes, "state" may become inconsistent and must reconcile
- Persistency (Present and absent)
  - Non-persistent
    - At most 1 object sent over TCP connection, and immediately closed
    - Requires 2 RTT per object (1 to initiate TCP, 1 to request & transmit file)
    - OS overhead for each TCP
    - Often browsers open parallel TCP connections
  - Persistent
    - Multiple objects sent over TCP, connection is not closed immediately (expecting more) after sending until a timeout. 
    - As little as 1 RTT for all referenced objects of delay (due to one TCP shared)

Roundtrip Time (RTT): Time for a small packet to travel from client to server and back (2x end-to-end delay)

![Client-server HTTP request-response](/notes-blog/assets/img/network/client_server_http.png)

### HTTP messages

#### Request

1. Request line (`GET`, `POST`, `HEAD` etc. commands)
2. Header lines (Header field + Corresponding value)
3. Carriage return
4. Entity body for `POST`, `PUT` commands etc.

```http
GET /Panopto/Pages/Viewer.aspx?id=956d6d43-941c-45a0-8c3e-ae230083e49c HTTP/2\r\n
Host: mediaweb.ap.panopto.com\r\n
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:96.0) Gecko/20100101 Firefox/96.0\r\n
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8\r\n
Accept-Language: en-GB,en;q=0.5\r\n
Accept-Encoding: gzip, deflate, br\r\n
Referer: https://mediaweb.ap.panopto.com/Panopto/Pages/Sessions/List.aspx?embedded=1\r\n
Connection: keep-alive\r\n
Cookie: ...
\r\n

{...} (entity body)
```

Notes:

- `Connection: keep-alive`: Persistent TCP 
- Uploading form input: via entity body (`POST` method), or via URL field `(/.../...search?foo&bar)`

#### Response

1. Status line (status code)
2. Header lines
3. Carriage return
4. Response data

```http
HTTP/2 200 OK\r\n
date: Fri, 21 Jan 2022 01:17:02 GMT\r\n
content-type: text/html; charset=utf-8\r\n
content-length: 32750\r\n
cache-control: private, no-store\r\n
content-encoding: gzip\r\n
vary: Accept-Encoding\r\n
\r\n

{...} (data)
```

### Cookies

Main method of saving state (despite statelessness of HTTP).

1. Cookie header line of response message (from server to unique end system)
2. Cookie header line of end system's next HTTP request
3. End system's local storage of cookie file
4. Backend database of the website storing the cookies

![Cookies](/notes-blog/assets/img/network/cookie.png)

| `set-cookie`                                                 | `cookie`                                                  |
| ------------------------------------------------------------ | --------------------------------------------------------- |
| ![set-cookie](/notes-blog/assets/img/network/set-cookie.png) | ![cookie](/notes-blog/assets/img/network/send-cookie.png) |

## Web-caches

Satisfy client request without involving origin server. 

When client makes a request:

- First goes through a **proxy** server
- Checks if the content is cached there
- If cached and updated: Return from cache
- Else:  Request object from **origin** server and returns the object to client

Advantages:

- Usually faster as proxies are closer to end-systems.
- Prevent overloading a server for a highly requested resource, by distributing service over many proxy servers.

### Conditional GET

1. Client process $\rightarrow$​ Server process: `if-modified-since: <date>`
2. Server process $\rightarrow$​ Client process: If cached copy is not modified since, `HTTP/1.0 304 Not modified` otherwise contains most updated data.

No object transmission delay, less link utilization.

## Domain name system (DNS)

**Host identifiers**: IP address (32 bit), name (human readable string). These two must be mapped to each other uniquely.

Distributed storage of this mapping

### DNS

![DNS](/notes-blog/assets/img/network/dns_tree.png)

> A **network domain** is an administrative grouping of multiple private [computer networks](https://en.wikipedia.org/wiki/Computer_network) or local hosts within the same infrastructure.
>
> -- Wikipedia (https://en.wikipedia.org/wiki/Network_domain)

Client searches for www.amazon.com. 

1. Client queries root server for .com DNS server
2. Client queries .com DNS server for amazon.com DNS server.
3. Client queries amazon.com DNS server for www.amazon.com

Levels of DNS Servers

- Root name servers (13 logical root name servers, what does logical mean here?)
- Top-level domain servers (.com, .net, .edu, country TLDs)
- Authoritative DNS server (the organization's own DNS server)

### Local DNS Server

1. Host makes DNS query
2. Query first sent to **local** DNS server
3. Search local cache of name-address pairs
4. Forwards query (as a proxy) into the 

Does not belong to the hierarchy of distributed database that stores mapping permanently.

### Root DNS Server

#### Iterated query

Local DNS server receives the next server to contact from the previous server, and calls them one by one until the authoritative DNS is found.

![Iterated Query](/notes-blog/assets/img/network/iterated_query.png)

#### Recursive query

Local DNS server requests for the authoritative DNS from the root DNS server, where if it isn't cached, requests from the TLD, which if it isn't cached, it requests from the next level domain etc. and passes the information back to the local DNS server.

![Recursive Query](/notes-blog/assets/img/network/recursive_query.png)

This is problematic:

- Every request has to pass through the root server, which may overload the root server.
- In the iterated structure, the responsibility of each layer is a lot clearer.

#### Caches

Satisfies client request without involving origin server.

- Page cached: If not expired, return cache
- Otherwise: Request cache from origin server and return to client.

Once (any) name server learns mapping, it caches mapping, but mappings **expire** after some Time To Live (TTL).

Cached entries might be out of date. (If a named host changes IP address, this won't be universally known until all TTLs expire.)

Response failure?

