# Backend Architecture Review Skill

## Purpose

Provide a structured framework for evaluating whether a backend architecture is **production-grade, secure, observable, resilient, and latency-aware**.

A backend system is incomplete if it cannot explain:

- WHAT it does
- HOW it works
- WHEN it fails
- HOW it self-recovers

This skill ensures backend designs are:

- measurable
- enforceable
- observable
- failure-aware
- control-tunable

---

# When To Use This Skill

Use this skill when evaluating or designing:

- backend architectures
- API systems
- distributed systems
- microservices
- database-backed applications
- production infrastructure
- system design proposals
- architecture review documents

Activate this skill when a request involves:

- system design
- scalability
- reliability
- backend performance
- infrastructure planning
- production readiness

---

# Core Principles

All production systems must follow these rules:

- Security must be **fail-closed**
- Latency is a **product feature**
- Tail latency must be treated as **first-class**
- All operational claims must be **measurable**
- Observability must enable **causal debugging**
- Systems must minimize **blast radius**
- Critical subsystems must expose **control knobs**

---

# Evaluation Process

When reviewing a backend architecture, evaluate the system using the following sections.

---

# 1. System Classification

## Workload Profile

Verify that the system defines:

- workload type  
  (read-heavy / write-heavy / streaming / batch / hybrid)

- traffic shape  
  (steady / spiky / diurnal / adversarial)

- current peak QPS

- projected 12-month QPS

- worst-case spike multiplier

If these are missing → **architecture is incomplete**

---

## Consistency Model

Check that the design explicitly defines:

- strong consistency zones
- eventual consistency zones
- async boundaries
- reconciliation mechanism
- maximum tolerated staleness window

Undefined boundaries → **invalid design**

---

## Scaling Strategy

Verify the system defines:

- horizontal vs vertical scaling
- shard key (if applicable)
- cache topology
- read replica strategy
- backpressure strategy

---

## Failure Mode Matrix

Every critical component must document:

| Component | Failure Mode | Detection | Mitigation | User Impact |
|----------|--------------|-----------|-----------|-------------|

If missing → **not production ready**

---

# 2. Latency Discipline

## Latency Budget

Check for a defined latency budget.

Example structure:

Total Budget: 100ms

- network
- TLS
- auth
- business logic
- database
- serialization
- queueing

All latency components must be **measured in production**.

---

## Latency SLO

Verify the system defines:

- p50 target
- p95 target
- p99 ceiling
- error budget

CI/CD should trigger alerts when:

- p95 exceeds SLO
- p99 exceeds 2× SLO

---

## Tail Latency Controls

Confirm monitoring exists for:

- event loop lag
- thread pool saturation
- lock contention
- GC pauses
- connection pool wait time
- retry amplification

---

## External Call Discipline

Every external dependency must implement:

- timeout
- retry cap
- exponential backoff with jitter
- circuit breaker

Retry traffic must remain:

**≤ 20% of baseline traffic**

---

## Cold Start Measurement

Services must expose:

- startup latency
- first request latency

---

# 3. Database Mechanics

## Query Discipline

Ensure rules exist:

- no `SELECT *` in production queries
- indexed access for high-QPS queries
- bounded result sets
- pagination required

---

## Query Guardrails

System should implement:

- slow query logging
- automatic EXPLAIN capture
- full table scan detection
- query fingerprint tracking
- per-query latency metrics

Target:


p95 DB query latency < 50ms


---

## N+1 Query Detection

Detection methods may include:

- query count per request
- ORM instrumentation
- SQL tracing

---

## Connection Pool Control

Pools must expose metrics:

- active connections
- wait queue
- timeout count
- saturation percentage

---

## Hot Partition Detection

If sharded, monitor:

- QPS per shard
- storage imbalance
- latency per shard

---

# 4. Security Enforcement

Security must be **fail-closed**.

---

## Transport Security

Verify:

- TLS 1.2+
- secure cipher suites
- HSTS enabled for public endpoints

---

## Authentication

JWT validation must include:

- signature verification
- expiration
- not-before
- issuer
- audience
- algorithm allow list
- key rotation support

Failure → **reject request**

---

## Authorization

Authorization must use **policy-based evaluation**.

Example model:


allow = policyEngine.evaluate(
subject,
action,
resource,
context
)


Role-only checks are insufficient for critical paths.

---

## Input Validation

Inputs must enforce:

- schema validation
- rejection of unknown fields
- Unicode normalization
- JSON depth limits
- array size limits
- regex backtracking protection
- payload size limits

---

## Rate Limiting

Verify support for:

- per-IP limits
- per-user limits
- per-token limits
- per-endpoint limits

Burst control and adaptive throttling must exist.

---

## Secrets Management

Check that:

- secrets are stored externally
- secrets never appear in logs
- secrets are not stored in repos
- automatic rotation exists

---

# 5. Caching Mechanics

## Cache Hierarchy

Design should define:

- L1 cache (in-process)
- L2 cache (distributed)
- L3 cache (persistent)

---

## Cache Contracts

Each cached object must specify:

- TTL
- jitter
- staleness tolerance
- invalidation authority
- write strategy

---

## Stampede Protection

System must implement at least one:

- request coalescing
- single-flight
- mutex lock
- probabilistic refresh

---

# 6. Threat Model

## Assets

Verify the system identifies critical assets:

- user data
- credentials
- authentication tokens
- payment information
- internal APIs

---

## Attacker Classes

Threat model must include:

- anonymous internet attackers
- authenticated malicious users
- botnets
- credential stuffing attackers
- insiders
- compromised dependencies

---

## Trust Boundaries

Architecture must define boundaries:

- public → API
- API → services
- services → database
- services → third parties

---

## Worst Case Breach Scenario

Document:

- maximum blast radius
- exposed data
- detection time
- containment time
- customer impact

---

# 7. Observability

## Golden Signals

Every service must track:

- latency
- traffic
- errors
- saturation

---

## Required Metrics

Expose:

- p50
- p95
- p99
- max latency
- error rate
- retry rate
- queue depth

---

## Structured Logging

Required fields:


request_id
trace_id
user_id
endpoint
latency_ms
status_code


Sensitive data must be redacted.

---

## Distributed Tracing

Trace context must propagate across services.

Trace spans should include:

- service name
- operation name
- dependency latency
- retry attempts
- errors

---

# 8. Resilience and Control Loops

Systems must implement:

- circuit breakers
- load shedding
- backpressure
- graceful degradation
- retry budget enforcement

Each must expose runtime tuning knobs.

---

# 9. Deployment Hardening

## Container Security

Containers must run with:

- non-root user
- minimal base image
- read-only filesystem
- dropped capabilities
- seccomp profile
- resource limits

---

## Network Isolation

Verify:

- database in private subnet
- no public database access
- service allowlists
- egress restrictions
- least privilege security groups

---

## Service-to-Service Security

If multiple services exist:

- mTLS required
- verified service identity
- short-lived certificate rotation

---

# Production Readiness Criteria

A backend system is **not production ready** unless:

- latency SLO defined
- failure modes documented
- observability implemented
- threat model completed
- rate limiting enforced
- runbooks documented
- security controls verified

Deployment should be **blocked** if these requirements are not met.

---

# Output Format When Using This Skill

When applying this skill, Claude should produce:

1. Architecture evaluation summary
2. Missing requirements
3. Identified risks
4. Recommended improvements
5. Production readiness score
