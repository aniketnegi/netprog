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

- **Data Encapsulation** - a packet is born (data) -> it is wrapped in a header by the first protocol (say, *tftp*) -> the whole thing is encapsulated again by the next protocol (say, *UDP*) -> then the next (*IP*) -> then finally by the protocol on the hardware layer (*ethernet*)
    - when another component receives the packet, it is stripped one by one - the hw strips the *ethernet* header -> the kernel strips the *IP* and *UDP* headers -> the *tftp* program strips the *tftp* headers and finally the data is obtained.
- **Layered Network Model** (ISO / OSI) - some advantages over other models ([ref](https://utsa.pressbooks.pub/networking/chapter/overview-of-network-models/)). socket programs can be written in the same way without caring about how the data will be transmitted physically, cause the lower level programs will take care of that.
- following are the layers of a full-blown network model:
    - Application
    - Presentation
    - Session
    - Transport
    - Network
    - Data Link
    - Physical
- The Physical Layer is the hardware (serial, Ethernet, etc.). The Application Layer is the place where users interact with the network.
- A layered model more consistent with Unix might be:
    - Application Layer (telnet, ftp, etc.)
    - Host-to-Host Transport Layer (TCP, UDP)
    - Internet Layer (IP and routing)
    - Network Access Layer (Ethernet, wi-fi, or whatever)
- further study on routing and the *IP* layer - [RFC 791](https://datatracker.ietf.org/doc/html/rfc791)

## 3.0 IP Addresses, structs & Data Munging

### 3.1 IPv6 and IPv6

- old network routing system called The Internet Protocol Version 4, also called IPv4. It had addresses made up of four bytes (A.K.A. four “octets”), and was commonly written in “dots and numbers” form, like so: `192.0.2.111`
- as devices grew, people were afraid that we'll run out of IPv4 addresses. in the beginning, when there were only a few computers and everyone thought a billion was an impossibly large number, some big organizations were generously allocated millions of IP addresses for their own use. (Such as Xerox, MIT, Ford, HP, IBM, GE, AT&T, and some little company called Apple, to name a few.)
- so IPv6 was born, much to the pleasure of Vint Cerf.
- IPv6 has 128 bits; that about  340 trillion trillion trillion addresses. Overcompensation imho.
- IPv6 is represented as hex, with every 2 byte chunk separated by a colon - `2001:0db8:c9d2:aee5:73e3:934a:a5ae:9551`
- IP address with lots of zeros in it can be compressed them. you can leave off leading zeros for each byte pair. For instance, each of these pairs of addresses are equivalent:
    - `2001:0db8:c9d2:0012:0000:0000:0000:0051` == `2001:db8:c9d2:12::51`
    - `2001:0db8:ab00:0000:0000:0000:0000:0000` == `2001:db8:ab00::`
    - `0000:0000:0000:0000:0000:0000:0000:0001` == `::1`
- `::1` is the loopback address. It means "current machine". In IPv4, the loopback address is `127.0.0.1`
-  IPv4-compatibility mode for IPv6 addresses: `192.0.2.33` == `::ffff:192.0.2.33`
-  Creators of IPv6 have quite cavalierly lopped off trillions and trillions of addresses for reserved use.

#### 3.1.1 subnets
