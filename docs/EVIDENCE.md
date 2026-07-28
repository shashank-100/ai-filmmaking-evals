# Evidence

Every number in the README, with where it came from and what you can check yourself.

**Read this first.** The pipeline runs on one machine against my own vendor accounts. Its
logs are not in this repo and won't be — they carry schedule, account, and third-party
detail. So most rows below say *not independently verifiable*. That is the honest state of
a single-operator project. What you *can* verify is the code: the probes, the guards, and
the thresholds are all here and all readable.

## What each number is counted from

| Number | Counted from | Verifiable by a reader? |
|--------|--------------|-------------------------|
| Runs on schedule, 10 of 10 days | scheduler log | No, log not published |
| Full chain 4 of 7 | per-run stage markers | No |
| Quality gate 0 true positives in 7 | the gate's decision log | No |
| Constraint held 14/15, first attempt 6/15 | run transcripts | No |
| Daily cost 2 credits | vendor usage page | No |
| Nine invariants, 13 probes | this repository | **Yes** — `ls probes/` |
| Three of four guards fail open | this repository | **Yes** — method below |

## The two numbers that matter most

`14 of 15` and `6 of 15` are the same rule counted two ways:

- **14 of 15** — how many runs *ended* correct. What a dashboard shows.
- **6 of 15** — how many runs *attempted* the correct thing without being stopped.

The difference is a pre-call hook rejecting a wrong first attempt. Only the second tells
you whether the instruction worked, and it's the number a dashboard never shows — by the
time anything is recorded, the outcome is already correct. **An outcome metric cannot
distinguish a system that complied from a system that was stopped. Count attempts.**

## The one thing you can reproduce

The fail-open finding needs no logs. Take any guard in `guards/`, remove the file it
depends on, feed it input it should reject, and read the exit code:

```
IDENTITY_PINS=/nonexistent/pins.json bash guards/block_unpinned_identity.sh < a-payload.json
echo $?
```

Three of four exit 0 and print nothing alarming. `ship_gate.sh` exits 64 — it's the only
one that already had its incident. See [ENFORCEMENT.md](ENFORCEMENT.md).

## Sample sizes

n=15 for compliance, n=7 for chain completion and the gate, n=4 for latency, n=1 for the
benchmark. These are tallies, not reliability estimates — a rate from 7 observations has a
confidence interval you could drive a truck through. Dollar cost and time saved are
deliberately absent; see [NOT-MEASURED.md](NOT-MEASURED.md).
