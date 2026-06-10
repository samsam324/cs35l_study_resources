# Networking, Client-Server & Protocols (CS35L Lecture 5)

Source: Lecture 5 -- Client Server & Node.js (Tobias Durschmid).
Scope: This guide covers ONLY the networking / client-server / protocols content from Lecture 5. JavaScript, Node.js, Express, npm, HTML, JSON content lives in `nodejs-javascript.md`.

---

## 1. Client-Server Architecture

Client-Server Architectures **define two roles**:

- **Client** -- the *consumer* of resources.
- **Server** -- the *provider* of resources.

Communication pattern:

```
Client (consumer) --- Request  ---> Server (provider)
Client (consumer) <-- Response --- Server (provider)
Client
Client
```

Key properties:

- **Multiple clients can use the same server.**
- **Connections are initiated by the client, not the server.**
- **Centralized architecture** -- all communication flows via the server.

The client sends a **Request** to the server. The server replies with a **Response**.

---

## 2. Peer-to-Peer (P2P) Architecture

Alternative architecture: in **Peer-to-Peer Architectures (P2P)** clients directly interact with other clients.

- **Decentralized Architecture.**
- **Peers are equally privileged participants in the network.**
- **Peers are both suppliers and consumers of resources.**
- In practice, there are more **hybrid architectures** than pure P2P architectures. Hybrid means that some communication happens via client-server, and some communication happens via P2P.

Diagram (fully-connected mesh of peers):

```
Peer --- Peer
  X   X
Peer --- Peer
  X   X
Peer --- Peer
```

(All peers connect to all other peers; lines cross.)

### Worked Example: Zoom -- Client-Server or P2P?

> "Would you implement Zoom via Client-Server or via P2P?"

It's **hybrid, starting with client-server**. For **video & audio of 1-on-1 calls**, Zoom **attempts peer to peer communication** and **uses client server as fallback**.

---

## 3. Throughput vs. Latency (Quality Attributes for Client-Server Systems)

- **Throughput** measures the **volume of work, data, or messages** processed by a system, network, or process within a specific period of time.
  - *Example:* "the system processes 100 requests per second."
- **Latency / Response time** measures the **time it takes for a single request to receive a reply**.
  - *Example:* "the average response time is 10ms."
- **Latency can be improved** by making the implementation **more efficient**.
- **Throughput can be improved** by **improving latency** and/or **duplicating servers** so that more requests can be processed at the same time.

---

## 4. How Do Requests Reach the Server?

Conceptually: a client (e.g., a computer on Wi-Fi behind a router) sends a request, which travels through the internet (a "cloud" of many interconnected servers/routers), and eventually reaches the destination server (e.g., a server rack).

(Lecture image: client computer + Wi-Fi router with a package -> cloud full of server racks -> destination server rack with a package.)

---

## 5. The TCP/IP Stack (4 Layers)

The TCP/IP stack is a **layered architecture**. Layers, from highest to lowest:

| Layer | Example Protocols | Purpose |
|-------|-------------------|---------|
| **Application Layer** | HTTP, HTTPS, SSH, DNS, POP, TLS/SSL (also FTP) | Provides **an interface for applications to access network services** and handles actual data exchange between applications. |
| **Transport Layer** | TCP, UDP | Provides **end-to-end communication between applications** running on different hosts. |
| **Internet Layer** | IPv4, IPv6 | Enables communication between networks through **addressing** and **routing data packets**. |
| **Link Layer** | Ethernet, Wifi, MAC | Handles the **physical transmission of data** over underlying local network hardware. |

Key idea: **Messages from higher layer protocols are wrapped in messages from lower layer protocols.** (Think Russian nesting dolls / Matryoshka.)

### Layered Architecture Properties (revisited)

- TCP/IP is an instance of a **layered architecture** in which higher-level layers use only lower-level layers.
- This increases **reusability** of lower-layer implementations.
- It also allows **flexibility** as you can simply replace one layer with a different implementation to get different results.

---

## 6. Messages: Header + Payload

Each message has a **Header** and a **Payload**.

```
+--------+----------------------------+
| Header |        Payload             |
+--------+----------------------------+
            Message
```

- **Headers contain meta information** -- e.g., destination, origin, content type, index number, checksum, ...
- The **payload** portion contains the **content** of the message.
- **Different protocols define different headers.**

---

## 7. Package Wrapping

- **Higher-layer protocols use the protocols directly below them to send messages.**
- Whenever this happens, the higher-layer **message might get split up**. Then the individual messages get placed in the **payload portion of the lower-layer messages**.

Example for an HTTP message being sent over Ethernet:

```
+-----------+--------+--------+--------+------------------+
| Ethernet  |  IP    |  TCP   |  HTTP  |     Payload      |
| Header    | Header | Header | Header |                  |
+-----------+--------+--------+--------+------------------+
                            Message
```

Reading order from outside to inside: Ethernet -> IP -> TCP -> HTTP -> Payload. Each lower layer wraps the layer above it.

