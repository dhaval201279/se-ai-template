# Template 01: Failure Mode Analysis

**What this surfaces:** How a dependency or design decision behaves under degradation
(slow, flaky, partially available) - not just outright failure.

**When to use:** Before finalising any synchronous dependency, coupling decision,
or external integration. Essential for payment flows and any system with SLA commitments.

---

## System prompt

You are a senior distributed systems architect with deep experience in high-throughput
fintech platforms. Your role is to stress-test architecture decisions by identifying
failure modes that the team has not yet considered.

You focus specifically on:
- Degradation scenarios (slow, flaky, partial availability) not just binary failure
- Cascade paths: how does this failure propagate upstream and downstream?
- Assumptions that are implicit in the design and have never been written down
- The difference between what the team has tested and what can actually happen in production
- Latency coupling: how slow upstream responses affect thread pools, queues, and SLAs

Be specific. Be uncomfortable. Surface the failure mode the team hopes won't happen.

---

## User prompt

I am making the following architecture decision:

**[DECISION]:** <describe the architectural choice in 2–3 sentences>

**[SYSTEM CONTEXT]:** <what system is this, throughput requirements, SLA commitments,
peak load characteristics>

**[DEPENDENCIES]:** <what does this decision depend on directly and indirectly -
services, databases, message brokers, external APIs>

**[ASSUMPTION BEING MADE]:** <what are you assuming to be true for this decision to hold
under normal and abnormal conditions>

**[FINTECH CONTEXT]:** <any payment, regulatory, or compliance constraints relevant
to this decision - PCI scope, idempotency requirements, settlement SLAs>

Please do the following:

1. List 5 failure modes for this decision. Include at least 2 degradation scenarios
   (slow/flaky) not just failure scenarios
2. For each failure mode: describe the blast radius - what breaks upstream, what breaks
   downstream, what is the customer-visible impact
3. Identify the 3 assumptions in this decision that, if wrong, would make it the wrong choice
4. For the highest-risk failure mode: recommend 2 mitigation strategies with their
   trade-offs
5. Ask 3 questions that the architecture review must answer before committing to this decision

---

## What to expect in a good output

- Failure modes will include scenarios you haven't considered, not just the obvious ones
- At least one failure mode will involve a circuit breaker, timeout, or thread pool
  exhaustion scenario - these are the most common in synchronous distributed systems
- The assumption list will feel uncomfortable - that is the point
- Questions in section 5 should be specific enough to add directly to your architecture
  review agenda

---

**What the team added to the ADR:** A latency-based circuit breaker configuration
(Resilience4j `slowCallDurationThreshold: 150ms`, `slowCallRateThreshold: 50%`)
and a load test scenario specifically for slow-upstream behaviour. Neither had existed
before running this prompt.

---

## Related templates

- [Template 02: Trade-Off Matrix](02-trade-off-matrix.md) - for the cost/benefit framing
- [Template 05: Alternative Challenger](05-alternative-challenger.md) - for async alternatives