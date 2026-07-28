# Build your own: voice, character, pins

This pipeline runs on two vendor accounts and a pair of pinned identities: one cloned
voice, one generated character. Below is the order that works. Tier recommendations are
in [COST.md](COST.md).

**Consent first.** Clone only your own voice, or one whose owner gave written consent.
The character should be generated, not a likeness of a real person.

## 1. Clone the voice (ElevenLabs)

1. Buy the smallest plan with instant voice cloning (the free tier lacks it).
2. Record 1–3 minutes of clean speech: one quiet room, one mic, natural pace.
   Consistency of the sample beats quantity.
3. Create the instant clone and note its `voice_id` — treat it as a secret.
4. Synthesize with `eleven_v3`, scripts above ~250 characters.
5. **Pin the model and settings** (stability 0.6, similarity_boost 0.92, style 0.3), and
   treat a change as a code change. A different model doesn't error — it returns a flatter
   reading of the correct words, and every probe still passes. Nine clips shipped that way
   before a person said it sounded flat. See [EVALS.md](EVALS.md).
6. **Draw three takes, keep the median by duration, then transcribe and diff against the
   script.** Synthesis is non-deterministic (6–37% length spread), and TTS costs no render
   credits, so a single blind draw is strictly worse at the same cost.

## 2. Create the character (HeyGen)

Two paths, same result:

- **Text prompt** — HeyGen's prompt avatar turns a description into a character; looks in
  the same avatar group stay consistent. The default path: one vendor, one identity chain.
- **Reference image** — generate one clean still with a frontier image model and create a
  photo avatar from it. Use when you need art direction a text prompt won't hold.

Note the **avatar group id** (the character) and the **look id** (one appearance). The
render engine exposes no seed, but animating a fixed look is deterministic, so **the look
id is the seed.**

**Render on the flat tier** (`avatar_iii`): 1 credit per render, flat with length. The
premium tiers scale up to 58 credits for output the probes here were built to judge. The
narration video *is* the avatar render — you need no separate text-to-video model.

## 3. Pin both identities

Write the four ids into a pins file and point the guards at it:

```
IDENTITY_PINS=/path/to/pins.json    # read by guards/block_unpinned_identity.sh
VOICE_TAKE=/path/to/your-tts-step   # the sanctioned voice entrypoint
REFRESH_PINS=/path/to/pin-refresh   # how a NEW look gets allowlisted
PROP_GATE / PIPELINE_PROBES         # see guards/pre_render_sanity.sh, guards/ship_gate.sh
```

The pins file stays out of git. The guard then refuses any render whose voice or avatar
id isn't the pinned one, on every egress path including raw HTTP. Prompt-level rules held
6 of 15 first attempts here; the hook held every time. See [ENFORCEMENT.md](ENFORCEMENT.md).

## 4. Order of operations per render

```
script (>250 chars)  ->  3 voice takes, median pick   (characters, cheap)
                     ->  look check, prop gate          (free, blocks bad spend)
                     ->  ONE render on the flat tier     (1 credit)
                     ->  probes + ship gate              (free)
```

Synthesis before render, always: the audio drives the video, and every gate that can
fail does so before the credit is spent.
