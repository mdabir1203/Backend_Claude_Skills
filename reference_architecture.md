# 🧭 Reference Production Architecture (Control-Aware)

---

## Target Profile

| Attribute | Value |
|-----------|-------|
| Workload | Hybrid (read-heavy API + write transactions) |
| Traffic shape | Spiky + diurnal |
| Initial scale | 5k QPS peak |
| 12-month projection | 40k QPS |
| Worst-case spike | ×8 |
| Consistency | Hybrid |
| Region | Single-primary, multi-AZ |

---

# 🏗 High-Level Architecture

---

## SECTION 1 — EDGE LAYER

**Components:**

- CDN  
- WAF  
- Bot protection  
- Coarse rate limiting  

**Latency Budget:** < 15ms added  

**Controls:**

- Geo filtering  
- IP reputation checks  
- Request size cap (e.g., 1 MB)  
- TLS termination (TLS 1.3)  

**Failure Mode:**  
If CDN fails → traffic bypass allowed but WAF must remain.

---

## SECTION 2 — API GATEWAY / INGRESS

**Responsibilities:**

- Routing  
- Coarse rate limiting  
- Request size enforcement  
- Auth pre-check (optional)  
- Request ID injection  

**Hard Limits:**

- Max header size: 16 KB  
- Max body: service-defined  
- Idle timeout: 60s  

**Observability:**  
Emit metrics:

- `request_rate`  
- `rejected_rate`  
- `upstream_latency`

---

## SECTION 3 — APPLICATION SERVICE

**Stateless containers**  
Runtime choice: Go or Node (non-blocking)  
HTTP/2 enabled, keep-alive enabled  

### Concurrency Model

**Node:**  
- Avoid synchronous APIs  
- Worker pool for CPU tasks  

**Go:**  
- Bounded goroutines for external calls  
- Context cancellation enforced  

### Latency Budget (App Layer)

**Target:** ≤ 30ms p95  

**Example Breakdown:**

| Component | Latency |
|-----------|--------|
| Auth | 5ms |
| Business logic | 15ms |
| Serialization | 10ms |

### External Call Policy (MANDATORY)

- `timeout = min(upstream_p95 × 1.5, 250ms)`  
- Retries: 2  
- Backoff: exponential + jitter  
- Circuit breaker: enabled  

### Connection Pool (App → Postgres)

- Pool size = min((CPU cores × 4), DB max_conn × 0.25)  
- Connection max lifetime: 30m  
- Idle timeout: 5m  

**Telemetry Required:**

- `pool_wait_ms`  
- `pool_in_use`  
- `pool_timeouts`

---

## SECTION 4 — REDIS CACHE (L2)

**Use Cases:** hot reads, session data, rate limit counters, idempotency keys  

**Performance Targets:**  
- p95 latency: < 3ms  
- Hit rate: > 80%  

**Protections:**  

- Stampede protection via single-flight or request coalescing  
- TTL strategy: base 300s ±20% jitter  
- Memory policy: allkeys-lru, fragmentation monitoring  

**Failure Mode:**  
If Redis unavailable → degrade gracefully, never hard-fail critical reads unless correctness requires it.

---

## SECTION 5 — PRIMARY DATABASE (Postgres)

**Topology:** primary (write) + 1–2 read replicas, multi-AZ  

**Target Performance:**  
- p95 query < 50ms  
- p99 query < 120ms  

**Connection Limits:**  
- Example: DB max_conn = 400  
- App pool total ≤ 200  
- Admin/reserved = 50  

**Index Discipline:**  
- Every high-QPS query has supporting index  
- EXPLAIN reviewed  
- Rows scanned bounded  

**Query Guardrails (MANDATORY):**

- Slow query log (>100ms)  
- Auto EXPLAIN capture  
- Sequential scan alerts  
- Query fingerprint metrics  

**N+1 Detection:**  
- Instrumentation emits `queries_per_request`  
- Alert thresholds: warn >20, critical >50

---

## SECTION 6 — ASYNC QUEUE

(Example: Kafka / RabbitMQ / SQS class)  

**Use For:** email, analytics, webhooks, heavy processing (non-blocking user requests)  

**Processing Guarantees:** at-least-once OR exactly-once (rare)  

**Worker Controls:**  
- Visibility timeout  
- Dead-letter queue  
- Retry cap  
- Idempotency check  

---

## SECTION 7 — SECURITY IMPLEMENTATION

**JWT Validation Flow:** strict order

