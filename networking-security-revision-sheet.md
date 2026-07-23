# 🎯 Interview Revision Sheet — Networking, OS, Linux & Security (§6 + §7)

> Fast-recall sheet. Built from my own notes.
> ⚠️ = correction/clarification vs. my original notes.
> 🔑 = interview keyword the interviewer wants to hear.

---

## Section 6 — Networking, OS & Linux

### Cluster A — Networking Fundamentals

========================================================

## 1. OSI + TCP/IP model — `T1`

**🪝 OSI = 7-layer reference map · TCP/IP = 4-layer reality the internet runs on.**

| # | OSI Layer | Protocol | Unit (PDU) | Device |
|---|---|---|---|---|
| 7 | Application | HTTP, DNS, FTP, SMTP | data | — |
| 6 | Presentation | **TLS**, encoding | data | — |
| 5 | Session | — | data | — |
| 4 | Transport | **TCP / UDP** | **Segment** | L4 LB |
| 3 | Network | **IP** | **Packet** | **Router** |
| 2 | Data Link | **MAC**, Ethernet | **Frame** | **Switch** |
| 1 | Physical | cable, radio | **Bits** | Hub |

**TCP/IP (4 layers, actually used):**
```
Application / Presentation / Session ==> Application
Transport ===========================> Transport
Network =============================> Internet
Data link / Physical ================> Network Access
```

**Encapsulation flow (down = +header, up = −header):**
```
L7  App        |             DATA            |
L4  Transport  |[TCP hdr]    DATA            |  → SEGMENT
L3  Network    |[IP][TCP]    DATA            |  → PACKET
L2  Data Link  |[MAC][IP][TCP] DATA [CRC]    |  → FRAME
L1  Physical    101011010100011101...           → BITS ─► wire
```

**Mnemonic (L7→L1):** *All People Seem To Need Data Processing*

🪤 **Trap — MAC vs IP across hops:**
- Destination **IP stays constant** the whole journey (end-to-end logical)
- Destination **MAC changes at every hop** (link-local, next device only)
- 💡 **IP = whole journey, MAC = one hop**

⚠️ Don't say "the internet runs on OSI" → it runs on **TCP/IP**. OSI is the reference model.
⚠️ TLS straddles L5/L6 → say *"between transport and application"*, don't force one box.

⚡ **OSI = 7-layer map · TCP/IP = 4-layer reality · segment → packet → frame → bits.**

========================================================

## 2. TCP vs. UDP — `T1`

**🪝 TCP = phone call (connected, checked, ordered) · UDP = postcard (fire and forget).**

```
TCP:  [handshake] → [send+ack, send+ack, ...] → [teardown]
                       ↑ ordered, reliable, slower

UDP:  [send] [send] [send] [send]
                       ↑ no setup, no ack, fast, some may vanish
```

| | **TCP** | **UDP** |
|---|---|---|
| Connection | **Connection-oriented** | **Connectionless** |
| Delivery | **Guaranteed** (ack + retransmit) | **Best-effort** |
| Order | **Ordered** (sequence numbers) | No guarantee |
| Speed | Slower (overhead) | **Faster** |
| Header | ~20 bytes | ~8 bytes |
| Flow/congestion control | ✅ | ❌ |
| Use case | bank transfer, file download, HTTP, DB | video call, gaming, live stream, DNS |

🔑 **Reliability mechanics (TCP only):** sequence numbers + acknowledgments + retransmission + flow control.

🪤 **Trap — "just use TCP everywhere, it's reliable":** in a live video call, retransmitting a *lost old frame* stalls the stream — you want the **next** frame, not the missing one.
💡 **Pick TCP for correctness, UDP for freshness.**

⚡ **TCP = handshake + ack + retransmit (reliable, slower) · UDP = fire-and-forget (fast, no guarantee).**

========================================================

## 3. TCP three-way handshake — `T2`

```
Client                          Server
  |──────── SYN ──────────────►|   "Can we talk? Here's my starting sequence #."
  |◄─────── SYN-ACK ───────────|   "Yes, and here's mine — I heard you."
  |──────── ACK ───────────────|   "Confirmed. Connection open."
  |◄══════ data flows ════════►|
```

- **SYN** = synchronize → each side sends a random **initial sequence number**
- **Why 3, not 2?** 2 messages prove **one direction** only. The 3rd ACK proves **bidirectional**.
- ⚠️ **Teardown is separate & 4-step:** FIN → ACK → FIN → ACK
- ⚠️ This handshake is *why* HTTPS is slow to first byte: **TCP handshake + TLS handshake stacked**

⚡ **2 messages prove one direction; 3 prove both.**

========================================================

## 4. Sockets & ports — `T2`

**Socket = IP + Port + Protocol** ⚠️ (protocol matters — TCP:443 ≠ UDP:443)

```
IP address = "which building?"    Port = "which apartment?"
142.250.190.14  :  443    →  one SOCKET (one endpoint)
A connection = 2 sockets: (client IP:port) ⇄ (server IP:port)
```

**Well-known ports:** 80 HTTP · 443 HTTPS · 22 SSH · 53 DNS · 5432 Postgres · 27017 Mongo · 6379 Redis

🪤 **Trap — "one server has only port 443, how does it serve 10,000 users?"**
Connections are identified by the **4-tuple** (src IP, src port, dst IP, dst port). Server port is shared; each client's side is unique.
💡 **Server port is shared; the connection is unique by the 4-tuple.**

========================================================

### Cluster B — The Web Stack

========================================================

## 5. HTTP — `T1`

**🪝 Stateless request/response contract: client asks with method + URL, server answers with status + body.**

- HTTP runs on **TCP**
- **Stateless** = each request is independent, server keeps no memory → 🔑 **this is what makes horizontal scaling easy**
- State faked back in via **cookies / tokens**