---

## 8. Application Layer Protocols

Your application uses protocols in the **Application Layer**:

- **HTTP/HTTPS** -- to **request files / documents from a server**.
- **FTP** -- to **request static files from a server**.
- **POP/SMTP** -- to **send emails**.
- ... (and others: SSH, DNS, TLS/SSL).

---

## 9. HTTP (Hypertext Transfer Protocol)

- The **foundation of data communication on the World Wide Web**.
- **Stateless protocol**:
  - *Each request is independent.*
  - The server **doesn't remember any information about previous requests** from the same client.

Communication pattern:

```
Client --- HTTP REQUEST  ---> Server (e.g., www.ucla.edu)
Client <-- HTTP RESPONSE ---- Server
```

### 9.1 HTTP Verbs

HTTP Requests can use different **Verbs**:

- **GET**
  - **Requests a resource** (e.g., a web page, data entry, image, file, ...).
  - **Response:** The content of the resource & status code.
- **POST**
  - **Sends data to the server to create or update a resource** (e.g., a direct message, file upload, complex request objects, ...).
  - **Response:** status code.
- **PUT** -- **Updates an existing resource** on the server.
- **DELETE** -- **Deletes a resource** on the server.
- **HEAD** -- **Retrieves only headers of a resource, not body**.

### 9.2 URL (Uniform Resource Locator)

A URL is the **web address of a resource on the internet**.

General form:

```
{protocol}://{domain}(:{port})?(/{resource})?
```

(The `(...)?` parts are optional.)

Examples:

```
http://localhost:8080/users/1
https://myapp.com/about.html
```

Mapping for `http://localhost:8080/users/1`:
- protocol = `http`
- domain = `localhost`
- port = `8080`
- resource = `/users/1`

Mapping for `https://myapp.com/about.html`:
- protocol = `https`
- domain = `myapp.com`
- (no explicit port)
- resource = `/about.html`

### 9.3 Common HTTP Status Codes

- **2xx Successful responses** -- the request was successfully received, understood, and accepted.
  - **200 OK:** The request was successful.
  - **201 Created:** request fulfilled, and new resource created.
- **4xx Client error responses** -- the client's request contains an error.
  - **400 Bad Request:** invalid request (e.g., malformed syntax).
  - **404 Not Found:** The server cannot find the requested resource.
  - Others: **401 Unauthorized**, **403 Forbidden**, ...
- **5xx Server error responses** -- the server failed to fulfill a valid request.
  - **500 Internal Server Error:** A generic error message indicating an unexpected condition on the server.
  - Others: **502 Bad Gateway**, **503 Service Unavailable**, ...

(Status code families also include **1xx informational** and **3xx redirect** -- only 2xx, 4xx, 5xx were elaborated in the lecture.)

Reference: https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status

### 9.4 Common HTTP Header Fields

- **Content-Type** (the media type of the resource)
  - `"text/html; charset=utf-8"` for HTML files in UTF-8 encoding.
  - `"text/plain"` for plain text.
  - `"application/json"` for json files (used for API requests / responses).

Reference: https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Type

(Other header examples in scope: Authorization, etc.)

---

## 10. HTTPS (Hypertext Transfer Protocol Secure)

- Uses **Secure Sockets Layer (SSL)** / **Transport Layer Security (TLS)** to **encrypt communication**.
- **Very important whenever sensitive data is transferred** (e.g., passwords, personally-identifiable information, private data, private messages, ...).
- Has become the **default for all public web pages, even for non-sensitive data**.

---

## 11. Transport Layer: Application Layer Protocols Use Transport Layer Protocols

Application Layer protocols (HTTP, HTTPS, ...) use Transport Layer protocols (**TCP, UDP**) to transmit messages.

(Same wrapping rule applies: the higher-layer message may be split and embedded inside the payload of lower-layer messages.)

---

## 12. UDP (User Datagram Protocol)

UDP "just throws" messages at the receiver.

> *"In UDP, Bob simply delivers packages to Alice's address without waiting for a response."*
>
> Bob (UDP): "Here are some packages for you" -- throws the package through Alice's window.
>
> "The End. Did she receive the packages? We don't know. But it was fast!"

UDP characteristics:

- **Fast.**
- **Connectionless.**
- **Lightweight** communication.
- **Does not guarantee delivery, order, or error checking.**

So UDP is a **simple, unreliable, but fast & high-throughput protocol**.

---

## 13. TCP (Transmission Control Protocol)

TCP is a **more complicated, but reliable** protocol.

It uses an analogy with a TCP delivery person ("Bob") and a recipient ("Alice"). The full lifecycle:

### 13.1 Three-Way Handshake (Connection Establishment)

1. **SYN** -- To **initiate the connection**, Bob sends a **synchronize message (SYN)**.
   - "Hello! I have some packages for you. Would you like them now?"
2. **SYN-ACK** -- Alice **confirms the connection** by sending a **SYN-ACK** message to Bob.
   - "Yes, I'm ready to receive the packages!"
