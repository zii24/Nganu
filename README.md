# XFY Outfit — Frontend MVP

A premium, curated second-hand fashion marketplace frontend, built for Indonesia and beyond. This is a **frontend-only prototype**: all data is mocked/local, there is no real backend, authentication, payment processing, or shipping API integration yet. The code is structured so each of those can be dropped in later without a rewrite.

## Stack

- **Next.js 14** (App Router) + **TypeScript**
- **Tailwind CSS** with a small custom design-token system (`app/globals.css`, `tailwind.config.ts`)
- Hand-implemented **shadcn/ui-style** primitives in `components/ui/` (Button, Dialog, Sheet, Accordion, Tabs, Table, etc.) — styled the same way shadcn components are, but written directly rather than pulled from the shadcn CLI/Radix, so the project has zero extra runtime dependencies beyond `class-variance-authority`, `clsx`, `tailwind-merge`, and `lucide-react`. You can swap any of these for the official shadcn/Radix versions later with no structural changes.
- **lucide-react** for icons

## Getting started

```bash
npm install
npm run dev
```

Then open `http://localhost:3000`.

> This project was authored in a sandboxed environment without npm registry access, so the code has **not** been run through `npm install` / `next build` here. It's been written carefully and reviewed by hand, but please run a build locally and let me know if anything needs a fix.

## Project structure

```
app/                    Routes (App Router)
  page.tsx              Home
  shop/                 Shop (filters, sort, grid)
  product/[slug]/       Product detail
  wishlist/             Wishlist (localStorage)
  cart/                 Cart (localStorage)
  checkout/             Shipping → Payment → Confirmation
  order/[id]/           Order tracking (timeline, review, return)
  order/                Order-ID lookup landing page
  journal/              Editorial articles
  about/                Brand story + Buyer Protection + Shipping + Returns
  admin/                Admin prototype (dashboard, products, orders, shipping)

components/
  ui/                   Design-system primitives
  layout/               Header, Footer, mobile nav, search, currency/language toggle
  home/                 Homepage sections
  shop/                 Filter sidebar/drawer, sort, shop content
  product/              Gallery, condition badge/modal, measurements, authenticity,
                         shipping modal, offer modal, checkout options, product card/grid
  cart/, checkout/, order/, admin/, shared/

context/                Currency, Language (EN/ID), Wishlist, Cart, Recently Viewed
                        — all localStorage-backed, SSR-safe

lib/
  types.ts              All domain types (Product, Order, Offer, ShippingRegion, ...)
  mock-data.ts           12 mock products across all 11 categories
  mock-orders.ts          Mock orders covering different tracking states
  mock-offers.ts           Mock offers for the admin dashboard
  mock-shipping.ts         14 configurable shipping regions
  mock-journal.ts          Editorial articles
  i18n.ts                  EN/ID UI string dictionary
  order-store.ts           localStorage order persistence (simulates a future orders API)
  admin-draft-store.ts     localStorage draft-product persistence for the admin prototype
  utils.ts, use-local-storage.ts
```

## Where the backend plugs in later

Everything reads from `lib/mock-*.ts` and writes to `localStorage` for now. To connect a real backend:

1. **Products** — replace `getPublishedProducts()`, `getProductBySlug()`, etc. in `lib/mock-data.ts` with fetch calls to your product API. Keep the same function signatures and every page keeps working.
2. **Offers** (`components/product/OfferModal.tsx`) — the minimum-offer comparison currently runs client-side, which means the minimum is technically present in the client bundle. This is flagged in a code comment. Move this evaluation to a server route (e.g. `POST /api/offers`) so the minimum never ships to the browser.
3. **Orders** (`lib/order-store.ts`) — swap the `localStorage` read/writes for real API calls. `app/checkout/page.tsx` and `app/order/[id]/page.tsx` are the only two places that touch this file.
4. **Payment** (`components/checkout/PaymentForm.tsx`) — UI is ready for Xendit; wire the "Place Order" handler in `app/checkout/page.tsx` to create a Xendit invoice/VA and redirect or poll for confirmation.
5. **Shipping rates** (`lib/mock-shipping.ts`, `app/admin/shipping/page.tsx`) — move the region table to a database; the admin page already demonstrates the enable/disable + fee/estimate editing UI, it just needs a save endpoint.
6. **Auth** — there is currently no login. The header's account icon and admin section are open by design for this prototype; add real auth (NextAuth, Clerk, etc.) and gate `/admin/*` with middleware when ready.
7. **Wishlist / Cart / Recently Viewed** — currently per-browser via `localStorage` (as requested). To sync across devices, replace the `context/*Context.tsx` persistence layer with API-backed state once accounts exist.

## Notable product decisions

- **New Drop badge** is computed relative to `Date.now()` (72-hour window from `published_at`), so it always demos correctly regardless of when you run the project.
- **One-of-one inventory**: every product is a single physical item, so cart/quantity is always 1, and `Reserved`/`Sold` items stay visible but are not purchasable anywhere in the UI.
- **Condition scoring** is a fixed 6–10 scale (`lib/types.ts` → `ConditionScore`), with the explanation modal reusable everywhere the score badge appears.
- **Currency** never auto-converts — every product carries manually-entered `priceIdr`/`priceUsd`, and the UI only ever renders the currently-selected currency's field.
- **No brand/style filters** on Shop, per spec — brand is listing metadata only, shown on product cards/detail but not filterable/browsable as a taxonomy.
- All images are placeholder photography from `picsum.photos` (seeded, so they're stable across reloads) — swap for real product photography URLs when available; `next.config.mjs` is already configured to also allow `images.unsplash.com` if you want to use that instead.

## Known gaps (by design, for a frontend MVP)

- No real authentication or accounts.
- No real payment processing (clearly labeled as demo in the UI).
- Admin "Save" actions persist to `localStorage` only, clearly labeled as demo-only in the UI where relevant.
- No automated tests.
