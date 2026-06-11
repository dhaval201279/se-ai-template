# Template 03: Ten-Year Regret Test

**What this surfaces:** The decisions you will wish you had made differently once
your system has lived in production at 2x, 5x, and 10x current load - and the
locally rational optimisations that compound into structural problems.

**When to use:** Any storage model, data access pattern, or infrastructure choice
where the decision is being made at current load levels. Critical for database
architecture, indexing strategy, and schema design in high-TPS fintech systems.
Also use for any decision where the team has said "we can optimise later" without
specifying what metric triggers the optimisation.

---

## System prompt

You are a principal architect reviewing a data storage or access pattern decision
with the benefit of hindsight. You have watched this system run in production
through 3 growth inflection points. You know which early storage decisions aged
well and which ones were re-architected under production pressure - and the
specific Oracle and relational database behaviours that make some decisions
dangerous at scale.

Your specific expertise:
- Oracle database behaviour at high row counts (500K–5M rows) and write TPS (500–2000)
- The gap between query performance at 100K rows and at 1M+ rows
- How "locally rational" optimisations - virtual columns, JSON extraction, denormalisation -
  transfer costs rather than eliminate them, and where those costs surface under
  write-heavy, settlement-batch, or DDL-event conditions
- The write/read trade-off in every storage optimisation, and how it shifts under
  settlement batch jobs, reconciliation workloads, and peak transaction bursts

Your job: surface the cost transfer the team has not made explicit. Every benefit
claimed in a storage decision has a corresponding cost deferred to a higher load,
a different access pattern, or a DDL operation under live traffic. Find it before
production does.

---

## User prompt

I am making the following data storage or access pattern decision:

**[DECISION]:** <describe the storage model or access pattern choice - e.g. JSON
column, virtual columns, denormalised table, partitioning strategy>

**[SYSTEM CONTEXT]:** <what system, current row count, current TPS, read/write
ratio, peak load characteristics, Oracle / Postgres / other>

**[THE PROBLEM BEING SOLVED TODAY]:** <what query or access pattern is broken or
slow at current load - be specific about the query shape and the latency observed>

**[WHAT WE CHOSE AND WHY]:** <the approach selected and the explicit reasoning -
including what alternatives were rejected and why>

**[LOAD GROWTH ASSUMPTION]:** <what do you expect row count and TPS to look like
in 12 months and 24 months>

**[ACCESS PATTERNS]:** <what queries run most frequently, what runs in scheduled
batch jobs (settlement, reconciliation, reporting), what runs ad-hoc for operations>

Please do the following:

1. For every benefit claimed in this decision, identify the corresponding cost that
   is being deferred, transferred, or made invisible by the current load level.
   Name the specific database behaviour - not generic trade-off language
2. Describe how this decision behaves at 2× current TPS and 5× current row count.
   Which component degrades first and by what mechanism?
3. Identify the DDL or operational event - index rebuild, statistics refresh,
   bulk UPDATE, new column addition - most likely to cause a production incident
   with this storage model under live traffic
4. Write the post-mortem paragraph for this decision as it will read 18 months from
   now, if the team optimises within this model rather than replacing it
5. What is the single question your DBA and your on-call engineer would both want
   answered before committing to this approach?

---

## What to expect in a good output

- Section 1 (cost transfer) names specific database behaviours - CBO selectivity
  estimation on virtual columns, redo log amplification on computed columns, JSON
  path evaluation overhead - not generic trade-off language. If it could apply to
  any system, it's not specific enough
- Section 2 (behaviour at scale) identifies the specific bottleneck component and
  mechanism, not just "performance degrades"
- Section 3 (DDL risk) is consistently the most underestimated risk in storage
  decisions on live, high-TPS tables. Pay particular attention to this one
- Section 4 (past-tense post-mortem) surfaces the "locally rational optimisation"
  compounding pattern more clearly than any future-tense analysis

---

## Related templates

- [Template 01: Failure Mode Analysis](01-failure-mode-analysis.md) - for runtime degradation risks
- [Template 02: Trade-Off Matrix](02-trade-off-matrix.md) - for explicit cost/benefit documentation
```

---