# Cost

Two vendors meter this pipeline: HeyGen bills **credits per render**, ElevenLabs bills
**characters per synthesis**. Everything below is in those native units. There are no
currency figures here (see [NOT-MEASURED.md](NOT-MEASURED.md)): no credit-to-currency
rate was recorded at measurement time. Check the vendors' pricing pages against the
consumption model below.

## The render schedule, as measured

| engine tier | ~11s | ~126s | ~169s | shape |
|-------------|------|-------|-------|-------|
| `avatar_iii` (flat) | **1** | **1** | **1** | flat with length |
| `avatar_iv` | 5 | 43 | **58** | scales, not knowably linear |
| `avatar_v` | 5 | 43 | **58** | same as iv at every point |

Each cell is a balance delta read before and after a real render — none interpolated.
The 169s column was measured twice on independent pairs, 58 credits each time.

**Default every scheduled render to the flat tier.** A 2-minute daily clip on
`avatar_iii` costs the same as a 9-second one. The premium tiers cost up to 43× more
for output the eval harness in this repo was built to judge.

## Why the table has holes instead of a formula

The router once emitted a flat "5 credits" estimate for the premium tiers. A batch of 8
premium renders then billed **344 credits** — an 8.6× understatement. Fitting a line
through the two points was rejected on arithmetic (the implied slope can't reproduce the
43-credit observation). So the router returns **NULL for any duration it hasn't billed**.
A NULL makes the caller ask; a confident 5 makes it spend 43.

New measurements are cheap: read the meter, render one clip, read again, record the
delta only. The absolute reading is account state and stays out of the repo.

## Voice synthesis

- Metered **per character** of script.
- The pipeline draws **3 takes per script** and keeps the median, so character spend is
  `3 × script length`.
- Scripts are kept above ~250 characters — shorter ones measured ~110 Hz brighter and
  less consistent.
- Audio is uploaded to the render vendor, so **TTS draws cost zero render credits**.
  Draw voice takes liberally, render once.

## The daily model

For 2 governed renders/day on the flat tier, one ~2-minute narration each:

```
renders:     2/day × 1 credit             =  2 credits/day   ->  ~60 credits/month
characters:  2 scripts × ~1,900 × 3       = ~11,400/day      -> ~350k chars/month
```

## Which plan to buy

- **HeyGen** — any plan whose monthly credits cover `renders/day × 30`, with API access.
- **ElevenLabs** — instant cloning needs a paid plan; size by `scripts/day × chars × 3 × 30`.

Verify both against current vendor pricing before buying. The consumption model is what
this repo can promise; the quota tables are theirs.
