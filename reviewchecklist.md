Security & Latency Approval Gate (Final)

1. Architecture

[ ] Workload classification documented
[ ] Traffic shape defined
[ ] Consistency boundaries defined
[ ] Scaling strategy documented
[ ] Failure mode matrix complete
[ ] Mechanism explanations included

2. Latency Discipline

[ ] Latency budget defined
[ ] p95 within SLO
[ ] p99 < 2× SLO
[ ] Tail latency metrics instrumented
[ ] All external calls have timeouts
[ ] Retries capped and jittered
[ ] Retry budget enforced
[ ] Circuit breakers implemented
[ ] Blocking I/O avoided
[ ] Cold start measured
[ ] Payload size limits enforced

3. Database Mechanics

[ ] All high-QPS queries indexed
[ ] EXPLAIN plans reviewed
[ ] No full table scans in hot paths
[ ] Query guardrails active
[ ] No N+1 patterns
[ ] p95 DB query < 50ms (or justified)
[ ] Connection pool telemetry exposed
[ ] Pool saturation monitored
[ ] Hot shard detection (if sharded)

4. Security Enforcement

[ ] TLS enforced
[ ] JWT signature verified
[ ] exp validated
[ ] nbf validated
[ ] issuer validated
[ ] audience validated
[ ] algorithm allow-listed
[ ] Explicit permission checks
[ ] Input schema enforced
[ ] Unknown fields rejected
[ ] Payload size limited
[ ] Rate limiting active
[ ] Secrets externally managed
[ ] No secrets in logs

5. Threat Model

[ ] Assets identified
[ ] Attacker classes defined
[ ] Entry points listed
[ ] Trust boundaries mapped
[ ] Worst-case breach documented
[ ] Blast radius analyzed
[ ] Mitigations documented

6. Observability

[ ] Golden signals instrumented
[ ] p50/p95/p99 monitored
[ ] 4xx monitored
[ ] 5xx monitored
[ ] Auth failures monitored
[ ] Retry rate monitored
[ ] Structured logging enabled
[ ] Trace propagation verified
[ ] Alert matrix defined
[ ] Alerts tested

7. Resilience

[ ] Circuit breakers active
[ ] Load shedding implemented
[ ] Backpressure enforced
[ ] Graceful degradation defined
[ ] Control knobs exposed
[ ] Retry storm protection active

8. Deployment Hardening

[ ] Non-root containers
[ ] Minimal base image
[ ] Read-only filesystem
[ ] seccomp applied
[ ] no-new-privileges set
[ ] Resource limits defined
[ ] Security groups restricted
[ ] No public database access
[ ] Network allowlists enforced
[ ] mTLS enabled (if multi-service)

FINAL GATE

Security: PASS / FAIL
Latency: PASS / FAIL
Observability: PASS / FAIL
Resilience: PASS / FAIL
Control Loops: PASS / FAIL
Tail Safety: PASS / FAIL
Mechanism Verified: PASS / FAIL

If any FAIL → deployment blocked.
