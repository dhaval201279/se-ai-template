# se-ai-templates

> **AI-augmented prompt templates for Staff and Principal engineers in fintech and distributed systems.**

A growing library of Claude prompt templates across six engineering disciplines - built from 20+ years of production
experience in high-throughput payment infrastructure (1000+ TPS, sub-100ms p99, multi-region) and released openly as part
of a multiple-week series on AI-augmented engineering.

These are not generic AI prompts. Every template is calibrated for the constraints that fintech and distributed systems
engineers actually face: idempotency, regulatory scope, latency coupling, settlement batch workloads, and the kind of
failure modes that only surface under production load.

---

## What this repo is not

This is not a collection of "ChatGPT prompts for developers."

The templates here are designed around a specific workflow: AI generates the questions, you supply the answers, the
combination produces better decisions and documentation than either could alone. The value is not in the AI output - it
is in the questions the AI output forces you to answer honestly.

---

## The six disciplines

| # | Discipline                                            | Templates              | Status         | Blog post                                                                                                                                     |
|---|-------------------------------------------------------|------------------------|----------------|-----------------------------------------------------------------------------------------------------------------------------------------------|
| 1 | [Architecture Decision Records (ADR)](./adr/)         | 5 templates            | ✅ Available    | [Three Fintech Architecture Post-Mortems: What AI-Augmented Review Would Have Caught](https://dhaval-shah.com/fintech-post-mortem-ai-review/) |
| 2 | [Performance Engineering](./performance-engineering/) | GC Comparison Template | ✅ Available | []()                                                                                                                                          |
| 3 | [Database Optimisation](./database/)                  | Coming Soon            | 🔜 In progress | Query plans, index strategy, storage models                                                                                                   |
| 4 | [Site Reliability Engineering](./sre/)                | Coming Soon            |  | Post-mortems, runbooks, SLO design                                                                                                            |
| 5 | [AI-Augmented Coding](./coding/)                      | Coming Soon            | | Code review, refactoring, test generation                                                                                                     |
| 6 | [FinOps & Cost Engineering](./finops/)                | Coming Soon            | | Cloud cost, provisioning, unit economics                                                                                                      |

Star this repo to follow the series. A new discipline will be added in coming weeks.

---

## How to use any template in this repo

Every template follows the same three-part structure:

```
1. SYSTEM PROMPT
   Paste this into Claude's system prompt or Project Instructions.
   It sets the context, expertise level, and what Claude should focus on.

2. USER PROMPT
   Fill in the [PLACEHOLDERS] with your specific decision, system, or scenario.
   The more specific your input, the more specific the output.

3. WORKED EXAMPLE
   A real (or realistic anonymised) production scenario showing actual Claude output
   and what the engineering team added to their decision record as a result.
```

**The workflow that makes this useful:**

```
Your decision or scenario
        ↓
Fill in the User Prompt with your specific context
        ↓
Run it - treat the output as questions, not answers
        ↓
Answer Claude's questions from your own production experience
        ↓
Those answers are your documentation, your ADR, your runbook
        ↓
The gap between Claude's output and your answers = your engineering judgment
```

---

## What Claude consistently gets wrong (read before using)

These templates are calibrated to surface this. But knowing the blind spots in advance makes you a better user:

**1. Claude over-indexes on technical failure modes.**
It will miss regulatory and compliance forcing functions - PCI-DSS scope isolation, RBI guidelines, GDPR data
residency - unless you explicitly provide them in your user prompt context. Always add your compliance horizon.

**2. Claude assumes greenfield conditions.**
Its architectural alternatives will often be correct in principle and wrong given your team's current operational
maturity, deployment cadence, and on-call structure. The "Alternative Challenger" templates in particular need your
operational reality as a filter.

**3. Claude doesn't know your organisation.**
Stakeholder objections it generates are technically grounded but organisationally generic. The most dangerous objections
in your actual review will be political, historical, or budget-driven. You have to add those yourself.

**4. Claude's scale intuitions are approximate.**
Specific numbers - TPS thresholds, row count inflection points, latency budgets - in Claude's output are informed
estimates, not measurements. Always validate against your own load test results and profiling data.

The templates in this repo are designed with these blind spots in mind.

---

## Fintech and distributed systems context

These templates are calibrated for constraints that generic engineering templates miss:

- **Idempotency** - payment flows require exactly-once semantics; Claude's default distributed systems advice often
  assumes at-least-once. Templates flag this explicitly.
- **Regulatory scope isolation** - PCI-DSS, GDPR, AML/KYC boundaries are external forcing functions that arrive on
  compliance timelines, not engineering timelines. Templates surface this as an architectural risk class.
- **Latency coupling** - slow ≠ failed, but both violate SLAs. The difference matters for circuit breaker design, thread
  pool sizing, and timeout strategy. Templates distinguish degradation from failure throughout.
- **Settlement and batch workloads** - high-TPS payment systems have a dual workload: OLTP inserts during the day, bulk
  settlement processing at end-of-day. Storage and indexing decisions that look correct for OLTP often break under
  batch. Templates account for this.
- **Multi-region consistency** - financial data has regulatory residency requirements that constrain consistency model
  choices. Templates include this as a constraint axis.

---

## Production credentials

The worked examples in these templates are drawn from experience building and operating:

- Payment orchestration platforms at 1000+ TPS sustained, sub-100ms p99 latency
- Multi-region Azure infrastructure for financial transaction processing
- High-volume e-commerce payment systems on Oracle (1M+ rows, 1000+ TPS)
- KYC user profile systems at scale with regulatory audit trail requirements

All examples are anonymised. The failure patterns are real.

---

## About the Repository

This repo is the living artefact of a multi-week public series on AI-augmented engineering for fintech and distributed
systems practitioners.

Periodically:

- One blog post on [dhaval-shah.com](https://dhaval-shah.com) - deep-dive with post-mortems, worked examples, and the "
  where AI gets it wrong" section
- New templates added to this repo
- LinkedIn and Twitter threads with the compressed, shareable version

| Week | Topic                                            | Blog                              | Templates                      |
|------|--------------------------------------------------|-----------------------------------|--------------------------------|
| 1    | AI-Augmented Architecture Decisions              | [Read →](https://dhaval-shah.com) | [adr/](./adr/)                 |
| 2    | AI-Augmented Performance Engineering             | Coming                            | [performance/](./performance/) |
| 3    | AI-Augmented Database Optimisation               | Coming                            | [database/](./database/)       |
| 4    | AI-Augmented SRE & Incident Response             | Coming                            | [sre/](./sre/)                 |
| 5    | AI-Augmented Coding & Code Review                | Coming                            | [coding/](./coding/)           |
| 6    | AI-Augmented FinOps & Cost Engineering           | Coming                            | [finops/](./finops/)           |

---

## Contributing

The most valuable contributions are worked examples from real (anonymised) production scenarios - not more prompt
variations.

**What makes a good contribution:**

- A specific decision, system type, and scale context
- The actual Claude output (paraphrased is fine, but substance matters)
- What the prompt surfaced that the team hadn't considered
- What Claude missed or got wrong
- What the team added to their decision record as a result

Generic prompt variations without production grounding will not be merged.

See [CONTRIBUTING.md](./CONTRIBUTING.md) for the full guide.

---

## About

Built by [Dhaval Shah](https://dhaval-shah.com) - Principal Software Architect with 20+ years in fintech and distributed
systems, including payment infrastructure at Mastercard.

Featured in Spring Weekly Newsletter (2018, 2021, 2024, 2025).

[LinkedIn](https://linkedin.com/in/dhavalshah201279) · [Twitter / X](https://twitter.com/dhaval201279) · [Blog](https://dhaval-shah.com) · [GitHub](https://github.com/dhaval201279)

---

## Licence

MIT. Use freely. Attribution appreciated but not required.

If these templates save you a production incident or a bad architecture decision, consider sharing what you learned -
open a PR.
