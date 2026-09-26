# Diagrams

System and pipeline diagrams supporting the documentation.

## What to include here

| Suggested filename | What it should show |
|---|---|
| `system-architecture.png` | End-to-end runtime path: storefront → serverless worker → result |
| `dataset-pipeline.png` | Flat-lay synthesis: on-model photo → isolated garment → paired sample |
| `training-pipeline.png` | Dataset → full UNet fine-tune → checkpoint → deployed worker |
| `brand-onboarding.png` | How an existing catalog becomes try-on-ready with no re-shoot |

The last one is worth drawing even though it is the simplest. It is the commercial argument in a single picture — existing photos in, working try-on out, no studio session — and that is the point most reviewers will remember.

## Note

ASCII versions of the architecture and dataset pipelines are embedded in [`../../README.md`](../../README.md), [`../../docs/system-architecture.md`](../../docs/system-architecture.md) and [`../../docs/dataset-construction.md`](../../docs/dataset-construction.md), so the documentation is readable even before image diagrams are added here.
