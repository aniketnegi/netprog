# NOTES

## 2.0 What is a socket?

- `socket`s are a way to speak to other programs using standard Unix file descriptor.
- When Unix programs do any sort of I/O, they do it by reading or writing to a file descriptor. A file descriptor is simply an integer associated with an open file.
- calling the `socket()` system routine returns a socket file descriptor. using `send()` & `recv()` we can communicate with the fd.

### 2.1 Two types of Internet Sockets

- lookup "[Raw Sockets](https://www.man7.org/linux/man-pages/man7/raw.7.html)", they are very powerful.
- Two main types of sockets are - "Stream Sockets" & "Datagram Sockets"(connectionless sockets)
- commonly referred to as `SOCK_STREAM` & `SOCK_DGRAM`
- "Stream Sockets" - reliable two-way connected communication streams. Items arrive in order. ERROR-FREE.
    - stream sockets are used by *telnet* and *ssh*. also used by HTTP to fetch pages. 
    - if you telnet to a web site on port 80, and type “GET / HTTP/1.0” and hit RETURN twice, it’ll dump the HTML back at you!!! (tested this and it works :>)
    - to achieve such high level of data transmission quality, TCP (Transmission Control Protocol) is used. ([RFC 793](https://datatracker.ietf.org/doc/html/rfc793) has detailed info on TCP).
    - “TCP/IP” where “IP” stands for “Internet Protocol” (see [RFC 791](https://datatracker.ietf.org/doc/html/rfc791)). IP deals primarily with Internet routing and is not generally responsible for data integrity.
- "Datagram Sockets" -  if you send a datagram, it may arrive. It may arrive out of order. If it arrives, the data within the packet will be error-free
    - use IP for routing. Instead of TCP, they use UDP (User Datagram Protocol) ([RFC 768](https://datatracker.ietf.org/doc/html/rfc768))
    - connectionless? -  you don’t have to maintain an open connection as you do with stream sockets. You just build a packet, slap an IP header on it with destination information, and send it out. No connection needed.
    - generally used when TCP is not available or a few dropped packets don't matter - like a video call, *tftp*(trivial file transfer protocol), *dhcpcd*(a DHCP client), multiplayer games, streaming audio & video.
    - BUT...BUT...BUT *tftp* abd *dhcp* are used to transfer important data from one host to another! how do dropped packets not matter then??!
        - tftp has its own protocol on top of UDP. the tftp protocol says that for each packet, the recipient has to send back a packet that says, “I got it!” (an “ACK” packet). If the sender gets no reply in "x" amount of time, he’ll re-transmit until he finally gets an ACK. 
        - This acknowledgment procedure is very important when implementing reliable SOCK_DGRAM applications.
        - For unreliable applications like games, audio, or video, you just ignore the dropped packets, or perhaps try to cleverly compensate for them.
    - why use an unreliable data transfer protocol? SPEED
    -  It’s way faster to fire-and-forget than it is to keep track of what has arrived safely and make sure it’s in order and all that.
    - TCP is good for applications like chats, but is useless if data being sent to too large too fast, and all the data integrity stuff just can't keep up.

### 2.2 Low Level Nonsense and Network Theory

- 

