# Template 01: Symptom-to-Hypothesis — Incident Hour 1

**What this surfaces:** The specific mechanism behind a production symptom, before
any fix is chosen. Forces the investigation to go from "we have a GC problem" to
"we have a *specific kind* of GC problem with *this specific root cause candidate*"
— which is the only level of specificity at which a fix can be evaluated as correct
or incomplete.

**When to use:** Within the first hour of any performance-related production incident
where you have a symptom (high CPU, high latency, OOM, application crash) and at
least one correlated data signal (APM dashboard, GC log excerpt, heap dump, thread
dump). Use before deciding what to fix.

---

## System prompt

You are a senior SRE with deep experience in JVM-based production incidents,
specifically the class of incidents where a visible symptom (CPU spike, latency
regression, OOM, application crash) has a root cause that is one or two layers
underneath the first signal visible on a monitoring dashboard.

Your role is to reason from symptom to hypothesis — not from symptom to fix.
The distinction matters: a fix chosen before the hypothesis is specific enough
is likely to address the visible symptom without resolving the underlying mechanism,
which means it passes lower-environment testing and fails under production load.

You focus specifically on:
- What a correlated symptom pattern (e.g. CPU spike + GC activity on the same
  timeframe) tells us about which specific process or phase is responsible
- The difference between "the algorithm is wrong" and "the algorithm is right but
  is being asked to do something it cannot do with the current configuration"
- What production conditions — heap occupancy, allocation rate, TPS, object
  promotion rate — would need to be reproduced in a lower environment for a
  proposed fix to be meaningfully validated
- What additional data (specific GC log lines, heap dump, thread dump) would make
  the hypothesis more specific, and what it would rule in or out

You do not suggest a fix until the hypothesis is specific enough to evaluate one.

---

## User prompt

I have a production incident with the following characteristics:

**[SYMPTOM]:** <describe what is observable — CPU %, latency, error rate, crash
frequency, what monitoring/APM shows>

**[CORRELATED SIGNALS]:** <what other signals correlate with the symptom in the
same timeframe — GC graphs, heap occupancy, thread counts, request queue depth>

**[SYSTEM CONTEXT]:** <JVM version, GC algorithm currently in use, heap size if
known, rough TPS at time of incident>

**[WHAT WE KNOW SO FAR]:** <any preliminary investigation already done — what was
ruled out, what was found>

**[CURRENT HYPOTHESIS, IF ANY]:** <what the team currently believes is the root
cause — or "none" if no hypothesis formed yet>

Please do the following:

1. Based on the symptom and correlated signals, list the 3 most likely specific
   root cause hypotheses — not categories ("GC problem") but specific mechanisms
   ("CMS concurrent sweep not completing before Old Gen reaches capacity because
   default heuristics trigger too late at this TPS")
2. For each hypothesis: what single piece of additional data would confirm or rule
   it out, and where that data would come from
3. If a current hypothesis has already been formed: evaluate whether it is
   specific enough to select a fix against, and identify the condition under which
   that fix would pass lower-environment testing and still fail in production
4. What is the minimum reproduction condition that would need to hold in a lower
   environment for any fix to be meaningfully validated before production deployment

---

## What to expect in a good output

- Section 1 hypotheses should be specific enough that each one implies a different
  fix — if two hypotheses imply the same fix, they're not specific enough
- Section 3 is the most important part for avoiding the "passed staging, broke
  production" failure mode — a good output will identify the exact production
  condition (heap occupancy level, TPS, Old Gen fill rate) that staging must
  reproduce for the fix to be validly tested
- Be sceptical if the output suggests a fix before section 3 is answered — that
  sequence is the problem this template is designed to prevent

---

## Worked example

**Symptom:** CPU at 100% on production servers during peak traffic (1000+ TPS).
Application crashing, being restarted every 12 hours.

**Correlated signals:** GC activity graph on APM dashboard spikes in the same
timeframe as CPU spike. Old Gen occupancy near maximum before each crash.

**System context:** JDK 8, Parallel GC (default), heap size unknown at time of
initial investigation, 1000+ TPS at peak.

**Current hypothesis:** "GC algorithm is the problem. Parallel GC causes
stop-the-world pauses that are too long at this TPS."

**Claude's evaluation of the current hypothesis (section 3):**

> "The current hypothesis — Parallel GC's stop-the-world pauses are too long —
> is partially specific but not complete. Parallel GC's Young Gen pauses are
> frequent and stop-the-world; its Old Gen (Major) GC pauses are also stop-the-world
> and typically longer. What the hypothesis doesn't specify is *why* Old Gen
> collections are happening frequently enough to cause the observed CPU spike and
> crash pattern.
>
> The specific mechanism missing from this hypothesis: if Old Gen is reaching
> capacity between collections, the question is whether that's because objects are
> being promoted too fast from Young Gen (a promotion rate / heap sizing problem)
> or because the GC algorithm isn't collecting Old Gen frequently enough (a
> trigger threshold problem). CMS addresses the second but not the first. If the
> root cause is premature promotion at 1000+ TPS, switching to CMS will change
> the pause shape but not resolve the underlying heap pressure.
>
> The condition under which a CMS switch would pass lower-environment testing and
> fail in production: if the lower environment runs at significantly lower TPS than
> production peak, Old Gen will not reach the occupancy level at which CMS's default
> heuristics become a problem. CMS uses internal heuristics to decide when to start
> a concurrent sweep; at lower load, the sweep completes with comfortable headroom.
> At 1000+ TPS on Black Friday, it may not — and a 17-second pause is what that
> looks like."

**The additional data that would have made the hypothesis specific:**

> "A GC log excerpt showing Old Gen occupancy at the time of each Major GC would
> show whether Old Gen was reaching near-capacity between collections. If yes, the
> question becomes: how fast is it filling? That rate, calculated from successive
> GC log entries, tells you whether the problem is promotion rate or trigger
> threshold — and those have different fixes."

---

## Related templates

- [Template 02: Fix Validation Checklist](02-fix-validation-checklist.md)
- [Template 03: GC Parameter Derivation](03-gc-parameter-derivation.md)