| Method | Job | Safe? | Idempotent? |
|---|---|---|---|
| GET | read | ✅ | ✅ |
| POST | create | ❌ | ❌ |
| PUT | full replace | ❌ | ✅ |
| PATCH | partial update | ❌ | ❌ |
| DELETE | remove | ❌ | ✅ |

⚠️ **Safe** = doesn't change state · **Idempotent** = same result if repeated. Matters for **retry safety** (POST needs an idempotency key).

**Status groups:** 2xx success · 3xx redirect · 4xx client fault · 5xx server fault
- **401 → unauthenticated** ("I don't know you")
- **403 → unauthorized** ("I know you, answer is no")
- 409 conflict · 422 unprocessable · 502 bad gateway

⚡ **HTTP = stateless (method + URL + headers + body) → (status + headers + body). 401 = who are you, 403 = you can't.**

========================================================

## 6. HTTP/1.1 vs 2 vs 3 (QUIC) — `T2`

```
HTTP/1.1 : [req]→[res] then [req]→[res]   one at a time (head-of-line blocking)
HTTP/2   : [req|req|req] all at once ═══► multiplexed over ONE TCP conn
                                          (1 lost packet stalls all — TCP HOL)
HTTP/3   : [req|req|req] over QUIC/UDP  ► multiplexed, 1 lost packet
                                          stalls only ITS stream
```

**My HTTP/3 example:**
```
Track 1 (HTML)  ─► [HTML 2] ─► [HTML 1] ─► BROWSER READS INSTANTLY ✅
Track 2 (CSS)   ─► [CSS 2]  ─► [❌ LOST CSS 1] 🏎️ retrying ─► frozen ⏳
Track 3 (Image) ─► [IMG 2]  ─► [IMG 1]  ─► BROWSER READS INSTANTLY ✅
```

| | 1.1 | 2 | 3 |
|---|---|---|---|
| Transport | TCP | TCP | **QUIC over UDP** |
| Parallel reqs | ❌ | **Multiplexed** | **Multiplexed** |
| Head-of-line block | app layer | **still at TCP layer** | **gone** |
| Headers | plain text | **HPACK compressed** | QPACK |

🪤 **Trap:** *"HTTP/3 uses UDP so it's unreliable"* → **QUIC rebuilds reliability per-stream** on top of UDP, and folds in the TLS handshake for faster setup.

⚡ **1.1 = one lane · 2 = many lanes, one road · 3 = many independent lanes (QUIC/UDP).**

========================================================

## 7. DNS resolution — `T1`

**🪝 The internet's phonebook: name → IP.**

```
You type youtube.com
   │
   ▼ (check browser cache → OS cache → resolver cache FIRST) ⚠️
Resolver (recursive) ─► Root      → "ask the .com TLD"
                     ─► TLD (.com) → "ask youtube's authoritative"
                     ─► Authoritative → "142.250.x.x" ✅
   │
   ▼ cached at every level for TTL duration
```

- **Recursive** = resolver's job (keeps asking until it has the answer)
- **Iterative** = what root/TLD/authoritative do (they just **refer** you down one step)
- ⚠️ **DNS uses UDP by default, falls back to TCP** for large responses (zone transfers, DNSSEC)
- **Records:** A (IPv4) · AAAA (IPv6) · CNAME (alias) · MX (mail) · NS (nameserver)

🪤 **Trap:** *"the root server returns the IP"* → **NO.** Root only knows where the **.com TLD** is. Only the **authoritative** server holds the real record.

⚡ **DNS = referral chain (resolver → root → TLD → authoritative), cached by TTL, UDP-first with TCP fallback.**

========================================================

## 8. HTTPS / TLS handshake — `T1`

**🪝 HTTPS = HTTP + TLS. Slow asymmetric crypto ONCE to agree a fast symmetric key — then symmetric for everything.**

```
ASYMMETRIC (slow, once)          SYMMETRIC (fast, all data after)
 public key locks       ───►      one shared session key
 private key unlocks               encrypts AND decrypts
 used to AGREE a key               used for the actual traffic
```

**Handshake steps:**
1. **ClientHello** — supported ciphers/versions
2. **ServerHello + Certificate** — server's **public key**, signed by a **CA**
3. **Client verifies the certificate** (trusted CA + domain match) ← 🔑 **authentication**
4. **Key exchange** → both derive a shared **session key**
5. **Finished** → switch to symmetric, real data flows

🔑 **TLS gives 3 things: encryption + integrity + authentication.**

🪤 **Trap:** *"why not asymmetric for everything?"* → far too **slow** for bulk data. It's a **performance** decision, not a security compromise.

