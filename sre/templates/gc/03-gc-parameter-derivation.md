# Template 03: GC Parameter Derivation from Production Log Data

**What this surfaces:** A principled starting estimate for a JVM GC tuning
parameter, derived from what the production GC log actually shows about the
application's memory behaviour — rather than sourced from documentation defaults
or blog recommendations that may not apply to this application's specific allocation
and promotion pattern.

**When to use:** Any time you need to set a GC tuning parameter and you have access
to a production GC log. Specifically designed for threshold parameters like
CMSInitiatingOccupancyFraction, G1HeapWastePercent, G1MixedGCLiveThresholdPercent,
and similar parameters where the right value depends on application-specific heap
behaviour, not a universal recommendation.

---

## System prompt

You are a JVM performance engineer specialising in GC parameter tuning for
production systems. Your approach to parameter selection is measurement-first:
the right value for any threshold or heuristic parameter comes from what the
application's production GC log shows about its memory behaviour, not from
documentation defaults or general recommendations.

You understand:
- How to read GC log data to extract Old Gen occupancy at collection trigger time,
  Old Gen fill rate between collections, and concurrent sweep duration
- How to use those measurements to derive a principled starting estimate for
  threshold parameters like CMSInitiatingOccupancyFraction
- The difference between a parameter value that works (empirically found to reduce
  incidents) and a parameter value that is correct (derived from measurement and
  understood to work for a specific reason)
- That a derived starting estimate still requires empirical validation — but it
  gives the tuning process a principled anchor rather than a documentation lookup

You are clear about what the GC log data can and cannot tell you, and you flag when
additional data (a second log capture, a load test result) would be needed to
increase confidence in a derived value.

---

## User prompt

I need to derive a principled starting estimate for a JVM GC tuning parameter,
based on production GC log data.

**[PARAMETER TO DERIVE]:** <name of the JVM flag and what it controls>

**[GC LOG DATA]:** <paste relevant excerpts from the production GC log — ideally
showing Old Gen occupancy before and after each Major GC, and timestamps to allow
fill rate calculation>

**[SYSTEM CONTEXT]:** <JVM version, GC algorithm, heap size, approximate peak TPS
at time of log capture>

**[WHAT WENT WRONG WITHOUT TUNING]:** <describe the observed failure — e.g. 17-second
max GC pause, concurrent mode failure, Old Gen fully occupied>

Please do the following:

1. From the GC log data provided, extract: Old Gen occupancy at the time each
   Major GC triggered, the interval between successive Major GC events, and the
   estimated Old Gen fill rate per minute at peak load
2. Using those measurements, calculate a principled starting estimate for the
   requested parameter. Show the reasoning, not just the number
3. State the assumption this calculation depends on — what would need to be true
   about the application's behaviour for this estimate to be correct
4. State what you would expect to see in the next GC log capture if this estimate
   is right — give a specific, measurable indicator, not "improvement in GC behaviour"
5. Identify what additional data would increase confidence in this estimate before
   treating it as a stable tuning value

---

## What to expect in a good output

- Section 1 should extract actual numbers from the log data you provided, not
  describe the extraction process in the abstract
- Section 2's calculation should be explicit enough to check — if you can't follow
  the reasoning, it hasn't shown its work
- Section 3's assumption statement is the honest acknowledgment that this is a
  derived estimate, not a measured fact — it should be specific, not "assumes stable
  load patterns"
- Section 4's expected indicator should be specific enough to evaluate against the
  next GC log capture — a number or range, not a direction

---

## Worked example

**Parameter to derive:** CMSInitiatingOccupancyFraction — the percentage of Old
Generation occupancy at which CMS should begin its concurrent sweep.

**GC log data provided (representative excerpts, JDK 8 CMS format):**

[GC (CMS Initial Mark) 4291M->4291M(5120M), 0.8234560 secs] [CMS-concurrent-mark: 12.456/14.832 secs] [CMS-concurrent-sweep: 8.234/9.112 secs] [GC (CMS Final Remark) 4987M->4987M(5120M), 17.234 secs]


Approximate timestamps between successive Major GC events: 8-12 minutes at peak
load. Old Gen at trigger time: approximately 84% (4,291M of 5,120M). Old Gen at
final remark: approximately 97% (4,987M of 5,120M).

**System context:** JDK 8, CMS GC (default heuristics, no CMSInitiatingOccupancyFraction
set), 5,120M heap, 1000+ TPS at peak. Observed failure: 17-second Final Remark pause,
concurrent mode failures during peak traffic.

**Claude's extraction (section 1):**

> "From the log data:
> - Old Gen occupancy at CMS Initial Mark trigger: 4,291M of 5,120M = **83.8%**
> - CMS concurrent mark duration: 14.8 seconds
> - CMS concurrent sweep duration: 9.1 seconds
> - Total concurrent phase duration: approximately 24 seconds
> - Old Gen at Final Remark: 4,987M = **97.4%** — grew by 696M (13.6%)
    >   during the 14.8-second concurrent mark phase
> - Old Gen fill rate during concurrent mark: approximately 696M / 14.8s =
    >   **47M per second**, or roughly **2.8% of total heap per second** at peak load"

**Claude's calculation (section 2):**

> "To avoid concurrent mode failure, CMS must complete its concurrent phases before
> Old Gen reaches capacity. The concurrent mark phase takes approximately 14.8
> seconds. During that time, at 2.8% of heap per second, Old Gen grows by
> approximately 41 percentage points (2.8% × 14.8s).
>
> Working backward: if CMS must start when Old Gen has at least 41 percentage
> points of headroom remaining before reaching 100%, the latest safe trigger
> point is approximately 100% - 41% = **59%** Old Gen occupancy.
>
> Adding a safety margin (because fill rate is not perfectly constant and the
> concurrent sweep also has duration): trigger at **55-60%**.
>
> The value of 70 sourced from documentation gives less headroom than this
> calculation suggests is needed for this application's fill rate at 1000+ TPS.
> A starting estimate of **60** would be more conservative and more appropriate
> given the observed fill rate. The team can validate upward from 60 if it causes
> more frequent collections than necessary — that's a better starting point than
> validating downward from 70 after observing concurrent mode failures."

**Claude's assumption (section 3):**

> "This calculation assumes the Old Gen fill rate during concurrent mark (47M/s)
> is representative of Black Friday peak load. If the fill rate is higher during
> the absolute peak of the traffic spike than during the log capture period, a
> lower threshold (50-55%) would be safer. The log capture conditions should be
> confirmed to represent the peak TPS at which concurrent mode failures occurred."

**Claude's expected indicator (section 4):**

> "If CMSInitiatingOccupancyFraction=60 is correct, the next GC log should show
> CMS Initial Mark triggering at approximately 55-62% Old Gen occupancy (allowing
> for some heuristic variation with UseCMSInitiatingOccupancyOnly=true also set).
> Old Gen occupancy at Final Remark should be below 90%, with concurrent sweep
> completing with at least 10% headroom remaining. If Final Remark still shows
> Old Gen above 95%, the fill rate during the concurrent mark phase is faster
> than this calculation assumed and the threshold needs to move lower."

---

## Related templates

- [Template 01: Symptom to Hypothesis](01-symptom-to-hypothesis.md)
- [Template 02: Fix Validation Checklist](02-fix-validation-checklist.md)