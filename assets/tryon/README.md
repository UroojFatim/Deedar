# Try-On Output Samples

19 real outputs from the deployed Deedar system — `1.jpeg` through `19.jpeg`.

**The reading guide, per-sample notes and weak-case list are in [`../../results/README.md`](../../results/README.md).** Start there; this file only covers the image format.

## Format

Each file is a single three-panel strip:

```
┌──────────────┬──────────────┬──────────────┐
│    LEFT      │    MIDDLE    │    RIGHT     │
│ shopper's    │ catalog      │ generated    │
│ input photo  │ garment      │ try-on       │
│ (wearing a   │ (synthesised │ output       │
│  different   │  flat-lay)   │              │
│  outfit)     │              │              │
└──────────────┴──────────────┴──────────────┘
```

The left panel is the point: the person arrives wearing something else entirely. The system removes that outfit and renders the target garment on the same body, same pose, same face. So the test is simply — *does the right panel show the middle garment on the left person?*

One composite per sample rather than three separate files, so a reviewer reads each result at a glance instead of pairing filenames.

## Why the filenames are bare numbers

`1.jpeg`–`19.jpeg` carry no meaning on purpose. [`../../results/README.md`](../../results/README.md) maps every number to its garment, its quality note, and whether it is a featured or weak case. Keeping the mapping in one document means it can be corrected or extended without renaming committed files.

## Coverage and gaps

Represented: block prints, all-over florals, dense embroidery, ornamental borders, patchwork scene prints, plain fabrics, and two-piece sets where kameez and trousers differ (see `13.jpeg`).

Not represented — and documented as known limitations in [`../../docs/limitations.md`](../../docs/limitations.md):

- **No dupatta in any sample.** Every strip is a kameez-and-trouser two-piece. Semi-transparent draped layers are the weakest case and are a dedicated workstream.
- No off-frontal or seated poses, cluttered backgrounds, extreme lighting, or low-resolution uploads.

## Privacy

Only person photographs the project has the right to publish are included. This repository is public.