1. Parse token  
2. Verify signature  
3. Check algorithm allow-list  
4. Validate `exp`, `nbf`, `iss`, `aud`  
5. Attach identity  

> Failure at any step → reject  

**Authorization:** policy engine required, no raw role checks in business handlers  

**Rate Limiting Stack:** layered

- Edge: coarse IP  
- Gateway: per-IP  
- App: per-user/token  
- Redis: sliding window / token bucket  

---

## SECTION 8 — OBSERVABILITY STACK

**Metrics Backend (Prometheus-class):**

- p50/p95/p99 latency  
- Error rates  
- Saturation  
- DB latency  
- Cache hit rate  
- Retry rate  

**Distributed Tracing:**  
- Required propagation: `trace_id`, `span_id`, `request_id`  

**Logging:** structured JSON only, sensitive fields auto-redacted  

**SLO Alerting:** burn-rate alerts  

- Fast burn: 5 min  
- Slow burn: 1 hour  

---

## SECTION 9 — RESILIENCE CONTROLS

- Circuit breakers (DB optional soft, external APIs, cache optional soft)  
- Load shedding when CPU > 85% sustained, queue depth exceeds threshold, pool wait exceeds threshold  
- Backpressure via HTTP 429, queue pause, worker concurrency reduction  
- Graceful degradation per endpoint: optional data dropped first, stale cache window, feature flags as kill-switch  

---

## SECTION 10 — DEPLOYMENT HARDENING

**Container Spec:**

- Non-root user  
- Distroless / minimal base  
- Read-only root FS  
- No-new-privileges  
- seccomp profile, drop NET_RAW  
- CPU / memory limits  

**Network Layout:**

- DB & Redis in private subnets  
- Services in private subnets  
- Only ingress public  
- Egress allowlist enforced  

**Service-to-Service Security (multi-service):**

- mTLS required  
- Cert rotation ≤ 24h  
- Identity-based auth  

---

## 🧪 Expected Production Numbers (Reference)

| Metric | Target |
|--------|--------|
| API p95 | ≤ 90ms |
| API p99 | ≤ 180ms |
| DB p95 | ≤ 50ms |
| Redis p95 | ≤ 3ms |
| Error rate | < 0.1% |
| Retry amplification | < 15% |
| Cache hit rate | > 80% |

---

## ✅ Architecture Checklist Pass Criteria

- Metrics prove SLOs  
- Guardrails are automated  
- Failure modes documented  
- Controls are runtime-tunable  
- No hidden blocking paths  
- Blast radius bounded



---

## SECTION 1 — EDGE LAYER

**Components:**

- CDN  
- WAF  
- Bot protection  
- Coarse rate limiting  

**Latency Budget:** < 15ms added  

**Controls:**

- Geo filtering  
- IP reputation checks  
- Request size cap (e.g., 1 MB)  
- TLS termination (TLS 1.3)  

**Failure Mode:**  
If CDN fails → traffic bypass allowed but WAF must remain.

---

## SECTION 2 — API GATEWAY / INGRESS

**Responsibilities:**

- Routing  
- Coarse rate limiting  
- Request size enforcement  
- Auth pre-check (optional)  
- Request ID injection  

**Hard Limits:**

- Max header size: 16 KB  
- Max body: service-defined  
- Idle timeout: 60s  

**Observability:**  
Emit metrics:

- `request_rate`  
- `rejected_rate`  
- `upstream_latency`

---

## SECTION 3 — APPLICATION SERVICE

**Stateless containers**  
Runtime choice: Go or Node (non-blocking)  
HTTP/2 enabled, keep-alive enabled  

### Concurrency Model

**Node:**  
- Avoid synchronous APIs  
- Worker pool for CPU tasks  

**Go:**  
- Bounded goroutines for external calls  
- Context cancellation enforced  

### Latency Budget (App Layer)

**Target:** ≤ 30ms p95  

**Example Breakdown:**

| Component | Latency |
|-----------|--------|
| Auth | 5ms |
| Business logic | 15ms |
| Serialization | 10ms |

### External Call Policy (MANDATORY)

- `timeout = min(upstream_p95 × 1.5, 250ms)`  
- Retries: 2  
- Backoff: exponential + jitter  
- Circuit breaker: enabled  

### Connection Pool (App → Postgres)

- Pool size = min((CPU cores × 4), DB max_conn × 0.25)  
- Connection max lifetime: 30m  
- Idle timeout: 5m  

**Telemetry Required:**

