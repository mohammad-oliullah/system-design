# Grading: Part 1 Assignment

**Total marks: 25** (5 marks per question)

---

### Q1 — Idempotency in Practice — **2 / 5**

You're right that `POST` isn't idempotent by default — good. But the "fix" misses the mark. Removing the body and hardcoding the DTO doesn't make the endpoint idempotent — it just breaks the feature (as you noticed yourself). The hint was **idempotency keys**: the client generates a unique key per logical request (e.g., a UUID) and sends it in a header (`Idempotency-Key: xyz`). The server checks if it's already processed that key — if yes, it returns the cached result instead of re-sending the OTP. The body stays exactly as-is; you're not removing data, you're deduplicating repeated attempts of the _same_ request.

---

### Q2 — Pagination Tradeoff — **4 / 5**

Solid. Correct choice (cursor-based), correct reasoning (offset breaks down with high-volume, frequently-changing data due to skip/duplicate issues). One point missed: you could've also mentioned _why_ offset is slow at scale — `OFFSET 500000` still forces the database to scan and discard 500K rows before returning results, which is a performance problem on top of the accuracy problem.

---

### Q3 — Protocol Choice — **4 / 5**

Correct answer and good that you've actually built this. Reasoning is right — persistent, full-duplex connection avoids the overhead of repeated requests. Lost a point for not explicitly addressing _why_ the alternatives fail: HTTP polling means repeated request/response overhead and delay (not truly real-time), and UDP isn't even usable directly from a browser — WebSockets _use_ TCP underneath, not UDP. Naming that contrast explicitly would've made the answer complete.

---

### Q4 — TLS Termination Reasoning — **1 / 5**

This one's got it backwards. The question was about terminating TLS _at the load balancer_ (not the backend) — meaning traffic is encrypted from client → load balancer, then **decrypted and sent as plain HTTP from load balancer → backend servers**. The security assumption being made is: the internal network between the load balancer and your app servers is trusted/private (not exposed to the public internet), so sending plain HTTP there is considered safe. Your answer concluded the opposite — that LB termination is "most secure" without explaining the actual mechanism or the trust assumption involved. Worth re-reading this one.

---

### Q5 — Status Code Audit — **1 / 5**

Too thin to grade well. "All correct, didn't change anything" isn't an audit — I can't tell if you actually checked, or what the 5 endpoints even were. A real audit needs: endpoint name, what it does, what status it currently returns, and why that's correct (or what you'd change). Redo this one with actual endpoints and status codes listed.

---

## Total: **12.5 / 25**

Strongest: Q2, Q3 (solid reasoning, correct conclusions).
Needs rework: Q1 (revisit idempotency keys), Q4 (redo with correct direction), Q5 (needs real detail, not a one-liner).
