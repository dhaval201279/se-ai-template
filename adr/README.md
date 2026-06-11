# adr-ai-templates

5 Claude prompt templates for AI-augmented Architecture Decision Records.

Built for Staff and Principal engineers working on fintech and distributed systems.
Retrospectively crafted based on learnings from fintech platforms that operate at high 
throughput and low latency

---

## Why this exists

Architecture decisions fail silently.

By the time you feel the pain - the cascading timeout, the schema migration, the 
PCI-scope extraction - you've already built your fintech platform on top of the 
wrong foundation.

The failure mode isn't due to bad engineers. It's because of assumptions that were never 
written down and never challenged.

These prompts surface failure modes, unstated assumptions, and regrets 
before you write a single line of code. They don't replace architectural judgment. 
They make your assumptions visible so your judgment has something to work on.

---

## The 5 templates

| # | Template | What it surfaces |
|---|----------|-----------------|
| 01 | [Failure Mode Analysis](templates/01-failure-mode-analysis.md) | How this decision fails under degradation, not just outright failure |
| 02 | [Trade-Off Matrix](templates/02-trade-off-matrix.md) | What you are actually giving up - written down, not assumed |
| 03 | [Ten-Year Regret Test](templates/03-ten-year-regret-test.md) | The question your future self will wish someone had asked |
| 04 | [Stakeholder Objection Simulator](templates/04-stakeholder-objection.md) | Every pushback before the review room |
| 05 | [Alternative Architecture Challenger](templates/05-alternative-challenger.md) | The 3 architectures you didn't fully consider |

---

## How to use

1. Copy the **System Prompt** from your chosen template into Claude's system prompt (or Project Instructions)
2. Fill in the **User Prompt** placeholders with your decision context
3. Run the prompt - treat the output as questions to answer, not answers to accept
4. Document your answers in your ADR. The questions are the value; your answers are the record.

### The Claude workflow

```
Decision identified
↓
Run Template 01 (Failure Mode Analysis) - what breaks this?
↓
Run Template 03 (Ten-Year Regret Test) - what will we wish we'd done?
↓
Run Template 05 (Alternative Challenger) - what didn't we fully consider?
↓
Write the ADR with answers to every question Claude raised
↓
Run Template 04 (Stakeholder Objection Simulator) - rehearse the review
↓
Architecture review - bring Claude's questions, bring your answers
```

**Time investment:** 20–30 minutes per decision. Versus 2-hour review meetings 
where the same 3 failure modes ship to production anyway.

---

## Fintech-specific context

These templates are calibrated for fintech and distributed systems constraints 
that generic architecture templates miss:

- **Idempotency requirements** in payment flows
- **PCI-DSS / regulatory scope isolation** as an external forcing function
- **Latency coupling** (slow ≠ failed, but both kill SLAs)
- **Schema evolvability** across multi-consumer event-driven systems
- **Multi-region consistency** trade-offs under financial data requirements

The "worked examples" in each template are drawn from real production scenarios 
in high-throughput payment infrastructure.

---

## What Claude gets wrong (read this)

These prompts are powerful. They also have consistent blind spots:

1. **Claude over-indexes on technical failure modes.** It will miss regulatory and 
   compliance forcing functions unless you explicitly provide them in context.

2. **Claude assumes greenfield.** Its alternatives will often be architecturally 
   correct and operationally impossible given your team's current maturity. 
   Filter accordingly.

3. **Claude doesn't know your organisation.** The stakeholder objections it generates 
   are generic. The most dangerous objections in your review will be political, not 
   technical. You have to add those yourself.

The gap between what Claude produces and what only you can add - that gap is your 
architectural judgment. Protect it. That's the moat.

---

## Reference

Full blog post with before/after case study: [dhaval-shah.com](https://dhaval-shah.com)  
Multi part series on AI-augmented engineering across architecture, performance, 
database optimisation, SRE, and FinOps.

LinkedIn: [linkedin.com/in/dhavalshah201279](https://linkedin.com/in/dhavalshah201279)  
Twitter: [@dhaval201279](https://twitter.com/dhaval201279)

---

## Contributing

Contributions welcome. If you have a worked example from a real (anonymised) 
production scenario, open a PR against `/examples/`. 

See [CONTRIBUTING.md](CONTRIBUTING.md).

## Licence

MIT
```

---