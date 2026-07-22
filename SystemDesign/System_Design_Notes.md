# System Design — Interview Notes

Source: [AlgoMaster Top 30 Concepts](https://algomaster.io/learn/system-design/top-30-system-design-concepts) + supporting resources.
Format: concept → diagram → interview Qs (with concise answers).

---

# GROUP 1 — Networking Foundations

Before designing any system, understand how machines talk to each other. Every request in a distributed system crosses a network. These are the layers everything else stands on.

---

## 1. Client-Server Model

Every web app follows the same shape: **the client asks, the server answers.**

```
┌──────────────────┐
│      CLIENT      │
│  Browser / App   │
└────────┬─────────┘
         │  HTTP Request
         ▼
┌──────────────────┐
│      SERVER      │
│     Backend      │
└────────┬─────────┘
         │  Query
         ▼
┌──────────────────┐
│     DATABASE     │
└──────────────────┘
   (response flows back up the same path)
```

**Key points:**
- **Client** = anything that initiates the request (browser, mobile app, IoT, or even another server).
- **Server** = listens for requests, processes them, returns responses.
- **Separation matters:** you can build a mobile app without touching the server, scale the server independently, or serve multiple client types from one backend.
- Almost every diagram you'll draw in an interview starts here.

> 💡 **"Evolve & scale independently" — what it means:**
> The **API contract** between client and server is what matters — as long as it stays stable, each side can change on its own.
> - **Evolve:** swap the frontend from React to Vue → backend untouched. Rewrite backend from Node to Python → frontend doesn't notice. Web + Mobile + Desktop can all sit on the **same** backend.
> - **Scale:** more backend traffic → add more **backend servers**. Many mobile users → upgrade the **mobile app** only. Each side scales on its own.
> **Golden line:** *"as long as the API contract is stable, both sides can change independently."*

### Interview Qs
- **What is the client-server model?** A pattern where the client initiates a request and the server responds. It separates concerns → client and server can evolve/scale independently.
- **Can a server also be a client?** Yes — service A calling service B makes A a client of B. Very common in microservices.

---

## 2. IP Address

Every device on the internet has a unique address, like a house has a street address. Without an IP, packets have nowhere to go.

```
┌────────────────────┐
│    Your Device     │
│   192.168.1.10     │
└──────────┬─────────┘
           │  Packet
           ▼
┌────────────────────┐
│   Router Gateway   │  ← translates private ↔ public IP (NAT)
└──────────┬─────────┘
           │
           ▼
┌────────────────────┐
│  Internet Backbone │  ← ISPs + global fiber network
└──────────┬─────────┘
           │  Delivers
           ▼
┌────────────────────┐
│      SERVER        │
│   142.250.80.46    │
└────────────────────┘
```

**Key points:**
- **IPv4** — `192.168.1.1` → ~4.3 billion addresses (running out).
- **IPv6** — `2001:0db8:85a3::8a2e:0370:7334` → virtually unlimited.
- **Private IPs** (`192.168.x.x`, `10.x.x.x`) live inside your local network. Your router assigns them.
- **Public IPs** are unique on the internet.
- In system design: every server, load balancer, DB node has an IP that others use to reach it.

### The path from device to server
```
Your Device → Router (home) → ISP (Vodafone/WE) → Internet Backbone → Server's ISP → Server
```
- **ISP** = Internet Service Provider — connects you to the backbone.
- **Internet Backbone** = huge cross-country/undersea cables that carry global traffic between ISPs.

### Interview Qs
- **What's the difference between IPv4 and IPv6?** IPv4 = 32-bit, ~4.3B addresses. IPv6 = 128-bit, virtually unlimited. IPv6 exists because IPv4 ran out.
- **Public vs private IP?** Public = unique on the internet. Private = local network only (behind your router via NAT).

---

## 3. DNS (Domain Name System)

**DNS is the phone book of the internet.** Nobody wants to memorize `142.250.80.46` — DNS translates `google.com` → IP.

```
┌──────────────────┐
│      BROWSER     │  "I need google.com"
└────────┬─────────┘
         │ 1. Query
         ▼
┌──────────────────┐
│   DNS RESOLVER   │  (usually your ISP's — Vodafone/WE)
│   ISP / Local    │
└────────┬─────────┘
         │
         │ 2. "Who handles .com?"
         ▼
┌──────────────────┐
│   ROOT SERVER    │  → "Ask the .com TLD server"
└────────┬─────────┘
         │
         │ 3. "Who handles google.com?"
         ▼
┌──────────────────┐
│   TLD SERVER     │  → "Ask google's authoritative NS"
│      .com        │
└────────┬─────────┘
         │
         │ 4. "What's the IP of google.com?"
         ▼
┌──────────────────┐
│ AUTHORITATIVE NS │  → "142.250.80.46"
│  (google.com)    │
└────────┬─────────┘
         │
         │ 5. IP returned back up the chain
         ▼
    Browser gets: 142.250.80.46
```

**Note:** each level **caches** the answer with a **TTL** → next lookup often skips most steps.

### Resolution process (step-by-step)
1. **Browser cache** — check locally. Hit? done.
2. **OS cache** — check the operating system cache.
3. **DNS Resolver** (your ISP's) — ask them.
4. Resolver asks the **Root** server: "who handles `.com`?"
5. Root points to the **TLD (`.com`) server**.
6. TLD points to `google.com`'s **Authoritative Name Server**.
7. Authoritative NS returns the actual IP.
8. Every level **caches** the answer (with a **TTL**) → next lookup skips the chain.

### DNS is also used for
- **Load balancing** — same domain returns different IPs → spread traffic across servers.
- **Failover** — primary down? DNS returns backup IP.
- **Geo-routing** — return the IP of the server nearest to the user (Egypt user → Cairo edge server).

### Interview Qs
- **Walk through what happens when you type `google.com` in your browser.**
  DNS lookup (browser → OS → resolver → root → TLD → authoritative) → returns IP → browser opens TCP connection → sends HTTP request → server responds → browser renders.
- **What is a TTL in DNS?** Time-to-live — how long a DNS record can be cached. Short TTL = fresher, more lookups. Long TTL = fewer lookups, slower to react to changes.
- **How is DNS used for load balancing?** DNS can return multiple IPs for the same domain (round-robin) or return the "closest" one (GeoDNS). Users hit different servers.
- **What happens if the authoritative name server is down?** Cached entries still work until their TTL expires. After that → resolution fails.

---

## 🔄 The Full Request Flow (putting 1+2+3 together)

```
1. You type "google.com" in the browser
             │
             ▼
2. DNS lookup: "google.com" → 142.250.80.46
   (Browser cache → OS cache → ISP Resolver → Root → TLD → Authoritative NS)
             │
             ▼
3. Browser opens TCP connection to 142.250.80.46
             │
             ▼
4. HTTP Request travels:
   Your Device → Router → ISP → Internet Backbone → Google's ISP → Google's Server
             │
             ▼
5. Server processes request (backend logic + DB if needed)
             │
             ▼
6. HTTP Response returns the same path back
             │
             ▼
7. Browser renders the page
```

### The interview classic — "what happens when you type URL and press Enter?"
The canonical answer covers:
1. **DNS resolution** — name → IP.
2. **TCP handshake** — 3-way (SYN, SYN-ACK, ACK) between browser and server.
3. **TLS handshake** (if HTTPS) — cert exchange + key negotiation.
4. **HTTP request** sent.
5. **Server processes** — may hit cache, DB, other services.
6. **HTTP response** returned.
7. **Browser renders** — parses HTML, fetches CSS/JS/images (which may repeat DNS + TCP for other domains), builds DOM, paints.

---

---

## 4. Proxy vs Reverse Proxy

**Proxy = a middleman between two parties.** Two flavors, opposite jobs.

### 🟢 Forward Proxy — sits in front of CLIENTS, hides them

```
┌──────────┐
│ Client A │──┐
└──────────┘  │
              │
┌──────────┐  │     ┌───────────────┐         ┌────────────┐
│ Client B │──┼────▶│ Forward Proxy │────────▶│  Internet  │
└──────────┘  │     │   (single     │         │  / Server  │
              │     │   choke pt)   │         └────────────┘
┌──────────┐  │     └───────────────┘
│ Client C │──┘         ↑
└──────────┘            Server sees ONLY the proxy's IP
```

- Server sees the **proxy's IP**, not the client's.
- **Single choke point** = one place to monitor, filter, block, or log.
- Use cases:
  - **VPN** — hide your real location from websites.
  - **Corporate proxy** — company filters/logs employee traffic.
  - **Content filtering** — schools block sites.
  - **Bypass geo-restrictions** — appear from another country.

### 🔵 Reverse Proxy — sits in front of SERVERS, hides them

```
                                       ┌──────────┐
                                    ┌─▶│ Server 1 │
                                    │  └──────────┘
┌──────────┐    ┌───────────────┐   │
│  Client  │───▶│ Reverse Proxy │───┤  ┌──────────┐
└──────────┘    │ (Nginx / LB)  │   ├─▶│ Server 2 │
                └───────────────┘   │  └──────────┘
                        ↑           │
                Client thinks it    │  ┌──────────┐
                talks to ONE server └─▶│ Server 3 │
                                       └──────────┘
```

- Client sees the **proxy's IP** — has no idea there are 3 (or 300) servers behind it.
- Server IPs are **not exposed** to the internet.

### What Reverse Proxy actually does (interview gold)

1. **Load balancing** — spreads incoming traffic across backend servers so none is overwhelmed.
2. **SSL termination** — handles HTTPS decryption; backend runs plain HTTP internally (see below).
3. **Caching** — stores common responses so the backend isn't hit every time.
4. **Compression** — gzip/brotli the response before sending to the client → smaller payload, faster.
5. **Security / DDoS protection** — hides backend IPs, rate-limits, blocks known bad IPs.
6. **Rewrite URLs / route by path** — `/api` → backend 1, `/static` → backend 2.

### Side-by-side

| | Forward Proxy | Reverse Proxy |
|--|---------------|---------------|
| Sits in front of | Clients | Servers |
| Hides | Client identity | Server identity |
| Configured by | Client / network admin | Server owner |
| Common tools | VPN, Squid | **Nginx, HAProxy, Cloudflare** |

**One-liner:**
> "Forward proxy hides the **client**. Reverse proxy hides the **server**."

---

## 5. DDoS Protection via Reverse Proxy

**DDoS (Distributed Denial of Service)** = flood of requests from thousands of hijacked machines aiming to crash your server.

Without a proxy → attacker hits your server directly → it dies.
With a reverse proxy → the proxy absorbs the hit.

**How the proxy defends:**
1. **Hides backend IP** — attacker can't target your server directly.
2. **Rate limiting** — X requests per IP per minute, else block.
3. **IP blacklists** — known botnet IPs blocked instantly.
4. **CAPTCHA challenges** — bots fail, real users pass.
5. **Behavior analysis** — 1000 IPs all hitting the same URL → attack pattern → block.

**Result:** the backend never even sees the attack traffic.

---

## 6. Nginx

**Nginx** (pronounced "Engine-X") — the most-used reverse proxy on the internet (~40% of websites).

### Why it's fast — event-driven architecture

- **Old model (Apache):** 1 request = 1 thread. 10K users = 10K threads = memory + context-switching hell.
- **Nginx:** one **event loop** per worker handles thousands of connections. When one is waiting on I/O, the worker serves another.
- Same idea as **Node.js event loop**.

Result: 10× more traffic per server, less memory.

### What Nginx does (all in one binary)

- Static file serving (HTML/CSS/JS/images)
- Reverse proxy → forward traffic to backend apps
- Load balancing (round-robin, least-connections, IP-hash)
- SSL termination
- Caching
- Compression (gzip / brotli)
- Rate limiting

### Typical setup

```
    Internet users
         │
         ▼
   ┌───────────┐
   │Cloudflare │  ← DDoS + CDN (edge)
   └─────┬─────┘
         ▼
   ┌───────────┐
   │   Nginx   │  ← SSL termination, load balancing, cache
   └─────┬─────┘
    ┌────┼────┐
    ▼    ▼    ▼
   Node Node Node  ← backend apps (business logic)
    │    │    │
    └────┼────┘
         ▼
    PostgreSQL
```

Used by: Netflix, Airbnb, Instagram, GitHub, WordPress.

---

## 7. SSL / TLS + SSL Termination

### SSL/TLS in one line
> **A protocol that encrypts traffic between client and server.** HTTPS = HTTP over TLS.

- **SSL** = old name. **TLS** = modern replacement. People still say "SSL" but everything is TLS.
- Marker: URL starts with **`https://`** + 🔒 icon.

### What TLS provides
1. **Encryption** — no one on the network can read the traffic.
2. **Authentication** — client verifies the server is really `google.com` (via **certificate** signed by a trusted **Certificate Authority**).
3. **Integrity** — any tampering is detected.

### TLS handshake (before the encrypted HTTP request)
```
1. Client → "Hi, I support these encryption methods..."
2. Server → "Let's use X. Here's my certificate."
3. Client → Verifies certificate, sends encrypted session key.
4. Both  → Now everything is encrypted with the session key.
```
The handshake is **CPU-expensive** (cryptographic math + multiple round trips).

---

### 🔑 SSL Termination

**Idea:** the **reverse proxy** does all the TLS work; the backend servers run plain HTTP internally.

```
Client                Reverse Proxy               Backend Servers
  │                          │                          │
  │──── HTTPS 🔒 ────────>   │                          │
  │  (encrypted)             │                          │
  │                          │ ← decrypts here          │
  │                          │                          │
  │                          │──── HTTP (plain) ─────>  │
  │                          │  (internal network only) │
  │                          │                          │
  │                          │<─── HTTP response ────── │
  │                          │                          │
  │                          │ ← re-encrypts            │
  │<─── HTTPS 🔒 ─────────── │                          │
```

**"Termination" = the encryption ends at the proxy.** Beyond it, traffic is plain HTTP.

### Why do it
- **Backend is lighter** — no crypto CPU cost, focuses on business logic.
- **One certificate to manage** — on the proxy, not on every backend server.
- **Proxy can read the content** — needed for caching, routing, load balancing decisions.
- **Renewals happen in one place.**

### Trade-off
- Internal traffic (proxy → backend) is **unencrypted**. Acceptable because it's on a **private, isolated network**.
- If total encryption is required end-to-end → **SSL Passthrough** (proxy just forwards encrypted traffic) or **re-encryption** (decrypt then re-encrypt to backend).

---

## 8. Latency

**Latency = the time data takes to travel from A to B, measured in milliseconds.** Low latency = the goal — UX depends on it.

**How much matters:**
- < 100ms → barely noticeable
- ~1s → feels sluggish
- > 3s → users leave

### 4 sources of latency
1. **Network distance** — speed of light in fiber (physics, can't beat it).
2. **Serialization** — converting objects to bytes before sending.
3. **Server processing** — the work the backend does.
4. **Queuing** — if the server is busy, requests wait.

### Solutions (each targets a different source)
| Problem | Fix |
|---------|-----|
| Distance | **CDN** (static content near users), **multi-region** deployment |
| Serialization overhead | **For internal service-to-service** calls: Protobuf/gRPC (smaller + faster). **Keep JSON** for public APIs (browser/mobile clients expect it). |
| Processing time | Better code, **indexes**, **caching** |
| Queuing | Scale up, **load balancing**, more servers |

### ⚠️ Latency vs Throughput (don't confuse)
- **Latency** = time for **one** request (ms).
- **Throughput** = requests per second (RPS).

**Highway analogy:**
- Latency = how long the trip takes (12 hours from Egypt to US).
- Throughput = how many cars pass per hour (10,000).

Both can be high at the same time — they're independent.

### Interview line
> "Latency is the delay for a single request. Sources: network distance (physics — speed of light), serialization, processing, queuing. Fixes: **CDNs** and **multi-region deployment** for distance, **caching** to skip DB round trips, **load balancing** for queuing. Not the same as throughput — throughput is requests/sec."

---

## 9. HTTP / HTTPS

**HTTP (HyperText Transfer Protocol)** = the language clients and servers use to talk on the web. Defines how requests and responses are structured. **HTTPS = HTTP over TLS** (encrypted — see section 7).

### 🔑 HTTP is STATELESS ⭐ (interview keyword)
Each request is **independent** — the server doesn't remember previous requests.

- **Good:** any server can handle any request → easy to scale horizontally (load balancers work).
- **Trade-off:** you need **cookies, tokens, or sessions** to maintain state across requests (login, cart, etc).

### HTTP Methods (verbs)
| Method | Purpose |
|--------|---------|
| **GET** | Read data (no side effects) |
| **POST** | Create new resource |
| **PUT** | Update/replace entire resource |
| **PATCH** | Update part of a resource |
| **DELETE** | Remove resource |

### HTTP Status Codes (the categories)
| Range | Meaning | Examples |
|-------|---------|----------|
| **1xx** | Info | 100 Continue |
| **2xx** | Success | **200 OK**, 201 Created, 204 No Content |
| **3xx** | Redirect | 301 Moved Permanently, 304 Not Modified |
| **4xx** | Client error | **400 Bad Request**, **401 Unauthorized**, **403 Forbidden**, **404 Not Found**, 429 Too Many Requests |
| **5xx** | Server error | **500 Internal Server Error**, 502 Bad Gateway, 503 Service Unavailable |

**Memorize:** 200 / 201 / 400 / 401 / 403 / 404 / 500 / 503.

### HTTP Headers
Metadata sent with every request/response.
- **`Authorization`** → auth tokens (Bearer, Basic).
- **`Content-Type`** → format (`application/json`, `text/html`).
- **`Cache-Control`** → caching rules (`no-cache`, `max-age=3600`).
- **`Cookie` / `Set-Cookie`** → session/state.

### Request example
```
POST /api/orders HTTP/1.1
Host: shop.com
Authorization: Bearer eyJhbGc...
Content-Type: application/json

{"product_id": 42, "qty": 2}
```

### Response example
```
HTTP/1.1 201 Created
Content-Type: application/json
Cache-Control: no-store

{"order_id": 1005, "status": "confirmed"}
```

### Interview line
> "**HTTP** is a stateless request-response protocol. **Stateless** = the server doesn't remember prior requests → any server can handle any request → easy horizontal scaling; state is added via **cookies/tokens/sessions**. Key methods: **GET/POST/PUT/DELETE**. Status codes grouped by first digit: **2xx** success, **3xx** redirect, **4xx** client error, **5xx** server error. **HTTPS** = HTTP over TLS — adds one TLS handshake but encrypts everything in transit."

---

## 10. TCP vs UDP ⭐⭐ (most-asked)

Two transport protocols. Same job (move bytes A→B). Opposite trade-offs.

### TCP — the reliable one
- **Connection-oriented** — needs a **3-way handshake** before sending anything.
- **Reliable delivery** — guarantees every packet arrives.
- **Ordered** — packets arrive in the same order sent.
- **Flow + congestion control** — slows down if network is busy.
- **Higher overhead** — header 20–60 bytes + all the extra bookkeeping.

**Use cases:** HTTP, databases, file transfer, email, SSH — anything where losing data is unacceptable.

### UDP — the fast one
- **Connectionless** — no handshake, just fire packets.
- **Unreliable** — no guarantee packets arrive.
- **Unordered** — packets may arrive out of order.
- **No congestion control.**
- **Minimal overhead** — header is just 8 bytes.

**Use cases:** DNS, video streaming, VoIP, online gaming, IoT sensors — anything where **speed > guarantees**.

### Side-by-side

| Aspect | TCP | UDP |
|--------|-----|-----|
| Connection | 3-way handshake first | None |
| Reliability | Guaranteed | Best-effort |
| Ordering | Maintained | Not guaranteed |
| Header size | 20–60 bytes | **8 bytes** |
| Speed | Slower | Faster |
| Use cases | HTTP, DBs, file transfer | DNS, streaming, gaming, VoIP |

### 3-Way Handshake (TCP setup)
```
Client                    Server
  │                         │
  │────── SYN ─────────────>│   "I want to talk"
  │                         │
  │<───── SYN-ACK ──────────│   "OK, I hear you"
  │                         │
  │────── ACK ─────────────>│   "Great, let's start"
  │                         │
  │────── DATA ────────────>│   (now data flows)
```

### Why "who handles reliability?" is the real question
Sometimes you want UDP's speed **but need reliability for certain messages**. Solution: build reliability at the **application layer** for only the messages that need it.

**Game example — 3 message types:**
1. **Unreliable:** player position (60/sec — old data is useless)
2. **Reliable unordered:** chat messages (must arrive, order doesn't matter)
3. **Reliable ordered:** game state changes (must arrive in order)

Game engines use UDP underneath but add sequence numbers/acks only where needed.

### QUIC + HTTP/3 (why this matters now)
- **QUIC** = a modern protocol built on **UDP** that adds reliability, congestion control, encryption, and multiplexing in user space.
- **HTTP/3 runs on QUIC** (instead of TCP).
- Why? faster connection setup + no head-of-line blocking (TCP's weakness).

### Interview line
> "**TCP** = connection-oriented, reliable, ordered — used by HTTP, DBs, file transfer. Costs a **3-way handshake** and larger headers. **UDP** = connectionless, unreliable, unordered, minimal 8-byte header — used by **DNS, video streaming, VoIP, gaming, IoT** where **speed matters more than guaranteed delivery**. The real question isn't 'do I need reliability?' but '**who** should handle it?' — sometimes the app handles reliability selectively (e.g. **QUIC / HTTP/3** builds it on top of UDP)."

---

## 🎯 Group 1 self-test (say out loud)
1. What is the client-server model?
2. What's the difference between IPv4 and IPv6?
3. What is DNS? Walk through the resolution process.
4. Two ways DNS is used besides name → IP translation?
5. What happens end-to-end when you type `google.com` and hit Enter?
6. Forward Proxy vs Reverse Proxy — one line for each.
7. Name 4 things a reverse proxy does.
8. How does a reverse proxy help with DDoS?
9. Why is Nginx so fast?
10. What's SSL termination and why do it?
11. What is latency? Name its 4 sources.
12. Latency vs throughput — the difference.
13. Why is HTTP called stateless? What are the implications?
14. TCP vs UDP — trade-offs + one use-case for each.
15. Walk through the TCP 3-way handshake.
16. Why does DNS use UDP not TCP?
17. What is QUIC and how does HTTP/3 use it?
14. HTTP status code categories — what do 2xx, 4xx, 5xx mean?
15. Difference between PUT and PATCH?

---

# GROUP 2 — APIs & Communication

Network moves data. APIs define **what data moves, how, and in what shape.** Every backend you build exposes an API. Every frontend you build consumes one.

---

## 11. What is an API?

**API = contract** between client and server. Defines:
- What requests can be made
- How to make them
- What responses to expect

```
     Client
       │
       │  Request (following API contract)
       ▼
   ┌────────────────────────┐
   │         API            │
   │  • Endpoints           │
   │  • Methods             │
   │  • Request/Response    │
   │    format              │
   └────────────────────────┘
       │
       │  Response
       ▼
     Server
```

**Two key characteristics:**
- **Abstraction** — hides implementation details, exposes functionality only
- **Service Boundaries** — clear interface between components (frontend ↔ backend, service ↔ service)

**Why it matters:** without API contract, client and server are tightly coupled. Any internal change breaks everything. API = agreement that both sides respect → evolve independently.

---

## 12. Core API Styles

| | REST | GraphQL |
|--|------|---------|
| **Model** | Resource-based (nouns) | Query language |
| **Transport** | HTTP | HTTP (single endpoint) |
| **Data format** | JSON (text) | JSON (text) |
| **Endpoints** | Many (`/users`, `/posts`) | One (`/graphql`) |
| **Operations** | GET/POST/PUT/DELETE | Query / Mutation / Subscription |
| **Best for** | Web & Mobile apps | Complex UIs (dashboards) |
| **Superpower** | Simple, cacheable, standard | Client gets exactly what it asks for |

### When to use what?

- **REST** → default choice. Simple CRUD, public APIs, mobile apps
- **GraphQL** → complex UI needing data from many resources in one request (dashboards, social feeds)

---

## 13. REST API (Deep Dive)

**REST = REpresentational State Transfer.** Not a protocol — a set of **constraints** for designing APIs over HTTP.

### Core Principles

1. **Resource-based** — everything is a resource identified by URL
2. **Stateless** — each request carries ALL info needed (no server memory between requests)
3. **Uniform Interface** — standard HTTP methods map to operations
4. **Client-Server separation** — evolve independently
5. **Cacheable** — responses can declare themselves cacheable
6. **Layered** — client doesn't know if talking to server directly or through proxy/LB

### Resources & URLs

```
Resource = noun (user, order, product)
URL = address of that resource

GET    /api/v1/users          → list all users
GET    /api/v1/users/123      → get user 123
POST   /api/v1/users          → create new user
PUT    /api/v1/users/123      → replace user 123 (full update)
PATCH  /api/v1/users/123      → partial update user 123
DELETE /api/v1/users/123      → delete user 123
```

**Nested resources:**
```
GET /api/v1/users/123/posts        → posts by user 123
GET /api/v1/users/123/posts/456    → specific post
```

### Good REST Design Rules

| Rule | Good | Bad |
|------|------|-----|
| Use nouns, not verbs | `/users` | `/getUsers` |
| Plural names | `/users` | `/user` |
| Versioning | `/api/v1/users` | `/api/users` |
| Nested for relationships | `/users/123/orders` | `/getUserOrders?id=123` |
| HTTP methods for actions | `DELETE /users/123` | `POST /deleteUser` |

### Status Codes (must know)

| Code | Meaning | When |
|------|---------|------|
| 200 | OK | Successful GET/PUT/PATCH |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Invalid input |
| 401 | Unauthorized | Not authenticated |
| 403 | Forbidden | Authenticated but no permission |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Duplicate / version conflict |
| 429 | Too Many Requests | Rate limited |
| 500 | Internal Server Error | Server bug |

### Pagination, Filtering, Sorting

```
GET /api/v1/users?page=2&limit=20              → pagination
GET /api/v1/users?role=admin                    → filtering
GET /api/v1/users?sort=created_at&order=desc    → sorting
GET /api/v1/users?role=admin&sort=name&page=1   → combined
```

### REST Weaknesses

1. **Over-fetching** — GET `/users/123` returns ALL fields even if you only need name
2. **Under-fetching** — need user + posts + followers = 3 separate requests
3. **No real-time** — HTTP is request-response only. Need polling or WebSocket for live data
4. **Versioning headache** — breaking changes need new version (`/v1/` → `/v2/`)

---

## 14. GraphQL

**GraphQL = query language for APIs.** Client describes exactly what data it wants → server returns exactly that. No more, no less.

### The Problem GraphQL Solves

**REST scenario — building a user profile page:**
```
GET /api/v1/users/123              → { id, name, email, bio, avatar, ... }  (over-fetch)
GET /api/v1/users/123/posts        → [ {title, body, likes, ...}, ... ]     (2nd request)
GET /api/v1/users/123/followers    → [ {name, avatar, ...}, ... ]           (3rd request)
```
3 requests. Each returns fields you don't need.

**GraphQL — same page, ONE request:**
```graphql
query {
  user(id: "123") {
    name
    avatar
    posts {
      title
    }
    followers {
      name
    }
  }
}
```
**One request. Only requested fields returned.**

### Core Concepts

**1. Schema** — defines ALL available data and operations (like a contract)
```graphql
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
}

type Post {
  id: ID!
  title: String!
  author: User!
}
```
`!` = non-nullable (required field).

**2. Three Operation Types:**

| Operation | Purpose | REST Equivalent |
|-----------|---------|-----------------|
| **Query** | Read data | GET |
| **Mutation** | Write/update/delete data | POST/PUT/DELETE |
| **Subscription** | Real-time (WebSocket under the hood) | No equivalent |

```graphql
# Mutation example
mutation {
  createPost(input: { title: "Hello", body: "World" }) {
    id
    title
  }
}
```

**3. Single Endpoint**
```
REST:     GET /users, GET /posts, GET /comments  (many endpoints)
GraphQL:  POST /graphql                           (ONE endpoint, always POST)
```

**4. Resolvers** — server-side functions that fetch data for each field
```
Query.user(id) → DB lookup → return user
User.posts     → DB lookup → return posts for this user
```
Each field in schema has a resolver. GraphQL engine calls them and assembles response.

### REST vs GraphQL — Side by Side

| | REST | GraphQL |
|--|------|---------|
| Endpoints | Many (one per resource) | One (`/graphql`) |
| Data shape | Server decides | Client decides |
| Over-fetching | Common | Impossible |
| Under-fetching | Common (multiple requests) | Impossible (nested query) |
| Caching | Easy (HTTP caching, CDN) | Hard (single POST endpoint) |
| Versioning | `/v1/`, `/v2/` | Schema evolution (no versioning needed) |
| Learning curve | Low | Medium |
| Error handling | HTTP status codes | Always 200, errors in response body |
| File upload | Native (multipart) | Needs workaround |
| Real-time | Needs WebSocket separately | Built-in Subscriptions |

### GraphQL Weaknesses

1. **Caching is hard** — all POST to same URL. HTTP cache / CDN can't help. Need application-level caching
2. **N+1 problem** — nested query can trigger hundreds of DB calls (resolver per field). Fix: **DataLoader** (batching)
3. **Complexity** — powerful queries can be expensive. Need **query depth limiting** and **complexity analysis**
4. **Overkill for simple CRUD** — if you just have 5 resources with basic CRUD, REST is simpler

### When to Use GraphQL?

- **Yes:** complex UI, multiple data sources, mobile (bandwidth matters), rapid frontend iteration
- **No:** simple CRUD, file-heavy APIs, need HTTP caching, small team that doesn't want the overhead

---

## 15. WebSockets

**Problem:** HTTP = request-response. Client must ask every time. Server can NEVER push data on its own.

**What if server needs to push data?** (chat messages, live scores, stock prices, notifications)

### Bad Solutions

**Regular Polling — server responds immediately, client waits, asks again:**
```
Client                          Server
  │── "any updates?" ───────────▶│
  │◀── "no" ─── (immediate) ─────│
  │                              │
  │  wait 5s...                  │
  │                              │
  │── "any updates?" ───────────▶│
  │◀── "no" ─── (immediate) ─────│
  │                              │
  │  wait 5s...                  │
```
- 90% of requests return nothing → **wasted bandwidth**
- Latency = polling interval (5s means user waits up to 5s to see new data)

**Long Polling — server holds the request open until data arrives:**
```
Client                          Server
  │── "any updates?" ───────────▶│
  │                              │ Server HOLDS the request open!
  │       (waiting...)           │ Doesn't respond yet.
  │       (10 seconds pass)      │
  │                              │ 💥 New data arrived!
  │◀── "yes! new message" ───────│ NOW respond
  │                              │
  │── "any updates?" ───────────▶│ Client immediately asks again
  │       (waiting...)           │
```

**Key difference:** Regular polling → server replies instantly (even with "no"). Long polling → server waits until it has real data before replying.

**Better than regular polling** (lower latency, fewer wasted requests), **but still hacky:**
- **One-directional** — server can only respond; can't push. Client must ask first.
- **Connection overhead** — every message = new HTTP request (headers, TCP handshake).
- **Server resources** — many open connections consume memory.
- **Timeout issues** — firewalls may kill long-open connections.

WebSocket solves all of these: one persistent connection, two-way, no repeat requests.

### The Real Solution — WebSocket

**One persistent connection. Both sides send whenever they want.**

```
     Client                          Server
       │                               │
       │── GET /chat (Upgrade) ───────▶│  (1) Normal HTTP request +
       │   Connection: Upgrade         │      "Upgrade: websocket" header
       │   Upgrade: websocket          │
       │                               │
       │◀── 101 Switching Protocols ──│  (2) Server agrees
       │                               │
       ║═══════════════════════════════║  (3) Connection stays open!
       │                               │      Full-duplex (both send)
       │◀── "Ahmed: hi" ──────────────│  Server pushes anytime
       │── "Salma: hey" ──────────────▶│  Client sends anytime
       │◀── "Ahmed: how are you?" ────│  Server pushes again
       │                               │
       │── Close ─────────────────────▶│  (4) Either side closes
```

### Key Points

- **Starts as HTTP** → upgrades to WebSocket (same port 80/443)
- **Full-duplex** — both sides send whenever they want
- **Persistent connection** — stays open, no repeated handshakes
- **Tiny overhead** — after handshake, frames are 2 bytes header (vs HTTP's ~800 bytes)
- **Stateful** — server must track every open connection (harder to scale than REST)

### Code Example — Server (Node.js)

```javascript
const WebSocket = require('ws');
const server = new WebSocket.Server({ port: 8080 });

server.on('connection', (socket) => {
  console.log('New client connected');

  socket.on('message', (message) => {
    // Broadcast to all other connected clients
    server.clients.forEach((client) => {
      if (client !== socket && client.readyState === WebSocket.OPEN) {
        client.send(message);   // ← server pushes without being asked!
      }
    });
  });

  socket.on('close', () => console.log('Client disconnected'));
});
```

### Code Example — Client (Browser)

```javascript
const socket = new WebSocket('ws://localhost:8080');

socket.onopen = () => {
  socket.send('Hello from Salma');
};

socket.onmessage = (event) => {
  console.log('Received:', event.data);   // ← server pushed this!
};

socket.onclose = () => console.log('Disconnected');
```

### HTTP vs WebSocket

| | HTTP | WebSocket |
|--|------|-----------|
| **Direction** | Client → Server only | **Both ways (full-duplex)** |
| **Connection** | Closes after each response | **Stays open** |
| **Server push** | Impossible | **Yes** |
| **Overhead** | ~800 bytes headers per request | **2 bytes after handshake** |
| **State** | Stateless | **Stateful** (tracks connections) |

### Use Cases

| WebSocket | NOT WebSocket |
|-----------|--------------|
| Chat apps | CRUD operations |
| Live dashboards | Form submissions |
| Multiplayer games | Static content |
| Stock tickers | Search |
| Collaborative editing (Google Docs) | One-time requests |

### The Scaling Problem

**REST is stateless** → add 10 servers behind a load balancer, done.

**WebSocket is stateful** → each server holds live connections:
```
User A ──connected──▶ Server 1
User B ──connected──▶ Server 2
```
A sends message to B. It hits Server 1... but B is on Server 2!
**Server 1 doesn't know how to reach B directly.**

Solution: **Pub/Sub between servers** (next section).

### SSE (Server-Sent Events) — Middle Ground

| | Polling | SSE | WebSocket |
|--|---------|-----|-----------|
| Direction | Client → Server | **Server → Client only** | **Both ways** |
| Connection | Closes each time | Stays open | Stays open |
| Use case | Simple checking | Notifications, live feeds | Chat, games |

SSE = server pushes only. Simpler than WebSocket. Use for live notifications, news feeds.

### When to use what?

| Use Case | Solution |
|----------|----------|
| User profile page | REST (GET) |
| Chat app | **WebSocket** |
| Live sports scores | **SSE** |
| Web notifications | **SSE** |
| Multiplayer game | **WebSocket** |
| Form submission | REST (POST) |
| Stock ticker | **WebSocket** |

---

## 16. Message Queues, Pub/Sub, and Kafka

These 3 tools all pass messages between services, but they solve **different problems**. Confusing them is a classic interview trap.

### The Two Core Concepts

| Concept | Behavior |
|---------|----------|
| **Message Queue** | Message goes to **ONE consumer** → consumed → deleted |
| **Pub/Sub** | Message goes to **ALL subscribers** → each gets a copy |

**These are concepts, not tools.** Different tools implement them differently.

### The Three Tools

| Tool | What it is |
|------|-----------|
| **RabbitMQ** | Message Queue (can do pub/sub too) |
| **Redis Pub/Sub** | Pub/Sub only, **NO storage** (fire and forget) |
| **Kafka** | Hybrid — distributed log with **persistent storage** |

### Real-World Analogies

- **RabbitMQ** = office task box. Manager drops a task → one employee grabs it → task done, paper thrown away.
- **Redis Pub/Sub** = radio broadcaster. Whoever tuned in **right now** hears it. Not tuned in? You missed it — no recording.
- **Kafka** = daily newspaper archive. Every event printed. Any subscriber reads at their own pace. Old issues stay for weeks. Can re-read.

### Comparison Table

| | **RabbitMQ** | **Redis Pub/Sub** | **Kafka** |
|--|--------------|-------------------|-----------|
| **Storage** | Until consumer takes it | **None** | **Persistent (days)** |
| **Consumer offline** | Message waits in queue | **Lost** | Re-reads when back |
| **Who gets message** | **One** consumer | **All** subscribers online | **All** consumer groups |
| **Latency** | ~1ms | **~0.1ms** (fastest) | ~2-5ms |
| **Throughput** | Medium | Very high | **Highest (millions/sec)** |
| **Use case** | Task distribution | Real-time routing | Event streaming + storage |

### When to use what?

| The Need | The Tool |
|----------|----------|
| One task, one worker (order, email, PDF gen) | **RabbitMQ** |
| Broadcast to servers NOW (chat routing) | **Redis Pub/Sub** |
| Reliable event log (audit trail, multiple consumers) | **Kafka** |

### Kafka Consumer Groups — Important Detail

**Within the same group** → workers split the load (like RabbitMQ):
```
Kafka Topic: "orders"
  │
  ▼
Consumer Group "order-processors"
  ├── Worker 1 → takes order 1
  ├── Worker 2 → takes order 2
  └── Worker 3 → takes order 3
```

**Between different groups** → each group gets a full copy (like Pub/Sub):
```
Kafka Topic: "orders"
  ├──▶ Group "order-processors"  → save to DB
  ├──▶ Group "analytics"          → compute stats
  └──▶ Group "notifications"      → send emails
```

**Same order processed 3 times for different purposes, but NOT saved to DB twice.**

### RabbitMQ Real Example — E-commerce Order

**Without queue (bad):**
```
Customer clicks "Buy"
  1. Save order       (100ms)
  2. Charge card      (2000ms)   ← slow
  3. Update inventory (200ms)
  4. Send email       (1500ms)   ← slow
  5. Notify warehouse (500ms)
                      ────────
                      5100ms → Customer waits 5 seconds!
```

**With RabbitMQ:**
```
Customer clicks "Buy"
  1. Save order       (100ms)
  2. Push to queues   (10ms)
  3. Return success   ← Customer sees success in 110ms! ⚡
       │
       ▼
   RabbitMQ
       ├── Queue "payments"  → Payment Worker → Stripe
       ├── Queue "emails"    → Email Worker   → SendGrid
       └── Queue "warehouse" → Warehouse Worker
       (workers run in parallel in background)
```

**Benefits:**
- **Fast response** — customer doesn't wait
- **Failure isolation** — email service down doesn't fail order
- **Independent scaling** — email backed up? Add 5 email workers
- **Auto retry** — failed message goes back to queue
- **Dead Letter Queue (DLQ)** — messages that fail N times get parked for manual inspection

### Exchange Types (RabbitMQ)

| Type | Behavior | Example |
|------|----------|---------|
| **Direct** | 1:1 — routing key → specific queue | Order → order-processing queue |
| **Fanout** | 1:N — message goes to ALL bound queues | User signup → email + analytics + welcome |
| **Topic** | Pattern matching on routing key | `order.created.US` matches `order.created.*` |

---

## 16b. Webhooks vs Polling

**Question:** You want to know when something happens. Do you ask, or do they tell you?

### Polling — "I keep asking"

```
Client → Server: "any updates?" → "no"
Client → Server: "any updates?" → "no"
Client → Server: "any updates?" → "yes!"
```

**Client asks server every X seconds. Server returns current state.**

### Webhook — "They tell me"

```
Client → Server: "when event happens, POST to my URL"
Server: "registered"

...silence...

Server → Client: POST /webhook  "event happened!"
```

**Server initiates the request when event occurs.**

### Real Example — Stripe Payment

```javascript
// One-time setup in Stripe dashboard:
// Webhook URL: https://myapp.com/stripe-webhook

// Your server endpoint:
app.post('/stripe-webhook', (req, res) => {
  const event = req.body;
  
  if (event.type === 'payment.succeeded') {
    updateOrderStatus(event.data.order_id, 'paid');
  }
  
  res.status(200).send();
});
```

### Comparison

| | Polling | Webhook |
|--|---------|---------|
| **Who initiates** | Client | **Server** |
| **Direction** | Client → Server | **Server → Client** |
| **Latency** | Depends on interval | **Instant** |
| **Wasted requests** | Many (mostly empty) | **Zero** |
| **Public URL needed?** | No | **Yes** (server must reach client) |
| **Reliability if client down** | Client polls again later | Message can be **lost** (unless server retries) |

### When to use what?

- **Polling** — no webhook available, client behind firewall, simple checks
- **Webhook** — rare/unpredictable events (payment, PR opened), real-time critical, third-party integrations (Stripe, GitHub, Twilio, Slack, Shopify)

### Webhook Security

Public URL = anyone can send fake requests. **Solution: signature verification.**

```javascript
const signature = req.headers['stripe-signature'];
const event = stripe.webhooks.constructEvent(
  req.body, 
  signature, 
  process.env.STRIPE_WEBHOOK_SECRET
);
// Throws if signature invalid → request wasn't really from Stripe
```

### Polling vs Webhook vs WebSocket

| | Polling | Webhook | WebSocket |
|--|---------|---------|-----------|
| **Connection** | New request each time | New request each time | **Always open** |
| **Bi-directional** | No | No | **Yes** |
| **Client needs public URL** | No | **Yes** (if backend) | No |
| **Use case** | Simple checks | Third-party events | Chat, live updates |

---

## 16c. WhatsApp — Full System Design (putting it all together)

This example uses everything from Group 2: WebSocket, Pub/Sub, Kafka, DB, Push Notifications.

### Requirements

**Functional:** 1-on-1 chat, groups, delivery/read receipts, online/last seen, media, E2E encryption

**Non-functional:** low latency (ms), high availability (99.99%), message ordering, 2B users / 100B messages/day

### High-Level Architecture

```
┌──────────────┐          ┌──────────────┐
│  Salma 📱    │          │  Ahmed 📱    │
└──────┬───────┘          └──────▲───────┘
       │ WebSocket               │ WebSocket
       ▼                         │
┌────────────────────────────────────────┐
│           Load Balancer                │
└──────┬───────────────────▲─────────────┘
       │                   │
       ▼                   │
┌────────────┐      ┌────────────┐
│  Chat      │      │  Chat      │
│  Server 1  │◄────►│  Server 2  │
│  (Salma)   │Redis │  (Ahmed)   │
└─────┬──────┘Pub/Sub└─────┬──────┘
      │                    │
      ▼                    ▼
┌────────────────────────────────────────┐
│               Kafka                    │
└──────────────┬─────────────────────────┘
               │
      ┌────────┼────────┐
      ▼        ▼        ▼
┌────────┐┌────────┐┌────────┐
│Cassandra││Push    ││S3      │
│(Messages)││(FCM/APNs)│(Media) │
└────────┘└────────┘└────────┘
```

### Complete Message Flow — Salma sends "Hey!" to Ahmed

```
1. Salma types "Hey!" and hits send
      │
      ▼
2. Message travels through open WebSocket to Chat Server 1
      │
      ▼
3. Chat Server 1:
   ├── Generate message_id, add timestamp, encrypt (E2E)
   ├── Push to Kafka (fire-and-forget, ~2ms)
   └── ACK to Salma → shows ✓ (sent)     ← Fast response!
      │
      ▼
4. Kafka distributes to consumers IN PARALLEL:
   
   ┌── Consumer A: Save to Cassandra
   │       └── Message persisted forever
   │
   ├── Consumer B: Route to recipient
   │       ├── Ask Redis: "Where is Ahmed?"
   │       ├── Redis: "Ahmed is on Chat Server 2, online"
   │       ├── Publish to Redis Pub/Sub channel "ahmed"
   │       ├── Chat Server 2 (subscribed) receives it
   │       └── Server 2 pushes via WebSocket → Ahmed's phone
   │            └── Ahmed's phone ACKs → Salma sees ✓✓ (delivered)
   │
   └── Consumer C: Send Push Notification (ALWAYS!)
           └── FCM/APNs pushes to Ahmed's phone OS
                └── Phone decides: banner? silent? open chat?
```

### Why This Design?

| Component | Why? |
|-----------|------|
| **WebSocket** | Real-time bidirectional between phone ↔ server |
| **Kafka** | Reliability — never lose a message. Consumers can retry |
| **Redis Pub/Sub** | Speed — routing between servers in ~0.1ms |
| **Redis (Connection Manager)** | "Where is user X?" lookup for routing |
| **Cassandra** | 100B writes/day, horizontal scale (SQL can't handle this) |
| **S3** | Media storage (cheap, CDN, phone downloads directly) |
| **FCM/APNs** | Reach the phone even when app is closed |

### Kafka vs Pub/Sub — Both Used, Different Roles

- **Kafka** = "Save this message reliably, process it durably"
  - Consumers: save to DB, push notification, analytics
  - If a consumer crashes → reads from same position when back
- **Redis Pub/Sub** = "Route this to Server 2 RIGHT NOW"
  - Speed > reliability (message is already safe in Kafka)
  - Fire and forget

### Offline User — What Happens?

```
Ahmed is offline. Salma sends "Hey!"
   │
   ▼
Kafka Consumer B: "Where is Ahmed?" → Redis: "offline"
   │
   ▼
Skip Pub/Sub (nobody would receive it anyway)
   │
   ▼
Message is safe in Cassandra (Consumer A saved it)
   │
   ▼
Push Notification sent via FCM (Consumer C)
   │
   ▼
Ahmed opens app hours later:
   ├── WebSocket connect to some server (say Server 3)
   ├── Server 3: SELECT * FROM messages 
   │            WHERE to='ahmed' AND delivered=false
   ├── Send all pending messages via WebSocket
   └── Update status → delivered → Salma sees ✓✓
```

### Push Notification is NOT a Replacement

WhatsApp sends push **even when user is online**. Why?
- **WebSocket** delivers into the app (shows in chat instantly)
- **Push** delivers to the phone's OS (banner, sound, lock screen)

**The phone decides what to show:**
- App open + in this chat → silent (message already visible)
- App open + in different chat → banner at top
- App closed → full notification + sound

### Read Receipts

| Status | Trigger | Icon |
|--------|---------|------|
| SENT | Server ACKed to sender | ✓ |
| DELIVERED | Recipient's phone ACKed | ✓✓ |
| READ | Recipient opened the chat | ✓✓ (blue) |

### Group Chat

Small groups (WhatsApp max 1024): **fan-out on write** — copy the message per member, each with independent delivery status.

Massive groups (Telegram 200K+): **fan-out on read** — store once, each member reads from group timeline.

### E2E Encryption

```
Salma encrypts with Ahmed's PUBLIC key
     │
     ▼
Server sees: "x8f2k$#@!..." (unreadable)
     │
     ▼
Ahmed decrypts with his PRIVATE key → "Hey!"
```

**Even WhatsApp servers can't read messages.** They just route encrypted blobs.

### Interview One-Liner

> "WhatsApp uses **WebSocket** for real-time delivery between phone and Chat Server, **Kafka** for reliable async processing (save to DB, push notifications, analytics), **Redis Pub/Sub** for fast routing between chat servers when the recipient is on a different server, **Cassandra** for high-write message storage (100B writes/day), **S3** for media, **FCM/APNs** for push notifications that work even when the app is closed, and **E2E encryption** so even the server can't read messages."

---

## 17. API Design Best Practices

### Design Process
1. Identify core use cases and user stories
2. Define scope and boundaries
3. Determine performance requirements
4. Consider security constraints

### Design Approaches
- **Top-down** — start from high-level requirements, design API, then implement
- **Bottom-up** — start from existing data models, expose capabilities
- **Contract-first** (recommended) — define API contract (OpenAPI/Swagger spec) before writing any code. Frontend and backend develop in parallel

### Essential Practices

1. **Use HTTPS everywhere** — never expose API over plain HTTP
2. **Authenticate** — API keys, JWT, OAuth (covered in Auth notes)
3. **Rate limiting** — prevent abuse (429 Too Many Requests)
4. **Idempotency** — see deep dive below
5. **Versioning** — URL path (`/v1/`) or header (`Accept: application/vnd.api.v1+json`)
6. **Pagination** — never return unbounded lists. Use cursor-based or offset-based
7. **Consistent error format:**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Email is required",
    "field": "email"
  }
}
```

### Idempotency — Deep Dive

**Definition:** Sending the same request N times has the **same effect** as sending it once. No duplicate side effects.

**Why it matters — the classic bug:**
```
Salma pays $100
   │
   ▼
POST /api/pay { amount: 100 }
   │
   ▼
Internet slow... timeout ❌
Salma: "it failed, let me retry"
   │
   ▼
POST /api/pay { amount: 100 }   ← same request twice!
   │
   ▼
Server charged $200! 😱
```

**Solution — Idempotency Key:**
```
POST /api/pay 
Headers: Idempotency-Key: abc-123
Body:    { amount: 100 }
   │
   ▼
Server:
  1. Check DB: "Have I seen key abc-123 before?"
  2. NO  → execute payment, save key + result
  3. YES → skip payment, return the saved response
```

Retry with the **same key** N times → charged **once**, returns same response every time.

**Which HTTP methods are idempotent?**

| Method | Idempotent? | Why? |
|--------|-------------|------|
| **GET** | ✓ | Read-only, no changes |
| **PUT** | ✓ | `PUT /users/123 {name:"Salma"}` — same result every call |
| **DELETE** | ✓ | Already deleted → deleting again = no-op |
| **PATCH** | ⚠️ Depends | `set x = 5` idempotent. `increment x` NOT idempotent |
| **POST** | ✗ | Each call creates something new (order, payment) |

**POST is the problem** — needs an Idempotency-Key header for sensitive operations.

**Real-world usage:**
- **Stripe** — every `/v1/charges` accepts `Idempotency-Key` header
- **Uber** — prevents double-booking the same ride
- **Amazon** — prevents duplicate orders from double-clicks

---

## Group 2 — Interview Questions

1. What is an API? Why is it important in system design?
2. REST vs GraphQL — when would you choose each?
3. What makes an API RESTful? Name the constraints.
4. What is over-fetching and under-fetching? How does GraphQL solve them?
5. Explain the N+1 problem in GraphQL and how to fix it.
6. What is idempotency? Which HTTP methods are idempotent?
7. How do WebSockets work? How do they differ from HTTP?
8. When would you use polling vs WebSocket vs SSE?
9. What is a message queue? Why use one instead of direct calls?
10. Fanout vs Direct vs Topic exchange — explain each.
11. How do you handle API versioning?
12. What is rate limiting and why is it important?
13. How does GraphQL handle real-time data?
15. What is a Dead Letter Queue?

---
