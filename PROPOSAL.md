# Deedar — AI Virtual Try-On for Pakistani Eastern-Wear

**Submission:** 4th International AI Championship (AIEF)
**Category:** AI in Retail and E-Commerce
**Status:** Working prototype — deployed and running live on serverless GPU infrastructure
**Team:** 2 members

---

## 1. Project Overview & Problem Statement

### Overview

Deedar is a deployed AI virtual try-on service built for one of Pakistan's largest consumer retail categories: women's eastern-wear. A shopper uploads a single full-body photo, selects an outfit from a store's catalog, and Deedar returns a photorealistic image of that exact garment worn by her — her face, body shape, pose and skin tone preserved, the garment's real cut, drape, colour and embroidery transferred onto her.

It is not a research demo. The system runs end-to-end today: a live Next.js storefront calls a serverless GPU inference worker, which runs our domain fine-tuned diffusion model and returns the result to the shopper inside the shopping flow.

### The industry problem

**Eastern-wear is a huge online category that online shopping serves badly.**

Unstitched and stitched eastern-wear is the default clothing purchase for the majority of Pakistani women, and it is one of the most heavily marketed categories in Pakistani e-commerce — brand lawn drops, seasonal collections, and a long tail of Instagram and WhatsApp sellers. But the purchase decision in this category is driven almost entirely by *how the outfit falls on the buyer specifically*: kameez length against her height, trouser break, how the dupatta sits, how embroidery reads at her body scale, whether the colour works with her complexion.

A flat product photo answers none of that. A photo on one studio model answers it for that model only.

The business consequences are concrete and sit with the seller:

| Cost centre | What actually happens |
|---|---|
| **Lost conversion** | Shoppers hesitate at the product page and abandon. The uncertainty is at the exact moment of purchase intent. |
| **Manual sales labour** | Brand and boutique pages absorb enormous volumes of DMs — "will this suit my height?", "how does it look on someone fair/dark?", "is the kameez long?" — answered one-by-one by humans. |
| **Returns and exchanges** | Appearance-driven returns are the expensive kind: reverse logistics, restocking, and in a market that runs heavily on cash-on-delivery, often a fully refused delivery. |
| **Catalog spend** | Brands shoot more and more model photography trying to cover body-type and complexion variation, which multiplies production cost without ever covering the actual customer. |

**And the existing AI solutions do not work on this clothing.**

Virtual try-on models in the public ecosystem are trained overwhelmingly on Western fashion datasets built around tight-fitting upper-body garments. Applied to eastern-wear they fail in specific, reproducible ways:

- **Silhouette failure** — a kameez is a long, loose, torso-to-knee garment; models trained on cropped tops truncate it or bleed it into the background.
- **Two-piece failure** — kameez and trousers are one visual unit; treating the garment as a single upper-body patch destroys the outfit.
- **Layering failure** — the dupatta is a semi-transparent, freely-draped layer that standard garment-mask pipelines do not represent at all.
- **Detail failure** — dense embroidery, mirror work, gota and block prints are high-frequency textures that get smoothed away or hallucinated.
- **Demographic failure** — the poses, body types and skin tones in the source datasets are not the customer base.

**The blocker nobody addresses: the catalog data does not exist.**

Every image-based try-on model needs *paired* data — a clean isolated flat-lay of the garment, plus a photo of a person wearing it. Pakistani brands almost never have the flat-lay. They shoot on-model editorial photography and nothing else.

So even a perfect model is undeployable in this market, because the input the model requires is missing from every catalog it would be pointed at. **This data gap, not the model gap, is why virtual try-on has not reached Pakistani retail.** It is the problem Deedar solves first.

---

## 2. Proposed Solution

Deedar is a three-part system: a data pipeline that makes local catalogs usable, a model fine-tuned for this clothing, and a production service that a brand can actually switch on.

### 2.1 Flat-lay synthesis — the pipeline that unlocks real catalogs

