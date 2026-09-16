# Evaluation Protocol

What we measure, how, and what the numbers currently say.

---

## 1. Metrics

Evaluation runs through a reproducible harness, `evaluate_tryon.py`, built on the [`piq`](https://github.com/photosynthesis-team/piq) library.

| Metric | Type | What it captures | Direction |
|---|---|---|---|
| **SSIM** | Pixel / structural | Structural similarity to the reference image | higher is better |
| **LPIPS** | Perceptual | Learned perceptual distance — closer to human similarity judgement | lower is better |
| **DISTS** | Perceptual | Structure + texture similarity; sensitive to texture realism | lower is better |
| **FID** | Distributional | Distance between generated and real image distributions | lower is better |

### Why all four

SSIM alone is misleading for generative try-on. A blurry, over-smoothed output can score well on pixel structure while looking obviously wrong to a person — and in this category, texture *is* the product. Embroidery, gota and block print are exactly what a pixel metric under-weights and what a shopper looks at first.

LPIPS and DISTS carry the perceptual signal; DISTS in particular is sensitive to texture fidelity. FID indicates whether outputs sit in the right distribution overall rather than being individually close to one reference.

---

## 2. Comparison design

The comparison is **zero-shot base vs. our fine-tuned checkpoint**, on the eastern-wear test set, under identical inference settings.

### What is explicitly not a baseline

**No competing architecture was trained as a quantitative baseline, and none is presented as one.** Candidate architectures were compared during base-model *selection* only — a structured human comparison on cross-garment eastern-wear cases, which is how IDM-VTON was chosen. That human-preference result belongs to base-model selection. It is not a fine-tune evaluation result and is not reported as one.

We state this plainly because reading a selection result as a trained baseline would materially overstate the work.

---

## 3. Current measured position

> **The fine-tuned model and the zero-shot base are statistically tied on pixel-level metrics, with only a borderline perceptual improvement.**

That is the finding. We are not framing it as anything else.

| Metric | Zero-shot base | Fine-tuned | Status |
|---|---|---|---|
| SSIM ↑ | — | — | full run in progress |
| LPIPS ↓ | — | — | full run in progress |
| DISTS ↓ | — | — | full run in progress |
| FID ↓ | — | — | full run in progress |

*Table to be completed with final figures on the current evaluation run. The qualitative position above is the established finding and will not be restated more favourably than the numbers support.*

### Open issue on the run

LPIPS and DISTS download a ~500 MB VGG weight file on first use. This is blocked in restricted network environments and requires a GPU node with unrestricted outbound access — the reason the full run is being completed in the live GPU environment rather than locally.

---

## 4. Our diagnosis

Two causes, both addressable, neither flattering:

**1. The training run is thin.** One epoch over ~1.5k pairs is a small amount of signal for a full UNet fine-tune. A full fine-tune has a large parameter surface to move; a single pass over a modest dataset is unlikely to move it decisively.

**2. The synthetic flat-lays are partly the wrong target.** Systematic artifacts in generated garment images — colour drift, smoothed embroidery, simplified folds — are consistent rather than random, so the model can fit *to the artifacts*. In that case the fine-tune is partly learning to reproduce synthesis noise instead of learning eastern-wear garment behaviour, which would suppress exactly the perceptual gain we are looking for.

Both are consistent with what we observe: no pixel-metric separation and only a marginal perceptual shift.

---

## 5. What follows from it

| Change | Rationale |
|---|---|
| Multi-epoch training | Directly addresses insufficient training signal |
| Dataset expansion and rebalancing | More pairs, better coverage across garment types and poses |
| Stricter flat-lay quality gating | Removes the worst synthetic artifacts from the training target |
| Tighter garment masking | Reduces background and adjacent-region leakage into the learned mapping |
| Re-measure after each change individually | So we know which change did what |

The last row matters most methodologically. Changing four things and retraining once tells us nothing about which change helped. Each change is measured on its own.

---

## 6. Planned: stratified reporting

A single aggregate score hides the failure mode that matters most for this product.

Deedar's value proposition is serving shoppers whom catalog photography ignores — a range of body types, heights and complexions wider than any brand's model roster. If output quality degrades unevenly across those segments, the aggregate number can look acceptable while the product fails precisely the users it exists for.

Planned: a stratified test set with **per-segment quality reporting** across body types and skin tones, rather than one pooled figure.

---

## 7. Beyond metrics

Pixel and perceptual metrics do not fully capture try-on quality. Our qualitative protocol complements them:

- **Human preference review** on paired outputs
- **Failure-case cataloguing** — documented and shown, not filtered out (see [`limitations.md`](limitations.md))
- **Attribute checks** — garment length, kameez–trouser coupling, identity preservation, embroidery legibility

---

## 8. Reporting commitment

The honest-reporting position applies to every output from this project, including the proposal, this repository and the live demonstration:

- Marginal results are reported as marginal.
- A selection result is never presented as a trained baseline.
- The fine-tune is described as a full UNet fine-tune, never as a parameter-efficient method.
- Dataset size is reported as 1,502 paired samples, with 1,482 used for training.
- Known weak cases are shown rather than avoided.

We expect to defend these numbers, not conceal them. A working system with an honest marginal result and a concrete plan is a stronger position than an inflated claim that collapses under a single question.
