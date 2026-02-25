# Backend Systems Skill
Secure, low-latency, production-grade backend architecture framework.

This document defines:
- Architectural requirements
- Security guarantees
- Latency discipline
- Fundamental syntax-level implementation rules
- Measurable outcome expectations

A backend system is incomplete if it cannot explain both:
1. WHAT it does
2. HOW it works at the protocol, runtime, and query level

---

# Core Principles

1. Secure by default (fail closed).
2. Latency is a feature.
3. Every performance claim must be measurable.
4. Minimize surface area.
5. Observability is mandatory.
6. Mechanism clarity > framework abstraction.

---

# SECTION 1 — SYSTEM CLASSIFICATION

Before writing code, define:

## Workload Type
- Read-heavy API
- Write-heavy transactional
- Streaming
- Batch
- Hybrid

## Consistency Model
- Strong (synchronous commit)
- Eventual (async propagation)
- Hybrid (boundaries documented)

## Scale Expectation
- < 10k daily users
- 10k–1M
- 1M+
- Unbounded

Reason:
Consistency and scale determine replication strategy,
locking strategy, and caching topology.

---

# SECTION 2 — LATENCY FUNDAMENTALS

## Define a Latency Budget

Example:

Total Budget: 100ms
- Network handshake: 20ms
- TLS negotiation: 5ms
- Auth validation: 5ms
- Business logic: 30ms
- DB query: 30ms
- Serialization: 10ms

---

## Fundamental Latency Equation

Total Latency =
Network +
Crypto +
Application Logic +
DB Execution +
Serialization +
Queueing Delay

If any component is unknown → system is unmeasured.

---

## Syntax-Level Performance Rules

### 1. Avoid Blocking I/O

Bad (Node):
fs.readFileSync()

Good:
await fs.promises.readFile()

Reason:
Blocking calls halt the event loop → increases tail latency.

---

### 2. Bound External Calls

Every outbound call must include:

- Timeout
- Retry cap
- Circuit breaker

Example (pseudo):

request({
  timeout: 200ms,
  retries: 3,
  backoff: exponential
})

Unbounded calls destroy p99 latency.

---

### 3. Query Structure Discipline

Bad:
SELECT * FROM users;

Good:
SELECT id, email FROM users WHERE id = ?;

Reason:
Minimize row width → reduces memory copy + serialization cost.

---

### 4. Index Fundamentals

Query:
SELECT id FROM orders WHERE user_id = ?;

Required index:
CREATE INDEX idx_orders_user_id ON orders(user_id);

Without index:
→ Full table scan
→ O(n)
With index:
→ B-tree lookup
→ O(log n)

---

# SECTION 3 — SECURITY FUNDAMENTALS

Security is enforced at:

1. Transport layer
2. Authentication layer
3. Authorization layer
4. Input validation layer
5. Data storage layer

---

## Authentication Syntax Fundamentals

JWT must validate:

- Signature
- Expiration
- Issuer
- Audience

Pseudo-flow:

1. Extract token
2. Verify signature (RS256 public key)
3. Validate exp < now
4. Validate aud matches service
5. Attach identity context

Failure → reject request immediately.

---

## Authorization Fundamentals

Never:
if (user.role == "admin") allow

Always:
checkPermission(user, "WRITE_REPORT")

Reason:
Role != permission.
Explicit permission prevents privilege escalation.

---

## Input Validation Mechanics

Validation must:

- Enforce schema
- Reject unknown fields
- Enforce max size

Reason:
Unknown fields → injection surface
Large payloads → DoS vector

---

## Rate Limiting Fundamentals

Use token bucket or leaky bucket.

Key design:
limit = 100 req/min
burst = 20

Mechanism:
- Each request consumes a token
- Tokens refill per time window

Prevents:
- brute force
- enumeration
- resource exhaustion

---

# SECTION 4 — DATABASE PERFORMANCE MECHANICS

## Query Planning

Every critical query must be evaluated using:

EXPLAIN ANALYZE

Review:
- Index usage
- Cost estimate
- Row scan count
- Execution time

---

## N+1 Pattern Breakdown

Bad:

for user in users:
  SELECT * FROM orders WHERE user_id = ?

Effect:
1 + N queries
Latency grows linearly.

Solution:
JOIN or IN clause batching.

---

## Connection Pool Fundamentals

Why:
Opening DB connection = TCP handshake + auth

Rule:
Pool size tuned to CPU cores and DB limits.

Too small → queueing delay
Too large → DB thrashing

---

# SECTION 5 — CACHING MECHANICS

## Cache Hierarchy

L1: In-process (microseconds)
L2: Redis (1–3ms)
L3: Database (5–30ms)
L4: External API (30–200ms)

---

## Cache Invalidation Rules

Each cached endpoint must define:

- TTL
- Write-through or write-behind
- Explicit invalidation event

No invalidation strategy → stale data risk.

---

# SECTION 6 — THREAT MODEL FUNDAMENTALS

Define:

Assets:
- User data
- Payment data
- Tokens
- Internal APIs

Attackers:
- Anonymous internet
- Authenticated user
- Insider
- Compromised dependency

Entry Points:
- HTTP endpoints
- Background workers
- Webhooks

Trust Boundaries:
- Public → API
- API → DB
- Service → Service

Worst Case:
Define full breach scenario.

---

# SECTION 7 — OBSERVABILITY MECHANICS

Latency metrics must include:

- p50
- p95
- p99
- max

Why:
Average latency hides tail risk.

---

## Structured Logging Syntax

Log format (JSON):

{
  request_id,
  user_id,
  endpoint,
  latency_ms,
  status_code
}

Never log:
- passwords
- raw tokens
- secret keys

---

# SECTION 8 — DEPLOYMENT HARDENING

Containers must:

- Run as non-root
- Use minimal base image
- Use read-only filesystem
- Drop unused capabilities

Network must:

- Restrict DB to private subnet
- Enforce service-level isolation

---

# PRODUCTION READINESS DEFINITION

A backend is production-ready only if it includes:

1. Architecture overview
2. Latency budget
3. Threat model
4. Caching strategy
5. Index strategy
6. Failure handling plan
7. Observability plan
8. Scaling strategy
9. Syntax-level explanation of performance and security mechanisms

Missing any → incomplete system.
