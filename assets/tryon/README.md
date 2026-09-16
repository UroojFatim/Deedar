# Try-On Output Samples

Raw try-on outputs from the system. Put the image files here; the reading guide and what-to-look-for notes live in [`../../results/README.md`](../../results/README.md).

## Suggested naming

Use a consistent scheme so a reviewer can pair inputs with outputs without guessing:

```
sample-01-person.png      ← shopper input photo
sample-01-garment.png     ← catalog garment (synthesised flat-lay)
sample-01-output.png      ← generated try-on result

sample-02-person.png
sample-02-garment.png
sample-02-output.png
...
```

If a side-by-side composite already exists, `sample-01-comparison.png` works on its own and is easier for a reviewer to read at a glance.

## Include at least one weak case

Name it clearly — `sample-XX-weakcase-dupatta.png` — and reference it in [`../../docs/limitations.md`](../../docs/limitations.md).

Showing a known failure alongside good results reads as control over the system. Showing only successes invites the reviewer to go looking for the failure themselves.

## Privacy

Only include person photographs you have the right to publish. This repository is public.
