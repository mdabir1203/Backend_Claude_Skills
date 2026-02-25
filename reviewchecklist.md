# Backend Review Checklist
Security & Latency Approval Gate (Mechanism Verified)

This checklist verifies both:
- Outcome compliance
- Underlying mechanism correctness

Deployment is blocked if any critical item is unchecked.

---

# 1. Architecture

[ ] System classification documented
[ ] Consistency model defined
[ ] Latency budget defined
[ ] Scaling strategy documented
[ ] Failure modes documented
[ ] Mechanism explanations included (not just framework names)

---

# 2. Latency Discipline

[ ] p95 within budget
[ ] p99 < 2x budget
[ ] All external calls have timeouts
[ ] Retries capped
[ ] Circuit breakers implemented
[ ] Blocking I/O avoided
[ ] Cold start measured
[ ] Payload minimized

---

# 3. Database Mechanics

[ ] All queries indexed
[ ] EXPLAIN reviewed
[ ] No full table scans
[ ] No N+1 patterns
[ ] p95 DB query < 50ms
[ ] Connection pooling tuned

---

# 4. Security Enforcement

[ ] JWT signature verified
[ ] Expiration validated
[ ] Audience & issuer validated
[ ] Explicit permission checks
[ ] Input schema validation enforced
[ ] Unknown fields rejected
[ ] Payload size limited
[ ] Rate limiting enforced
[ ] Secrets managed externally
[ ] No secrets in logs

---

# 5. Threat Model

[ ] Assets identified
[ ] Attackers defined
[ ] Entry points listed
[ ] Trust boundaries mapped
[ ] Worst-case breach scenario defined
[ ] Mitigations documented

---

# 6. Observability

[ ] p50/p95/p99 monitored
[ ] 4xx/5xx monitored
[ ] Auth failures monitored
[ ] Structured logging enabled
[ ] Request IDs propagated
[ ] Alert thresholds defined
[ ] Alert thresholds tested

---

# 7. Deployment Hardening

[ ] Non-root containers
[ ] Minimal base image
[ ] Read-only filesystem
[ ] Security groups restricted
[ ] No public database access
[ ] mTLS for service-to-service (if multi-service)

---

# Final Gate

Security: PASS / FAIL  
Latency: PASS / FAIL  
Observability: PASS / FAIL  
Resilience: PASS / FAIL  
Mechanism Verified: PASS / FAIL  

If any FAIL → deployment blocked.