Rather than asking a brand to re-shoot its entire catalog as flat-lays, Deedar **synthesises the flat-lay from the on-model photograph the brand already has**, using FLUX.1-Kontext on A100-80GB hardware. The pipeline takes an existing editorial shot and produces the isolated, front-facing garment image the try-on model needs, preserving cut, colour and print.

The commercial consequence is the whole point:

> **Brand onboarding cost drops from "re-shoot 500 products" to "give us access to the photos you already have."**

That is the difference between a feature a brand might consider next season and a feature it can enable this week. It is also the part of Deedar that generalises beyond this category — any apparel market without flat-lay catalog imagery has the same blocker.

### 2.2 A model adapted to this clothing

- **Base architecture: IDM-VTON**, selected after a structured comparison of candidate open try-on architectures on cross-garment eastern-wear cases, where human evaluators consistently preferred its handling of loose, full-length garments. *(Competing architectures were assessed for selection purposes only; none were trained as quantitative baselines and none are claimed as such.)*
- **Full UNet fine-tune** on our own eastern-wear dataset — not LoRA, not any parameter-efficient adapter — producing a ~12 GB domain-specialised checkpoint (1 epoch, 1,482 training pairs, NVIDIA A40, Diffusers 0.25 / Transformers 4.36.2).
- **Caption-free inference.** A fixed textual conditioning prompt is used rather than per-item captions. Operationally this matters a great deal: a brand does not have to write a description for every SKU to get try-on working on it.

### 2.3 A dataset built for this market

Using the synthesis pipeline we constructed a **semi-synthetic paired dataset of 1,502 samples** — real person photographs paired with AI-generated flat-lays — covering the garment types, drapes, prints and poses actually present in Pakistani eastern-wear. To our knowledge no comparable paired dataset for this category exists publicly.

### 2.4 Production deployment

- **Inference:** RunPod **serverless** GPU worker. GPU capacity is billed per request, so a brand carries no idle GPU cost — the unit economics scale with traffic instead of ahead of it.
- **Storefront:** live Next.js store ("Mahila") with a real catalog, upload flow and try-on result view.
- **Integration contract:** the storefront talks to the worker over a simple request/response API, so the same worker can sit behind any other store, a brand's existing site, or a marketplace listing page.

### 2.5 Measured, not asserted

We built a reproducible evaluation harness (`evaluate_tryon.py`) on the `piq` library covering **SSIM, LPIPS, DISTS and FID** — and we report what it says rather than what we would prefer. Current position: our fine-tuned model and the zero-shot base are **statistically tied on pixel metrics, with only a borderline perceptual improvement.** Section 8 states our diagnosis and the specific changes that follow from it. We would rather defend a working system with an honest number than an inflated one.

---

## 3. Objectives & Expected Outcomes

### Objectives

| # | Objective | Success criterion |
|---|---|---|
| **O1** | Make virtual try-on work on full-length, loose, two-piece eastern-wear | Garment length, silhouette and kameez–trouser coupling preserved; shopper's identity and pose unchanged |
| **O2** | Remove the flat-lay data dependency that blocks market adoption | Any brand onboardable from existing on-model photography alone, with no re-shoot |
| **O3** | Operate as a commercial service, not a demo | Live storefront → serverless GPU → result, with measured latency and a known cost per inference |
| **O4** | Quantify quality transparently | Full SSIM / LPIPS / DISTS / FID suite run and reported without selective framing |
| **O5** | Demonstrate business impact with a real seller | Pilot with a live brand, instrumented against pre-purchase query volume and conversion |

### Expected outcomes

**For shoppers** — a first-person answer to "will this suit me?" in seconds, before payment rather than after delivery. Particularly meaningful for buyers outside the narrow body-type and complexion range that catalog photography represents.

**For sellers** — fewer pre-purchase queries to answer manually, higher confidence at checkout, and fewer appearance-driven returns. Reduced pressure to fund ever more model photography to cover variation.

**For the market** — a reusable method for bringing try-on to any apparel category that lacks flat-lay catalog data, and the first documented evaluation baseline for eastern-wear try-on.

