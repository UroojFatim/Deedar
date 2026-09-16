# Storefront Screenshots — Mahila

Screenshots of the live Next.js storefront that consumes the Deedar inference service.

## What to include here

| Suggested filename | What it should show |
|---|---|
| `01-catalog.png` | Catalog / product listing page |
| `02-product-detail.png` | A single product page with the "Try On" entry point visible |
| `03-upload.png` | The photo upload step |
| `04-tryon-result.png` | The try-on result shown inside the shopping flow |
| `05-mobile.png` | Mobile view — this is where the actual customer is |

The mobile screenshot matters more than it looks. This category is bought on phones, and a reviewer who only sees desktop screenshots has not seen the real product.

## Note

The storefront is publicly reachable and browsable. Interactive try-on is not accepting live requests at present — the GPU worker is not kept warm continuously. See [`../../docs/limitations.md`](../../docs/limitations.md).
