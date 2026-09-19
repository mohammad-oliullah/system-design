# Part 1 — Foundations (Expanded) || 20 Sep - 23 Sep

## Day 1-2: Core Concepts

### What system design actually is (and isn't)

- **Is**: the practice of deciding how components of a software system fit together to handle real load, stay reliable, and remain maintainable as usage grows. Central question: "what happens when this needs to work for 100 users vs. 100 million?"
- **Isn't**: not algorithm/DSA problem-solving, not memorizing famous architectures, not finding "the correct answer" — it's about navigating tradeoffs (consistency vs. availability, cost vs. performance, simplicity vs. flexibility) given specific constraints.
- Two levels: **HLD** (services, databases, how they connect) and **LLD** (class structure, schemas, specific implementations).

### Client-server model

- Client initiates requests, server listens and responds. Communication happens over a network — inherently slower and less reliable than in-memory calls.
- Core constraint: one server has finite CPU, memory, and connections. Nearly everything in system design exists because of this fact.

### Request/response lifecycle

- Client sends a request → server processes it → server sends a response. Each request is a self-contained unit of work.
- Full path: DNS resolution → TCP connection (+TLS if HTTPS) → HTTP request sent → server routes to handler → business logic runs → database queried → response built → sent back → connection closes or stays alive.

### IP addresses

- Every device/server reachable on a network has an IP address (IPv4: `104.21.5.12`, IPv6: longer hex format, built for the exhaustion of IPv4 space).
- Servers are addressed as IP + port (`104.21.5.12:443`). A domain is just a human-friendly alias for an IP.

### DNS

- Translates domains into IPs. Flow: browser → resolver → root servers → TLD servers → authoritative nameserver → IP returned to browser.
- Results are cached based on **TTL** (time-to-live) — this caching is why DNS changes (like pointing a domain to a new server) take time to propagate.
- Common record types: `A` (domain → IPv4), `AAAA` (→ IPv6), `CNAME` (alias to another domain), `MX` (mail server).

### TCP/IP basics

- TCP/IP is the layered model underlying internet communication: IP handles addressing/routing packets, TCP handles reliable, ordered delivery on top of that.
- Before any HTTP request, TCP does a **three-way handshake**: SYN (client) → SYN-ACK (server) → ACK (client). Only after this is the connection "open."
- TCP guarantees packets arrive in order and retransmits lost ones — this reliability has a cost (latency), which is part of why HTTP/2 and HTTP/3 exist (more below).

### HTTP/HTTPS

- HTTP: plain-text application-layer protocol for request/response communication, built on top of TCP.
- HTTPS: HTTP + TLS encryption. Adds a TLS handshake before data flows, so a request now needs a TCP handshake _and_ a TLS handshake before the actual HTTP request goes out.

### HTTP methods

- `GET` — read, no side effects, safe to cache
- `POST` — create a resource, or trigger a non-idempotent action
- `PUT` — full replace of a resource
- `PATCH` — partial update
- `DELETE` — remove a resource
- `HEAD` — like GET but returns headers only, no body
- `OPTIONS` — used for CORS preflight checks, asks "what methods are allowed here?"

### HTTP status codes