3. **ACK** -- Bob **confirms that the connection has been established** by sending an **ACK (acknowledgment)** message to Alice.
   - "Great. Then I will give you the packages now."

### 13.2 Data Transfer (with per-message ACK)

- **Bob sends data to Alice in multiple ordered messages that contain a checksum** to allow Alice to **detect corrupted data** and **request retransmission** if needed.
- "Here is the first package!" (Data)
- If Alice has received the message and it does not contain errors, **she sends an acknowledgement (ACK)** to Bob ("I received the first package!").
- **If Bob does not receive the ACK until a given timeout, he attempts to send the message again.**
- Bob then sends the next message ("Here is the second package!") and Alice ACKs again.

This continues for every data message.

### 13.3 Connection Termination

- **FIN** -- To notify Alice that **Bob does not have any more messages to send**, he sends a **FIN** message ("That's all I have for you").
- **FIN ACK** -- Alice **acknowledges Bob's message (ACK)**. **She also does not have any more messages to send**, so she sends a **FIN message too** ("Okay. Goodbye").
- **ACK** -- Bob **acknowledges Alice's FIN message with an ACK message** ("Goodbye").

### 13.4 Message Count

> "To achieve reliability, **TCP sends at least 6 + 2N messages** for N messages of data."

Where the **6** breaks down as:
- **3** for connection establishment (SYN, SYN-ACK, ACK).
- **3** for connection termination (FIN, FIN-ACK, ACK).

And the **2N** are the N data messages plus their N ACKs.

Full timeline (TCP vs. UDP, for the same N=2 data messages):

```
Client          TCP            Server          Client     UDP    Server
  | -- SYN --------------------> |               | --Data1--------> |
  | <-- SYN-ACK ---------------- |               | --Data2--------> |
  | -- ACK --------------------> |   <- establishes connection
  | -- Data1 ------------------> |
  | <-- ACK[Data1] ------------- |   <- (Full Duplex) data transfer
  | -- Data2 ------------------> |       (both can send at any time)
  | <-- ACK[Data2] ------------- |
  | -- FIN --------------------> |
  | <-- FIN-ACK ---------------- |   <- terminates the connection
  | -- ACK --------------------> |
```

Notes attached to the diagram:

- **Establishes the Connection** -- to ensure both parties are **ready** to send / receive data.
- **(Full Duplex) Data Transfer** -- both can send messages at any time.
- **Terminates the Connection** -- to **free up resources** on both the client and server systems and to **ensure reliable closure** of the communication session.
- Side comment on the diagram: *"UDP completes the transfer before TCP even sends the first data package."*

### 13.5 TCP vs. UDP Trade-Offs

| | TCP | UDP |
|---|---|---|
| Order of messages | **Preserved** | Messages can arrive in **any order** |
| Error detection | **Included** | **None** |
| Lost messages | **Get re-sent** | **Might get lost** |
| Delay | Protocol adds **additional overhead** | **No additional delay** |

Quick summary:
- TCP: order preserved, error detection included, lost messages re-sent, BUT additional overhead.
- UDP: messages can arrive in any order, no error detection, some messages might get lost, BUT no additional delay.

### 13.6 TCP vs. UDP Use Cases

**TCP works well for:**
- **Text Messaging** (e.g., Slack, Discord).
- **Web Browsing.**
- **File Transfer.**

**UDP works well for:**
- **Live Video Streaming** (e.g., Webcasts).
- **Real-Time Voice Chat** (e.g., Audio Call).

### 13.7 Online Gaming -- Hybrid Use Case

"What about online gaming? Talk to your Neighbor(s)!"

- **TCP side:**
  - **Ideal for round-based games** (e.g., Chess, Go).
  - In hybrid games, TCP is used for **reliable, non-time-sensitive data where loss is fatal** (e.g., chat messages, logging into the server, inventory transactions, scoring).
- **UDP side:**
  - **High-speed real-time events** (e.g., player positions, physics).
  - **Game updates should not require previous messages**, so they need to include the **absolute positions of all movable objects**.

**Many games use a hybrid approach** (TCP + UDP).

---

## 14. IP Addresses (Internet Layer)

**Challenge: Identification.** How to identify a particular sender/receiver out of the **billions of computers** connected to the internet?

**Solution idea: IP Addresses.** Have a **portion of the address that represents the network** (like a "city") and a **portion that represents the host** (like a "street address").

### 14.1 IPv4

- IPv4 addresses range from `0.0.0.0` to `255.255.255.255`.
- (Implicitly 32-bit -- four 8-bit octets separated by dots.)
- Example addresses from the lecture: `8.8.4.4`, `120.28.1.4`, `10.80.14.4`, `210.82.4.14`.
- There are some **special IP addresses**, e.g., **`127.0.0.1` is "local host"**, which is internet-speak for the literal words *"your machine"*.

### 14.2 IPv6