- `pool_wait_ms`  
- `pool_in_use`  
- `pool_timeouts`

---

## SECTION 4 — REDIS CACHE (L2)

**Use Cases:** hot reads, session data, rate limit counters, idempotency keys  

**Performance Targets:**  
- p95 latency: < 3ms  
- Hit rate: > 80%  

**Protections:**  

- Stampede protection via single-flight or request coalescing  
- TTL strategy: base 300s ±20% jitter  
- Memory policy: allkeys-lru, fragmentation monitoring  

**Failure Mode:**  
If Redis unavailable → degrade gracefully, never hard-fail critical reads unless correctness requires it.

---

## SECTION 5 — PRIMARY DATABASE (Postgres)

**Topology:** primary (write) + 1–2 read replicas, multi-AZ  

**Target Performance:**  
- p95 query < 50ms  
- p99 query < 120ms  

**Connection Limits:**  
- Example: DB max_conn = 400  
- App pool total ≤ 200  
- Admin/reserved = 50  

**Index Discipline:**  
- Every high-QPS query has supporting index  
- EXPLAIN reviewed  
- Rows scanned bounded  

**Query Guardrails (MANDATORY):**

- Slow query log (>100ms)  
- Auto EXPLAIN capture  
- Sequential scan alerts  
- Query fingerprint metrics  

**N+1 Detection:**  
- Instrumentation emits `queries_per_request`  
- Alert thresholds: warn >20, critical >50

---

## SECTION 6 — ASYNC QUEUE

(Example: Kafka / RabbitMQ / SQS class)  

**Use For:** email, analytics, webhooks, heavy processing (non-blocking user requests)  

**Processing Guarantees:** at-least-once OR exactly-once (rare)  

**Worker Controls:**  
- Visibility timeout  
- Dead-letter queue  
- Retry cap  
- Idempotency check  

---

## SECTION 7 — SECURITY IMPLEMENTATION

**JWT Validation Flow:** strict order

1. Parse token  
2. Verify signature  
3. Check algorithm allow-list  
4. Validate `exp`, `nbf`, `iss`, `aud`  
5. Attach identity  

> Failure at any step → reject  

**Authorization:** policy engine required, no raw role checks in business handlers  

**Rate Limiting Stack:** layered

- Edge: coarse IP  
- Gateway: per-IP  
- App: per-user/token  
- Redis: sliding window / token bucket  

---

## SECTION 8 — OBSERVABILITY STACK

**Metrics Backend (Prometheus-class):**

- p50/p95/p99 latency  
- Error rates  
- Saturation  
- DB latency  
- Cache hit rate  
- Retry rate  

**Distributed Tracing:**  
- Required propagation: `trace_id`, `span_id`, `request_id`  

**Logging:** structured JSON only, sensitive fields auto-redacted  

**SLO Alerting:** burn-rate alerts  

- Fast burn: 5 min  
- Slow burn: 1 hour  

---

## SECTION 9 — RESILIENCE CONTROLS

- Circuit breakers (DB optional soft, external APIs, cache optional soft)  
- Load shedding when CPU > 85% sustained, queue depth exceeds threshold, pool wait exceeds threshold  
- Backpressure via HTTP 429, queue pause, worker concurrency reduction  
- Graceful degradation per endpoint: optional data dropped first, stale cache window, feature flags as kill-switch  

---

## SECTION 10 — DEPLOYMENT HARDENING

**Container Spec:**

- Non-root user  
- Distroless / minimal base  
- Read-only root FS  
- No-new-privileges  
- seccomp profile, drop NET_RAW  
- CPU / memory limits  

**Network Layout:**

- DB & Redis in private subnets  
- Services in private subnets  
- Only ingress public  
- Egress allowlist enforced  

**Service-to-Service Security (multi-service):**

- mTLS required  
- Cert rotation ≤ 24h  
- Identity-based auth  

---

## 🧪 Expected Production Numbers (Reference)

| Metric | Target |
|--------|--------|
| API p95 | ≤ 90ms |
| API p99 | ≤ 180ms |
| DB p95 | ≤ 50ms |
| Redis p95 | ≤ 3ms |
| Error rate | < 0.1% |
| Retry amplification | < 15% |
| Cache hit rate | > 80% |

---

## ✅ Architecture Checklist Pass Criteria

- Metrics prove SLOs  
- Guardrails are automated  
- Failure modes documented  
- Controls are runtime-tunable  
- No hidden blocking paths  
- Blast radius bounded
