# 🚀 Production Readiness Gate – GitHub Checklist Template

**Purpose:** Ensure backend systems meet all security, latency, resilience, and operational standards before production deployment.

---

## 1️⃣ Architecture Review

- [ ] Workload classification documented
- [ ] Traffic shape defined
- [ ] Consistency boundaries defined
- [ ] Scaling strategy documented
- [ ] Failure mode matrix complete
- [ ] Mechanism explanations included

### Workload Documentation
- [ ] Workload type (read-heavy / write-heavy / streaming / batch / hybrid)
- [ ] Traffic shape (steady / spiky / diurnal / adversarial)
- [ ] Peak QPS defined
- [ ] Projected 12-month QPS defined
- [ ] Worst-case spike multiplier defined

---

## 2️⃣ Latency Discipline

- [ ] Latency budget defined
- [ ] p95 latency within SLO
- [ ] p99 latency < 2× SLO
- [ ] Tail latency metrics instrumented

### External Dependencies
- [ ] Timeouts configured for all external calls
- [ ] Retries capped with exponential backoff & jitter
- [ ] Retry budget enforced
- [ ] Circuit breakers implemented
- [ ] Retry traffic ≤ 20% of baseline

### Runtime Performance
- [ ] Blocking I/O avoided
- [ ] Cold start latency measured
- [ ] Payload size limits enforced

---

## 3️⃣ Database Mechanics

- [ ] All high-QPS queries indexed
- [ ] EXPLAIN plans reviewed
- [ ] No full table scans in hot paths
- [ ] Query guardrails active
- [ ] No N+1 query patterns
- [ ] p95 DB query < 50ms (or justified)
- [ ] Connection pool telemetry exposed
- [ ] Pool saturation monitored
- [ ] Hot shard detection (if sharding) implemented

---

## 4️⃣ Security Enforcement

### Transport Security
- [ ] TLS enforced

### Authentication
- [ ] JWT signature verified
- [ ] JWT `exp` validated
- [ ] JWT `nbf` validated
- [ ] Issuer validated
- [ ] Audience validated
- [ ] Algorithm allow-listed

### Authorization
- [ ] Explicit permission checks implemented

### Input Validation
- [ ] Input schema enforced
- [ ] Unknown fields rejected
- [ ] Payload size limited

### Abuse Protection
- [ ] Rate limiting active (per-IP / per-user / per-token / per-endpoint)

### Secrets Management
- [ ] Secrets externally managed
- [ ] No secrets in logs or repos

---

## 5️⃣ Threat Model

- [ ] Assets identified
- [ ] Attacker classes defined
- [ ] Entry points listed
- [ ] Trust boundaries mapped
- [ ] Worst-case breach documented
- [ ] Blast radius analyzed
- [ ] Mitigations documented

---

## 6️⃣ Observability

### Golden Signals
- [ ] Latency instrumented
- [ ] Traffic monitored
- [ ] Error rate monitored
- [ ] Saturation monitored

### Metrics Coverage
- [ ] p50, p95, p99 monitored
- [ ] 4xx / 5xx rates monitored
- [ ] Auth failures monitored
- [ ] Retry rate monitored

### Logging
- [ ] Structured logging enabled
- [ ] Required fields: `request_id`, `trace_id`, `user_id`, `endpoint`, `latency_ms`, `status_code`
- [ ] Sensitive fields redacted

### Distributed Tracing
- [ ] Trace propagation verified

### Alerting
- [ ] Alert matrix defined
- [ ] Alerts tested

---

## 7️⃣ Resilience Controls

- [ ] Circuit breakers active
- [ ] Load shedding implemented
- [ ] Backpressure enforced
- [ ] Graceful degradation defined
- [ ] Runtime control knobs exposed
- [ ] Retry storm protection active

---

## 8️⃣ Deployment Hardening

### Container Security
- [ ] Non-root containers
- [ ] Minimal base image
- [ ] Read-only filesystem
- [ ] seccomp applied
- [ ] no-new-privileges enabled

### Resource Safety
- [ ] CPU limits defined
- [ ] Memory limits defined

### Network Security
- [ ] Security groups restricted
- [ ] No public database access
- [ ] Network allowlists enforced

### Service Security
- [ ] mTLS enabled (if multi-service)

---

## 9️⃣ Chaos Engineering Tests

- [ ] Database outage simulation
- [ ] Cache failure simulation
- [ ] Dependency timeout simulation
- [ ] Traffic spike simulation
- [ ] Queue backlog simulation

### Chaos Evaluation Criteria
- [ ] Service degradation is graceful
- [ ] Cascading failures prevented
- [ ] Automatic recovery occurs
- [ ] Alerts trigger correctly

---

## 🔟 Production Readiness Scorecard

| Category | Score (0-100) |
|----------|---------------|
| Security |               |
| Latency |               |
| Observability |         |
| Resilience |            |
| Infrastructure Hardening | |

**Minimum required score:** 85

---

## ✅ Automated CI/CD Gate

- [ ] `ci/security-check`
- [ ] `ci/latency-slo-check`
- [ ] `ci/observability-check`
- [ ] `ci/container-security-check`
- [ ] `ci/dependency-scan`
- [ ] `ci/static-analysis`

> If any check fails → deployment blocked.

---

## 🛠 Final Approval Gate

| Category | Status (PASS/FAIL) |
|----------|------------------|
| Security |                  |
| Latency |                  |
| Observability |            |
| Resilience |               |
| Control Loops |            |
| Tail Safety |              |
| Mechanism Verified |       |

### Deployment Decision
- [ ] Approved for Production
- [ ] Deployment Blocked

### Reviewer Sign-Off

| Role | Name | Date |
|------|------|------|
| Architecture Reviewer | | |
| Security Reviewer | | |
| SRE Reviewer | | |
| Final Approver | | |

---

## 📝 Post-Deployment Requirements (within 7 days)

- [ ] Review system latency
- [ ] Validate error budgets
- [ ] Repeat chaos testing
- [ ] Verify incident response readiness

> Failure to complete post-deployment validation requires a risk review.