- After **running out of all IP addresses**, a new standard (**IPv6**) was created.
- (IPv6 uses 128-bit addresses, vastly expanding the address space.)

---

## 15. Historical Note: Inventors of TCP/IP

**TCP/IP was invented by Robert Kahn & Vinton Cerf.** Vinton Cerf went to **UCLA** ("I went to UCLA! Go Bruins").

Further reading on TCP: https://www.geeksforgeeks.org/computer-networks/what-is-transmission-control-protocol-tcp/

---

## 16. Quick Recap Cheat Sheet

- **Client-Server:** centralized, client initiates, multiple clients per server.
- **P2P:** decentralized, peers are equal, both suppliers and consumers; hybrid is more common in practice (e.g., Zoom).
- **Throughput** = volume / time. **Latency** = time per single request. Improve latency by making code efficient; improve throughput by improving latency and/or duplicating servers.
- **TCP/IP stack (top-down):** Application (HTTP, HTTPS, SSH, DNS, POP, TLS/SSL, FTP) -> Transport (TCP, UDP) -> Internet (IPv4, IPv6) -> Link (Ethernet, Wifi, MAC). Higher-layer messages are wrapped inside lower-layer payloads.
- **Message = Header (meta info) + Payload (content).**
- **HTTP:** stateless, foundation of the WWW. Verbs: GET, POST, PUT, DELETE, HEAD. URL = `{protocol}://{domain}(:{port})?(/{resource})?`. Status: 2xx success (200, 201), 4xx client error (400, 401, 403, 404), 5xx server error (500, 502, 503).
- **HTTPS:** HTTP + SSL/TLS encryption; default on modern web.
- **TCP:** reliable, ordered, error-checked, has 3-way handshake (SYN, SYN-ACK, ACK) and FIN-based teardown (FIN, FIN-ACK, ACK); ~6+2N messages for N data messages.
- **UDP:** connectionless, unreliable, fast, no order/error/delivery guarantees.
- **IP:** identifies hosts; IPv4 dotted-decimal 0.0.0.0--255.255.255.255 (special `127.0.0.1` = localhost), IPv6 created after IPv4 exhaustion.


---

# Appendix: Lecture 5 Slides (raw extracted text)

The following is the full extracted text of Lecture 5. Preserved verbatim so no information is lost.

