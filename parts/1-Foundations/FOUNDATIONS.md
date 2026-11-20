# Part 1: Foundations, Full Lesson

## 📌 Lacking / Previous Weakness

These three tripped you up last time, so this lesson goes deeper on them. Look for the 🎯 marker:

- 🎯 **Status codes**: knowing _why_ a code fits
- 🎯 **Idempotency**: what it is and how idempotency keys work
- 🎯 **TLS termination**: where encryption ends and what's trusted after that

**Suggested pacing:** Day 1 = Section A, Day 2 = Section B, Day 3 = Section C + assignment.

---

# Section A: Core Concepts

## A1. What system design is (and isn't)

**It is** deciding how the pieces of a system fit together so it keeps working as users, data, and traffic grow. You're choosing components (servers, databases, caches, queues) and how they talk to each other.

**It isn't** DSA, memorizing Netflix's architecture, or finding "the one right answer."

Every design is judged on **non-functional requirements**, the qualities beyond "does the feature work":

- **Scalability**: handles 10x traffic
- **Availability**: stays up when parts fail
- **Latency**: feels fast
- **Reliability**: doesn't lose or corrupt data
- **Consistency**: everyone sees the same data
- **Maintainability**: the team can change it safely

Improving one usually costs another, and those trade-offs are the real subject. Two levels:

- **HLD** (high-level design): services, databases, queues, and how they connect
- **LLD** (low-level design): classes, schemas, and code structure inside one component

## A2. Client-server model

- **Client**: starts the conversation (browser, mobile app, or another service)
- **Server**: waits, processes, and replies

Your stack is a chain of client-server pairs: browser → Next.js → NestJS → MongoDB/Redis. Each arrow is a **network call**, and network calls are slower and flakier than function calls.

Rough latency feel:
| Operation | Order of magnitude |
|---|---|
| Read from memory | ~100 nanoseconds |
| SSD read | ~100 microseconds |
| Round trip inside one data center | ~0.5 milliseconds |
| Round trip across continents | ~100–150 milliseconds |

You don't need exact numbers. The point is that **network hops dominate**, so fewer hops and fewer round trips usually means faster.

## A3. Request/response lifecycle

What happens when someone opens a property page:

```
1. Browser checks caches for the IP (browser → OS → resolver)
2. DNS lookup (if not cached)           → gets IP
3. TCP handshake                        → 1 round trip
4. TLS handshake (HTTPS)                → 1 round trip (TLS 1.3)
5. HTTP request sent                    → GET /properties/123
6. Load balancer picks a server
7. App runs: auth → validation → logic → DB query
8. HTTP response returned               → 200 + JSON
9. Connection kept open (keep-alive) or closed
10. Browser renders
```

Every step can be slow or fail. Most of system design is shaving time off these steps and surviving their failures.

## A4. IP addresses and ports

- **IPv4**: `104.21.5.12` (32-bit, running out). **IPv6**: longer hex format, practically unlimited.
- **Public IP**: reachable from the internet. **Private IP** (`10.x.x.x`, `192.168.x.x`, `172.16–31.x.x`): only inside a private network. Your database should live on a private IP.
- **Port**: which program on the machine gets the traffic. Common ones: `80` HTTP, `443` HTTPS, `5432` Postgres, `27017` MongoDB, `6379` Redis.
- A server is addressed as **IP:port**.

## A5. DNS

DNS translates `bhalobasha.com` into an IP address.

**Lookup chain:** browser cache → OS cache → your ISP's resolver → root server → TLD server (`.com`) → the domain's authoritative name server → IP returned and cached.

**Common records:**
| Record | Meaning |
|---|---|
| A | Domain → IPv4 |
| AAAA | Domain → IPv6 |
| CNAME | Domain → another domain (alias) |
| MX | Where the domain's email goes |
| TXT | Free-form text (verification, SPF) |
| NS | Which name servers are authoritative |

**TTL** says how long a record may be cached. That's why DNS changes "propagate" slowly: old answers sit in caches until their TTL expires. Tip: lower the TTL before a planned server migration.

