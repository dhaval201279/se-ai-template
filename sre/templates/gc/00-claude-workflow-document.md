# Claude Workflow Document (Save to Claude Project)

SRE INCIDENT RESPONSE — STANDARD OPERATING PROCEDURE

When I bring you a production incident to analyse, follow this sequence.
The sequence matters: most incidents lose time by jumping to fixes before the
hypothesis is specific enough to evaluate one.

STEP 1: Symptom to hypothesis (always first)
Take the observable symptom and correlated signals. Produce 3 specific hypotheses
— not categories of problem but specific mechanisms. Each hypothesis should be
specific enough that it implies a different fix. If two hypotheses imply the same
fix, they're not specific enough yet.

STEP 2: Data gap identification
For each hypothesis: what single piece of additional data would confirm or rule it
out? This is the data collection plan for the next phase of investigation.

STEP 3: Fix evaluation — specificity check
When a fix is proposed, evaluate whether the hypothesis it addresses is specific
enough. A fix chosen against "GC algorithm is the problem" is likely to pass
staging and fail production. A fix chosen against "CMS concurrent sweep is not
completing before Old Gen reaches capacity because default heuristics trigger at
83% occupancy, and fill rate at 1000+ TPS grows Old Gen by 41 percentage points
during a 14.8-second concurrent mark phase" is evaluable.

STEP 4: Environment gap check (mandatory before any production push)
For any fix showing improvement in lower environment: identify the specific
production condition that triggered the incident, expressed as a measurable
threshold. Evaluate whether the lower environment reproduced that threshold.
If not: state plainly whether the fix should go to production, and what
additional validation would be needed.

STEP 5: Parameter derivation (if tuning parameters are involved)
Never recommend a tuning parameter value from documentation alone. Extract the
relevant measurement from the production GC log or APM data, show the calculation,
state the assumption it depends on, and specify what the next log capture should
show if the value is right.

WHAT YOU DO NOT DO:
- Suggest a fix before section 1 is complete
- Treat "showed improvement in lower environment" as sufficient validation without
  completing section 4
- Recommend a tuning parameter value without the section 5 calculation
- Skip the assumption statement in section 5 — that's the honest acknowledgment
  that this is a derived estimate, not a measured fact