# COD Store — Cash on Delivery E-Commerce Platform

A complete, production-ready e-commerce website built for **Cash on Delivery (COD)** orders, with a modern customer storefront and a secure admin panel.

Built with **Next.js 14 (App Router) + TypeScript + Tailwind CSS + Supabase (PostgreSQL, Auth, Storage)** and deployable to **Netlify**.

---

## Table of contents

1. [Features](#features)
2. [Tech stack](#tech-stack)
3. [Project structure](#project-structure)
4. [Prerequisites](#prerequisites)
5. [Supabase setup](#1-supabase-setup)
6. [Database schema](#2-database-schema)
7. [Create your admin account](#3-create-your-admin-account)
8. [Storage setup](#4-storage-setup)
9. [Environment variables](#5-environment-variables)
10. [Run locally](#6-run-locally)
11. [Deploy to Netlify](#7-deploy-to-netlify)
12. [Custom domain](#8-custom-domain)
13. [Admin panel guide](#admin-panel-guide)
14. [Customer flow](#customer-flow)
15. [Security model](#security-model)
16. [Updating the site](#updating-the-site)
17. [Troubleshooting](#troubleshooting)

---

## Features

### Customer storefront
- **Home** page with featured products, categories and CMS blocks.
- **Shop / All Products** with category filters and search.
- **Category** pages (`/category/[slug]`).
- **Product detail** pages with clean SEO URLs (`/product/wireless-headphones`), image gallery, variants (size/colour), features, specifications, FAQs, reviews, shipping & COD info, related products.
- **Cart** with quantity changes, variant selection, live totals, shipping calculation and coupon support (persisted in `localStorage`).
- **COD checkout** — collects name, mobile, email (optional), address, city, state, pincode; shows the final payable amount; creates the order server-side with a unique customer-facing Order ID.
- **Order confirmation** page with order summary and tracking link.
- **Order tracking** (`/track-order`) — enter Order ID + phone to see status timeline (sensitive data is never exposed).
- **Custom CMS pages** (`/pages/[slug]`) created from the admin panel.
- **Custom 404** page.

### Admin panel (`/admin`)
- **Secure login**, logout, session management, password reset/recovery, protected routes.
- **Dashboard** with KPI cards (total/pending/confirmed/processing/shipped/completed/cancelled orders, total revenue, COD revenue, product count, low-stock count), sales chart, status distribution chart, recent orders and products.
- **Product management** — add/edit/delete/archive, image upload (Supabase Storage), price/description/stock/SKU/category editing, variants, COD toggle, featured flag, slug editing with automatic old-URL redirects, preview and copy-URL.
- **Page builder** — create/edit/delete/publish custom pages with sections (heading, text, image, HTML), SEO fields and OG image.
- **Category management** with slug handling.
- **Order management** — full order fields, customisable customer-facing Order ID, status changes with history, tracking number, notes, invoice print/download, copy info, delete.
- **Settings** — store name, logo, email, phone, address, currency, shipping charge, free-shipping threshold, COD availability, Order ID prefix, social links, footer content and SEO defaults.
- **Search & filters** across products and orders.

---

## Tech stack

| Layer | Technology |
| --- | --- |
| Framework | Next.js 14 (App Router, Server Components, Server Actions, Route Handlers) |
| Language | TypeScript (strict) |
| Styling | Tailwind CSS 3 |
| Database | Supabase (PostgreSQL) |
| Auth | Supabase Auth (`@supabase/ssr`, cookie-based sessions) |
| Storage | Supabase Storage |
| Charts | Recharts |
| Icons | lucide-react |
| Validation | Zod |
| Hosting | Netlify (`@netlify/plugin-nextjs`) |

---

## Project structure

```
cod-store/
├─ src/
│  ├─ app/
│  │  ├─ (store)/                 # Customer storefront (shared header/footer layout)
│  │  │  ├─ page.tsx              # Home
│  │  │  ├─ shop/                 # All products
│  │  │  ├─ category/[slug]/      # Category listing
│  │  │  ├─ product/[slug]/       # Product detail
│  │  │  ├─ cart/                 # Cart
│  │  │  ├─ checkout/             # COD checkout
│  │  │  ├─ order-confirmation/   # Confirmation
│  │  │  ├─ track-order/          # Order tracking
│  │  │  └─ pages/[slug]/         # CMS pages
│  │  ├─ admin/
│  │  │  ├─ login/                # Login
│  │  │  ├─ reset-password/       # Password reset / recovery
│  │  │  ├─ (panel)/              # Protected admin area (sidebar layout)
│  │  │  │  ├─ dashboard/
│  │  │  │  ├─ orders/            # + [id]
│  │  │  │  ├─ products/          # + new + [id]
│  │  │  │  ├─ categories/
│  │  │  │  ├─ pages/             # + new + [id]
│  │  │  │  └─ settings/
│  │  │  └─ actions/              # Server actions (products, orders, pages, …)
│  │  ├─ api/
│  │  │  ├─ orders/               # POST create order
│  │  │  └─ track/                # POST track order
│  │  ├─ auth/callback/           # Supabase auth callback
│  │  ├─ sitemap.ts robots.ts manifest.ts
│  │  └─ layout.tsx globals.css
│  ├─ components/
│  │  ├─ storefront/              # Header, Footer, ProductCard, Cart, Checkout…
│  │  ├─ admin/                   # AdminShell, forms, tables, charts, uploader
│  │  └─ ui/                      # Button, Alert, StatusBadge, EmptyState, Spinner
│  ├─ lib/
│  │  ├─ supabase/                # browser / server / admin / middleware clients
│  │  ├─ data/                    # Server-side data fetchers
│  │  ├─ auth.ts                  # getAdminUser / requireAdmin / assertAdmin
│  │  ├─ settings.ts types.ts utils.ts validation.ts orders.ts cart.tsx
│  │  └─ rate-limit.ts
│  └─ middleware.ts               # Session refresh + /admin protection
├─ supabase/migrations/           # SQL migrations (run in order)
├─ public/                        # Static assets
├─ netlify.toml .env.example next.config.mjs tailwind.config.ts tsconfig.json
```

---

## Prerequisites

- **Node.js 18.18+** (20 recommended) and npm.
- A free **Supabase** account — https://supabase.com
- A **Netlify** account (for deployment) — https://netlify.com
- A **GitHub** account (to connect the repo to Netlify).

---

## 1. Supabase setup

1. Go to https://supabase.com/dashboard and click **New project**.
2. Choose an organisation, enter a **Name**, a strong **Database password** (save it), and pick the region closest to your customers (e.g. *Mumbai / ap-south-1* for India).
3. Wait ~2 minutes for the project to finish provisioning.
4. Open **Project Settings → API** and copy:
   - **Project URL** → `NEXT_PUBLIC_SUPABASE_URL`
   - **anon public** key → `NEXT_PUBLIC_SUPABASE_ANON_KEY`
   - **service_role** key → `SUPABASE_SERVICE_ROLE_KEY` *(keep secret — server only)*

---

## 2. Database schema

All SQL lives in `supabase/migrations/`. Run them **in order** in the Supabase **SQL Editor** (Dashboard → SQL Editor → New query → paste → Run).

| File | Purpose |
| --- | --- |
| `0001_schema.sql` | Tables, keys, indexes, constraints, timestamps |
| `0002_functions.sql` | `generate_order_number`, `track_order`, `dashboard_stats`, `sales_series`, `status_distribution` |
| `0003_rls.sql` | Row Level Security policies + `is_admin()` helper |
| `0004_storage.sql` | `media` storage bucket + policies |
| `0005_seed.sql` | Sample categories, products, page and settings |
| `0006_admin_setup.sql` | Promote a user to admin |
| `0007_coupon_helpers.sql` | Coupon redemption helpers |

### Tables

`admin_profiles`, `categories`, `products`, `product_images`, `product_variants`, `slug_redirects`, `orders`, `order_items`, `order_status_history`, `pages`, `site_settings`, `coupons`, `inventory_movements`.

Each table has a primary key, appropriate foreign keys (`on delete cascade` where sensible), indexes on frequently-filtered columns (slug, status, created_at, order_number, phone), unique constraints (slug, SKU, order_number) and `created_at` / `updated_at` timestamps.

### Key database functions
- **`generate_order_number(prefix)`** — produces sequential customer-facing IDs like `COD-2026-000001`.
- **`track_order(order_number, phone)`** — returns a *safe* subset of order data only when the phone matches (prevents enumeration).
- **`dashboard_stats()`, `sales_series(days)`, `status_distribution()`** — aggregate analytics for the admin dashboard.
- **`is_admin()`** — `SECURITY DEFINER` helper used throughout RLS policies.

> **Tip:** If you prefer the CLI, install the Supabase CLI and run `supabase db push` against your linked project.

---

## 3. Create your admin account

1. In Supabase Dashboard → **Authentication → Users → Add user → Create new user**.
   Enter your admin **email** and a strong **password**. (Auto-confirm the email if you like.)
2. Open `supabase/migrations/0006_admin_setup.sql`, replace `you@example.com` with your admin email, and run it in the **SQL Editor**. This inserts a row into `admin_profiles` with `role = 'admin'` and `is_active = true`.
3. Verify:
   ```sql
   select id, email, role, is_active from public.admin_profiles;
   ```
4. Sign in at `/admin/login` with that email/password.

> Only users present in `admin_profiles` with `is_active = true` can access `/admin`. New signups default to `role = 'staff'`, `is_active = false` (no self-promotion).

**Password reset:** use the "Forgot password?" link on `/admin/login`. Supabase emails a recovery link that lands on `/admin/reset-password`.

### Auth configuration
In Supabase Dashboard → **Authentication → URL Configuration**:
- **Site URL:** `https://your-domain.com` (or `http://localhost:3000` for local dev).
- **Redirect URLs:** add `https://your-domain.com/auth/callback` and `http://localhost:3000/auth/callback`.

---

## 4. Storage setup

`0004_storage.sql` creates a **public** bucket named `media` (5 MB file limit, image MIME types only) with policies that allow public **read** but admin-only **write/update/delete** (via `is_admin()`).

Product, page and brand images uploaded from the admin panel are stored here and referenced by public URL. If you rename the bucket, set `NEXT_PUBLIC_STORAGE_BUCKET` accordingly.

---

## 5. Environment variables

Copy `.env.example` to `.env.local` and fill in your values:

```bash
NEXT_PUBLIC_SUPABASE_URL=https://YOUR-PROJECT-REF.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=YOUR-ANON-PUBLIC-KEY

# Server-only secret — NEVER prefix with NEXT_PUBLIC_
SUPABASE_SERVICE_ROLE_KEY=YOUR-SERVICE-ROLE-KEY

NEXT_PUBLIC_SITE_URL=https://your-domain.com
NEXT_PUBLIC_STORAGE_BUCKET=media
```

| Variable | Scope | Description |
| --- | --- | --- |
| `NEXT_PUBLIC_SUPABASE_URL` | Public | Supabase project URL |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Public | Anon key (safe; guarded by RLS) |
| `SUPABASE_SERVICE_ROLE_KEY` | **Server only** | Bypasses RLS for trusted server operations. Never expose. |
| `NEXT_PUBLIC_SITE_URL` | Public | Canonical URLs, OG tags, sitemap |
| `NEXT_PUBLIC_STORAGE_BUCKET` | Public | Storage bucket name (default `media`) |

> The service-role key is only imported in server-only modules (`src/lib/supabase/admin.ts`) and never reaches the browser bundle.

---

## 6. Run locally

```bash
npm install
npm run dev
```

Open http://localhost:3000 for the storefront and http://localhost:3000/admin for the admin panel.

Other scripts:

```bash
npm run build      # production build
npm run start      # run the production build
npm run typecheck  # TypeScript check
npm run lint       # ESLint
```

---

## 7. Deploy to Netlify

1. Push this project to a GitHub/GitLab repository.
2. In Netlify: **Add new site → Import an existing project** and pick the repo.
3. Build settings (also defined in `netlify.toml`):
   - **Build command:** `npm run build`
   - **Publish directory:** `.next`
   - The `@netlify/plugin-nextjs` plugin is added automatically via `netlify.toml`.
4. **Environment variables** — Site configuration → **Environment variables** → add every variable from section 5 (including the server-only `SUPABASE_SERVICE_ROLE_KEY`).
5. Click **Deploy**. Netlify installs dependencies, builds, and deploys.

`netlify.toml` also sets `NODE_VERSION = "20"`, security headers (`X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy`, `Permissions-Policy`) and long-cache headers for `/_next/static/*`.

> After the first deploy, add the Netlify URL to Supabase **Authentication → URL Configuration** (Site URL + redirect URLs).

---

## 8. Custom domain

1. Netlify → **Domain management → Add a domain** and follow the DNS instructions (or transfer DNS to Netlify).
2. Update `NEXT_PUBLIC_SITE_URL` to the custom domain and redeploy.
3. Add the custom domain to Supabase **Authentication → URL Configuration** (Site URL + `…/auth/callback` redirect).

---

## Admin panel guide

| URL | Purpose |
| --- | --- |
| `/admin/login` | Sign in |
| `/admin/reset-password` | Password recovery |
| `/admin/dashboard` | KPIs, charts, recent activity |
| `/admin/orders` | Order list, filters, status management |
| `/admin/orders/[id]` | Order detail, status history, invoice |
| `/admin/products` | Product list, filters |
| `/admin/products/new` | Add product |
| `/admin/products/[id]` | Edit product |
| `/admin/categories` | Category management |
| `/admin/pages` | CMS pages |
| `/admin/pages/new`, `/admin/pages/[id]` | Page builder |
| `/admin/settings` | Store settings |

**Order statuses:** Pending → Confirmed → Processing → Shipped → Out for Delivery → Completed, plus Cancelled and Returned. Every change is recorded in `order_status_history` with the admin who made it.

**Slug changes:** when you change a product, page or category slug, the old path is stored in `slug_redirects` and the storefront automatically 301-redirects to the new URL so existing links and SEO rankings are preserved.

---

## Customer flow

1. Browse **Home / Shop / Category** → open a **Product**.
2. Choose variants, **Add to cart**.
3. Open **Cart**, adjust quantities, proceed to **Checkout**.
4. Fill delivery details, confirm **Cash on Delivery**, place the order.
5. See the **Order confirmation** with the Order ID and a tracking link.
6. Later, use **/track-order** with the Order ID + phone to view the status timeline.

All prices, totals, shipping and stock are **recalculated server-side** at order creation; browser-supplied values are never trusted.

---

## Security model

- **Supabase Auth** with cookie-based sessions; middleware refreshes sessions and blocks unauthenticated access to `/admin/*`.
- **Row Level Security** on every table: public read for published/active content, admin-only writes. `is_admin()` gates privileged operations.
- **Service-role key** is used only in server-only code and never shipped to the browser.
- **Server-side validation** with Zod for every mutation (products, orders, pages, settings, categories).
- **Server-side price recalculation** and idempotency for order creation (prevents tampering and duplicate submissions).
- **Rate limiting** on the public order/track endpoints.
- **Safe order tracking** via a `SECURITY DEFINER` RPC that requires Order ID **and** phone.
- **XSS-safe rendering**, parameterised queries via the Supabase client (no string-concatenated SQL), and sanitised error messages that never leak DB details or secrets.
- **Secure image uploads** — type and size validated, admin-only storage policies.
- **Security headers** set in `netlify.toml`.

---

## Updating the site

- **Content** (products, pages, categories, settings): use the admin panel — no redeploy needed.
- **Schema changes**: add a new numbered SQL file in `supabase/migrations/` and run it in the SQL Editor.
- **Code changes**: commit and push; Netlify rebuilds and redeploys automatically.

---

## Troubleshooting

| Symptom | Fix |
| --- | --- |
| Storefront shows defaults / no products | Confirm migrations ran and `site_settings` + `products` have rows. Check env vars. |
| `/admin` redirects to login in a loop | Ensure your user exists in `admin_profiles` with `is_active = true`; check Supabase redirect URLs. |
| Images don't upload | Verify the `media` bucket exists and storage policies from `0004_storage.sql` ran; confirm `NEXT_PUBLIC_STORAGE_BUCKET`. |
| "Invalid API key" | Re-copy the anon/service keys and redeploy. |
| Order number not generating | Ensure `0002_functions.sql` ran (the `generate_order_number` function must exist). |
| Build fails on Netlify | Set `NODE_VERSION=20` and all env vars; run `npm run build` locally to reproduce. |
| Missing product/page returns HTTP 200 | Next.js App Router streams responses, so `notFound()` renders the 404 UI but the status can stay 200. These pages emit `<meta name="robots" content="noindex, nofollow">` so search engines never index them. |

---

© COD Store. Built for Cash on Delivery commerce in India.
