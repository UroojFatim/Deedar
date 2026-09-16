# System Architecture

How Deedar works end to end, from a shopper tapping "try on" to the returned image.

---

## 1. Runtime path

```
┌──────────────────────────────────────────────────────────────┐
│  Shopper (mobile browser)                                    │
│  1. browses catalog                                          │
│  2. uploads one full-body photo                              │
│  3. taps "Try On" on a product                               │
└───────────────────────────┬──────────────────────────────────┘
                            │ HTTPS
                            ▼
┌──────────────────────────────────────────────────────────────┐
│  Mahila storefront — Next.js                                 │
│  · product catalog + garment images                          │
│  · upload handling and input validation                      │
│  · try-on request construction                               │
│  · result display in the shopping flow                       │
└───────────────────────────┬──────────────────────────────────┘
                            │ request / response API
                            ▼
┌──────────────────────────────────────────────────────────────┐
│  RunPod serverless GPU worker                                │
│  · loads fine-tuned checkpoint (~12 GB)                      │
│  · person-image preprocessing                                │
│  · garment conditioning from catalog flat-lay                │
│  · diffusion inference with fixed textual prompt             │
│  · returns generated try-on image                            │
└──────────────────────────────────────────────────────────────┘
```

### Why serverless

GPU inference for diffusion models costs GPU-seconds per image. An always-on GPU means a fixed monthly bill regardless of traffic — which is exactly the cost structure a small eastern-wear seller cannot absorb.

A serverless worker bills per request, so the cost curve follows actual usage instead of running ahead of it. This is a commercial design decision as much as a technical one: it is what makes the service sellable to a boutique rather than only to a large brand.

**The trade-off we accept:** cold starts. A worker that has scaled to zero pays a model-load penalty on the next request. Mitigations under development are listed in [`limitations.md`](limitations.md).

### Integration contract

The storefront communicates with the worker over a plain request/response API — person image, garment image, and inference parameters in; generated image out. There is no coupling between the worker and the Mahila front end.

The practical consequence: the same worker can serve any other store, a brand's existing website, or a marketplace listing page. Mahila is the reference implementation and proving ground, not a dependency.

---

## 2. Model layer

| Property | Value |
|---|---|
| Base architecture | IDM-VTON |
| Adaptation | **Full UNet fine-tune** — not LoRA, not any parameter-efficient adapter |
| Training data | 1,482 pairs (from the 1,502-sample dataset) |
| Epochs | 1 |
| Hardware | NVIDIA A40 |
| Framework | Diffusers 0.25 / Transformers 4.36.2 |
| Checkpoint size | ~12 GB |
| Text conditioning | A single fixed prompt describing the garment class, identical for every catalog item |

### Base architecture selection

IDM-VTON was chosen after a structured comparison of candidate open try-on architectures on cross-garment eastern-wear cases, where human evaluators consistently preferred its handling of long, loose, full-length garments over alternatives built around fitted upper-body clothing.

**Scope of that result:** it belongs to base-model selection in a cross-garment setting. It is not a fine-tune evaluation result and is not presented as one. No competing architecture was trained as a quantitative baseline, and none is claimed as one anywhere in this project.

### Why caption-free inference matters

A single fixed textual prompt is used for every garment, rather than per-item captions.

This is an operational choice with direct commercial weight. A model requiring per-SKU text descriptions imposes ongoing manual work on every brand that adopts it — someone has to write and maintain a caption for every product, forever. A fixed prompt means a brand can point the service at a 500-item catalog and get try-on on all of it with zero metadata authoring.

---

## 3. Catalog layer

This is the part most try-on work assumes away.

A brand arrives with on-model editorial photography and no flat-lays. Before any try-on can happen, the garment side of the input has to be constructed:

```
Brand's on-model catalog photo
            │
            ▼
   Flat-lay synthesis pipeline
   (FLUX.1-Kontext, A100-80GB)
            │
            ▼
   Isolated front-facing garment image
            │
            ▼
   Usable as try-on garment input
```

The same pipeline serves two purposes: it built the training dataset, and it onboards new brands at inference time. Method detail in [`dataset-construction.md`](dataset-construction.md).

---

## 4. Evaluation layer

A reproducible harness (`evaluate_tryon.py`) built on the `piq` library computes SSIM, LPIPS, DISTS and FID over the eastern-wear test set. Protocol and current measured position: [`evaluation-protocol.md`](evaluation-protocol.md).

---

## 5. Design decisions, and what they cost

| Decision | Why | What it costs |
|---|---|---|
| Full UNet fine-tune over LoRA | Maximum adaptation capacity for a domain far from the base training distribution | ~12 GB checkpoint; heavier training and storage; slower iteration |
| Serverless inference | Per-request economics; no idle GPU cost for the seller | Cold-start latency |
| Synthetic flat-lays | Makes real catalogs usable at all — the alternative is no product | Synthetic artifacts enter training data |
| Fixed prompt over per-item captions | Zero per-SKU work for brands | Loses per-garment textual steering |
| Single-photo input | No measurements, no calibration, no app install | No true 3D fit reasoning — visual appearance only |

Each row is a trade we made deliberately and can defend. None of them is free, and we do not present them as free.

---

## 6. What is not published here

Worker source, training code, preprocessing and mask-pipeline internals, and synthesis prompts and hyperparameters are held privately pending IP review. This document describes the architecture at design level — enough to evaluate the engineering, not enough to reproduce the pipeline.
