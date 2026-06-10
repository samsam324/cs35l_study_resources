# Networking & Client-Server Cheatsheet (candidate items)

## Architectures
### Client-Server
- **Client** = consumer of resources
- **Server** = provider of resources
- Multiple clients to one server
- Connections initiated by client, not server
- **Centralized** — all communication through server

### Peer-to-Peer (P2P)
- Peers are equally privileged
- Each is both supplier AND consumer
- **Decentralized**
- Most real-world systems are HYBRID (e.g., Zoom: client-server with P2P fallback for 1-on-1)

## Throughput vs Latency
- **Throughput** — volume of work / data per time unit ("100 req/sec")
- **Latency** (Response time) — time for ONE request → reply ("10 ms")
- Improve latency → faster code per request
- Improve throughput → faster latency OR more servers

## TCP/IP Stack (4 layers)
| Layer | Examples | Purpose |
|---|---|---|
| **Application** | HTTP, HTTPS, SSH, DNS, POP, TLS/SSL, FTP | App-to-app communication |
| **Transport** | TCP, UDP | End-to-end between hosts |
| **Internet (Network)** | IPv4, IPv6 | Addressing + routing packets |
| **Link** | Ethernet, Wi-Fi, MAC | Physical transmission |

**Key idea:** higher-layer messages WRAPPED IN lower-layer messages (nesting dolls).

## Messages: Header + Payload
- **Header** — metadata (destination, origin, content type, checksum)
- **Payload** — content
- Different protocols → different headers

## Package wrapping example
```
+-----------+--------+--------+--------+----------+
| Ethernet  |  IP    |  TCP   |  HTTP  | Payload  |
+-----------+--------+--------+--------+----------+
```
Outermost = link layer; innermost = application data.

## Application Layer Protocols
- **HTTP / HTTPS** — request files from server
- **FTP** — file transfer
- **POP / SMTP** — email
- **SSH** — secure shell
- **DNS** — domain → IP lookup
- **TLS / SSL** — encryption layer

## HTTP basics
- Stateless (no server-side session)
- Each request independent
- Verbs: **GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS**

## HTTP status codes
| Class | Range | Meaning |
|---|---|---|
| 1xx | Informational | Continue |
| 2xx | Success | 200 OK, 201 Created, 204 No Content |
| 3xx | Redirect | 301 Moved, 304 Not Modified |
| 4xx | Client error | 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 429 Too Many Requests |
| 5xx | Server error | 500 Internal, 502 Bad Gateway, 503 Unavailable |

## URL structure
```
{protocol}://{domain}(:{port})?(/{resource})?
http://localhost:8080/users/1
https://api.example.com/items?id=42
```
- Default ports: HTTP 80, HTTPS 443, SSH 22, FTP 21

## HTTPS / TLS
- HTTP over TLS (encrypted)
- Uses public-key crypto for handshake, then symmetric encryption for data
- Prevents eavesdropping + MITM
- Authenticates server identity via certificate

## TCP vs UDP
| | TCP | UDP |
|---|---|---|
| Connection | Yes (3-way handshake) | No |
| Reliable | Yes (acks, retries) | No |
| Ordered delivery | Yes | No |
| Speed | Slower | Faster |
| Use | Web, email, file transfer | Video calls, gaming, DNS |

## TCP 3-way handshake
1. **SYN** — client → server: "want to talk"
2. **SYN-ACK** — server → client: "ok, sync"
3. **ACK** — client → server: "confirmed"
Then data flows. **FIN** to close.

## IP addresses
- **IPv4** — 32-bit (e.g., `192.168.1.1`) — ~4.3B addresses, exhausted
- **IPv6** — 128-bit (e.g., `2001:0db8:85a3::8a2e:0370:7334`) — practically unlimited

## DNS (Domain Name System)
- Translates `example.com` → `93.184.216.34`
- Distributed hierarchy (root → TLD → authoritative)
- Cached at multiple levels

## REST basics
- Use HTTP verbs as actions on resources
- See `api-design-and-interoperability.md` for details

## Common ports
| Port | Service |
|---|---|
| 22 | SSH |
| 25 | SMTP |
| 53 | DNS |
| 80 | HTTP |
| 443 | HTTPS |
| 3306 | MySQL |
| 5432 | PostgreSQL |
| 6379 | Redis |
| 8080 | HTTP (alt) |

## Layered architecture properties (revisited)
- TCP/IP = layered architecture (higher uses lower)
- Increases **reusability** of lower layers
- Allows **flexibility** — swap a layer's implementation

