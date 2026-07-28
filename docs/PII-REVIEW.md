# Pre-publish PII review

This repository was assembled from a private working tree. Publication is irreversible, so
it went through a two-layer gate before the first commit. This file is the record: what
each layer found, what changed, and which findings were dismissed and why. The dismissals
are the point — a gate whose false alarms get silently waved through is not a gate.

## The two layers

| | catches | how it fails |
|---|---------|--------------|
| `tools/pii_scan.sh` | shapes: keys, hosts, hex ids, times, name tokens | deterministic, blind to anything shapeless |
| `tools/pii_llm_review.sh` | meaning: a third party in prose, a routine, a quoted message | slow, over-flags |

Both run in `.githooks/pre-commit`; only the deterministic layer runs in CI.

## What the deterministic layer found

125 findings on the first pass, 38 blocking — all fixed or suppressed. It now exits 0 with
4 suppressions and 0 skipped checks. Two worth recording:

- **A bug let real content through.** The owner-identity pattern was matched
  case-sensitively, so a name in capitals scored zero while the same name in ordinary case
  was caught next to it. Fixed with an opt-in `-i` flag for that rule only.
- **A suppression I wrote hid a real finding.** I allowlisted wall-clock times in the
  README as schedule descriptions — true of all but one, a sentence pairing a time with a
  presence claim. The judgement layer caught it after the deterministic gate went green.
  The sentence was rewritten and the suppression narrowed.

## What the judgement layer found

- **Persona handles** — two-letter identifiers for the character appeared across guards
  and probes, linking her to a private toolchain by name. Removed.
- **Private tooling filenames** referring to scripts not in this repo. Parameterized.

Both classes were then written into the local rule file, so the cheap layer catches them
for free — the intended direction: the expensive reader finds a new class, the cheap
matcher inherits it.

## Dismissed, with reasons

Dismissed by hand, not by loosening the prompt:

| flagged as | actual | verdict |
|------------|--------|---------|
| "internal codename `avatar_iii`" | published vendor engine strings | dismissed |
| "internal tool name `ship_gate`" | a file in this repo | dismissed |
| "hardware inventory" | product categories, not an inventory | dismissed |
| "reference to a specific female subject" | the generated character, disclosed | dismissed |
| "internal incident history" | the derivation behind each threshold — the artifact | dismissed |
| "consent-gate workflow with a training video" | documents the gate, identifies nobody | dismissed |

The last is the interesting one: a reviewer tuned for confidentiality reads "here is the
incident that produced this number" as a leak. In a repo whose whole claim is that
thresholds were derived rather than typed, deleting the derivations would leave the numbers
unfalsifiable. The incidents stay.

## Two rules that survived

- **Recall lives in the prompt, precision lives in the post-filter.** Listing dismissed
  classes in the prompt taught the reviewer to ignore adjacent real findings. The ledger is
  applied only after the model answers and is never shown to it.
- **A per-call-billed privacy gate prices itself out of use.** The reviewer runs on the
  operator's flat-rate subscription, because a gate with a marginal cost per run is a gate
  you learn to skip.

## Honest limits

At sufficient caution every technical detail reads as internal, so the reviewer over-reads
at the advisory tier; the blocking tier is the calibrated one. The deterministic layer is
the gate wired to block a commit, and the one that must stay at zero. The roster used for
name matching only covers enrolled names — which is why the highest-risk source files were
excluded from this repository entirely rather than redacted.