**As a product** — two separable commercial assets: the **Deedar inference service** (sold to brands and marketplaces as an API) and the **Mahila storefront** (a direct-to-consumer channel that proves the feature in production).

---

## 4. Target Users

### Primary — small and mid-size eastern-wear sellers

Brands and boutiques selling through Instagram, WhatsApp and their own storefronts. This is where the pain is sharpest and where no solution exists: they have catalog photography, real demand, and no ML capability, no GPU budget and no data team. They carry the uncertainty cost directly, as manual DM handling and exchange logistics. Deedar reaches them as an API plus a drop-in storefront component — no model training, no infrastructure, no per-SKU captioning.

### Primary — the shopper

Pakistani women and the wider South Asian diaspora buying eastern-wear online, overwhelmingly on mobile, who currently cannot tell from a product page whether an outfit will work for their height, body shape or complexion.

### Secondary

- **Marketplaces and aggregators** — try-on as a category-level feature across many sellers, where the flat-lay problem is worst because catalog quality is uncontrolled.
- **Resellers and drop-shippers** — selling stock they never photographed on a model at all.
- **Larger brands** — as a way to cut catalog production spend while increasing apparent coverage of body types and complexions.
- **Stylists and tailors** — pre-stitch visualisation for unstitched fabric purchases.

---

## 5. Key Features / Deliverables

### Product features

1. **Single-photo try-on** — one full-body photo. No measurements, no calibration, no depth capture, no app install.
2. **Full-outfit rendering** — kameez and trousers treated as one coupled garment, not an upper-body patch.
3. **Identity preservation** — face, hair, body shape, skin tone and pose retained from the shopper's own photo.
4. **Detail fidelity** — embroidery, prints and colour carried through from the garment image.
5. **In-journey experience** — browse catalog, tap try-on, see the result without leaving the shopping flow.
6. **Zero per-SKU setup** — caption-free inference, so brands add no metadata work.
7. **Existing-catalog onboarding** — flat-lay synthesis from the brand's current photography.
8. **Pay-per-use economics** — serverless GPU, no idle cost for the seller.

### Deliverables

| Deliverable | Description | State |
|---|---|---|
| Fine-tuned try-on checkpoint | Full UNet fine-tune of IDM-VTON, ~12 GB, eastern-wear specialised | **Complete** |
| Eastern-wear paired dataset | 1,502 semi-synthetic paired samples | **Complete** |
| Flat-lay synthesis pipeline | FLUX.1-Kontext based on-model → flat-lay generation | **Complete** |
| Serverless inference worker | RunPod GPU worker + request/response API | **Deployed** |
| Mahila storefront | Live Next.js store with catalog and try-on flow | **Live** |
| Evaluation harness | `evaluate_tryon.py` — SSIM, LPIPS, DISTS, FID via `piq` | Built; full run in progress |
| Technical documentation | Method, dataset construction, evaluation protocol, limitations | In progress |
| Championship demo | End-to-end live walkthrough with failure cases shown honestly | For 24 October finale |

---

## 6. Implementation Plan & Timeline

### Already delivered

**Phase 1 — Domain study & architecture selection.** Surveyed current try-on and garment-transfer work (2024–2026): CatVTON, MuGa-VTON, TryOffDiff, MGT, BD-VITON, BootComp, Garments2Look, FLUX.1-Kontext. Ran a structured human comparison of candidate architectures on cross-garment eastern-wear cases; selected IDM-VTON.

**Phase 2 — Data pipeline & dataset.** Built and tuned the FLUX.1-Kontext flat-lay synthesis pipeline on A100-SXM4-80GB; assembled and cleaned the 1,502-sample paired dataset.

**Phase 3 — Fine-tuning.** Full UNet fine-tune on 1,482 pairs, NVIDIA A40 → ~12 GB checkpoint.