```text
CS 35L Software
Construction
Lecture 5 ­ Client
Server & Node.js
Assistant Teaching Professor
Computer Science Department
ACM ICPC proudly presents:
 CodeSprint LA 2026!
Embark on a quest through the Kingdom of Hyrule! Our Legend of Zelda themed
competitive programming contest features $2,100+ in total prizes, including $750
reserved exclusively for UCLA Beginners.  Team up to solve challenging, out-
of-the-box programming problems and win BIG! The event runs from 9:00 AM to
5:00 PM and includes:
· Morning: Beginner Workshop, Opening Ceremony, and Lunch .
· The Contest: 11:30 AM ­ 4:30 PM
· Evening: Closing Ceremony and Awards .
When: Saturday, May 9th, 2026 Where: De Neve Plaza Room
Format: Teams of up to 3
Register by May 3rd: https://codesprintla.uclaacm.com/register
For the full schedule and more details, visit https://codesprintla.uclaacm.com/
Now this Course is Starting to Get Hard...
This lecture:
· Will give a very short intro to JavaScript & Node.js
· Will teach you the most important concepts of client-server interactions that
  Node.js implements
· Include some examples code snippets of how this is implemented in Node.js
· Will not replace going through a thorough tutorial of JavaScript & Node.js
  (This is not CS31 with JavaScript / Node.js)
We built a tutorial for this. Please go through the tutorial in enough detail for you to
become familiar with Node.js. I recommend to do this before Thursday since we'll
look into React.js & have a React tutorial & React homework then.
If you struggle with JS & Node, please ask on Piazza and/or LA/TA office hours
               CS 35L Software Construction: Lecture 5 ­ Client Server & Node.js          3
               Tobias Dürschmid
CS 35L Software Construction: Lecture 5 ­ Client Server & Node.js  4
Self-Study Tips   to Become a Superhero
· Don't just watch video tutorials. Code yourself!
   · You only learn by going & quizzing yourself afterwards
   · That's why we build the interactive tutorials!
· Feeling the effort in the learning process is what makes it so effective!
· Regularly practice concepts from previous lectures in the SE Gym
We are happy to help you, particularly if you have not had much programming
experience yet. If you need help, please come to us directly
Client Server Architectures Define Two Roles
· Multiple clients can use the same server
· Connections are initiated by the client, not the server
· Centralized architecture (all communication flows via the server)
    Client        Request                                            Server
(consumer of      Response                                         (provider of
                                                                   resources)
 resources)
    Client
Client
Alternative: In Peer-to-Peer Architectures (P2)
Clients Directly Interact with other Clients
· Decentralized Architecture          Peer                         Peer
· Peers are equally privileged        Peer                               Peer
  participants in the network
· Peers are both suppliers and        Peer                         Peer
  consumers of resources
· In practice, there are more hybrid
architectures than pure P2P architectures. Would you implement Zoom via
                                            Client-Server or via P2P?
Hybrid means that communication
happens via client-server, and some   It's hybrid, starting with client-server. For video &
communication happens via P2P         audio of 1-on-1 calls, Zoom attempts peer to peer
                                       commutation and uses client server as fallback
Throughput & Latency are important Quality
Attributes for Client-Server Systems
· Throughput measures the volume of work, data, or messages processed
  by a system, network, or process within a specific period of time
  (e.g., the system processes 100 requests per second)
· Latency / Response time measures the time it takes for a single request
  to receive a reply (e.g., the average response time is 10ms)
· Latency can be improved by making the implementation more efficient
· Throughput can be improved by improving latency and/or duplicating
  servers so that more requests can be processed at the same time
How do Requests reach the Server?
                                    Messages from higher layer protocols are
TCP/IP Stack wrapped in messages from lower layer protocols
 Application Layer              Provides an interface for applications to
                              access network services and handles actual
(e.g., HTTP, HTTPS, SSH,
   DNS, POP,TLS/SSL)                data exchange between applications
  Transport Layer            Provides end-to-end communication between
                                  applications running on different hosts
      (e.g., TCP, UDP)
Internet Layer                  Enables communication between networks
                             through addressing and routing data packets
 (e.g., IPv4, IPv6)
      Link Layer             Handles the physical transmission of data over
                                      underlying local network hardware
(e.g., Ethernet, Wifi, MAC)
Each Message has a Header and a Payload
  Headers contain meta information
  (e.g., destination, origin, content type, index number, checksum, ...)
· The payload portion contains the content of the message
· Different protocols define different headers
                                            Message
Header            Payload
Package Wrapping
· Higher-layer protocols use the protocols     Application Layer
  directly below them to send messages.
                                                (e.g., HTTP, HTTPS)
· Whenever this happens, the higher-layer
                                                Transport Layer
message might get split up.
                                                   (e.g., TCP, UDP)
Then it the individual message
                                                 Internet Layer
get placed in the payload portion
                                                   (e.g., IPv4, IPv6)
of the lower-layer messages
                                                   Link Layer
                                   Message
                                                (e.g., Ethernet, MAC)
Ethernet IP       TCP HTTP
                                            Payload
Header Header Header Header
Your Applications use Protocols in the
Application Layer
· Your application can use:             Application Layer
  · HTTP/HTTPS to request files
    documents a server                   (e.g., HTTP, HTTPS,
  · FTP to request static files from a       SSH, DNS,FTP
    server                                   POP,TLS/SSL)
  · POP/SMTP to send emails
  ·...                                    Transport Layer
                                            (e.g., TCP, UDP)
                                           Internet Layer
                                            (e.g., IPv4, IPv6)
                                             Link Layer
                                         (e.g., Ethernet, MAC)
HTTP (Hypertext Transfer Protocol)
· The foundation of data                             www.ucla.edu
  communication on the World
  Wide Web                        HTTP REQUEST
                                    HTTP RESPONSE
· Stateless protocol
   · Each request is independent
   · the server doesn't remember
     any information about
     previous requests from the
     same client
HTTP Requests can use different Verbs
· GET
   · Requests a resource (e.g., a web page, data entry, image, file, ...)
   · Response: The content of the resource & status code
· POST
   · Sends data to the server to create or update a resource (e.g., a direct
     message, file upload, complex request objects, ...)
   · Response: status code
· PUT: Updates an existing resource on the server
· DELETE: Deletes a resource on the server
· HEAD: Retrieves only headers of a resource, not body
A URL (Uniform Resource Locator) is the web
address of a resource on the internet
         http://localhost:8080/users/1
           {protocol}://{domain}(:{port})?(/{resource})?
         https://myapp.com/about.html
Common HTTP Status Codes
· 2xx Successful responses (the request was successfully received,
  understood, and accepted)
   · 200 OK: The request was successful.
   · 201 Created: request fulfilled, and new resource created
· 4xx Client error responses (client's request contains an error)
   · 400 Bad Request: invalid request (e.g., malformed syntax)
   · 404 Not Found: The server cannot find the requested resource.
   · Others: 401 Unauthorized, 403 Forbidden, ...
Read more here: https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status
Common HTTP Status Codes
· 5xx Server error responses (the server failed to fulfill a valid request)
   · 500 Internal Server Error: A generic error message indicating an
     unexpected condition on the server.
   · Others: 502 Bad Gateway, 503 Service Unavailable, ...
Read more here: https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status
Common HTTP Header Fields
· Content-Type (the media type of the resource)
   · "text/html; charset=utf-8" for HTML files in UTF-8 encoding
   · "text/plain" for plain text
   · "application/json" for json files (used for API requests / responses)
Read more here: https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Type
HTTPS (Hypertext Transfer Protocol Secure)
· Uses Secure Sockets Layer (SSL) / Transport Layer Security (TLS) to
  encrypt communication
· Very important whenever sensitive data is
  transferred (e.g., passwords, personally-identifiable
  information, private data, private messages, ...)
· Has become the default for all public web pages,
  even for non-sensitive data
Application Layer Protocols use Transport Layer
Protocols to Transmit Messages
· Higher-layer protocols use the protocols                         Application Layer
  directly below them to send messages.
                                                                   (e.g., HTTP, HTTPS)
· Whenever this happens, the higher-layer
  message might get split up.                                      Transport Layer
  Then it the individual message
  get placed in the payload portion                                   (e.g., TCP, UDP)
  of the lower-layer messages
                                                                     Internet Layer
                                                                      (e.g., IPv4, IPv6)
                                                                       Link Layer
                                                                   (e.g., Ethernet, MAC)
UDP (User Datagram Protocol)
Just "throws" Messages at the Receiver
                                                          Data
In UDP, the Bob simply delivers
packages to Alice's  Here are some
address               packages for
without waiting
                            you
for a response.
UDP Is a Simple, Unreliable,
but Fast & High Throughput Protocol
UDP provides fast,          The End
connectionless, and
lightweight communication.
It does not guarantee       Did she receive
delivery, order, or error   the packages?
checking.                   We don't know.
                            But it was fast!
TCP (Transmission Control Protocol) is a more
Complicated, but Reliable Protocol
· To initiate the connection, Bob   SYN  Hello!
                                         I have some packages
sends a synchronize message (SYN).       for you. Would you like
                                                 them now?
TCP (Transmission Control Protocol) is a more
Complicated, but Reliable Protocol
                                                      SYN-ACK
· Alice confirms the connection by
sending a SYN-ACK message to                                       Yes, I'm ready
Bob.                                                               to receive the
                                                                     packages!
TCP (Transmission Control Protocol) is a more
Complicated, but Reliable Protocol
                                                      ACK    Great. Then I
· Bob confirms that the connection                         will give you the
                                                           packages now.
  has been established by sending an
  ACK (acknowledgment) message to
Alice.
        CS 35L Software Construction: Lecture 5 ­ Client Server & Node.js     26
        Tobias Dürschmid
TCP (Transmission Control Protocol) is a more
Complicated, but Reliable Protocol
· Bob sends data to Alice in multiple  Data  Here is the
  ordered messages that contain a
                                             first package!
checksum to allow Alice to detect
corrupted data and request
retransmission if needed.
TCP (Transmission Control Protocol) is a more
Complicated, but Reliable Protocol
· If Alice has received the message.       ACK
  Since it does not contain errors, she
                                                I received the
                                                first package!
sends an acknowledgement (ACK) to
Bob.
· If Bob does not receive the ACK until a
  given timeout, he attempts to send
  the message again.
      CS 35L Software Construction: Lecture 5 ­ Client Server & Node.js  28
      Tobias Dürschmid
TCP (Transmission Control Protocol) is a more
Complicated, but Reliable Protocol
· Bob sends data to Alice in multiple  Data  Here is the
  ordered messages that contain a
                                             second package!
checksum to allow Alice to detect
corrupted data and request
retransmission if needed.
TCP (Transmission Control Protocol) is a more
Complicated, but Reliable Protocol
· If Alice has received the message.       ACK
  Since it does not contain errors, she
                                                      I received the
                                                   second package!
sends an acknowledgement (ACK) to
Bob.
· If Bob does not receive the ACK until a
  given timeout, he attempts to send
  the message again.
      CS 35L Software Construction: Lecture 5 ­ Client Server & Node.js  30
      Tobias Dürschmid
TCP (Transmission Control Protocol) is a more
Complicated, but Reliable Protocol
                                                       FIN   That's all I
· To notify Alice that Bob does not                         have for you
  have any more messages to send,
  he sends a FIN message.
TCP (Transmission Control Protocol) is a more
Complicated, but Reliable Protocol
· Alice acknowledges Bob's message   FIN ACK
  (ACK). She also does not have any
  more messages to send.                        Okay.
                                              Goodbye
So, she sends a FIN message too.
TCP (Transmission Control Protocol) is a more
Complicated, but Reliable Protocol
· Bob acknowledges Alice's FIN  ACK
message with an ACK message.         Goodbye
To achieve reliability, TCP sends at least 6+2N
messages for N messages of data
Client TCP Server                                       Client UDP Server
SYN       SYN-ACK        Establishes the                Data1
                                                        Data2
ACK    ACK[Data1]        Connection
Data1  ACK[Data2]
                         (to ensure both parties are
Data2                    ready to send / receive data)
                         (Full Duplex)          UPD completes the transfer
                         Data Transfer          before TCP even sends the first
                                                data package
                         (both can send
                         messages at any time)
FIN    FIN-ACK           Terminates the Connection
ACK
                         (to free up resources on both the client and server systems
                         and to ensure reliable closure of the communication session)
       CS 35L Software Construction: Lecture 5 ­ Client Server & Node.js               34
       Tobias Dürschmid
TCP and UDP Offer Different Trade-Offs
TCP                    Message Order is preserved
                       Error-Detection included
                       Lost messages get re-sent
                       Protocol adds additional overhead
UDP                     Messages can arrive in Any Order
                        No Error-Detection included
                        Some messages Might Get Lost
                        No Additional Delay
     CS 35L Software Construction: Lecture 5 ­ Client Server & Node.js  35
     Tobias Dürschmid
TCP and UDP Work Well For Different Use Cases
TCP                    Text Messaging (e.g., Slack, Discord)
                       Web Browsing
                       File Transfer
UDP                    Live Video Streaming (e.g., Webcasts)
                       Real-Time Voice Chat (e.g., Audio Call)
     CS 35L Software Construction: Lecture 5 ­ Client Server & Node.js  36
     Tobias Dürschmid
What about online gaming? Talk to your Neighbor(s)!
TCP and UDP Work Well For Different Use Cases
TCP                    · Ideal for round-based games (e.g., Chess, Go)
                          Many games use a hybrid approach:
                       · Reliable, non-time-sensitive data where loss
                           is fatal (e.g., chat messages, logging into the
                           server, inventory transactions, scoring)
UDP                    · High-speed real-time events
                          (e.g., player positions, physics)
                       · Game updates should not require previous
                          messages, so they need to include the absolute
                          positions of all movable objects
     CS 35L Software Construction: Lecture 5 ­ Client Server & Node.js      37
     Tobias Dürschmid
IP Addresses
Challenge: Identification             8.8.4.4 120.28.1.4 10.80.14.4 210.82.4.14
How to identify a particular sender/receiver out of the billions of
computers connected to the internet?
  Solution Idea: IP Addresses
Have a portion of the address that represents the network (like a "city")
and a portion that represents the host (like a "street address")
IPv4 addresses range from 0.0.0.0 to 255.255.255.255
There are some special IP addresses, e.g., 127.0.0.1 is "local host",
which is internet-speak for the literal words "your machine"
After running out of all IP addresses, a new standard (IPv6) was created
                                    Messages from higher layer protocols are
TCP/IP Stack wrapped in messages from lower layer protocols
 Application Layer           · TCP/IP is an instance of a layered
                                architecture in which higher-level
(e.g., HTTP, HTTPS, SSH,        layers use only lower-level layers
   DNS, POP,TLS/SSL)
                             · This increases reusability of lower-
  Transport Layer               layer implementations
      (e.g., TCP, UDP)       · It also allows flexibility as you can
                                simply replace one layer with a
    Internet Layer              different implementation to get
                                different results
      (e.g., IPv4, IPv6)
      Link Layer
(e.g., Ethernet, Wifi, MAC)
TCP-IP was invented by     I went to UCLA!
Robert Kahn & Vinton Cerf     Go Bruins
Read more about TCP here: https://www.geeksforgeeks.org/computer-networks/what-is-transmission-control-protocol-tcp/
HTML (Hyper Text Markup Language)
HTML is the most common markup language used to create web pages.
It supports structured content. HTML is not a programming language
<!DOCTYPE html>  Head contains metadata read by the browser
<html>           Body contains the content to be rendered
<head>
<title>Page Title</title>
</head>                      h1 indicates the top heading (others: h2 ­ h6)
<body>
<h1>This is a Heading</h1>
<p>This is a paragraph.</p>
</body>
</html>
         CS 35L Software Construction: Lecture 5 ­ Client Server & Node.js   41
         Tobias Dürschmid
Java Script (JS)
· Interpreted, dynamically typed programming language (similar to Python)
· Can be hooked into HTML
// A function that takes two arguments
function add(num1, num2) {
   let sum = num1 + num2;
   return sum; // Returns the result
}
let result = add(5, 7); // result is now 12
console.log(result); // Output: 12
JSON (JavaScript Object Notation)
human-readable, lightweight data-interchange   {
format used to transfer data between a server     "address-dict": {
and a web page, or between applications.              "city": "New York",
                                                      "residential": true,
                                                      "postal_code": 10023
                                                  },
                                                  "phone_numbers": [
                                                      {
                                                         "type": "home",
                                                         "number": "212 555-1234"
                                                      },
                                                      {
                                                         "type": "office",
                                                         "number": "646 555-4567"
                                                      }
                                                  ],
                                                  "children": [
                                                      "Catherine",
                                                      "Thomas",
                                                      "Trevor"
                                                  ]
                                               }
Node.js
· JavaScript runtime for asynchronous events
· Allows you to run JavaScript outside of the browser
· Is not a programming language
  (you are writing code in Java Script to be run within the Node.js environment)
· Fundamentally single-threaded for its main execution of JavaScript code
· Works via callbacks and event handlers
CS 35L Software Construction: Lecture 5 ­ Client Server & Node.js  45
CS 35L Software Construction: Lecture 5 ­ Client Server & Node.js  46
Blocking vs. Non-Blocking Code
                           require imports the fs (file system) module from node
                           standard library (similar to import fs as fs)
const fs = require('node:fs');
const declares a constant (cannot be changed after initialization).
Use constants to prevent bugs in you code!
const data = fs.readFileSync('/file.md');
// blocks here until file is read
Read more here: https://nodejs.org/en/learn/asynchronous-work/overview-of-blocking-vs-non-blocking
Blocking vs. Non-Blocking Code
const fs = require('node:fs');
                         Blocking Code
const data = fs.readFileSync('/file.md');
// blocks here until file is read
            Non-Blocking Code                                                Node.js philosophy:
                                                                             use non-blocking
fs.readFile('/file.md', (err, data) => {                                     code for all non-
                                                                             instant tasks to
if (err) {                                                                   maintain high
                                                                             performance and
     ...    Defines an asynchronous callback                                 scalability
}           that executes after the data has been
});         completely stored in the data variable
// following code is executed instantly
          CS 35L Software Construction: Lecture 5 ­ Client Server & Node.js  48
          Tobias Dürschmid
Define Callbacks Functions
for Long Non-Blocking Code
function handleMarkdownRead(err, data) {  Callbacks Functions
   if (err) {                             allow you to reuse
        ...                               non-blocking code,
   }                                      give it a name, and
                                          make your code
}                                         easier to read
fs.readFile('/file.md', handleMarkdownRead);
// following code is executed instantly
CS 35L Software Construction: Lecture 5 ­ Client Server & Node.js  50
                                                                            See: https://www.w3schools.com/nodejs/nodejs_http.asp
What does it do? Talk to your Neighbor(s)!
const http = require('http'); Imports http module
const PORT = 3000;     Creates an HTTP server object by defining an callback to be
                       called whenever the server receives an HTTP requires (req)
const server = http.createServer((req, res) => {
res.writeHead(200, { 'Content-Type': 'text/plain' });
                 Sets HTTP response header to status code 200 (success) and plain text content
res.end('Hello, World!\n');
});  Writes 'Hello, World!' into the HTTP response body and sends the message
                     Starts the server on localhost to listen on port 3000
server.listen(PORT, 'localhost', () => {
console.log(`Server running at http://localhost:${PORT}/`);
});                    Like f-strings in Python
     CS 35L Software Construction: Lecture 5 ­ Client Server & Node.js         51
     Tobias Dürschmid
