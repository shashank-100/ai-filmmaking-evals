# Enforcement

I wrote rules telling the AI what not to do, then measured whether it followed them. It
mostly did — but only because code was stopping it, not because the rules worked. The
code in [`../guards/`](../guards/) is what actually runs.

## The measurement

An unattended run calls a metered API. The prompt carries numbered hard constraints in
the strongest form available. Across 15 governed runs:

| | |
|---|---|
| Runs that *ended* with exactly one paid render | **14 of 15** |
| Runs that *attempted* exactly one paid render | **6 of 15** |
| Runs where a pre-call hook rejected the first attempt | **8** |

The first number is what a dashboard shows. The second is what the model actually did.
The gap between them is a piece of code. Prompt-level constraints held ~40% of the time;
the runtime guard brought effective compliance to 93%.

## The natural experiment

One run carried four constraints in identical form, same model, same context:

| constraint | mechanism | held |
|------------|-----------|------|
| refresh the pin allowlist before rendering | prompt only | yes |
| exactly one render call | prompt + pre-call hook | yes |
| at most two look generations | prompt only | **no** |
| poll inline rather than yielding | prompt only | **no** |

What separates them is **failure visibility**. The render constraint had a hook, so a
violation was blocked and loud. The pin-refresh was load-bearing for the next call, so
skipping it failed immediately. The other two fail silently and only cost money.

**Rank constraints by what happens when violated, not by how important they feel.** A
constraint whose violation is silent needs a mechanism; one whose violation blocks the
next step is safe in prose.

## The guards, and what they do when a dependency is gone

I fed each guard deliberately broken input:

| guard | file | on a missing dependency |
|-------|------|-------------------------|
| identity pin | `block_unpinned_identity.sh` | **fails open**, accidentally |
| pre-spend shim | `pre_render_sanity.sh` | **fails open, on purpose**, and says so |
| prop gate | `prop_gate.sh` | **fails open**, accidentally |
| ship gate | `ship_gate.sh` | **fails closed**, exit 64 |

Three of four approve everything when a file goes missing, and **two of those three
don't know they're doing it.** The shim's fail-open is a documented trade (a host shim
that started deciding things would become a second, divergent gate); the other two have
no such note. The ship gate is the exception, and only because it already had its
incident and was rewritten to refuse.

**A guard that fails open is not a guard, it is a log line.** The danger isn't that it
fails — it's that it looks identical to passing.

## Coverage is separate from correctness

The identity pin read the right file and refused the right values, but was enforced on
only one of two egress paths: it matched on the SDK tool name, so a raw HTTP call with
the same id was never inspected. The fix keys on the field in any payload aimed at the
vendor host. While testing, the guard **blocked the test harness itself** — a live hook
firing on a payload nobody enumerated, which is the guard proving it works.

## What this does not claim

The compliance numbers come from 15 runs — a tally, not a reliability estimate. The
fail-open findings come from synthetic payloads, not production incidents. What is
measured is what the guards *would* do.
