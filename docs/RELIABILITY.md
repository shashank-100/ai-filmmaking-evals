# Reliability

I built a checker to catch bad videos. It never once caught a bad video. So I took away
its power to block, instead of tuning it until it agreed with me.

## The gate that never worked

One stage had a quality gate in front of it: if it judged the output unusable, the stage
fell back to a known-good clip. Over 7 evaluations it fired once, and the clip it rejected
was fine.

The tally: **0 true positives, 1 false positive, 7 evaluations.** Every time the gate
expressed an opinion, acting on it made the product worse — the fallback replaced today's
real content with a generic clip from another day.

## Why it was not tuned

The obvious move is to adjust the threshold until the gate agrees with me. A gate with no
true positives has never demonstrated it can detect the thing it exists to detect. Tuning
it changes how often it is wrong, not whether it works.

**A detector that has never fired correctly is not a mis-tuned detector. It is an
unvalidated one, and tuning hides that.**

## What replaced it: fail open, fail loud

The gate stopped blocking. It still runs and judges, but its verdict is advisory:

| | before | after |
|---|--------|-------|
| bad output detected | fallback shown, silently | real output shown, alert raised |
| gate wrong | real content silently destroyed | nothing happens, alert is noise |
| gate unavailable | fallback shown, silently | real output shown, alert raised |

The asymmetry is the argument. When the gate was wrong in blocking mode, it destroyed the
day's output and nobody found out. When it's wrong in advisory mode, the cost is one
unnecessary alert. Given a measured 100% false-positive rate, the second is strictly
cheaper.

## The alert test is inverted on purpose

The alert fires on **anything that is not a clean success**, not on a list of known
failures. Written the usual way, a failure nobody anticipated produces silence, and
silence reads as success. Inverted, a novel failure is loud the first time it happens.
The cost is more false alarms — the correct trade for an unattended system, where the
alternative to a false alarm is not finding out.

## What this does not claim

n=7. Seven evaluations can't prove a detector useless in general. It is enough to say this
one had produced no evidence of working when it was demoted, and that letting an
unvalidated detector destroy real output was the worse mistake.