**Phase 4 — Deployment.** Packaged the model as a RunPod serverless worker; built the Mahila Next.js storefront; wired the complete shopper-facing try-on flow.

### To submission — by 12 October 2026

| Work | Detail |
|---|---|
| **Full quantitative evaluation** | Run the complete `piq` suite (SSIM / LPIPS / DISTS / FID) on the eastern-wear test set in the live GPU environment; resolve the ~500 MB VGG weight dependency that LPIPS and DISTS pull on first run |
| **Documentation package** | Method, dataset construction, evaluation protocol and limitations, written to submission standard |
| **Prototype hardening** | Input validation, error and edge-case handling, mobile UX pass, latency measurement, cost-per-inference measurement |
| **Failure-case catalog** | Documented weak cases — dupatta/layering, off-standard poses, difficult backgrounds — prepared to be shown rather than hidden |

### To finale — by 24 October 2026

| Work | Detail |
|---|---|
| **Training iteration** | Act on the tied-metrics finding: multi-epoch training, rebalanced dataset, tighter garment masking, stricter synthetic-flat-lay quality filtering; re-measure after each change |
| **Demo build** | Stable live walkthrough: shopper journey, brand-onboarding-from-existing-photos demonstration, before/after metrics view |
| **Defence preparation** | Full rationale for architecture choice, dataset method, and the honest evaluation position |

### Post-championship roadmap

| Stage | Work | Duration |
|---|---|---|
| Quality workstream | Dedicated dupatta, layering and transparency handling | 2–3 weeks |
| Brand pilot | Onboard 1–2 real sellers from existing catalog photography; instrument query volume, conversion and return rate | 3–4 weeks |
| Commercial hardening | Result caching, cold-start reduction, usage metering and billing, self-serve onboarding portal | 4–6 weeks |
| Category expansion | Menswear (kurta/shalwar), bridal wear, multi-garment full-look try-on | Ongoing |

---

## 7. Resources or Requirements

### Compute

| Purpose | Hardware |
|---|---|
| Model fine-tuning | NVIDIA A40 (VRAM-bound; ~12 GB checkpoint output) |
| Flat-lay data generation | NVIDIA A100-SXM4-80GB |
| Production inference | RunPod serverless GPU workers (per-request billing) |
| Evaluation | GPU node with unrestricted network access — LPIPS and DISTS download a ~500 MB VGG weight file on first run |

### Software stack

PyTorch · Diffusers 0.25 · Transformers 4.36.2 · IDM-VTON (base architecture) · FLUX.1-Kontext (flat-lay synthesis) · `piq` (SSIM, LPIPS, DISTS, FID) · Next.js (storefront) · RunPod serverless runtime

### Data & storage

- 1,502-sample paired dataset plus intermediate synthesis artifacts
- Multi-GB checkpoint and model-weight storage per training run
- Brand catalog imagery for pilot onboarding, with written usage rights secured per brand

### Team

Two members, covering (a) model training, data pipeline and evaluation, and (b) full-stack development and deployment. Capacity is the binding constraint on how many workstreams run in parallel, which is why the plan above sequences evaluation before the training iteration rather than running both at once.

### Non-technical

- IP review completed ahead of any public technical disclosure
- Written image-rights agreements for brand catalogs and for any person photographs used in training or demonstration
- Documented data-retention and deletion policy for shopper-uploaded photos

---

## 8. Potential Challenges or Risks

### Technical

**1. The fine-tune gain is currently marginal — and we state it plainly.**
Our fine-tuned model and the zero-shot base are statistically tied on pixel metrics, with only borderline perceptual improvement. Our diagnosis: one epoch over ~1.5k pairs is thin for a full UNet fine-tune, and artifacts in the synthetic flat-lays are partly teaching the model the wrong target.
*Mitigation:* multi-epoch training, dataset expansion and rebalancing, stricter flat-lay quality gating, and re-measurement after each individual change rather than one blind retrain.

