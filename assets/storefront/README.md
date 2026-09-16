# Storefront Screenshots — Mahila

The live Next.js storefront that consumes the Deedar inference service.

**Live:** https://mahila-fyp.vercel.app

## Desktop

| File | Page |
|---|---|
| `01-home-hero.jpg` | Homepage hero — "Quiet elegance, worn daily", with the AI Try-On entry point |
| `02-home-collections.jpg` | Shop by Collection — Kalaam, Luxury, Semi Formal, Virasat |
| `03-shop-all-products.jpg` | Womenswear catalog listing (32 products) |
| `04-tryon-hero.jpg` | AI Try-On page hero — "See it on you, before you buy", with the status badge |
| `05-tryon-how-it-works.jpg` | How it works — the three-step flow, alongside a real rendered result |
| `06-tryon-stats.jpg` | Try-on metrics panel and positioning |
| `07-tryon-modal-result.jpg` | **The AI Virtual Mirror modal with a completed result** — your photo, the outfit, and the generated output side by side, in the product page where a shopper actually buys |

## Mobile

This category is bought on phones, so the mobile views are the real product.

| File | Page |
|---|---|
| `08-mobile-home.jpg` | Homepage hero on mobile, with the "See AI Try-On" call to action |
| `09-mobile-featured.jpg` | Featured Pieces grid — the browsing experience on a phone |
| `10-mobile-tryon-page.jpg` | AI Try-On page on mobile |
| `11-mobile-tryon-modal.jpg` | The AI Virtual Mirror modal in its empty state, before a photo is uploaded — shows the three-panel layout a shopper is presented with |

## The one to look at first

`07-tryon-modal-result.jpg`. Everything else shows a storefront; that screenshot shows the commercial moment — a shopper on a product page, her own photo on the left, the catalog outfit in the middle, and herself wearing it on the right, with a "Try It On" button and the disclaimer *"AI-generated preview for illustration. Actual fit and colour may vary."*

Try-on is inside the buying flow, not on a separate demo page. That is the point of the product.

## Note on the live demo

The storefront is publicly reachable and fully browsable. Interactive try-on is not accepting live requests at present — the GPU worker is not kept warm continuously, which is why `11-mobile-tryon-modal.jpg` shows the empty state. The site's own AI Try-On page carries the status badge *"Live on Mahila · Still actively in development"*.

Real outputs from the system — 19 of them — are in [`../tryon/`](../tryon/), with the reading guide in [`../../results/README.md`](../../results/README.md).