## A6. TCP/IP basics

- **IP** moves packets between machines (routing). **TCP** sits on top and adds reliability.
- **TCP guarantees:** data arrives complete and in order. It does this with sequence numbers, acknowledgements, retransmission of lost packets, and flow/congestion control.
- **Three-way handshake:** client `SYN` → server `SYN-ACK` → client `ACK`. Only then does data flow. This costs **one round trip** before you can send anything.
- Closing a connection uses `FIN` packets.

Reliability has a price: handshakes, retransmits, and ordering all add latency. Keep that in mind for TCP vs UDP later.

## A7. HTTP and HTTPS

HTTP is a **text-based request/response protocol**, and it's **stateless**: the server doesn't remember you between requests unless you send something that identifies you.

**Request:**

```
POST /properties HTTP/1.1
Host: api.bhalobasha.com
Authorization: Bearer eyJhbGci...
Content-Type: application/json

{"title":"2BHK in Mirpur","rent":18000}
```

**Response:**

```
HTTP/1.1 201 Created
Content-Type: application/json
Location: /properties/abc123

{"id":"abc123","title":"2BHK in Mirpur"}
```

Parts: **method + path + headers + optional body** going out, **status code + headers + optional body** coming back. **HTTPS** is the same thing wrapped in TLS encryption.

## A8. HTTP methods

| Method  | Meaning                         | Safe? | Idempotent? |
| ------- | ------------------------------- | ----- | ----------- |
| GET     | Read                            | ✅    | ✅          |
| HEAD    | Headers only                    | ✅    | ✅          |
| OPTIONS | What's allowed (CORS preflight) | ✅    | ✅          |
| PUT     | Replace the whole resource      | ❌    | ✅          |
| DELETE  | Remove                          | ❌    | ✅          |
| POST    | Create / trigger an action      | ❌    | ❌          |
| PATCH   | Partial update                  | ❌    | Depends     |