**2. Synthetic training data carries its own bias.**
Generated flat-lays are not photographs. Systematic artifacts — colour drift, smoothed embroidery, simplified folds — get baked into training.
*Mitigation:* quality gating on generated flat-lays, a real-flat-lay validation subset held aside, and human review of failure cases.

**3. Dupatta, layering and transparency.**
Semi-transparent, freely-draped layers are the hardest remaining case and currently the weakest part of output quality.
*Mitigation:* treated as a dedicated workstream rather than something expected to fall out of more training. Documented openly as a current limitation.

**4. Inference cost and latency at scale.**
Diffusion inference costs GPU-seconds per image, and serverless workers pay a cold-start penalty. A viral moment could produce a cost spike or a request queue — which, for a paid service, is a commercial risk, not just a technical one.
*Mitigation:* result caching, warm-pool tuning, step-count and resolution optimisation, and a measured cost-per-inference figure established before any pilot scale-up.

**5. Fairness across body types and skin tones.**
A model fine-tuned on a limited dataset can degrade unevenly across body shapes and complexions. In this product that is not a secondary quality concern — serving shoppers whom catalog photography ignores *is the value proposition*, so uneven quality would undercut the entire premise.
*Mitigation:* stratified test set with per-segment quality reporting rather than a single aggregate score.

### Product & operational

**6. Trust and consent around shopper photos.**
Users upload full-body photographs of themselves. Anything less than explicit consent, clear retention limits and no unsanctioned reuse breaks the product outright — especially with this user base in this market.
*Mitigation:* documented retention policy, user-initiated deletion, and no training on shopper uploads without opt-in.

**7. Misuse potential.**
Any garment-transfer model can in principle be used to generate images of people without their consent.
*Mitigation:* restriction to catalog garments only, input validation, and rate limiting.

**8. Brand onboarding friction in practice.**
Even with synthesised flat-lays, real catalog quality varies and some brands will need image cleanup.
*Mitigation:* pilot with 1–2 sellers first to measure genuine onboarding effort before promising a self-serve flow.

### Strategic

**9. Disclosure vs. IP sequencing.**
Public disclosure ahead of an IP filing can defeat novelty in most jurisdictions. This is an actively tracked risk: IP review is completed before any public technical disclosure.

**10. Upstream model licensing.**
The system builds on third-party open models. Commercial-use terms are verified per component before any revenue-generating deployment.

**11. Platform competition.**
A large platform could ship generic try-on at any time. Our defensibility is deliberately *not* the base architecture — it is the eastern-wear dataset, the flat-lay synthesis method that makes local catalogs usable, and the category knowledge behind both. A generic global try-on feature still fails on a kameez and still has no flat-lay to work from.

---

## 9. Additional Details

### Why this belongs in AI in Retail and E-Commerce

It is an AI system built for a retail transaction, deployed in a retail context, aimed at retail metrics — conversion, pre-purchase query volume, return rate, catalog production cost. The category it targets is one of the largest consumer apparel categories in Pakistan and one of the worst served by current online retail tooling.

### What is genuinely new

Most try-on work improves the model. **Our contribution sits upstream of the model: we make the data exist.** The flat-lay synthesis pipeline converts the photography this market already has into the paired format try-on requires. Without it, model quality is irrelevant here, because the catalog side of the input is simply absent from every Pakistani brand's asset library. That method transfers to any underserved apparel category, anywhere with the same gap.

### Execution position

The system is built, deployed and running. The catalog exists, the storefront is live, the worker serves requests. What remains between now and the finale is measurement, one training iteration, and demo hardening — not core construction.

### Reporting stance

We report measured results as measured. The fine-tune is currently tied with the zero-shot base on pixel metrics and only borderline better perceptually; that statement appears in every output about this project, including this proposal and including the live demo. We expect to defend it, not conceal it.

### Reference work surveyed

CatVTON · MuGa-VTON · TryOffDiff · MGT · BD-VITON · BootComp · Garments2Look · FLUX.1-Kontext (2024–2026)

---

*Product name pending final confirmation; "Deedar" used throughout.*
