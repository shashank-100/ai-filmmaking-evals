# Not measured

What this repository does not claim, and what it would take to claim it honestly. The
fastest way to lose a technical reader is one unearned number. Everything below is a
number I could have written and chose not to.

## Time saved

**Not claimed.** The pipeline runs end to end in a median of 33 minutes, across 4 timed
runs. That is a measurement of the pipeline, not a claim about time saved — I never
measured a human doing the same work by hand. Without a baseline there is no saving, only
a duration.

*To claim it:* a defined manual procedure, ≥5 timed human runs, the same quality bar
applied to both.

## Dollar cost

**Not claimed.** Daily render cost is 2 vendor credits, measured at 4 clip lengths.
Credits are not dollars, and I never recorded the credit-to-currency rate at measurement
time. Back-filling today's price onto an old measurement looks precise and isn't.

*To claim it:* record the plan and rate alongside each usage reading, at read time.

## Reliability as a rate

**Not claimed.** The README says 4 of 7, 14 of 15, 0 of 7 — never 57%, 93%, or 0%. Every
denominator is small enough that a percentage implies precision the sample can't support
(n=7 with 4 successes spans ~18%–90% at 95%). The counts are reported as counts on purpose.

## Quality of the output

**Not claimed.** Nine invariants score the output, each derived from labelled exemplars.
This measures agreement with **my own** labels — not whether the output is good, nor
whether the invariants cover every way it can fail.

*To claim it:* labels from someone who is not me.

## Generalization

**Not claimed.** One operator, one machine, one set of accounts, one display. Nothing here
has run on a second machine, which is exactly the class of problem the known guard
portability gaps ([ENFORCEMENT.md](ENFORCEMENT.md)) would surface immediately.

## The parallelism speedup

**Partly retracted.** The benchmark is real: 229s → 89s, byte-identical, n=1. The
*explanation* was wrong — I credited batched inference, but the setting meant to run ten
workers wasn't taking effect. The number survived; the causal story didn't. Both stay in
the repo, because the gap between them is the useful artifact.

## Why the lip-sync figure moves

**Not explained.** Across three looks, three voices, three tiers, dropped-onset rates
ranged 19–43% with no clean association to any variable. Two confident causes (the still,
then the engine) were both proven wrong. Two possibilities remain and can't be separated
here: the probe's 0.8s window is too tight for this voice, or every clip genuinely drops
phrases and the eye tolerates it. Until settled, the figure reports and does not gate.

## Known open bugs

Stated here rather than fixed silently:

1. The parallel worker setting doesn't take effect (4 of 6 runs ran serial).
2. Three of four guards fail open on a missing dependency, reporting success.
3. The depth speedup is attributed to the wrong mechanism.
4. Nothing checks the presenter is distinguishable from her background — a dark garment
   on a zero-luma fill passes every gate and ships a floating head.

A repo whose headline is "count attempts, not outcomes" would be a poor place to hide its
own open findings.
