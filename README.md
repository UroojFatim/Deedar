# Deedar — AI Virtual Try-On for Pakistani Eastern-Wear

**Supporting materials for the 4th International AI Championship (AIEF)**
**Category:** AI in Retail and E-Commerce · **Team size:** 2

> A deployed AI virtual try-on service for Pakistani women's eastern-wear. A shopper uploads one full-body photo, picks an outfit from a store catalog, and the system returns a photorealistic image of that garment worn by her — identity, pose and skin tone preserved, the garment's real cut, drape and embroidery transferred.

---

## What this repository is

This is the **evidence and documentation repository** for the Deedar submission. It contains the project proposal, system documentation, evaluation protocol, and visual results from the working prototype.

**It is not the full source repository.** Training code, the dataset, the fine-tuned checkpoint, and the data-generation pipeline internals are held privately pending an IP review. What is documented here is the architecture, the method at design level, and the measured results — everything needed to evaluate the work, without publishing a reproducible recipe. Where a detail is withheld, this is stated explicitly rather than glossed over.

### Related repositories

| Repository | Component | Access |
|---|---|---|
| `deedar` *(this repo)* | Documentation, results, proposal | Public |
| [`mahila`](https://github.com/USERNAME/mahila) | Next.js storefront — the reference implementation | Public |
| `deedar-api` | Inference backend and service endpoints | Private — pending IP review |
| `deedar-dashboard` | Brand-facing catalog and usage dashboard | Private |
| `deedar-widget` | Embeddable try-on widget for third-party sites | In development |

---

## Repository contents

| Path | What it is |
|---|---|
| [`PROPOSAL.md`](PROPOSAL.md) | Full championship proposal — problem, solution, objectives, users, plan, risks |
| [`docs/system-architecture.md`](docs/system-architecture.md) | How the deployed system fits together, request to result |
| [`docs/dataset-construction.md`](docs/dataset-construction.md) | The flat-lay synthesis method and how the paired dataset was built |
| [`docs/evaluation-protocol.md`](docs/evaluation-protocol.md) | Metrics, harness, test protocol, and current measured position |
| [`docs/limitations.md`](docs/limitations.md) | Known failure modes, stated openly |
| [`results/`](results/) | Try-on output samples |
| [`assets/storefront/`](assets/storefront/) | Live storefront screenshots |
| [`assets/diagrams/`](assets/diagrams/) | System and pipeline diagrams |

---

## The problem, briefly

Eastern-wear is one of Pakistan's largest consumer apparel categories, and one of the worst served by online retail. The purchase decision depends almost entirely on **how the outfit falls on the buyer specifically** — kameez length against her height, trouser break, dupatta drape, how embroidery reads at her body scale, whether the colour works with her complexion. A flat product photo answers none of that. A photo on one studio model answers it for that model only.

The cost lands on the seller: hesitation at the product page, enormous volumes of manual "will this suit me?" messages, and appearance-driven returns — which in a cash-on-delivery market often means a fully refused delivery.

**Existing virtual try-on models do not solve it.** They are trained overwhelmingly on Western fashion datasets built around tight-fitting upper-body garments, and they fail on eastern-wear in reproducible ways: they truncate the long loose kameez, break the kameez–trouser unit, have no representation for the semi-transparent dupatta layer, smooth away dense embroidery, and were fitted to poses, body types and skin tones that are not this customer base.

**And the blocker underneath all of that: the catalog data does not exist.** Every image-based try-on model needs *paired* data — a clean isolated flat-lay of the garment plus a photo of someone wearing it. Pakistani brands almost never have the flat-lay; they shoot on-model editorial photography and nothing else. So even a perfect model is undeployable here, because the input it requires is absent from every catalog it would be pointed at.

That data gap is what Deedar addresses first.

---

## The system

```
Brand's existing on-model photo
            │
            ▼
   ┌────────────────────┐
   │ Flat-lay synthesis │   FLUX.1-Kontext, A100-80GB
   │ (catalog unlock)   │   on-model photo → isolated garment image
   └────────────────────┘
            │
            ▼
   ┌────────────────────┐
   │ Paired dataset     │   1,502 semi-synthetic pairs
   │ (eastern-wear)     │   real person photos + synthesized flat-lays
   └────────────────────┘
            │
            ▼
   ┌────────────────────┐
   │ Full UNet          │   IDM-VTON base, 1,482 training pairs
   │ fine-tune          │   NVIDIA A40 → ~12 GB checkpoint
   └────────────────────┘
            │
            ▼
   ┌────────────────────┐        ┌──────────────────────┐
   │ RunPod serverless  │◄──────►│ Next.js storefront   │
   │ GPU worker         │  API   │ ("Mahila")           │
   └────────────────────┘        └──────────────────────┘
            │
            ▼
      Try-on result to shopper
```

### Three components

**1. Flat-lay synthesis — the part that makes this deployable.**
Instead of asking a brand to re-shoot its catalog, Deedar synthesises the flat-lay from the on-model photograph the brand already has. Brand onboarding cost drops from *"re-shoot 500 products"* to *"point us at the photos you already have."* That is the difference between a feature a brand might consider next season and one it can switch on this week. Details: [`docs/dataset-construction.md`](docs/dataset-construction.md).

**2. A model adapted to this clothing.**
IDM-VTON as base architecture, selected after a structured human comparison of candidate open try-on architectures on cross-garment eastern-wear cases. On top of it, a **full UNet fine-tune** — not LoRA, not any parameter-efficient adapter — on the eastern-wear dataset. Inference uses a single fixed textual prompt identical for every catalog item, so brands add no per-SKU metadata work.

**3. Production deployment.**
The model runs as a **serverless** GPU worker, billed per request, so a seller carries no idle GPU cost. A live Next.js storefront consumes it over a simple request/response API — meaning the same worker can sit behind any other store, a brand's own site, or a marketplace listing page. Details: [`docs/system-architecture.md`](docs/system-architecture.md).

---

## Current status

| Component | State |
|---|---|
| Flat-lay synthesis pipeline | Complete |
| Eastern-wear paired dataset (1,502 pairs) | Complete |
| Fine-tuned checkpoint (~12 GB) | Complete |
| Serverless GPU inference worker | Deployed |
| Mahila storefront | [Live](https://mahila-fyp.vercel.app) — **browse-only at present, see note below** |
| Evaluation harness (`evaluate_tryon.py`) | Built; full metric run in progress |
| Documentation package | In progress |

### ⚠️ Note on the live demo

The Mahila storefront is at **https://mahila-fyp.vercel.app** and is fully browsable — catalog, product pages, and the AI Try-On page. **Interactive try-on is not accepting live requests at the time of writing** — the GPU worker is not kept warm continuously. The site's own try-on page carries the status badge *"Live on Mahila · Still actively in development"*.

The 19 try-on outputs in [`results/`](results/) are real outputs from this system, generated through this pipeline.

A live interactive window will be available for the demonstration round.

---

## Results

Sample try-on outputs are in [`results/`](results/), with storefront screenshots in [`assets/storefront/`](assets/storefront/).

### Quantitative position — stated honestly

We evaluate with **SSIM, LPIPS, DISTS and FID** via the [`piq`](https://github.com/photosynthesis-team/piq) library. Our current measured position:

> The fine-tuned model and the zero-shot base are **statistically tied on pixel-level metrics**, with only a **borderline perceptual improvement**.

We report this rather than bury it. Our diagnosis and the specific changes that follow from it are in [`docs/evaluation-protocol.md`](docs/evaluation-protocol.md) and [`docs/limitations.md`](docs/limitations.md). In short: one epoch over ~1.5k pairs is thin for a full UNet fine-tune, and artifacts in the synthetic flat-lays are partly teaching the model the wrong target. Both are addressable and both are being addressed.

**What is not claimed anywhere in this project:**

- No competing architecture was trained as a quantitative baseline, and none is presented as one. Candidate architectures were compared for *base-model selection* only.
- The human-preference result from that selection process belongs to base-model selection in a cross-garment setting. It is not a fine-tune evaluation result.
- No parameter-efficient method (LoRA or otherwise) was used. The fine-tune is a full UNet fine-tune.

---

## Positioning our contribution precisely

We want to be exact about what is and is not new here, because the adjacent literature matters.

**Garment extraction from worn photographs is an active research area** — work such as TryOffDiff and Garments2Look addresses recovering canonical garment images from photos of people wearing them. We are not claiming to have invented that idea.

**What we claim is the combination, and the domain it unlocks:**

1. Applying flat-lay synthesis specifically as a **catalog-onboarding mechanism** for a market where flat-lay imagery structurally does not exist — making try-on deployable where it previously was not.
2. The resulting **eastern-wear paired dataset (1,502 samples)** — to our knowledge the first of its kind, in a category with no public dataset and no public benchmark.
3. A **working end-to-end deployment** — synthesis pipeline, domain fine-tune, serverless inference and a live storefront — rather than a model checkpoint alone.
4. A **documented evaluation baseline** for eastern-wear try-on, reported without selective framing.

Our defensibility is deliberately not the base architecture. A generic global try-on feature still fails on a kameez, and still has no flat-lay to work from.

---

## Stack

| Layer | Technology |
|---|---|
| Base try-on architecture | IDM-VTON |
| Flat-lay synthesis | FLUX.1-Kontext (NVIDIA A100-SXM4-80GB) |
| Fine-tuning | Full UNet fine-tune, NVIDIA A40, Diffusers 0.25 / Transformers 4.36.2 |
| Evaluation | `piq` — SSIM, LPIPS, DISTS, FID |
| Inference | RunPod serverless GPU worker |
| Storefront | Next.js |

---

## Literature surveyed (2024–2026)

CatVTON · MuGa-VTON · TryOffDiff · MGT · BD-VITON · BootComp · Garments2Look · FLUX.1-Kontext

---

## What is withheld, and why

Training code, dataset files, the fine-tuned checkpoint, synthesis prompts and hyperparameters, and mask-pipeline internals are **not published here**, pending completion of an IP review. Public disclosure ahead of a filing can defeat novelty in most jurisdictions, and this project has a potential commercialisation path.

Everything required to evaluate the work — architecture, method at design level, evaluation protocol, measured results, limitations and visual outputs — is in this repository. Reviewers who need deeper technical access for judging purposes are welcome to request it directly.

---

## Contact

Reviewers and judges who need access to withheld material — training code, dataset samples, or a live inference window — can request it at **CONTACT_EMAIL**.

Live storefront: **https://mahila-fyp.vercel.app**

---

## License

All rights reserved. See [`LICENSE`](LICENSE). This repository is published as submission evidence; it does not grant rights to reuse the method, dataset or models.
