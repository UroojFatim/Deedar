# Dataset Construction

How the eastern-wear paired dataset was built, and why it had to be built at all.

---

## 1. The requirement

Image-based virtual try-on is trained on **paired** data. Each training sample needs two images:

1. A clean, isolated, front-facing image of the garment — a **flat-lay**.
2. A photograph of a person **wearing that same garment**.

The model learns to map (person, flat-lay) → (person wearing garment). Without both halves, there is nothing to learn from.

---

## 2. Why no such dataset exists for this clothing

### The public datasets are the wrong clothing

Public try-on datasets are built around Western fashion — predominantly tight-fitting upper-body garments photographed on a narrow range of body types and poses. Training on them produces models that fail on eastern-wear in specific, reproducible ways:

| Failure | Cause |
|---|---|
| Kameez truncated or bled into background | Trained on cropped upper-body garments, not torso-to-knee lengths |
| Kameez–trouser unit broken | Garment treated as a single upper-body patch |
| Dupatta ignored or corrupted | No representation for a semi-transparent, freely-draped layer |
| Embroidery smoothed or hallucinated | High-frequency textures (gota, mirror work, block print) outside the training distribution |
| Degraded output on this customer base | Source poses, body types and skin tones are not representative |

### And the local catalogs have no flat-lays

This is the harder half of the problem, and it is structural rather than incidental.

Pakistani eastern-wear brands shoot **on-model editorial photography**. That is what sells the category — a model in the outfit, styled, in a location. Almost no brand maintains clean isolated garment flat-lays, because nothing in their existing sales process requires one.

So the garment side of every training pair, and of every inference request, is missing from every catalog the system would be pointed at.

> **This is why virtual try-on has not reached Pakistani retail. Not model quality — missing input data.**

---

## 3. The approach: synthesise the flat-lay

Rather than treating the missing flat-lay as a precondition somebody else must satisfy, we generate it.

```
Real on-model photograph
   (garment worn by a person)
            │
            ▼
   ┌──────────────────────────┐
   │  Flat-lay synthesis      │
   │  FLUX.1-Kontext          │
   │  NVIDIA A100-SXM4-80GB   │
   └──────────────────────────┘
            │
            ▼
   Isolated front-facing garment image
   preserving cut, colour and print
            │
            ├─────────────► paired with the source person photo → training sample
            │
            └─────────────► used as garment input at inference → brand onboarding
```

### Two uses, one pipeline

| Use | What it enables |
|---|---|
| **Dataset construction** | Building paired training data from real on-model photographs |
| **Brand onboarding** | A new seller becomes usable from their existing catalog, with no re-shoot |

The second is the commercially significant one. It collapses onboarding effort from *"re-shoot 500 products in a studio"* to *"give us access to the photos you already have."*

---

## 4. The resulting dataset

| Property | Value |
|---|---|
| Total paired samples | **1,502** |
| Used for fine-tuning | **1,482** pairs |
| Person images | Real photographs |
| Garment images | AI-generated flat-lays (FLUX.1-Kontext) |
| Composition | **Semi-synthetic** — real person photos, synthesised garment side |
| Coverage | Eastern-wear garment types, drapes, prints and poses present in this market |
| Generation hardware | NVIDIA A100-SXM4-80GB |

To our knowledge this is the first paired virtual try-on dataset for Pakistani eastern-wear. The category has no public dataset and no public benchmark.

---

## 5. Honest accounting of the method

A synthesised flat-lay is not a photograph, and pretending otherwise would undermine everything downstream.

### Known costs of this approach

**Systematic artifacts enter training.** Generated flat-lays can carry colour drift, smoothed embroidery, and simplified fold structure. Because these artifacts are systematic rather than random, the model can learn them as the target — which is part of our current diagnosis for why the fine-tune has not yet separated cleanly from the zero-shot base. See [`evaluation-protocol.md`](evaluation-protocol.md).

**Information genuinely lost in the worn photograph stays lost.** A garment region occluded by an arm, a fold, or a dupatta cannot be recovered — it can only be plausibly completed. Plausible is not the same as correct, and a confidently wrong completion is worse than a visible gap.

**Synthetic-to-real gap at inference.** The model is trained on synthesised flat-lays. Where a brand does supply a real flat-lay, that is a distribution the model has seen less of.

### Mitigations in place and planned

| Mitigation | State |
|---|---|
| Quality gating on generated flat-lays before they enter the dataset | In place |
| Real-flat-lay validation subset held aside | Planned |
| Human review of failure cases | Ongoing |
| Stricter filtering thresholds as part of the next training iteration | Planned |
| Dataset expansion and rebalancing | Planned |

---

## 6. Prior work, and what we actually claim

Recovering a canonical garment image from a photograph of someone wearing it is an active research direction — **TryOffDiff** and **Garments2Look** both address it. We are not claiming to have originated that idea, and we surveyed both as part of this work.

**What we claim:**

1. Using flat-lay synthesis as a **catalog-onboarding mechanism** for a market where flat-lay imagery structurally does not exist — turning try-on from undeployable into deployable here.
2. The **eastern-wear paired dataset** that resulted, in a category with no public dataset.
3. Integration of synthesis, domain fine-tune and serverless inference into **one working service**, rather than a component in isolation.

We would rather state this precisely and defend it than overclaim and be corrected in a defence round.

---

## 7. Ethics and rights

| Concern | Position |
|---|---|
| Person photographs in training data | Used with rights secured; no scraped identifiable individuals redistributed |
| Brand catalog imagery | Written usage rights secured per brand before onboarding |
| Shopper-uploaded photos | Not used for training without explicit opt-in; documented retention and deletion policy |
| Redistribution | The dataset is not published, and is not offered for redistribution |

---

## 8. What is withheld

Synthesis prompts, generation hyperparameters, quality-gating thresholds, data sourcing specifics, and the dataset files themselves are held privately pending IP review. This document states what the method is and what it costs; it is not a reproduction recipe.
