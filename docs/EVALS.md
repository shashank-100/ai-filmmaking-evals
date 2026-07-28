# Evals

A rendered human fails in ways no checksum can see. So I watched clips, wrote down
verdicts, and turned the verdicts into numbers that can block a render. The numbers
keep being wrong — and that is the interesting part.

```
label  ->  derive  ->  gate  ->  render  ->  relabel
```

## 1. Label

Verdicts come first, in plain language, on real takes:

| set | size |
|-----|------|
| Labelled stills | 113 |
| Labelled videos | 67 |
| Frame-level identity labels | 174 records (115 `her` / 59 `not_her`) |
| A/B verdict log | 677 records |
| Render ledger | 14 renders (7 kept, 3 rejected, 4 unrated) |

The labels are the only ground truth in the building — never smoothed as noise.

## 2. Derive

Every threshold in [`../probes/`](../probes/) is computed from a labelled pass and a
labelled fail exemplar. None is typed from intuition, because intuition lost repeatedly:

- **Ten models died in one day** — every one inverted on the labelled set. The
  survivor (`eye_eval.py`) puts its background bar (4.5) between the worst labelled
  pass (3.30) and the best labelled reject (5.32); `--validate` exits nonzero unless
  it agrees with the labels 100%.
- **A metric can encode "look like last time."** A brightness bar set to 8.0 because
  one engine's clip measured 7.9 secretly meant *resemble that engine*, and steered
  choices for six hours while gating nothing.
- **Normalization smuggles in values.** A spasm score that divided motion-in-silence
  by motion-in-speech punished excited delivery for being excited; now it reports the
  two terms separately.

## 3. Gate

Thresholds compile into guards that run before money is spent.

- **Stability before authority.** A lip-sync metric matched the labels 8/8 and was
  wired in as a blocker within minutes — then swung 6–10 frames against itself inside
  one clip and was demoted the same hour. *A metric that agrees with your labels is
  not yet a metric; it must be stable within a clip before it can gate anything.*
- **Blind judging.** Cases are duplicated under opaque names; the grader can't see the
  label it grades.
- **Rank by consequence.** The same constraint held 14/15 runs at the outcome but only
  6/15 at the first attempt — the gap is a pre-call hook. See [ENFORCEMENT.md](ENFORCEMENT.md).

## 4. Relabel

When eye and instrument disagree, the disagreement is the data:

- **Wrong physics.** Thirteen metrics correlating continuous motion against audio
  found nothing about gesture timing. One human sentence — "the movement lags the
  speech" — plus the literature (gesture aligns to pitch accents as discrete events)
  fixed it: a late gesture is caught at ~200ms, an early one forgiven.
- **Sign flipped.** A rest meter discarded as useless was reinstated inverted once
  robotic takes measured 2–8% rest against a human reference's 10–17%.
- **Stop steering when every predictor inverts.** On the day the ten models died,
  look selection was handed to seeded random and the human relabelled by hand.

## Two failures a metric cannot report

Both caught by a person, not the suite:

- **A wrong config does not error, it reads flatter.** Nine clips shipped on the wrong
  TTS model. Every probe passed, because none compares the output to the human recording
  the voice was cloned from. Measured after: the wrong model rested 11.4% of the time vs
  the human's 19.0%. A pipeline that only measures against thresholds can't catch a
  defect that shifts the whole distribution.
- **A confident number from a region with no information.** A backdrop-motion probe
  reported the same "24 px, 0 reversals" on three clips because it sampled a featureless
  black field and returned its own search boundary. Fix: emit INCONCLUSIVE when the
  region carries no signal.

## The failure class none of this catches

A missing metric has no second party, so no relabelling surfaces it. One shipped: the
matte fills the background black, a look was chosen wearing a black top, and the torso
dissolved into the fill — a floating head. Twelve probes passed; eleven scored the
subject and one scored the backdrop, and none held both sides of the boundary at once.
The lesson: **the fill colour and the wardrobe are one decision**, and the suite was
organized as though they were two.

## Case study: cloning a voice by ear

- **The defect was heard first.** Five draws of identical text gave first-formant
  values of 537, 616, 727, 573, 558 Hz — one draw in five sat 27% off its siblings.
- **First instrument, wrong.** A fixed formant profile failed; the redesign compares N
  draws of the *same* text against their own median (threshold 0.15, derived).
- **Retired to a cheap check.** Three voice-quality metrics all inverted the labels, so
  the whole benchmark was killed. What survived: transcribe the audio and diff it against
  the script before spending a render.
- **More data lost twice.** A 69-second stitched reference pitched the voice up and scored
  lower on timbre than a 10-second continuous original, then its clips were rejected by
  ear. Continuity of the source beat quantity of it.

## What this does not claim

The labels are one person's taste — internally consistent, externally unvalidated.
Sample sizes are small and stated as counts, never rates. The claim is not that these
numbers are correct, but that they are **traceable**, and when wrong the pipeline finds
out and records which direction it was wrong in.
