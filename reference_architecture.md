🧭 Reference Production Architecture (Control-Aware)
Target Profile

Workload: Hybrid (read-heavy API + write transactions)
Traffic shape: Spiky + diurnal
Initial scale: 5k QPS peak
12-month projection: 40k QPS
Worst-case spike: ×8
Consistency: Hybrid
Region: single-primary, multi-AZ

🏗 High-Level Architecture
Client
  ↓
Edge (CDN + WAF)
  ↓
API Gateway / Ingress
  ↓
Stateless App Service
  ↓            ↓
Redis Cache    Async Queue
  ↓            ↓
Postgres       Worker Service
SECTION 1 — EDGE LAYER
Components

CDN

WAF

Bot protection

Rate limiting (coarse)

Latency Budget

Target: < 15ms added

Controls

geo filtering

IP reputation

request size cap (e.g., 1 MB)

TLS termination (TLS 1.3)

Failure Mode

If CDN fails → traffic bypass allowed but WAF must remain.

SECTION 2 — API GATEWAY / INGRESS
Responsibilities

routing

coarse rate limiting

request size enforcement

auth pre-check (optional)

request ID injection

Hard Limits

max header size: 16 KB

max body: service-defined

idle timeout: 60s

Observability

Emit:

request_rate

rejected_rate

upstream_latency

SECTION 3 — APPLICATION SERVICE

Stateless containers.

Runtime Choice (example)

Go or Node (non-blocking)

HTTP/2 enabled

keep-alive enabled

Concurrency Model
Node

avoid sync APIs

worker pool for CPU tasks

Go

bounded goroutines for external calls

context cancellation enforced

Latency Budget (App Layer)

Target: ≤ 30ms p95

Breakdown example:

auth: 5ms

business logic: 15ms

serialization: 10ms

External Call Policy (MANDATORY)

Example:

timeout = min(upstream_p95 × 1.5, 250ms)
retries = 2
backoff = exponential + jitter
circuit_breaker = enabled
Connection Pool (App → Postgres)

Starting point:

pool size = min( (CPU cores × 4), DB max_conn × 0.25 )

connection max lifetime: 30m

idle timeout: 5m

Telemetry Required

pool_wait_ms

pool_in_use

pool_timeouts

SECTION 4 — REDIS CACHE (L2)
Use Cases

hot reads

session data

rate limit counters

idempotency keys

Performance Targets

p95 latency: < 3ms

hit rate target: > 80% for cacheable endpoints

Required Protections
Stampede Protection

Implement:

single-flight OR

request coalescing

TTL Strategy

Example:

base_ttl = 300s
jitter = ±20%
Memory Policy

eviction: allkeys-lru

fragmentation monitoring enabled

Failure Mode

If Redis unavailable:

system must degrade gracefully

never hard-fail critical reads unless correctness requires it

SECTION 5 — PRIMARY DATABASE (Postgres)
Topology

primary (write)

1–2 read replicas

multi-AZ

Target Performance

p95 query: < 50ms

p99 query: < 120ms

Connection Limits (example)

If DB max_conn = 400:

app pool total ≤ 200

admin/reserved = 50

margin = remaining

Required Index Discipline

Every high-QPS query must have:

supporting index

EXPLAIN reviewed

rows scanned bounded

Query Guardrails (MANDATORY)

Enable:

slow query log (>100ms)

auto EXPLAIN capture

sequential scan alerting

query fingerprint metrics

N+1 Detection

Instrumentation must emit:

queries_per_request

Alert threshold example:

warn: > 20

critical: > 50

SECTION 6 — ASYNC QUEUE

(Example: Kafka / RabbitMQ / SQS class)

Use For

email

analytics

webhooks

heavy processing

Never block user request on these.

Processing Guarantees

Document clearly:

at-least-once OR

exactly-once (rare)

Worker Controls

Workers must implement:

visibility timeout

dead-letter queue

retry cap

idempotency check

SECTION 7 — SECURITY IMPLEMENTATION
JWT Validation Flow

Strict order:

parse token

verify signature

check alg allow-list

validate exp

validate nbf

validate iss

validate aud

attach identity

Failure at any step → reject.

Authorization

Policy engine required.

No raw role checks in business handlers.

Rate Limiting Stack

Layered:

Edge: coarse IP

Gateway: per-IP

App: per-user/token

Redis: sliding window or token bucket

SECTION 8 — OBSERVABILITY STACK
Metrics Backend

(Example: Prometheus-class)

Must collect:

p50/p95/p99 latency

error rates

saturation

DB latency

cache hit rate

retry rate

Distributed Tracing

Required propagation:

trace_id

span_id

request_id

Across all services.

Logging

Structured JSON only.

Hard rule:

Sensitive fields auto-redacted.

SLO Alerting

Use burn-rate alerts.

Example:

fast burn: 5 min

slow burn: 1 hour

SECTION 9 — RESILIENCE CONTROLS
Circuit Breakers

Required for:

DB (optional soft)

external APIs

cache (optional soft)

Load Shedding

When:

CPU > 85% sustained

queue depth exceeds threshold

pool wait exceeds threshold

System must reject gracefully.

Backpressure

Must propagate via:

HTTP 429

queue pause

worker concurrency reduction

Graceful Degradation

Define per endpoint:

optional data dropped first

stale cache allowed window

feature flags for kill-switch

SECTION 10 — DEPLOYMENT HARDENING
Container Spec

Mandatory:

non-root user

distroless/minimal base

read-only root FS

no-new-privileges

seccomp profile

drop NET_RAW

CPU/memory limits

Network Layout

Mandatory:

DB in private subnet

Redis in private subnet

services in private subnets

only ingress public

egress allowlist enforced

Service-to-Service Security

If multi-service:

mTLS required

cert rotation ≤ 24h

identity-based auth

🧪 Expected Production Numbers (Reference)

For ~5k QPS service:

API p95: ≤ 90ms

API p99: ≤ 180ms

DB p95: ≤ 50ms

Redis p95: ≤ 3ms

error rate: < 0.1%

retry amplification: < 15%

cache hit rate: > 80%

✅ When This Architecture PASSES Your Checklist

You pass when:

metrics prove SLO

guardrails are automated

failure modes documented

controls are runtime-tunable

no hidden blocking paths

blast radius bounded