CS 35L Software Construction: Lecture 5 ­ Client Server & Node.js  52
                          See https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Server-
                          side/Express_Nodejs/Introduction
What does it do? Talk to your Neighbor(s)!
const express = require('express');  Imports the super helpful express module
const app = express();
const port = 8080;                   that helps you with routing URLs
app.get('/users/:userId', (req, res) => {           Route for all GET requests to
res.send(`GET request to user ${req.params.userId}`); localhost:8080/users/.*
}); Retrieves the user id from the GET request URL
app.post('/', (req, res) => {
res.send('POST request to the homepage'); Route for all POST requests
});                                                 to localhost:8080/
app.get('/about', (req, res) => {    Route for all GET requests to
   res.send('About page');           localhost:8080/about
});
app.all('*', (req, res) => {
   res.status(404).send('404 - Page not found'); Route for all other requests
});
     CS 35L Software Construction: Lecture 5 ­ Client Server & Node.js                                           53
     Tobias Dürschmid
CS 35L Software Construction: Lecture 5 ­ Client Server & Node.js  54
                                               See https://nodesource.com/blog/an-absolute-beginners-guide-to-using-npm
NPM ­ The Node Package Manager
· NPM allows you to install node packages and keep the version
  listed in your package.json
· Node has a large ecosystem of useful package
Common npm cpmmands
$ npm init (creates a new NPM package from your code)
$ npm install express (installs the express package)
$ npm install express@4 (installs version 4 of express)
$ npm install --save (creates a package.json)
$ npm install (installs all packages listed in the package.json)
NodeJS Tutorials
· Course tutorial
   · not a fresh start, builds on what you know from Python & C++
   · Interactive tutorial right in the browser on smaller individual tasks
   · Also teaches some basics of express, communication via HTTP, and JSON
   · https://tobiasduerschmid.github.io/SEBook/tools/nodejs-tutorial
· Video Tutorial
   · https://www.youtube.com/watch?v=32M1al-Y6Ag
   · Practically-minded "bootcamp" on how to build a complete server backend in
     NodeJS
Please Fill out your Exit Tickets on Bruin Learn!
Please summarize three insights you learned about Client Server, Networking,
and/or Node.js today.
Imagine you should implement the video feature for YouTube. Which Application Layer
protocol and which Transport Layer protocol would you use and why?
Please leave any questions that you have about today's materials and things that
are still unclear or confusing to you (if none, simply write N/A).
Credits: These slide use images from Flaticon.com (Creators: Freepik, syafii5758, kerismaker)

```