## Common protocols quick reference
- **HTTP** — web pages, REST
- **HTTPS** — encrypted HTTP
- **WebSocket** — bidirectional persistent connection
- **gRPC** — protobuf RPC over HTTP/2
- **MQTT** — lightweight pub-sub (IoT)
- **WebRTC** — P2P browser real-time

## Throughput formula (rough)
`Throughput ≈ 1 / Latency × Parallelism`
Improve either side to scale.

## Additional items (potentially missing)

### CORS (Cross-Origin Resource Sharing)
- Browser security: blocks JS from making requests to a different origin
- Origin = scheme + domain + port
- Server opts-in by sending headers:
  ```
  Access-Control-Allow-Origin: https://example.com
  Access-Control-Allow-Methods: GET, POST
  Access-Control-Allow-Headers: Content-Type
  ```
- **Preflight request** — OPTIONS request sent before non-simple cross-origin requests

### HTTP/2 vs HTTP/1.1 (brief)
- HTTP/2: binary, multiplexed, server push, header compression
- HTTP/1.1: text, one request per connection (without keep-alive)
- HTTP/3: over QUIC (UDP), faster handshake

### WebSockets
- Persistent bidirectional connection
- Starts as HTTP, upgrades via `Upgrade: websocket`
- Use case: chat, live updates, multiplayer games
- `ws://` (insecure) or `wss://` (secure)

### Polling vs WebSockets vs SSE
| | Polling | WebSocket | SSE (Server-Sent Events) |
|---|---|---|---|
| Direction | Client pulls | Bidirectional | Server pushes |
| Overhead | High | Low | Medium |
| Use case | Simple updates | Real-time chat, games | News feeds, notifications |

### REST URL conventions
| HTTP + URL | Action |
|---|---|
| `GET /users` | list |
| `GET /users/42` | single |
| `POST /users` | create |
| `PUT /users/42` | replace |
| `PATCH /users/42` | partial update |
| `DELETE /users/42` | remove |
| `GET /users/42/orders` | nested |

### Standard ports recap
| Port | Service |
|---|---|
| 22 | SSH |
| 23 | Telnet (insecure) |
| 25 | SMTP |
| 53 | DNS |
| 80 | HTTP |
| 110 | POP3 |
| 143 | IMAP |
| 443 | HTTPS |
| 465 | SMTPS |
| 587 | SMTP (submission) |
| 993 | IMAPS |
| 995 | POP3S |
| 3306 | MySQL |
| 5432 | PostgreSQL |
| 6379 | Redis |
| 8080 | HTTP (alt) |
| 27017 | MongoDB |

### Reserved IP ranges
- `10.0.0.0/8` private
- `172.16.0.0/12` private
- `192.168.0.0/16` private
- `127.0.0.0/8` loopback (`127.0.0.1` = localhost)
- `169.254.0.0/16` link-local

### DNS record types
- **A** — IPv4 address
- **AAAA** — IPv6 address
- **CNAME** — alias to another name
- **MX** — mail server
- **TXT** — arbitrary text (SPF, DKIM, verification)
- **NS** — name server
- **SOA** — start of authority

### Network troubleshooting tools
- `ping <host>` — is host reachable
- `traceroute <host>` — path packets take
- `dig <host>` / `nslookup <host>` — DNS lookup
- `curl -v <url>` — verbose HTTP
- `netstat -an` — open connections
- `lsof -i :<port>` — what's listening on port
- `tcpdump` — packet capture
- `wireshark` — graphical packet analyzer

### REST best practices recap
- Use nouns in URLs (not verbs)
- Plural resource names (`/users`, not `/user`)
- Status codes that match outcome
- Consistent error format
- Versioning (`/v1/`)
- Authentication via header (not URL)
- Rate limiting

### Cookies
- Set-Cookie header from server
- Sent automatically with subsequent requests to same domain
- Attributes: `HttpOnly`, `Secure`, `SameSite`, `Path`, `Domain`, `Expires`, `Max-Age`

### Stateful vs Stateless
- **Stateful** — server remembers client (sessions)
- **Stateless** — every request has all needed info (JWT, REST style)
- Stateless scales better (any server can handle any request)

### Common request headers
- `Accept: application/json` — what I want
- `Accept-Encoding: gzip` — supported compressions
- `Accept-Language: en-US` — preferred language
- `Authorization: Bearer <token>`
- `Content-Type: application/json`
- `Content-Length: 1234`
- `User-Agent: ...`
- `Cookie: name=value`
- `Referer: <url>` — previous page
- `Host: example.com`

### Common response headers
- `Content-Type: application/json`
- `Content-Length`
- `Cache-Control`
- `ETag`
- `Set-Cookie: ...`
- `Location: ...` (for 3xx)
- `Server: nginx/1.18`