⚠️ The **certificate doesn't encrypt** — it carries the public key and **proves identity**.
⚠️ Without the **CA system**, an attacker hands you *their* public key and you encrypt straight to them (secrecy with no idea who you're secret *with*).

⚡ **TLS = asymmetric (once, key exchange) + symmetric (always, data) + CA cert (identity).**

========================================================

## 10. Load balancer types (L4 vs L7) — `T2`

| | **L4 (Transport)** | **L7 (Application)** |
|---|---|---|
| Routes by | **IP + port only** (content-blind) | **URL path, host header, cookies** |
| Speed | **Faster** | Slower (parses request) |
| SSL termination | ❌ | ✅ |
| Path-based routing | ❌ | ✅ |
| Example | AWS NLB | AWS ALB, Nginx |

🔑 **"L4 or L7?" really means: "route on connection info, or on request content?"**
🔑 **SSL termination** = L7 LB does the TLS handshake so backends only see plain HTTP.

🪤 **Trap:** *"L7 is smarter, always use it"* → L4 is better for raw TCP traffic (game servers, DB proxies) where there's no HTTP semantics to read.

========================================================

### Cluster D — Linux

========================================================

## 23. Shell scripting + cron — `T3`

**Cron = time-based job scheduler.** Runs unattended, survives reboots, no session needed.

```
*  *  *  *  *  command
│  │  │  │  └── day of week (0-6, Sun=0)
│  │  │  └───── month (1-12)
│  │  └──────── day of month (1-31)
│  └─────────── hour (0-23)
└────────────── minute (0-59)
```

| Expression | Meaning |
|---|---|
| `0 * * * *` | every hour |
| `0 0 * * *` | daily at midnight |
| `*/15 * * * *` | every 15 min |
| `0 9 * * 1` | Mondays 9am |

**Use case (strongest interview answer): nightly DB backup**
```
0 2 * * * pg_dump mydb > /backups/mydb_$(date +\%Y\%m\%d).sql
```
Also: log rotation, cache warming, expired-session cleanup, weekly reports.

**Manage:** `crontab -e` (edit) · `crontab -l` (list)
⚠️ **Classic bug:** cron runs with a **minimal environment** (no shell profile, limited `PATH`) → "works manually, fails under cron." Fix = absolute paths.

⚡ **Cron = 5 time fields + command, scheduled by the OS, not a long-running process.**

---

## Section 7 — Security

### Cluster E — Identity: Authentication & Tokens

========================================================

## 24. Authentication vs. Authorization — `T1`

```
Request ──► [ Authentication ] ──► [ Authorization ] ──► Resource
              "who are you?"        "can you do this?"
              fails → 401           fails → 403
```

- **AuthN** = who you are → password, token, biometric, session, OAuth
- **AuthZ** = what you're allowed to do → RBAC, ABAC, ACL, PBAC
- 🔑 **Order is fixed:** can be authenticated but NOT authorized. Never the reverse.

**Authorization models:**

| Model | Decides by | Code shape |
|---|---|---|
| **RBAC** (Role) | fixed role → permission set | `if user.role == "admin": allow()` |
| **ABAC** (Attribute) | user + resource + context attrs, live | `if user.dept == resource.dept and is_business_hours(): allow()` |
| **ACL** (Access Control List) | list attached **to the resource** | `if resource.acl.has(user.id, 'WRITE'): allow()` |
| **PBAC** (Policy) | central policy engine | `if iam.evaluate(request): allow()` |

**PBAC — what's in the request & the policy:**
```json
request = { "user": "alice", "action": "s3:GetObject",
            "resource": "arn:aws:s3:::reports/q3.docx",
            "context": {"time": "14:00", "ip": "10.0.0.5"} }

policy  = { "Effect": "Allow", "Principal": "alice",
            "Action": "s3:GetObject", "Resource": "arn:aws:s3:::reports/*" }
```
⚠️ In RBAC/ABAC **your app code** does the check. In PBAC the app **asks an external engine** — that's the real difference.
⚠️ Laravel "Policies" are **not** true PBAC — they're RBAC/ACL logic organized per-model **inside** the app.

🪤 **Trap:** logged-in user blocked from deleting someone else's post = **403 (authorization)**, not 401.

⚡ **AuthN = who you are (401) · AuthZ = what you can do (403) · always in that order.**

========================================================

## 25. Hashing vs Encryption vs Encoding — `T1`

| | Reversible? | Needs key? | Purpose | Example |
|---|---|---|---|---|
| **Encoding** | ✅ (anyone) | ❌ | change **format** | Base64, URL encoding |
| **Encryption** | ✅ (with key) | ✅ | **confidentiality** | AES, RSA |
| **Hashing** | ❌ never | ❌ | **integrity / storage** | SHA-256, bcrypt |

- Hashing → fruit → smoothie, **no way back**
- Encryption → locked suitcase, key opens it
- Encoding → Morse code, anyone who knows it reads it

**Where each is used:**
```
hashing    → bcrypt     ==> passwords
encryption → AES, RSA   ==> TLS
encoding   → Base64     ==> JWT header + payload
```

**Decision test:** need the original back?
→ yes + no secret = **encoding** · yes + secret = **encryption** · never = **hashing**

🪤 **Trap:** *"JWT is signed so the payload is secret"* → **NO.** Payload is only **Base64-encoded** and readable by anyone. Signing = **integrity**, not confidentiality.

⚠️ Never say "we encrypted the passwords" → passwords are **hashed** (you never need them back).

⚡ **Encoding = reversible, no key · Encryption = reversible, needs key · Hashing = one-way, never back.**

========================================================

## 26. Password hashing (bcrypt, salting) — `T1`

**Before (broken):** `SHA256(pass) == stored_hash` → same input = same output
→ billions of guesses/sec + **rainbow tables** (`|12345|afaodljso...|`)

**Fix: salt + deliberate slowness** → bcrypt / Argon2 / scrypt

| | **SHA-256** | **bcrypt** |
|---|---|---|
| Speed | **fast** (billions/sec on GPU) | **deliberately slow** (tunable cost factor) |
| Salt | ❌ not built in | ✅ **automatic** |
| Built for | file integrity, checksums, HMAC | **passwords** |

**⚠️ What bcrypt actually stores — salt is INSIDE the string:**
```
$2b$12$N9qo8uLOickgx2ZMRZoMye6i7z2u8V.4S/8ZQhpJ8mZBZi9Sn0IJC
 │   │  └────── salt ──────┘└────────── hash ──────────┘
 │   └── cost factor (how slow)
 └── algorithm version
```
Login = `bcrypt.compare(sentPass, storedHash)` → the library **extracts the salt from the stored string**, re-hashes, compares. You never manage the salt manually.

🔑 **Salt** = random per user → identical passwords produce **different hashes** → kills rainbow tables.
🔑 **Cost factor** = raise it over time as hardware gets faster.
⚠️ Salt is **not secret** — it just needs to be **unique**. Don't encrypt it.

**Use case:** SHA → files/checksums/HMAC · bcrypt → passwords, always

⚡ **Password hashing = bcrypt/Argon2 (slow by design) + unique salt, compared by re-hashing — never decrypted.**

========================================================

## 27. Sessions vs JWT vs OAuth2 — `T1`

**My analogy:**
- **Session** → *the other knows me* → y3m na 3omda
- **JWT** → *I introduce myself* → "I'm Omda, ITI, 23 y.o"
- **OAuth** → a third party who knows us both introduces me to the second person

```
SESSION:  client [session_id] ──► server LOOKS IT UP in session store
JWT:      client [full signed token] ──► server just VERIFIES signature (no lookup)
OAUTH2:   app ──"let this user in via Google"──► provider ──► token
```

| | **Session** | **JWT** | **OAuth2** |
|---|---|---|---|
| Data lives | **server** (store) | **in the token** | delegated to provider |
| Server work | **DB/cache lookup** | signature verify only | n/a |
| Scaling | needs shared store | **stateless** ✅ | n/a |
| Revocation | **easy** (delete row) | **hard** (wait for exp / blocklist) | depends on provider |

⚠️ **OAuth2 is authorization, NOT authentication.** It answers *"what can this app access."*
**OIDC** (OpenID Connect) sits on top and adds identity via an **ID token** (itself a JWT).

**Session ID storage:**

| Where | Verdict |
|---|---|
| **RAM (local)** | fast, dies on restart, ❌ breaks with multiple servers |
| **Cache (Redis)** | fast + shared across servers → ✅ **industry standard** |
| **DB** | slower, survives restart, fine if you don't want Redis |

🔑 Local RAM breaks the moment a **load balancer** sends you to a different server.

**OAuth2 flow (Sign in with Google → Claude):**
```
1. Click "Sign in with Google" on Claude
2. Claude redirects your BROWSER to accounts.google.com
3. You log into GOOGLE directly (Claude never sees the password)
4. Google consent screen: "Claude wants: your name, email" → Allow
5. Google redirects back to Claude with a short-lived CODE
6. Claude's SERVER exchanges code + client_secret with Google (server-to-server)
7. Google returns an ID TOKEN (JWT): { "email": "...", "name": "Mo", "sub": "..." }
8. Claude reads it, knows it's really you, logs you in — no Claude password ever existed
```

**What each piece protects:**

| Piece | Why |
|---|---|
| Login on Google's page | Claude **never touches your password** |
| Short-lived **code** | passed via browser — useless without the secret |
| **client_secret** exchange | proves it's really Claude, not an impostor |
| **Scopes** | only what you approved — not your whole account |
| Revocable | remove access anytime, password unchanged |

🔑 **Best one-liner:** *"scoped, revocable, verified-recipient-only permission — without handing over my password."*
⚠️ "no password" is only **1 of 3** benefits. Scope + revocability are the other two.

🪤 **Trap:** *"just delete the JWT to log them out"* → you **can't**. It's client-held and valid until expiry.

⚡ **Session = server remembers · JWT = client carries proof · OAuth2 = delegated authorization (OIDC adds identity).**

========================================================

## 28. JWT internals & full auth flow — `T1`

```
header.payload.signature
eyJhbGc...  . eyJzdWIi...  .  SflKxwRJ...
(Base64)      (Base64)        (HMAC of header+payload, using SECRET)

header    = {"alg":"HS256","typ":"JWT"}                    ← Base64 (READABLE)
payload   = {"sub":"42","role":"admin","exp":1234}         ← Base64 (READABLE)
signature = HMAC-SHA256(header + "." + payload, SECRET)    ← the ONLY protected part
```

🔑 **Verification, not decryption:** server **recomputes** the signature with its `.env` secret and checks it **matches**. Edited payload → recomputed signature won't match → rejected. **No DB lookup.**

**Access vs Refresh:**

| | **Access token** | **Refresh token** |
|---|---|---|
| Lifespan | short (~15 min) | long (days/weeks) |
| Sent on | **every** request | only when access expires |
| Purpose | prove identity per-request | get a new access token |
| Revocable? | ❌ (wait for exp) | ✅ **checked against DB/Redis** |
| Storage | httpOnly cookie or memory | **httpOnly cookie only** |

**Refresh flow when access token fails:**
```
1. Request with access token → signature OK but exp PASSED → 401 "expired"
2. Client interceptor catches the 401
3. Client POSTs the refresh token to /auth/refresh
4. Server checks: (a) signature (b) expiry (c) ⚠️ REVOCATION in DB/Redis
5. All pass → sign a NEW access token
6. Client retries the ORIGINAL request — invisible to the user
```
⚠️ This is a **separate request cycle**, not a branch inside the same request.
⚠️ **Rotation (senior detail):** issue a new refresh token each use and invalidate the old one. If an already-used refresh token reappears → treat as **theft**, revoke the whole token family.

⚠️ Invalid signature → **401** (not 403).
🪤 **Trap:** JWT is **encoded, not encrypted** → never put passwords/secrets/PII in the payload.
⚠️ Encrypted variant exists (**JWE**) vs. normal signed (**JWS**) — rarely needed; nested sign+encrypt only for genuinely sensitive claims crossing multiple services.

⚡ **JWT = header.payload.signature · payload readable · signature = integrity · short access + long revocable refresh.**

========================================================

## 29. Token storage (localStorage / sessionStorage / httpOnly cookies) — `T1`

### 🍪 Cookies
- **Oldest** of the three, made for client ↔ server communication
- **Sent automatically** to the same domain with every request → small overhead on every call
- Holds credentials/tokens → this auto-sending is *why* it's used for auth
- **Size:** ~4KB · **Expiration:** set at creation (persistent or dies with browser)
- **Must-do:** `HttpOnly` → JS can't read it → protects from **XSS** theft

### 💾 Local Storage
- **Client-side only**, never sent to server automatically
- **Size:** ~5–10MB · **Expiration:** none (until deleted / incognito close)
- **Use case:** dark/light theme, non-sensitive prefs

### 🗂️ Session Storage
- **Client-side only**, never sent automatically
- **Size:** ~5MB · **Expiration:** dies when **that tab** closes
- **Use case:** temporary UI state ("was this popup shown?") — **not credentials**

**The security line:**
```
httpOnly cookies    → immune to XSS reading, VULNERABLE to CSRF
local/session store → immune to CSRF, VULNERABLE to XSS
```

⚠️ **localStorage vs sessionStorage is a LIFESPAN choice, not a security one** — both are equally JS-readable, both equally XSS-exposed. Only **httpOnly** removes JS's ability to read.
⚠️ **Why Web Storage is CSRF-immune:** CSRF works *only* because cookies are **auto-sent**. Web Storage is never auto-sent, and evil.com's JS can't read your origin's storage (Same-Origin Policy).
⚠️ **Never put a token in the URL** → lands in browser history, server logs, and the Referer header.

**Production setup:**
```
Refresh token → httpOnly + Secure + SameSite cookie
Access token  → same, OR in memory (JS variable, not localStorage)
```

⚡ **Web Storage = XSS-exposed, CSRF-immune · httpOnly cookie = XSS-safe, CSRF-exposed. No option is risk-free.**

========================================================

## 30. Cookie types & attributes — `T2`

```
Set-Cookie: token=abc123; Secure; HttpOnly; SameSite=Lax; Max-Age=3600
```

| Attribute | Does what |
|---|---|
| **Secure** | only sent over **HTTPS** — enforced on **every request**, not just at creation |
| **HttpOnly** | invisible to JS (`document.cookie` can't see it) → blocks XSS theft |
| **SameSite** | controls sending on **cross-site** requests → CSRF defense |
| **Max-Age / Expires** | makes it **persistent** (survives browser restart) |
| *(none of the above)* | **session cookie** ⚠️ — dies when the **browser** closes |

⚠️ **Session cookie dies on BROWSER close.** (`sessionStorage` dies on **tab** close — different things that just share the word "session," with zero shared machinery.)

**Combinations:**
```
Set-Cookie: token=xyz; HttpOnly              ← session + httpOnly
Set-Cookie: token=xyz; HttpOnly; Max-Age=999 ← persistent + httpOnly
Set-Cookie: token=xyz                        ← session + readable by JS
Set-Cookie: token=xyz; Max-Age=999           ← persistent + readable by JS
```
⚠️ Session-vs-persistent = **lifespan** axis. HttpOnly-vs-readable = **security** axis. Independent, they combine freely.

### ⏰ Max-Age vs JWT `exp` — two INDEPENDENT clocks
```
Cookie = the ENVELOPE   → Max-Age controls the envelope's life (BROWSER enforces)
JWT    = the LETTER      → exp controls the token's validity  (SERVER enforces)
```

| | Cookie `Max-Age` | JWT `exp` |
|---|---|---|
| Enforced by | **browser** | **server** |
| At expiry | browser **deletes** cookie, stops sending | server **rejects**, returns 401 |

**Mismatch cases:**
- `Max-Age` **>** `exp` → normal ✅ (server expiry is the real gate; refresh flow handles it)
- `Max-Age` **<** `exp` → ⚠️ **bug** — browser throws away a still-valid token → user logged out early for no security reason

🔑 **Rule: set `Max-Age` ≥ `exp`** (real systems set them equal, or omit Max-Age entirely).

⚡ **Max-Age = how long the browser keeps sending it · exp = how long the server accepts it.**

========================================================

### Cluster F — Attacks & Defenses

========================================================

## 31. XSS (Cross-Site Scripting) — `T1`

**🪝 The attacker's script runs AS the trusted site — not from another site, but INSIDE the real one.**

⚠️ **Key fact:** each origin sees only **its own** localStorage/sessionStorage. `evil.com`'s JS **cannot** read facebook.com's storage (Same-Origin Policy). XSS only works because the malicious script gets **injected into and executed by facebook.com itself**.

**Example:** you open Facebook, a post's comment contains a malicious `<script>`, the browser renders the page and **runs every `<script>` tag in it** — including the injected one.

| Type | Payload lives | Trigger |
|---|---|---|
| **Stored** | in the **DB** (comment, bio) | anyone who **views** the page — no click needed |
| **Reflected** | in the **URL / query string** | victim **clicks** a crafted link |
| **DOM-based** | never hits the server — client-side JS bug | page load / interaction |

⚠️ **Stored XSS is the most dangerous** — one payload, every future visitor.
⚠️ Why the attacker sends data to `evil.com`: the theft happens **on facebook.com**; evil.com is just the **drop-off mailbox** for exfiltration.

**What a payload can do:** read `document.cookie` (if not httpOnly) · read local/sessionStorage · make **authenticated requests as you** · deface / keylog / redirect

**Defenses (in order of impact):**
1. **Output encoding** → `<script>` renders as `&lt;script&gt;` (text, not code)
2. **CSP** → run scripts only from whitelisted sources, block inline scripts
3. Avoid **`eval()`, `innerHTML`, `document.write()`** on untrusted input
4. **Sanitize rich input** (DOMPurify) if HTML must be allowed
5. **httpOnly** → only **limits damage** to one cookie, ⚠️ does **not** prevent XSS

🪤 **Trap:** *"we escape input, so we're safe"* → escaping must match the **context** (HTML body vs attribute vs JS string vs URL). One missed context reopens the hole.
⚠️ React/Vue auto-escape — but `dangerouslySetInnerHTML` / `v-html` reopen it.

⚡ **XSS = untrusted input becomes executable code ON the trusted site. Fix at the injection point (encoding + CSP), not the storage layer.**

========================================================

## 32. CSRF (Cross-Site Request Forgery) — `T1`

**🪝 CSRF never sees your cookie — it just tricks your browser into USING it.**

```
Monday:  You log into facebook.com → cookie stored → you CLOSE the tab

Tuesday: NO Facebook tab open anywhere. You land on evil.com.
         evil.com auto-fires on page load:
           <form action="https://facebook.com/transfer-money" ...>
           <script>document.forms[0].submit()</script>
         Browser: "request going TO facebook.com — do I have its cookie?" → YES
         → attaches it automatically → request succeeds as if you did it
```

⚠️ **No open tab needed.** Only a **valid cookie sitting in the browser** from any earlier login.
⚠️ **No click needed** beyond landing on the malicious page — it auto-submits on load.
🔑 **Only cookies are vulnerable, because only cookies are sent automatically.**

### Solution 1 — SameSite

| | Triggered FROM own site | Link click FROM evil.com | Hidden POST FROM evil.com |
|---|---|---|---|
| **None** | ✅ sent | ✅ sent | ✅ sent → **attack works** 💀 |
| **Lax** | ✅ sent | ✅ sent (top-level nav) | ❌ blocked ✅ |
| **Strict** | ✅ sent | ❌ blocked | ❌ blocked |

🔑 **Lax rule:** *did the address bar change because I clicked a real link?*
→ **yes** = sent · **no** (POST, fetch, form, image, iframe, background) = **blocked**
→ **Lax is the modern browser default** — blocks the dangerous case, allows the useful one.

⚠️ **SameSite never changes the cookie's domain scoping.** A facebook.com cookie only ever travels **to facebook.com**. SameSite only controls **which site is allowed to trigger** that request.
⚠️ `SameSite=None` **requires** `Secure`. Only for legitimate third-party embeds (SSO, payment widgets).

### Solution 2 — CSRF token
Server generates a random token, **embeds it in its own page's HTML**, and stores it server-side per session:
```html
<form action="/delete-comment" method="POST">
  <input type="hidden" name="csrf_token" value="aZ7qW3xR...">
  <input type="hidden" name="comment_id" value="482">
  <button>Delete this comment</button>
</form>
```
**Why evil.com can't forge it:**
- The cookie is **auto-attached** → attacker doesn't need to know it
- The CSRF token is **NOT auto-attached** → must be read out of the real page's HTML
- ⚠️ **Same-Origin Policy blocks** evil.com's JS from **reading** facebook.com's response body → it can trigger the request but can never supply the token

🔑 **Cookie proves "this browser is logged in." CSRF token proves "this request came from the site's own page."**

🪤 **Trap:** *"our API only accepts JSON so we're CSRF-safe"* → helps, but not standalone. Pair with SameSite + CSRF tokens.
⚠️ Never use **GET** for state-changing actions — a plain `<img src>` triggers it.

⚡ **CSRF = attacker's site auto-triggers a request, browser auto-attaches the real cookie. Fix: SameSite + CSRF tokens.**

========================================================

## 33. SQL Injection — `T1`

**🪝 Same root cause as XSS: user input becomes the query STRUCTURE instead of data.**

```
query = "SELECT * FROM users WHERE username = '" + input + "'"

input = "mo"            → SELECT ... WHERE username = 'mo'            ✅ fine
input = "' OR '1'='1"   → SELECT ... WHERE username='' OR '1'='1'     ⚠️ ALWAYS TRUE → all users
input = "admin' --"     → SELECT ... WHERE username='admin' --'...'   → comments out the
                                                                        PASSWORD CHECK entirely
```

**Solution — parameterized queries:**
```javascript
// ❌ vulnerable
db.query(`SELECT * FROM users WHERE username = '${input}'`);

// ✅ safe
db.query('SELECT * FROM users WHERE username = ?', [input]);
```
🔑 **Why it works:** the SQL **structure** is sent and compiled **first, separately**. The value arrives **after**, purely as **data** — structurally impossible to change the query's shape.

⚠️ **ORMs are safe by default** because their normal methods (`.findOne`, `.where`, `.filter`) parameterize internally.
⚠️ **BUT** the raw-query escape hatch reopens it:
```javascript
sequelize.query(`SELECT * FROM users WHERE username = '${input}'`);  // ❌ same hole
```
🪤 **Trap:** *"we use an ORM so we're immune"* → only if **every** query goes through the ORM's query builder. One raw concatenated query = fully vulnerable.

⚠️ Don't rely on **escaping/blacklisting** `'` → encoding tricks and **second-order injection** route around it.

⚡ **SQLi = input treated as structure, not data. Fix = parameterized queries (structural), not escaping (patch).**

========================================================

## 34. CORS + Same-Origin Policy — `T1`

⚠️ **Correction:** the rule that stops a site reading another's responses is the **Same-Origin Policy** (always on, by default). **CORS is the SERVER deliberately relaxing it** for named origins — it's a **doorway**, not a wall.

```
DEFAULT (no CORS header):
  app.com JS ──fetch──► api.com  → server responds, but BROWSER
                                    blocks app.com's JS from READING it

WITH CORS:
  api.com responds with:
    Access-Control-Allow-Origin: https://app.com
  → browser lets app.com's JS read the response ✅
```

🔑 **The request is always SENT either way.** CORS only controls whether the **calling JS may read the response**.
⚠️ **This is why CORS is NOT a CSRF defense** — a forged request still executes server-side.

| | **Simple request** | **Preflighted request** |
|---|---|---|
| When | GET / POST, basic headers, standard content-types | **PUT, DELETE, custom headers, `Content-Type: application/json`** |
| Flow | sent directly, response checked | browser sends **`OPTIONS`** FIRST asking permission |

**Preflight:**
```
1. OPTIONS /api/data
     Origin: https://app.com
     Access-Control-Request-Method: PUT
2. Server: Allow-Origin / Allow-Methods / Allow-Headers
3. Allowed? → NOW send the real PUT.  Not allowed? → never sent at all.
```
🔑 Preflight exists so **side-effecting requests are checked BEFORE they happen**.

⚠️ **Credentials + CORS:** `credentials: 'include'` requires the **exact origin**, never `Allow-Origin: *`.
⚠️ **CORS doesn't apply to Postman / curl / server-to-server** — it's **browser-enforced only**, so it is **not** access control. Real protection = actual auth checks.

🪤 **Trap:** *"CORS secures our API"* → no. Anyone can hit the API directly and get a full response.

⚡ **SOP = default block · CORS = server opting specific origins in · controls response-READING, not request-sending.**

========================================================

### Cluster G — Framework & Hardening

========================================================

## 35. OWASP Top 10 (2025) — `T1`

Not a list of bugs — a ranked list of **categories of failure**, derived from **real breach data**.
`#1 = most commonly exploited / highest real-world impact · #10 = still serious, rarer as root cause`

⚠️ Finalized **January 2026** — first update since 2021. Reciting the 2021 list dates you instantly.

| # | Category | Note |
|---|---|---|
| A01 | **Broken Access Control** | now **absorbs SSRF** |
| A02 | **Security Misconfiguration** | ⚠️ jumped **#5 → #2** |
| A03 | **Software Supply Chain Failures** | ⚠️ **NEW** — deps + CI/CD pipelines |
| A04 | Cryptographic Failures | |
| A05 | Injection | ← SQLi (topic 33) lives here |
| A06 | Insecure Design | architecture flaws, not impl. bugs |
| A07 | Authentication Failures | ← all of Cluster E |
| A08 | Software/Data Integrity Failures | |
| A09 | Security Logging & **Alerting** Failures | |
| A10 | **Mishandling of Exceptional Conditions** | ⚠️ **NEW** — error handling |

🔑 **Strong answer:** *"the biggest shift is misconfiguration and supply chain now ranking above classic injection — real breaches trace to a forgotten setting or a compromised dependency more than a clever exploit."*

⚡ **OWASP = data-driven ranked categories, re-derived every few years — 2 new categories in 2025, misconfig #5→#2.**

========================================================

## 38. Securing a login application — end-to-end `T1`

**🪝 Every topic above is one layer of the same wall. No single control is enough.**

```
1. User submits credentials
   → over HTTPS ONLY (§8)
   → server-side input validation (never trust client-only)

2. Server checks password
   → bcrypt.compare() against salted hash (§26)
   → rate-limited: max N attempts per IP/account (§39)

3. Server issues tokens
   → access token (short exp) + refresh token (long exp, revocable)
   → signed with JWT_SECRET from a vault/.env (§37)

4. Tokens sent to client
   → httpOnly + Secure + SameSite cookies (§29-30)

5. Every subsequent request
   → server verifies signature + exp (§28)
   → CSRF token for state-changing actions (§32)

6. All rendered output
   → escaped/encoded, CSP enforced (§31)

7. All DB queries
   → parameterized, never concatenated (§33)
```

🪤 **Trap:** *"we hash with bcrypt and use JWT, so we're secure"* → covers password cracking + stateless auth only. Nothing about XSS, CSRF, brute force, or SQLi.
⚠️ **Most commonly forgotten layer: RATE LIMITING.** bcrypt slows each guess; only a rate limit stops **unlimited** guessing.
⚠️ Always say **HTTPS** out loud — skipping it reads as an oversight, not an assumption.

🔑 **Delivery tip:** don't list items — **narrate the request's journey** and name the control at each stage.

⚡ **Login security = HTTPS + bcrypt/salt + short access/revocable refresh + httpOnly/Secure/SameSite + CSP/encoding + parameterized queries + CSRF tokens + rate limiting + vault secrets.**

========================================================

## 40. Stateful vs Stateless

Named by **where the current session's data is remembered**.

### Stateful flow
```
1. Login (username/pass → server)
2. Verify (check credentials in DB)
3. Session Creation (server creates session in memory/DB → session_id)
4. Cookie handing (server sends cookie containing session_id)
5. Next req (browser auto-attaches the session_id cookie)
6. Lookup (server searches memory/DB — is it valid?)
7. Response
```
**Inside a cookie:** name & value, expiration, domain

⚠️ **Why it breaks at scale:** too many sessions in memory crashes servers, and with a **load balancer** Server B doesn't know Server A's session IDs → needs a **shared store** (Redis). That pressure is what produced tokens (JWT).

### Stateless flow
```
1. Login (username/pass → server)

2. Verify
   SHA:     SHA256(sentPass) == stored_hash
   bcrypt:  bcrypt.compare(sentPass, storedHash)   ← extracts salt from stored, re-hashes

3. Token Creation (access + refresh JWT)
   3.0 header:  {"alg":"HS256","typ":"JWT"}          // HS256 = HMAC
   3.1 payload: {"sub":"user123","role":"admin","exp":1234}   (access, short)
                {"sub":"user123","role":"admin","exp":1735689600} (refresh, long)
   3.2 signature = HMAC(header + "." + payload, SECRET in .env)
       NOTE: HMAC = hashing using an external secret only I know
             (not a random per-user salt like bcrypt)
   3.3 JWT = "header.payload.signature"  → sent to client

4. Client saves both in httpOnly cookies

5. Next req → attach JWT_access

6. Server validation
   6.1 expected_signature = HMAC(header + "." + payload, SECRET in .env)
   6.2 compare with the signature in the token (string compare)
       if equal   → check exp
                     ok      → serve the request
                     expired → 401 → silent refresh → retry the original request
       if not eq  → 401

7. Response
```

**Example JWT:**
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjo0MiwiZW1haWwiOiJhbGlAZXhhbXBsZS5jb20iLCJleHAiOjE3MTM5MTQ0MDB9.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

| Part | Contains |
|---|---|
| **Header** | algorithm used for the signature + type (JWT) |
| **Payload** | user data (id, name, exp) — **Base64-encoded, NOT encrypted.** Never put passwords/secrets here |
| **Signature** | header + payload + secret, mixed by the header's algorithm — must match on verify |

**Why two tokens?** You **can't invalidate a JWT** — once signed it's valid until it expires. Logout/ban has no row to delete. So access tokens stay **short-lived**: even if leaked, they die fast.

⚠️ Tampered token → **401** (not 403). 401 = signature/identity failed.

**The JWT logout problem:**
- **Sessions:** trivial — delete the row, the ID becomes meaningless
- **JWT frontend logout:** delete the token client-side → can't make requests **from this device**, but the token is still technically valid until expiry
- **JWT true logout:** blocklist the token in Redis/DB → but that reintroduces a **DB hit per request**, the very thing JWT avoided

⚡ **Stateful = server remembers (easy revoke, needs shared store) · Stateless = token carries proof (scales, hard to revoke).**

========================================================

## 41. Compromised user

**Stateful:** delete the session ID. Done.

**Stateless:**
```
Compromise detected
     │
     ▼
Refresh token → mark REVOKED in DB/Redis   ← the ONE thing you can control
     │
     ▼
Access token → do NOT blocklist it
               (would mean a DB check on EVERY request — kills statelessness)
     │
     ▼
Attacker's access token still works... but only until its own short exp
Attacker tries to refresh → refresh revoked → refresh FAILS
     │
     ▼
Attacker fully locked out within minutes
```

**What if the remaining window is enough to do damage?**
1. **Shorter exp** on the access token
2. **Step-up auth** for sensitive actions (money/admin) — re-verify per action, ~60s tokens
3. **Detect abuse during the window** — rate limiting, anomaly detection (IP change, impossible travel) → revoke immediately, don't wait for exp
4. **Reduce what a stolen token is worth** — narrow **scopes**, least privilege
5. **Prevent theft first** — HTTPS, httpOnly cookies, short-lived design

⚠️ **Honest framing:** short expiry doesn't **stop** the attack — it **caps the blast radius** after prevention already failed.
⚠️ Optional for high-security systems: blocklist access tokens by **`jti`** in Redis (cheap cache lookup, list stays small since they expire fast).

⚡ **Revoke the refresh token (your real control point) · let the access token expire naturally.**

---

## 🗂️ Not covered in this sheet
Cluster C (processes/threads, concurrency vs parallelism, race conditions, deadlock, thread pools, mutex vs semaphore, virtual memory, IPC) · Cluster D (Linux commands, `chmod`/`chown` permissions, `ps`/`top`/`kill`) · Topic 9 (WebSockets) · Topics 36–37, 39 (security headers/CSP, secrets management, rate limiting algorithms).

---

## ⚡ One-Line Recall Sheet

| # | Topic | Compressed |
|---|---|---|
| 1 | OSI/TCP-IP | 7-layer map vs 4-layer reality · segment→packet→frame→bits · IP = journey, MAC = hop |
| 2 | TCP vs UDP | handshake+ack+retransmit vs fire-and-forget · correctness vs freshness |
| 3 | 3-way handshake | 2 msgs prove one direction; 3 prove both · teardown is 4-step |
| 4 | Sockets/ports | socket = IP + port + protocol · connection = 4-tuple |
| 5 | HTTP | stateless · 401 = who are you, 403 = you can't · idempotent = retry-safe |
| 6 | HTTP versions | 1.1 one lane · 2 multiplexed on TCP · 3 QUIC/UDP, no global stall |
| 7 | DNS | referral chain root→TLD→authoritative · cached by TTL · UDP-first |
| 8 | TLS | asymmetric once (key exchange) + symmetric always (data) + CA cert (identity) |
| 10 | L4 vs L7 LB | IP:port (fast, blind) vs URL/header/cookie (smart, SSL termination) |
| 23 | Cron | 5 time fields + command · min hour dom month dow |
| 24 | AuthN/AuthZ | who you are (401) then what you can do (403) · RBAC/ABAC/ACL/PBAC |
| 25 | Hash/Encrypt/Encode | one-way · reversible-with-key · reversible-no-key |
| 26 | Password hashing | bcrypt = slow by design + auto salt (stored inside the hash string) |
| 27 | Session/JWT/OAuth | server remembers · client carries proof · delegated authorization |
| 28 | JWT | header.payload.signature · payload readable · signature = integrity |
| 29 | Token storage | Web Storage = XSS-exposed/CSRF-immune · httpOnly = reverse |
| 30 | Cookie attrs | Secure (HTTPS) · HttpOnly (no JS) · SameSite (cross-site) · Max-Age ≥ exp |
| 31 | XSS | input becomes code ON the trusted site · fix = encoding + CSP |
| 32 | CSRF | attacker triggers request, browser auto-attaches cookie · fix = SameSite + token |
| 33 | SQLi | input becomes query structure · fix = parameterized queries |
| 34 | CORS | SOP blocks by default; CORS opts origins in · controls READING, browser-only |
| 35 | OWASP 2025 | misconfig #5→#2 · supply chain + exceptional conditions are NEW |
| 38 | Secure login | layered: HTTPS + bcrypt + tokens + cookie flags + CSP + params + CSRF + rate limit |
| 40 | Stateful/Stateless | shared store + easy revoke vs scalable + hard revoke |
| 41 | Compromised | revoke refresh, let access expire |
