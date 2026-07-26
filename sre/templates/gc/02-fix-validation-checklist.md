# Template 02: Fix Validation Checklist — Before Production Push

**What this surfaces:** The specific conditions under which a fix that shows good
results in a lower environment could still fail in production — before the fix is
pushed, not after. Specifically targets the environment gap: the difference between
what staging measures and what production load actually exercises.

**When to use:** After a proposed fix has been tested in a lower environment and
shows improvement. Before deciding to push that fix to production. Mandatory for
any fix applied to a system where lower-environment load does not match production
peak load characteristics.

---

## System prompt

You are a senior SRE reviewing a proposed production fix before deployment. Your
specific responsibility is to identify the conditions under which a fix that shows
improvement in a lower environment could still fail, degrade, or behave differently
in production.

You focus on the environment gap — the specific differences between lower-environment
testing conditions and production conditions that could cause a fix to behave
differently. You are not reviewing whether the fix is technically correct; you are
reviewing whether the test conditions were sufficient to validate the fix for the
specific production failure mode it is intended to address.

You are particularly sceptical of:
- Fixes validated at lower TPS than the production incident occurred at
- Fixes involving JVM configuration that has threshold or heuristic behaviour
  (where the fix works well below the threshold and breaks above it)
- Fixes where the improvement metric in lower environment is real but is measuring
  a proxy rather than the actual production failure mode (e.g. lower max pause in
  staging ≠ lower max pause at production heap occupancy)

---

## User prompt

A production incident has occurred. A fix has been proposed, tested in lower
environment, and shows improvement. I want to validate whether the lower environment
test was sufficient before pushing to production.

**[PRODUCTION INCIDENT]:** <brief description of what happened in production —
symptom, conditions, scale>

**[PROPOSED FIX]:** <what change was made — algorithm, configuration, code>

**[LOWER ENVIRONMENT RESULTS]:** <what metrics improved, by how much, under what
load conditions>

**[LOWER ENVIRONMENT LOAD vs PRODUCTION LOAD]:** <how closely does lower environment
reproduce production TPS, heap pressure, object allocation rate, data volume>

**[FIX MECHANISM]:** <how does this fix work — what specifically does it change
about how the system behaves>

Please do the following:

1. Identify the specific production condition that triggered the original incident
   — expressed as a measurable threshold or state (not "high load" but "Old Gen
   occupancy exceeding X% at Y TPS")
2. Evaluate whether the lower environment test reproduced that condition — yes, no,
   or partially with explanation
3. Identify the specific condition under which this fix would break or degrade
   even if it shows improvement at lower-environment load
4. List the 3 things that must be true in production for this fix to hold — these
   become your post-deployment monitoring criteria
5. State plainly: is the lower environment validation sufficient to push this fix
   to production, or does it need additional validation, and what would that look like

---

## What to expect in a good output

- Section 1 should produce a specific, measurable threshold — not a description
  of the symptom but the underlying state that caused it
- Section 3 is the most important output — it should describe a specific condition,
  not "higher load" generically
- Section 5 should give a clear yes/no, not a hedge — the point of this checklist
  is to produce a decision

---

## Worked example

**Production incident:** CPU at 100%, application crashes, 1000+ TPS, JDK 8
Parallel GC, Black Friday peak load.

**Proposed fix:** Switch from Parallel GC to CMS GC.

**Lower environment results:** 50% reduction in allocation failures, 84% reduction
in max GC pause time.

**Lower environment load vs production:** Lower environment did not reproduce
1000+ TPS Black Friday load. Tested at significantly lower sustained TPS.

**Fix mechanism:** CMS replaces Parallel GC's stop-the-world Old Generation
collections with concurrent mark-and-sweep phases that run alongside application
threads. Reduces STW pause duration at the cost of higher CPU usage from concurrent
GC threads.

**Claude's specific production condition (section 1):**

> "The specific production condition triggering the incident: Old Generation occupancy
> reaching near-capacity between Major GC collections, causing Parallel GC to run
> increasingly frequent and long stop-the-world Major collections at 1000+ TPS.
> Expressed as a threshold: Old Gen occupancy exceeding ~85-90% between successive
> Major GC cycles."

**Claude's evaluation of lower environment validation (section 2):**

> "No — the lower environment did not reproduce the triggering condition. CMS GC
> uses internal heuristics to decide when to begin a concurrent sweep; at lower TPS,
> Old Gen does not reach the occupancy levels at which these heuristics trigger too
> late. The 84% reduction in max pause seen in lower environment reflects CMS's
> genuine improvement over Parallel GC at moderate heap pressure. It does not
> validate CMS's behaviour when Old Gen fills rapidly at 1000+ TPS Black Friday load,
> which is the specific condition the fix needs to address."

**Claude's specific break condition (section 3):**

> "This fix will break in production if Old Gen fills faster than CMS's concurrent
> sweep can complete — a condition known as 'concurrent mode failure.' When this
> happens, CMS falls back to a stop-the-world collection, which is slower than
> Parallel GC's equivalent and results in longer pauses than the original problem.
> At 1000+ TPS with CMS's default heuristics, Old Gen can reach capacity before
> the concurrent sweep starts, triggering exactly this failure mode. The 17-second
> pause observed in production is the signature of CMS concurrent mode failure."

**Claude's verdict (section 5):**

> "No — the lower environment validation is not sufficient to push this fix to
> production. The fix addresses the right problem (STW pause duration in Old Gen
> collection) but the test environment did not reproduce the specific condition
> (Old Gen fill rate at 1000+ TPS) under which CMS's default heuristics become
> a bottleneck. Minimum additional validation: run a load test at production-level
> TPS (1000+) in the lower environment and capture Old Gen occupancy at the moment
> each Major GC triggers. If occupancy is consistently above 80% at trigger time,
> add CMSInitiatingOccupancyFraction to the fix before production push."

---

## Related templates

- [Template 01: Symptom to Hypothesis](01-symptom-to-hypothesis.md)
- [Template 03: GC Parameter Derivation](03-gc-parameter-derivation.md)