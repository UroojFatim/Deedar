# Limitations & Known Failure Modes

Stated openly. This document exists because a submission that hides its weak cases is one question away from collapsing, and because these are the things we are actually working on.

---

## 1. Model quality

### 1.1 The fine-tune has not yet separated from the zero-shot base

**Status:** open, actively being worked.

The fine-tuned model and the zero-shot base are statistically tied on pixel metrics, with only a borderline perceptual improvement.

**Diagnosis:** one epoch over ~1.5k pairs is thin for a full UNet fine-tune, and systematic artifacts in the synthetic flat-lays are partly teaching the model the wrong target.

**Plan:** multi-epoch training, dataset expansion and rebalancing, stricter flat-lay quality gating, tighter garment masking — each measured individually. Detail in [`evaluation-protocol.md`](evaluation-protocol.md).

### 1.2 Dupatta, layering and transparency — the weakest case

**Status:** open. Currently the largest quality gap.

A dupatta is semi-transparent, freely draped, and positioned differently in every styling. Standard garment-mask pipelines have no representation for it at all. Results degrade where a dupatta is present, prominent, or layered over the kameez.

**Position:** this is treated as a dedicated engineering workstream, not something expected to fall out of more training. Nothing about more epochs teaches a model a layer representation it does not have.

### 1.3 Synthetic training data carries its own bias

Generated flat-lays are not photographs. Colour drift, smoothed embroidery and simplified fold structure appear systematically, so they can be learned as the target rather than averaged out.

**Mitigations:** quality gating before dataset entry; a real-flat-lay validation subset held aside; human review of failure cases.

### 1.4 Occluded garment regions cannot be recovered

Where the source on-model photograph hides part of the garment behind an arm, a fold, or a dupatta, that information is genuinely absent. The synthesis pipeline can only plausibly complete it.

A confident, plausible, wrong completion is worse than a visible gap — because a shopper will believe it. This is a hard limit of single-photo synthesis, not a tuning issue.

### 1.5 Input sensitivity

Quality degrades on:

- strongly off-frontal or seated poses
- heavy occlusion of the body
- cluttered or low-contrast backgrounds
- extreme or coloured lighting
- low-resolution or heavily compressed uploads

**Mitigation:** input validation and guidance at upload, so a user is told the photo will not work rather than shown a bad result.

### 1.6 No true fit or size reasoning

Deedar answers *"how does this look on me?"* — not *"will this fit me?"* It is a visual appearance model. There is no body measurement, no 3D reasoning, no size recommendation.

Presenting it as sizing guidance would be misleading, so we do not. Size guidance is roadmap, not current capability.

---

## 2. Fairness

### 2.1 Uneven quality across body types and skin tones

**Status:** open; measurement being built.

A model fine-tuned on a dataset of this size can degrade unevenly across body shapes, heights and complexions.

For this product that is not a secondary quality concern — **serving shoppers whom catalog photography ignores is the entire value proposition.** Uneven quality across those segments would undercut the premise of the product, not merely reduce its score.

**Plan:** stratified test set with per-segment quality reporting instead of a single pooled figure. A good aggregate number over a bad distribution is not an acceptable outcome here.

---

## 3. Operational

### 3.1 The live demo is browse-only at present

**Status:** known; will be addressed for the demonstration round.

The Mahila storefront is publicly reachable and its catalog and try-on interface can be browsed, but interactive try-on is not accepting live requests at present — the GPU worker is not kept warm continuously.

The outputs in [`../results/`](../results/) are real outputs from this system. A live interactive window will be available for the demonstration.

### 3.2 Cold-start latency

A serverless worker that has scaled to zero pays a model-load penalty on the next request, and a ~12 GB checkpoint is not a fast load.

This is the price of per-request economics, which we consider the right trade for the target customer. **Mitigations:** warm-pool tuning, result caching, step-count and resolution optimisation.

### 3.3 Cost at scale

Diffusion inference costs GPU-seconds per image. A traffic spike is a cost spike or a request queue — a commercial risk, not only a technical one.

**Mitigation:** a measured cost-per-inference figure established before any pilot scale-up, plus caching and rate limiting.

### 3.4 Brand onboarding friction in practice

Even with synthesised flat-lays, real catalog quality varies. Poor lighting, heavy styling, group shots and unusual crops all reduce synthesis quality, and some brands will need image cleanup.

**Mitigation:** pilot with one or two sellers first to measure genuine onboarding effort before promising a self-serve flow.

---

## 4. Trust, consent and misuse

### 4.1 Shopper photographs

Users upload full-body photographs of themselves. Anything less than explicit consent, clear retention limits and no unsanctioned reuse breaks the product outright — particularly with this user base in this market.

**Position:** documented retention policy, user-initiated deletion, and no training on shopper uploads without explicit opt-in.

### 4.2 Misuse potential

Any garment-transfer model can in principle be used to generate images of people without their consent.

**Mitigations:** generation restricted to catalog garments only; input validation; rate limiting.

---

## 5. Strategic

### 5.1 IP disclosure sequencing

Public disclosure ahead of an IP filing can defeat novelty in most jurisdictions. This is an actively tracked risk and the reason training code, dataset and checkpoint are not published in this repository.

### 5.2 Upstream model licensing

The system builds on third-party open models. Commercial-use terms are verified per component before any revenue-generating deployment.

### 5.3 Platform competition

A large platform could ship generic try-on at any time.

**Our position:** defensibility is deliberately not the base architecture. It is the eastern-wear dataset, the synthesis method that makes local catalogs usable, and the category knowledge behind both. A generic global try-on feature still fails on a kameez, and still has no flat-lay to work from.

---

## 6. Why this document exists

Two reasons.

**Because it is accurate.** Every claim in this project is stated at the level the evidence supports — marginal results as marginal, selection results as selection results, a full UNet fine-tune as a full fine-tune.

**Because it is the stronger position.** A working deployed system with an honest marginal result and a specific plan holds up under questioning. An inflated claim does not survive the first follow-up.
