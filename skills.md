Purpose

Define a secure, low-latency, production-grade backend architecture standard that is:

measurable

enforceable

observable

failure-aware

control-tunable

A backend system is incomplete if it cannot explain:

WHAT it does

HOW it works (protocol/runtime/query)

WHEN it fails

HOW it self-recovers

CORE PRINCIPLES

Secure by default (fail closed)

Latency is a product feature

Tail latency is first-class

Every claim must be measurable

Minimize attack surface and blast radius

Observability must enable causal debugging

Every critical subsystem must expose control knobs

SECTION 1 — SYSTEM CLASSIFICATION
Workload Profile (Required)

Document:

workload type (read-heavy / write-heavy / streaming / batch / hybrid)

traffic shape (steady / spiky / diurnal / adversarial)

current peak QPS

projected 12-month QPS

worst-case spike multiplier

Mechanism Requirement

Autoscaling policy must reference the workload shape.

Consistency Model (Required)

Define explicitly:

strong consistency zones

eventual consistency zones

async boundaries

reconciliation mechanism

maximum tolerated staleness window

Failure to define boundaries → design invalid

Scaling Strategy (Required)

Must specify:

horizontal vs vertical

shard key (if applicable)

cache topology

read replica policy

backpressure strategy

Failure Mode Matrix (Required)

Every critical component must document:

Component	Failure Mode	Detection Signal	Automatic Mitigation	User Impact

System without this → not production-ready

SECTION 2 — LATENCY DISCIPLINE
Latency Budget (Required)

Example structure:

Total Budget: 100ms

network: X ms

TLS: X ms

auth: X ms

business logic: X ms

DB: X ms

serialization: X ms

queueing: X ms

All components must be measured in production.

Latency SLO (Hard Requirement)

Define:

p50 target

p95 target

p99 hard ceiling

error budget

Enforcement

CI/CD must fail or alert when:

p95 exceeds SLO

p99 > 2× SLO

Tail Latency Controls (Required)

System must monitor and expose:

event loop lag / thread pool saturation

lock contention

GC pause time

connection pool wait time

retry amplification

External Call Discipline (Required)

Every outbound call must implement:

timeout (adaptive)

retry cap

exponential backoff with jitter

circuit breaker

Retry Budget Rule

Total retry traffic ≤ 20% of baseline traffic.

Cold Start Measurement (Required)

Services must export:

startup latency

first-request latency

Alert if regression exceeds threshold.

Payload Minimization (Required)

Each endpoint must define:

max request size

max response size

Requests exceeding limits → rejected early.

SECTION 3 — DATABASE MECHANICS
Query Discipline (Required)

Rules:

no SELECT * in production paths

indexed access for all high-QPS queries

bounded result sets

pagination required for collections

Query Guardrails (Required)

System must implement:

slow query log

automatic EXPLAIN capture

full table scan detector

query fingerprint tracking

per-query p95 monitoring

Hard SLO

p95 DB query latency < 50ms (unless justified).

N+1 Prevention (Required)

Detection via:

query count per request metric

ORM instrumentation or SQL tracing

Threshold breaches must alert.

Connection Pool Control

Pool must expose:

active connections

wait queue length

timeout count

saturation %

Auto-tuning policy must be documented.

Hot Partition Detection (if sharded)

Monitor:

QPS per shard

storage skew

p95 per shard

Automatic rebalancing trigger must exist.

SECTION 4 — SECURITY ENFORCEMENT

Security must be fail-closed.

Transport Security

Required:

TLS 1.2+

secure cipher suites

HSTS (for public endpoints)

Authentication (JWT Requirements)

Must validate:

signature (RS256 or stronger)

expiration (exp)

not-before (nbf)

issuer (iss)

audience (aud)

algorithm allow-list

key rotation support

clock skew tolerance

Failure → immediate reject.

Authorization (Policy-Based)

Must use explicit permission evaluation:

allow = policyEngine.evaluate(subject, action, resource, context)

Role-only checks are prohibited in critical paths.

Input Validation (Strict)

Validation must:

enforce schema

reject unknown fields

normalize Unicode

limit JSON depth

cap array lengths

guard against regex backtracking

enforce payload size limits

Rate Limiting (Multi-Dimensional)

Must support:

per-IP

per-user

per-token

per-endpoint

Must include burst control and dynamic throttling.

Secrets Management

Required:

secrets stored in external manager

no secrets in repo

no secrets in logs

automatic rotation supported

SECTION 5 — CACHING MECHANICS
Cache Hierarchy

Define L1/L2/L3 usage and expected latency per tier.

Cache Contract (Required)

Each cached object must define:

TTL

jitter

staleness tolerance

invalidation authority

write strategy (through/behind)

Stampede Protection (Required)

At least one must be implemented:

single-flight

request coalescing

mutex lock

probabilistic early refresh

SECTION 6 — THREAT MODEL
Assets

Must enumerate:

user data

credentials

tokens

payment data

internal APIs

Attacker Classes

Must include:

anonymous internet

authenticated user

botnet

credential stuffer

insider

compromised dependency

Trust Boundaries

Must map:

public → API

API → services

services → database

services → third parties

Worst-Case Breach Scenario

Document:

maximum blast radius

data exposed

time to detect

time to contain

customer impact

SECTION 7 — OBSERVABILITY
Golden Signals (Mandatory)

Track per service and dependency:

latency

traffic

errors

saturation

Metrics Requirements

Must expose:

p50 / p95 / p99 / max

4xx rate

5xx rate

auth failure rate

retry rate

queue depth

Structured Logging

Required JSON fields:

{
  request_id,
  trace_id,
  user_id,
  endpoint,
  latency_ms,
  status_code
}

Sensitive data must be redacted.

Distributed Tracing (Mandatory)

Trace context must propagate across services.

Alert Matrix (Required)

Alerts must include:

fast SLO burn

slow SLO burn

error spikes

dependency latency regression

auth anomaly detection

Alerts must be tested in staging.

SECTION 8 — RESILIENCE & CONTROL LOOPS

System must implement:

circuit breakers

load shedding

backpressure

graceful degradation

retry budget enforcement

Each must expose runtime tuning knobs.

SECTION 9 — DEPLOYMENT HARDENING
Container Requirements

non-root user

minimal base image

read-only filesystem

no-new-privileges

drop unused capabilities

seccomp profile

CPU/memory limits

Network Isolation Rules (Mandatory)

database in private subnet

no public DB access

service allowlists enforced

egress restrictions defined

security groups least-privilege

Service-to-Service Security (if multi-service)

mTLS required

service identity verified

short-lived cert rotation

PRODUCTION READINESS DEFINITION

Deployment is blocked unless all sections are satisfied and measurable.
