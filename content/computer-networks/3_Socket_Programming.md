# 3 -- Socket Programming

**Socket** is the interface between the application and transport layer.

**Port number** is a 16-bit integer (0 to 65535) identifier for a socket with IP address. *(Note: 0 to 1023 are reserved for standard use. (HTTP service, DNS service etc.)*

**IP address** is an identifier, not for a host but for a network interface. (TBC in IP layer)

![Client-Server communication](/notes-blog/assets/img/network/transport_layer.png)

2 socket types:

- UDP: Unreliable datagram, connectionless
- TCP: reliable byte stream, connection-oriented

## UDP

No connection:

- No handshaking, but every packet needs to include address information
- Sender (*client*) explicitly attaches IP **destination (server)** address and port number on every packet sent
- Receiver (*server*) extracts sender IP and port number from received packet
- Data may be lost, or received out of order.
  - Unreliable groups of bytes (datagrams)

*Note: The term datagrams is used to refer to data at the IP layer, in transport layer it's called segments, in application layer, messages. Different terminologies at different layers.*

![UDP](/notes-blog/assets/img/network/udp.png)

### High level program

![High level UDP](/notes-blog/assets/img/network/udphll.png)

### Client code

```python
from socket import *
serverName = 'hostname'
serverPort = 12000

# automatically creates random feasible port number for you here.
clientSocket = socket(AF_INET, SOCK_DGRAM)
message = input("input message: ")
# attach server name, port to header and send bytes to server
clientSocket.sendTo(message.encode(), (serverName, serverPort))
# 2048 is the upper limit on the size of the response/buffer size.
response, serverAddress = clientSocket.recvfrom(2048) 

print(response.decode())
clientSocket.close()
```

`AF_INET` is the address family for IPv4, `SOCK_DGRAM` to specify that we are creating a datagram socket for UDP, not TCP.

Does server not require client info (port number, IP)? No it does! The port number is generated in `socket(AF_INET, SOCK_DGRAM)`

### Server code

```python
from socket import *
serverPort = 12000

serverSocket = socket(AF_INET, SOCK_DGRAM)
# bind socket to localhost "", local port number 12000
serverSocket.bind(("", serverPort))

while True:
    # clientAddress = (IP address, port number)
    msg, clientAddress = serverSocket.recvfrom(2048)
    modifiedMessage = message.decode().upper()
    serverSocket.sendto(modifiedMessage.encode(), clientAddress)
```

*Note: Binding the port number to the socket is "assigning" anyone who sends a packet to 12000 to this socket. This ensures a fixed port number for senders to request from.* 

UDP server enters a while loop to receive and process packets. This omits a lot of the burdens that are on the developer to handle:

- order of the data to be read then, as there are a lot of processes reading from the same socket
- concurrency
- preventing starvation

DNS servers use UDP to send/received requests for IP addresses.

## TCP

For each client:

- after it sends the initial request through server's **welcoming socket**
- server must create **a unique connection socket** to welcome that client's contact
  - server uses the source port numbers (each unique socket's) to distinguish client.
- the client creates TCP socket, specifies IP address, port number of server: creating the socket establishes the connection to TCP server
- data in reliable, in-order byte-stream transfer

![TCP](/notes-blog/assets/img/network/tcp.png)

The welcoming socket is not the same as the socket for subsequent communicating with the client. Hence there is a three-way handshake.

![TCP sockets](/notes-blog/assets/img/network/tcp_sockets.png)

### Client code

```python
from socket import *
serverName = 'hostname'
serverPort = 12000

# SOCK_STREAM for stream connection instead of datagrams
clientSocket = socket(AF_INET, SOCK_STREAM)
# connect to establish TCP connection
clientSocket.connect((serverName, serverPort))
message = input("input message: ")
# send won't need to attach server name/port
clientSocket.send(message.encode())
response, serverAddress = clientSocket.recvfrom(2048) 

print(response.decode())
clientSocket.close()
```

### Server code

```python
from socket import *
serverPort = 12000

serverSocket = socket(AF_INET, SOCK_STREAM)
serverSocket.bind(("", serverPort))
# the welcoming/serverSocket listens for TCP requests
serverSocket.listen(1)

while True:
    # server blocks on accept until incoming request is received
    # a connectionSocket will be created.
    connectionSocket, addr = serverSocket.accept()
    message = connectionSocket.recvfrom(2048).decode()
    connectionSocket.send(message.upper().encode())
	connectionSocket.close()
```