- **Safe** = doesn't change server state. Caches and crawlers rely on this.
- **Idempotent** = doing it N times leaves the same end state as doing it once. (We'll go deep in Section B.)

## A9. 🎯 HTTP status codes

| Range | Meaning                   |
| ----- | ------------------------- |
| 2xx   | It worked                 |
| 3xx   | Look somewhere else       |
| 4xx   | **Client** made a mistake |
| 5xx   | **Server** failed         |

**The codes you must know, and _why_:**

| Code                        | Use it when                                                             |
| --------------------------- | ----------------------------------------------------------------------- |
| `200 OK`                    | Success, response has a body                                            |
| `201 Created`               | A new resource was created (add a `Location` header ideally)            |
| `202 Accepted`              | Received, but processing happens later (async job)                      |
| `204 No Content`            | Success, nothing to return (typical for DELETE)                         |
| `301 / 302`                 | Permanent / temporary redirect                                          |
| `304 Not Modified`          | Cached copy is still valid                                              |
| `400 Bad Request`           | Malformed request or failed validation                                  |
| `401 Unauthorized`          | **Not authenticated** ("I don't know who you are")                      |
| `403 Forbidden`             | **Authenticated but not allowed** ("I know you, and no")                |
| `404 Not Found`             | Resource doesn't exist                                                  |
| `409 Conflict`              | Clashes with current state (duplicate email, slot already booked)       |
| `422 Unprocessable Entity`  | Well-formed but semantically invalid (some teams use it for validation) |
| `429 Too Many Requests`     | Rate limit hit                                                          |
| `500 Internal Server Error` | Unhandled bug on our side                                               |
| `502 Bad Gateway`           | Proxy got a bad response from upstream                                  |
| `503 Service Unavailable`   | Overloaded or down for maintenance                                      |
| `504 Gateway Timeout`       | Proxy timed out waiting for upstream                                    |

**Quick decision flow when writing an endpoint:**

1. Is the input malformed or invalid? → `400`
2. Who are you? Missing/invalid token → `401`
3. Are you allowed? → `403`
4. Does the thing exist? → `404`
5. Does it clash with current state? → `409`
6. Too many requests? → `429`
7. Our bug? → `500`

**Judgment calls to know:**

- Sometimes you return `404` instead of `403` on purpose so you don't reveal that a private resource exists.
- Don't return `200` with `{ "success": false }` for errors. It breaks caches, monitoring, and clients.

**NestJS detail worth knowing:** `@Post()` returns **201 by default**, everything else returns **200**. So your `send-otp` endpoint returns `201 Created` even though nothing was created. Better:

```ts
@Post('send-otp')
@HttpCode(200)   // or 202 if the SMS is sent via a background job
sendOtp(@Body() dto: SendOtpDto) { ... }
```

Built-in exceptions map to codes: `BadRequestException` (400), `UnauthorizedException` (401), `ForbiddenException` (403), `NotFoundException` (404), `ConflictException` (409).

## A10. Headers, cookies, sessions

**Headers** are metadata on requests and responses:
| Header | Purpose |
|---|---|
| `Content-Type` | Format of the body (`application/json`) |
| `Authorization` | Credentials (`Bearer <token>`) |
| `Cache-Control` | How long it can be cached |
| `Set-Cookie` / `Cookie` | Server sets a cookie → browser sends it back every request |
| `Accept` | What formats the client wants |
| `Idempotency-Key` | Dedupes retries (Section B) |

**Cookie attributes that matter:** `HttpOnly` (JavaScript can't read it, which helps against XSS), `Secure` (HTTPS only), `SameSite` (helps against CSRF), `Max-Age`/`Expires`.

**Sessions:** the server stores who you are and gives the browser a **session ID** in a cookie. Each request carries the ID, and the server looks you up.

**The scaling trap:** if sessions live in one server's memory, request #2 landing on server B finds no session. Fixes: store sessions in **Redis**, or use **stateless tokens (JWT)** where the token itself carries identity. We'll cover this in Parts 2 and 8.

## A11. TLS basics

TLS gives you three things:

1. **Confidentiality**: nobody on the path can read the data
2. **Integrity**: nobody can alter it undetected
3. **Authentication**: you're really talking to `bhalobasha.com`

**Certificates:** the server presents a certificate signed by a **Certificate Authority** your browser already trusts. The browser verifies the chain, and that's how it knows the server is genuine.

**Handshake (simplified, TLS 1.3):**

```
Client → Server : "Hello, I support these ciphers" + key share
Server → Client : certificate + key share + "finished"
Both            : derive the same session key
Everything after: encrypted with that key
```

TLS 1.3 needs **one round trip** (TLS 1.2 needed two), and **session resumption** makes repeat visits even cheaper.

---

# Section B: APIs

## B1. What an API is

An API is a contract: _"send me this, I'll send back that."_ It defines the endpoints, inputs, outputs, and errors. Common styles: **REST** (most web apps), **GraphQL** (client picks fields), **gRPC** (fast service-to-service), and **WebSocket** (real-time).

## B2. REST

REST is an architectural style built on these ideas:

- **Resources** are nouns identified by URLs (`/properties/123`)
- Standard **HTTP methods** act on them
- **Stateless**: every request has everything the server needs (this is what allows horizontal scaling)
- **Representations**: usually JSON
- **Cacheable** responses and a **uniform interface**

## B3. RESTful API design rules

- Nouns, plural, lowercase: `/properties`, not `/getProperty`
- Nest only when the child belongs to the parent, and keep it shallow: `/users/42/bookings` is fine, 4 levels deep isn't
- Use the method for the action: `GET /properties`, `POST /properties`, `PUT /properties/:id`, `PATCH /properties/:id`, `DELETE /properties/:id`
- For actions that aren't CRUD, use a sub-resource: `POST /bookings/:id/cancel`
- Filtering, sorting, and paging go in **query params**
- Be consistent: naming, casing, response shape, error shape

## B4. Request lifecycle inside NestJS

```
Request
  → Middleware
  → Guards          (auth: "are you allowed?")
  → Interceptors    (before)
  → Pipes           (validate/transform input: DTOs)
  → Controller handler
  → Service → Repository/DB
  → Interceptors    (after: shape the response)
  → Response
(any thrown error → Exception filters → error response)
```

Knowing this tells you where each concern belongs: auth in guards, validation in pipes, response formatting in interceptors, and error shaping in exception filters.

## B5. API versioning

Needed because you can't break existing clients (your own mobile app, old frontend builds, third parties).

- **URI**: `/v1/properties` (most common, easy to debug)
- **Header**: `Accept: application/vnd.bhalobasha.v2+json`
- **Query**: `?version=2` (rare)

Non-breaking changes (adding optional fields) don't need a new version. Breaking changes (removing or renaming fields, changing meaning) do. NestJS supports this with `app.enableVersioning()`.

## B6. Pagination

Never return everything. Two main approaches:

**Offset-based:** `?page=3&limit=20` (or `skip`/`limit`)

- ✅ Simple, you can jump to page N
- ❌ Slow on big tables: `skip(500000)` makes the database walk past 500K rows first
- ❌ Unstable: if rows are inserted while the user pages, items get skipped or repeated

**Cursor-based:** `?limit=20&cursor=<opaque token>`

- ✅ Fast, because it uses an index ("give me rows after this one")
- ✅ Stable under inserts and deletes
- ❌ No jumping to an arbitrary page

MongoDB-style example:

```ts
const rows = await Model.find(cursor ? { _id: { $lt: cursor } } : {})
  .sort({ _id: -1 })
  .limit(limit + 1); // fetch one extra
const hasMore = rows.length > limit;
const data = rows.slice(0, limit);
const nextCursor = hasMore ? data[data.length - 1]._id : null;
```

Response shape:

```json
{ "data": [ ... ], "meta": { "limit": 20, "hasMore": true, "nextCursor": "64f1..." } }
```

Always cap `limit` server-side (for example max 100). If you sort by a non-unique field (like rent), your cursor needs that field **plus** `_id` as a tiebreaker.

## B7. Filtering and sorting

- Filtering: `GET /properties?district=dhaka&minRent=10000&maxRent=25000&type=flat`
- Sorting: `GET /properties?sort=-createdAt` (`-` = descending)
- **Allow-list** the fields clients may filter and sort on. Never pass raw query objects to the database (NoSQL injection risk).
- Every filter/sort combo you support should be backed by an **index** (Part 3), or it becomes a full scan at scale.

## B8. 🎯 Idempotency

**Definition:** an operation is idempotent if doing it once or many times leaves the system in the same end state.

- `PUT /users/5 {name:"Oli"}` → idempotent (same result every time)
- `DELETE /users/5` → idempotent (it's gone either way; the second response may be `404`, but the _state_ is the same)
- `POST /bookings` → **not** idempotent (each call creates another booking)
- `POST /send-otp` → **not** idempotent (each call sends another SMS)

**Why it matters:** retries are everywhere. Requests get duplicated by timeouts, flaky mobile networks, double-clicks, client libraries that auto-retry, and load balancers that retry. If the _first_ request actually succeeded but the response got lost, the retry causes a duplicate. For a booking or payment, that's a real bug.

**The idempotency key pattern:**

1. The client generates a unique key (UUID) **per logical action** and sends it: `Idempotency-Key: 550e8400-...`
2. The server atomically claims that key (e.g., Redis `SET key "in_progress" NX EX 60`)
3. First time seen → process the request, then store the **response** under that key (TTL ~24h)
4. Seen again, **in progress** → return `409` (or make it wait)
5. Seen again, **completed** → return the **stored response**, don't re-execute
6. Same key but a **different body** → reject (`422`/`409`), since it's a client bug

**Rules:**

- A **retry** reuses the same key. A genuinely new action (user clicks "resend OTP" on purpose) gets a **new** key.
- Scope keys per user and per endpoint, so different users or endpoints can't collide.
- Keys expire (TTL) so storage doesn't grow forever.

**Other ways to get idempotency without keys:**

- Use `PUT` instead of `POST` where it fits
- **Unique constraints** (a unique index makes a duplicate insert fail instead of duplicating)
- **Upserts**

**For `send-otp` specifically:** a per-phone cooldown or rate limit (e.g., one OTP per 60 seconds) is often the more practical protection, and an idempotency key complements it by handling network retries cleanly.

## B9. Error handling

Use **one consistent error shape** everywhere:

```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "Request validation failed",
    "details": [{ "field": "phone", "issue": "must be a valid BD number" }],
    "requestId": "req_8f3a21"
  }
}
```

- Right status code + a stable machine-readable `code` + a human message
- Include a `requestId` (correlation ID) so you can find the exact log line later
- **Never leak** stack traces, SQL/Mongo errors, or internal paths in production
- In NestJS, a global **exception filter** is where you enforce this shape

## B10. API documentation

- **OpenAPI/Swagger** is the standard. `@nestjs/swagger` generates it from your decorators (you already use `@ApiOperation`).
- Good docs list: endpoints, params, request/response examples, auth, error codes, rate limits.
- Add `@ApiResponse` for every status code an endpoint can return, since this forces you to think about them.

---

# Section C: Networking Basics

## C1. TCP vs UDP

|            | TCP                    | UDP                                  |
| ---------- | ---------------------- | ------------------------------------ |
| Connection | Handshake first        | None                                 |
| Delivery   | Guaranteed, ordered    | Best-effort, may drop or reorder     |
| Speed      | More overhead          | Very low overhead                    |
| Used for   | HTTP, databases, email | Video/voice calls, gaming, DNS, QUIC |

One-liner: **TCP trades speed for correctness, UDP trades correctness for speed.** Browsers can't send raw UDP from JavaScript, and WebSockets run on TCP.

## C2. Connection lifecycle

Opening a fresh HTTPS connection costs round trips before any data moves:

```
DNS (if uncached) → TCP handshake (1 RTT) → TLS 1.3 handshake (1 RTT) → request/response (1 RTT)
```

Over a 100 ms link, that's ~300 ms before the first byte arrives. This is exactly why connection reuse matters. Closing uses `FIN`, or the connection is dropped after being idle.

## C3. Keep-alive connections

Instead of closing after each response, the connection **stays open** for the next request, skipping the TCP and TLS handshakes. It's the default in HTTP/1.1. On the backend side, connection pooling does the same for HTTP clients, databases, and Redis (reusing connections instead of opening new ones).

## C4. HTTP/1.1 vs HTTP/2 vs HTTP/3

| Version      | Key idea                                                                      | Problem it solves                                                                                                           |
| ------------ | ----------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| **HTTP/1.1** | One request at a time per connection                                          | Browsers open ~6 parallel connections as a workaround; **head-of-line blocking**: a slow response blocks the ones behind it |
| **HTTP/2**   | **Multiplexing**: many requests share one connection, plus header compression | Removes HTTP-level blocking, but TCP-level blocking remains (one lost packet stalls all streams)                            |
| **HTTP/3**   | Runs on **QUIC over UDP**                                                     | Removes TCP head-of-line blocking, faster connection setup, survives network switches (Wi-Fi → mobile data)                 |

## C5. WebSockets

- Starts as a normal HTTP request with `Upgrade: websocket`; the server answers `101 Switching Protocols`, and the connection becomes **persistent and two-way**.
- Either side can push messages anytime, with no new request each time.
- Great for chat, live notifications, collaborative editing, and presence.
- Costs: every open connection uses server memory, you need **heartbeats** to detect dead connections, and scaling across multiple servers needs a shared layer like Redis pub/sub (Part 4).
- **vs SSE:** Server-Sent Events are one-way (server → client), simpler, and auto-reconnect. Use SSE when the client doesn't need to send over the same channel.

## C6. 🎯 TLS termination

**What it means:** _terminating TLS_ means decrypting HTTPS at some point on the path. Where you do it is the design decision.

```
A) Termination at load balancer (most common)
Client ══HTTPS══▶ Load Balancer ──plain HTTP──▶ App servers
                  (decrypts here)   (private network)

B) Passthrough
Client ══HTTPS══▶ Load Balancer ══HTTPS══▶ App servers
                  (can't read it)          (decrypt here)

C) Re-encryption (bridging)
Client ══HTTPS══▶ LB (decrypt, inspect, re-encrypt) ══HTTPS══▶ App servers
```

**Why terminate at the load balancer:**

- Certificates are managed in **one place**, not on every app server
- Decryption CPU cost moves off your app servers
- The LB can **read the request** (L7 routing by path/header, WAF, logging)
- Adding or removing app servers is simpler

**The security assumption:** traffic between the LB and the app servers is **plain HTTP**, so you're trusting that this internal network is **private and isolated**. If someone could sniff that segment, they'd see plaintext. When that trust isn't acceptable (compliance, zero-trust setups, shared networks), use **passthrough** or **re-encryption**.

**What changes in your NestJS app behind a TLS-terminating proxy:**

- The app sees plain HTTP, so it must read **`X-Forwarded-Proto`** to know the user was on HTTPS, and **`X-Forwarded-For`** to get the real client IP (otherwise every user looks like the LB's IP, which breaks per-IP rate limiting).
- With the Express adapter, enable `app.set('trust proxy', 1)` (adjust the number to your proxy hops).
- Secure cookies and redirect logic depend on this being right.

---

# Quick self-check (you should be able to explain each in 2–3 sentences)

- What happens between typing a URL and seeing the page?
- Safe vs idempotent, and which methods are which
- 401 vs 403, 400 vs 404 vs 409
- Why in-memory sessions break with multiple servers
- Offset vs cursor pagination, and why offset gets slow
- How an idempotency key works, including the "in-progress" and "different body" cases
- Why HTTP/2 helps and what HTTP/3 fixes on top of it
- TLS termination vs passthrough, and the trust assumption

---

# 📝 Part 1 Assignment

Do **A (written)** and **B (hands-on)**. Write answers in your own words, with specifics from your projects.

## A. Written

1. **Trace it:** Pick a real page in bhalobasha (e.g., property detail). Write the full lifecycle from click to render: DNS, TCP, TLS, HTTP, load balancer, NestJS pipeline (guard → pipe → controller → service → DB), response. Note which steps can be cached or skipped on a repeat visit.
2. **Status code table:** Give the best code and a one-line _why_ for each:
   (a) property ID doesn't exist (b) logged-in user edits another user's listing (c) no token on a protected route (d) invalid phone format (e) user registers an email that already exists (f) listing created successfully (g) 100th OTP request in a minute (h) your DB is unreachable (i) OTP sent via background job
3. **Idempotency:** Classify these as idempotent or not, and justify: `PUT /properties/1`, `DELETE /properties/1`, `POST /bookings`, `PATCH /properties/1 {rent: 20000}`, `PATCH /properties/1 {rent: "+1000"}`. Then describe the **full idempotency-key flow** for `POST /bookings`, including what happens on a concurrent duplicate and on a same-key/different-body request.
4. **TLS termination:** Draw the three modes (termination, passthrough, re-encryption). Explain in your own words _why_ terminating at the load balancer is common, what trust it assumes, and what you must configure in NestJS because of it.
5. **API design:** Design the listing endpoint for bhalobasha with filters (district, rent range, type), sorting, and cursor pagination. Show an example request, the response JSON, and the error response for an invalid filter.

## B. Hands-on (bhalobasha-api)

1. **Status code audit:** Take **5 real endpoints** and fill in a table: `endpoint | what it does | success code | failure codes | correct? | fix`. Fix at least two (hint: check `send-otp`).
2. **Global exception filter:** Add one that returns the consistent error shape from B9, including a `requestId`. Paste the code.
3. **Idempotency module:** Finish the `IdempotencyModule` you started and wire it into `send-otp`, including the in-progress state and the body-mismatch check. Test with Postman by sending the same key twice, and paste the results.

Send your answers when you're ready, either all at once or section by section. I'll grade them with marks like before, and then we'll move to **Part 2: Scaling Fundamentals**.

Want this lesson as a markdown file for your notes?
