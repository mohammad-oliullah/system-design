# 📌 Lacking / Previous Weakness — Part 1

## 1. Idempotency Keys

**Core idea, more carefully explained:**
An idempotency key has three possible states: **absent** (first time seeing this key), **in-flight** (request is currently being processed — matters for concurrent duplicate requests), and **complete** (already processed, has a stored result). The common mistakes to avoid: storing keys forever (causes unbounded storage growth — use a TTL, typically 24 hours), using the same key for different request bodies (the server should reject mismatched bodies rather than silently returning the wrong cached result), generating keys from non-unique data like a timestamp (causes collisions), and forgetting to scope keys per-user or per-tenant.

A subtlety worth knowing: true exactly-once delivery requires combining idempotency keys with something like a transactional outbox pattern — if the server crashes after executing the side effect but before storing the key, a retry will still duplicate the work. That's an advanced detail, good to be aware of but not something to worry about implementing yet.

**Video:** [What is API Idempotency and Why Is It Important?](https://www.youtube.com/watch?v=6pjGuuGsqxE) — short, direct, exactly the gap you have.

## 2. TLS Termination at the Load Balancer

**Read first (clarifies the direction issue from your answer):**
SSL offloading terminates HTTPS at the proxy and communicates with backends over HTTP: Client HTTPS → [Proxy: SSL/TLS] → Backend HTTP. The trust model: SSL termination at the load balancer is secure when the internal network segment between the load balancer and backend servers is isolated and trusted.

Compare that to the alternative so the contrast locks in: use TLS termination when you want the proxy to inspect, route, or log traffic, or to offload CPU-intensive decryption from backend servers; use TLS passthrough when you require true end-to-end encryption and don't need the proxy to read traffic. A common mistake is forgetting that termination exposes plaintext internally — so internal network security is critical.

**Video search:** Search YouTube for **"SSL termination vs SSL passthrough load balancer"** — Hussein Nasser and NetworkChuck both have solid explainer videos on this exact topic; I don't have a verified direct link to hand you, so search and pick the one with a clear diagram (visual really helps here).

## 3. HTTP Status Codes

For this one, honestly, a video won't fix the issue — the gap wasn't knowledge, it was depth of answer. Search YouTube for **"HTTP status codes explained"** (any 5–10 min crash course works, they're mostly interchangeable) just to refresh the list, then redo Q5 yourself by actually opening your code and checking.

---

# Part 2 — Scaling Fundamentals

## 2.1 Scaling Strategies

**Vertical scaling (scale up)**

- Adding more power to a single machine — more CPU, RAM, faster disk.
- Simple, requires no architectural changes.
- Hard limit: there's a ceiling to how big one machine can get, and it's a single point of failure — if it dies, everything dies.

**Horizontal scaling (scale out)**

- Adding more machines instead of making one bigger.
- No real ceiling (in theory), and failure of one machine doesn't kill the whole system.
- The cost: you now need a way to _distribute_ traffic across machines (load balancing) and your app needs to not depend on any one machine "remembering" things (statelessness).

**Stateless vs. stateful architecture**

- **Stateless**: a server doesn't store anything about a client between requests — every request carries everything needed to process it. This is what makes horizontal scaling easy: any server can handle any request.
- **Stateful**: a server holds onto client-specific data (e.g., in-memory session). This breaks horizontally — if a user's session lives only on Server A, and their next request lands on Server B, the server has no idea who they are.
- This is exactly the sessions problem you flagged back in Part 1 — the fix is usually externalizing state (e.g., session data in Redis, not in server memory), so _any_ server can serve _any_ request.

**Bottleneck identification**

- Before scaling anything, you need to know _what's actually slow_. Common bottleneck sources: database queries (unindexed, N+1 queries), a single server under CPU/memory pressure, network latency, a downstream service that's slow.
- Scaling the wrong thing (e.g., adding app servers when the database is the real bottleneck) doesn't help — it just moves the traffic jam closer to the database and makes it worse.

## 2.2 Load Balancing

**Why it's needed**

- Once you have multiple servers, something has to decide which server handles each incoming request. That's the load balancer's job — distribute traffic evenly (or intelligently) so no single server gets overwhelmed.

**Layer 4 vs Layer 7**

- **L4 (transport layer)**: routes based on IP + port, doesn't inspect the actual HTTP content. Fast, simple.
- **L7 (application layer)**: understands HTTP — can route based on URL path, headers, cookies. Slower per-request but far more flexible (e.g., send `/api/*` to one service, `/static/*` to another).

**Routing algorithms**

- **Round-robin**: requests go to servers in rotating order. Simple, assumes all servers and requests are roughly equal.
- **Weighted round-robin**: same idea, but some servers get more traffic (e.g., a bigger server gets a higher weight).
- **Least connections**: route to whichever server currently has the fewest active connections — better when request processing time varies a lot.
- **IP hashing**: route based on a hash of the client's IP, so the same client consistently hits the same server — useful when you need some continuity without full statefulness.

**Health checks**

- The load balancer periodically pings each server (e.g., a `/health` endpoint). If a server stops responding correctly, it's pulled out of rotation until it recovers — prevents routing traffic into a dead server.

**Session affinity / sticky sessions**

- Forces a client to keep hitting the _same_ server (often via a cookie) — usually used as a workaround when sessions are stored in server memory instead of being externalized.
- Worth noting: this is a workaround, not a best practice. It reintroduces the "what if that one server dies" problem and limits how evenly you can balance load. Externalizing state (Redis) is the cleaner fix.

## 2.3 Caching

**Why caching works**

- Most systems read data far more often than they write it, and a lot of that data doesn't change frequently. Caching stores a copy of expensive-to-compute or expensive-to-fetch data somewhere faster to access, so repeated requests don't hit the slow path (database, external API) every time.

**Layers of caching**

- **Browser caching**: the browser itself stores static assets (images, JS, CSS) locally based on `Cache-Control` headers.
- **CDN caching**: a geographically distributed network of servers caches content close to the user — critical for static assets and increasingly for API responses too.
- **Application-level caching**: your backend caches computed results or frequently-accessed data in memory or in something like Redis.
- **Database caching**: the database itself caches query results and frequently-accessed pages internally.

**Redis**

- An in-memory key-value store, the most common choice for application-level caching. Fast because it's in RAM, supports data structures beyond simple key-value (lists, sets, sorted sets — useful for things like leaderboards or rate limiting).

**Caching strategies**

- **Cache-aside (lazy loading)**: app checks cache first; on a miss, fetches from the database and writes the result into the cache for next time. Most common pattern.
- **Read-through**: the cache itself sits in front of the database and handles the fetch-on-miss logic, so the app only ever talks to the cache.
- **Write-through**: every write goes to the cache _and_ the database at the same time, keeping them in sync immediately.
- **Write-back (write-behind)**: writes go to the cache first, and are asynchronously flushed to the database later — faster writes, but risk of data loss if the cache fails before the flush happens.

**Cache invalidation** — famously one of the two hardest problems in computer science

- **TTL (time-to-live)**: cached data automatically expires after a set duration. Simple, but means you can serve stale data for up to the TTL window.
- Active invalidation: explicitly clearing/updating the cache entry the moment the underlying data changes — more accurate, more complex to implement correctly.

**Failure modes to know by name**

- **Cache stampede**: a popular cache entry expires, and a flood of simultaneous requests all miss the cache at once and hit the database simultaneously — can take the database down. Mitigated with locks or staggered expiry.
- **Cache penetration**: repeated requests for data that doesn't exist in the cache _or_ the database (e.g., an invalid ID), bypassing the cache entirely every time and hammering the database. Mitigated by caching "not found" results too (with a short TTL).
- **Cache avalanche**: a large number of cache entries expire at the same time (e.g., all set with the same TTL), causing a sudden spike of database load all at once. Mitigated by adding random jitter to TTLs so expiries spread out.

---

# Assignment — Part 2

1. **Stateless redesign**: Think about your realtime-chat-system (12–14k daily users). If you had to run it across 3 app servers behind a load balancer right now, what parts of your current design would break because they rely on one server "remembering" something? List them and describe how you'd externalize that state.

2. **Load balancer algorithm choice**: For your bhalobasha platform, would you pick round-robin, least connections, or IP hashing for the main API servers? Justify your choice based on what kind of traffic that app likely sees (short property lookups vs. longer operations).

3. **Caching strategy design**: Pick one read-heavy endpoint from bhalobasha (e.g., listing properties in a location). Design a caching approach for it — which strategy (cache-aside, write-through, etc.), what you'd use as the cache key, and what TTL you'd set. Explain your reasoning.

4. **Break it on purpose**: Describe a cache stampede scenario that could realistically happen on your realtime-chat-system or bhalobasha app. Then describe how you'd prevent it.

5. **Project task (hands-on)**: Pick any one `GET` endpoint in one of your real NestJS backends and actually implement caching for it with Redis (cache-aside pattern). Paste the code when done — we'll review it together.