- **1xx** — informational (rare in practice)
- **2xx** — success: `200 OK`, `201 Created`, `204 No Content`
- **3xx** — redirection: `301 Moved Permanently`, `304 Not Modified` (important for caching)
- **4xx** — client error: `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `429 Too Many Requests`
- **5xx** — server error: `500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable`, `504 Gateway Timeout`
- Knowing the difference between `401` (not authenticated) and `403` (authenticated but not authorized) trips people up constantly — worth nailing down.

### Headers, cookies, sessions

- **Headers**: key-value metadata sent with requests/responses. Examples: `Content-Type`, `Authorization`, `Cache-Control`, `Set-Cookie`.
- **Cookies**: small pieces of data the server asks the browser to store and send back on every subsequent request to that domain. Used for sessions, tracking, preferences.
- **Sessions**: server-side mechanism to remember a user across requests. Traditionally: server creates a session ID, stores session data server-side (in memory or a store like Redis), sends the ID to the client as a cookie. The client just sends the ID back each time.
- This is the seed of a huge system design theme: session data stored _on one server_ breaks the moment you have multiple servers behind a load balancer (you'll hit this properly in Part 2).

### TLS basics

- TLS (Transport Layer Security) encrypts data between client and server so it can't be read or tampered with in transit.
- TLS handshake (simplified): client and server agree on a cipher suite, server presents a certificate (proving identity, issued by a trusted Certificate Authority), a shared encryption key is negotiated.
- TLS 1.3 (current standard) reduced this to fewer round trips than TLS 1.2, which matters for latency at scale.

---

## Day 3-4: APIs

### What an API is

- A contract defining how two systems (client and server, or service and service) communicate — what endpoints exist, what data goes in, what comes back.

### REST

- An architectural style (not a protocol) built around **resources** (nouns) manipulated via standard HTTP verbs.
- Core principles: statelessness (no server-side session dependency per request), uniform interface, resources identified by URLs, representations (usually JSON) exchanged.

### RESTful API design

- Use nouns, not verbs, in URLs: `/users/123` not `/getUser?id=123`.
- Nest resources logically: `/users/123/orders` for a user's orders.
- Use plural resource names consistently: `/users`, not `/user`.
- Keep the API consistent — same conventions for naming, casing, and response shape across all endpoints.

### Request/response lifecycle (API-specific)

- Client builds a request with method, headers (e.g., `Authorization: Bearer <token>`), and body (if applicable).
- Server validates the request (auth, input shape), executes logic, queries data, builds a response.
- Response includes a status code, headers (e.g., `Content-Type: application/json`), and a body.

### API versioning

- Needed because APIs evolve but you can't break existing clients (like your Next.js frontend, or third-party consumers).
- Common strategies:
  - URL versioning: `/v1/users`, `/v2/users`
  - Header versioning: `Accept: application/vnd.myapi.v2+json`
  - Query param versioning: `/users?version=2`
- URL versioning is the most common in practice because it's the most visible and debuggable.

### Pagination

- Returning 10 million records in one response isn't viable. Two main approaches:
  - **Offset-based**: `?page=2&limit=20` — simple, but slow and unstable on large datasets (data shifting between pages causes duplicates/skips).
  - **Cursor-based**: `?after=<id_or_token>&limit=20` — more efficient and stable at scale, used by most large-scale APIs (Twitter, Stripe).

### Filtering and sorting

- Filtering: `?status=active&role=admin` — lets clients narrow results server-side instead of pulling everything and filtering client-side.
- Sorting: `?sort=createdAt&order=desc` — needs to map cleanly to indexed database columns, or it gets expensive fast (ties into Part 3, indexing).

### Idempotency

- An operation is **idempotent** if calling it multiple times has the same effect as calling it once.
- `GET`, `PUT`, `DELETE` should be idempotent by design. `POST` typically isn't (calling it twice creates two resources).
- Matters a lot in distributed systems: if a client doesn't get a response (network blip) and retries, idempotency prevents duplicate side effects — e.g., idempotency keys on payment APIs, so a retried "charge card" request doesn't charge twice.

### Error handling

- Consistent error response shape across the whole API — e.g., `{ "error": { "code": "USER_NOT_FOUND", "message": "..." } }`.
- Correct status codes matched to the situation (see status codes above).
- Don't leak internals (stack traces, database errors) in production responses — a real security concern, not just a style preference.

### API documentation

- Tools: **OpenAPI/Swagger** (most common — NestJS has built-in support via `@nestjs/swagger`), Postman collections, or hand-written docs.
- Good docs describe: endpoints, required/optional params, request/response shapes, auth requirements, error codes, and rate limits.

---

## Networking Basics

### TCP vs UDP

- **TCP**: connection-oriented, reliable, ordered delivery, retransmits lost packets. Used for HTTP, most application traffic.
- **UDP**: connectionless, no delivery guarantee, no ordering, no retransmission — but much lower overhead. Used where speed matters more than perfect reliability: video calls, live streaming, gaming, DNS queries.
- Tradeoff in one line: TCP guarantees correctness at the cost of latency; UDP guarantees speed at the cost of correctness.

### Connection lifecycle

- TCP handshake opens the connection (SYN → SYN-ACK → ACK).
- Data is exchanged.
- Connection closes via a four-way termination (FIN/ACK exchange) — or stays open if using keep-alive.

### Keep-alive connections

- Without keep-alive, every single HTTP request would require a fresh TCP (and TLS) handshake — expensive.
- `Connection: keep-alive` lets a single TCP connection handle multiple requests/responses sequentially, avoiding repeated handshake overhead. Default behavior in HTTP/1.1.

### HTTP/1.1 vs HTTP/2 vs HTTP/3

- **HTTP/1.1**: one request per connection at a time (head-of-line blocking) unless you open multiple connections in parallel (browsers typically cap this at ~6 per domain).
- **HTTP/2**: introduces **multiplexing** — multiple requests/responses share a single TCP connection simultaneously, no head-of-line blocking at the HTTP layer. Also adds header compression and server push.
- **HTTP/3**: replaces TCP with **QUIC** (built on UDP), eliminating TCP-level head-of-line blocking entirely and speeding up connection setup (fewer round trips, especially on reconnects). Increasingly the default for major platforms.

### WebSockets

- A protocol that upgrades an HTTP connection into a **persistent, full-duplex** connection — both client and server can send messages anytime, without a new request/response cycle each time.
- Used for real-time features: chat, live notifications, collaborative editing, live dashboards.
- Contrast with HTTP's request-response model: WebSockets stay open; HTTP typically doesn't (aside from keep-alive optimizations).

### TLS termination

- In production, TLS is often "terminated" at a load balancer or reverse proxy (e.g., Nginx, AWS ALB) rather than at each individual application server.
- This means: encrypted traffic comes in from the client, gets decrypted at the load balancer, and travels as plain HTTP internally between the load balancer and your app servers (inside a trusted private network).
- Why: centralizes certificate management, offloads the CPU cost of encryption/decryption from your app servers, simplifies scaling.

---

# Day 5: Assignment: Part 1 (Expanded)

1. **Idempotency in practice**: In your spoken-english-app or portfolio backend, pick one `POST` endpoint. Explain why it's _not_ idempotent by default, and describe one way you'd make it idempotent (hint: think about idempotency keys).

2. **Pagination tradeoff**: You're building an endpoint to list vocabulary words in your spoken-english-app, and the table could grow to 500K+ rows. Would you use offset-based or cursor-based pagination? Justify your answer.

3. **Protocol choice**: If you were adding a live "typing indicator" feature to a chat feature, would you use plain HTTP polling, WebSockets, or UDP directly? Explain why, referencing the tradeoffs above.

4. **TLS termination reasoning**: Explain in your own words why terminating TLS at a load balancer (rather than at every backend server) is a common pattern. What's the security assumption being made about the network between the load balancer and the app servers?

5. **Status code audit**: Open your NestJS backend and check 5 real endpoints. Are they returning the _correct_ status codes for success and failure cases? List any you'd fix and what you'd change them to.

Send these over when ready — then we'll move to Part 2: Scaling Fundamentals.
