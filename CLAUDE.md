# ShopMate POS Billing Manager
A local-first POS web app for small shops to create bills, manage inventory and customers, track dues, review sales, and print receipts.

## Masterplan

- Make **Create Bill** the fastest workflow: search products, adjust quantity/discount, select payment, complete sale, then print the invoice.
- Serve independent shop owners who need straightforward billing and stock visibility rather than accounting software.
- Require GenMB authentication and one-time shop setup before operational screens are accessible.
- Keep each user’s shop data isolated by authenticated user ID and synced from browser storage to GenMB KV.
- Deliberately remain single-shop and lightweight: no staff roles, suppliers, locations, purchase orders, GST filing, or accounting ledger.

## Tech Stack & Architecture

- **React 18 + TypeScript + Vite**
- **React Router DOM** with `HashRouter` in `src/main.tsx`.
  - URLs are hash-based: `#/bill`, `#/products`, `#/customers`, etc.
  - Do not change to `BrowserRouter` without configuring SPA fallback routing on the host.
- **Tailwind CSS v4**, enabled through `@tailwindcss/vite` in `vite.config.ts`.
- **Lucide React** for interface icons.
- **Inter** is loaded from Google Fonts in `index.html`.
- **GenMB Auth** is injected through `index.html` and provides:
  - Email/password signup, verification, sign-in, and password reset
  - Google sign-in
  - Magic-link sign-in
  - Session restoration, auth-state subscriptions, and logout
- **GenMB KV** is injected through `index.html` and is the only cloud persistence layer.

### Application organization

`src/App.tsx` is intentionally the main application module. It contains:

- Auth gate and login, signup, verification, reset, Google, and magic-link flows.
- First-time shop setup.
- Desktop sidebar and mobile navigation shell.
- Dashboard, POS, Products, Customers, Bills, Reports, Settings, and Invoice route views.
- Product/customer CRUD, confirmation modals, toasts, billing calculations, stock changes, and invoice print actions.
- Store synchronization through the `commit(updater)` pattern.

Shared UI is intentionally lightweight rather than using a component library:

- `src/components/ui.tsx` supplies `Button`, `Input`, `Field`, `Modal`, `StatusBadge`, `EmptyState`, and toast support.
- `src/components/AppErrorBoundary.tsx` prevents an uncaught render error from blanking the full app.

### State and persistence

There is no custom REST API, SQL database, server-side invoice renderer, or separate backend in this repository.

All business data is stored as one `Store` snapshot per GenMB user:

```ts
type Store = {
  setupCompleted: boolean;
  shop: Shop;
  products: Product[];
  customers: Customer[];
  bills: Bill[];
  updatedAt: number;
};
```

Data is scoped by authenticated user ID:

```txt
localStorage: shopflow-pos-store-v1:${userId}
GenMB KV:    shopflow:store:${userId}
```

Persistence is local-first:

1. `loadStore(userId)` restores the local browser snapshot.
2. `loadStoreFromKv(userId)` loads the GenMB KV snapshot.
3. `commit(updater)` in `src/App.tsx` creates the next immutable store state.
4. The resulting snapshot is saved through `saveStore()` and `saveStoreToKv()`.

**Important:** Never mutate `shop`, `products`, `customers`, `bills`, or nested `Bill.items` in place. Always return new arrays and objects from store updaters.

`normalizeStore()` in `src/lib/data.ts` is the persistence migration and validation boundary. It:

- Repairs malformed numeric values.
- Normalizes product fields and default units.
- Clamps discounts to valid limits.
- Recalculates bill subtotal, discount, and total from bill items.

Update `normalizeStore()` whenever a persisted model, invoice calculation, or discount rule changes.

## File Structure

```text
index.html                         # HTML shell, Inter font, and injected GenMB Auth/KV SDKs.
package.json                       # Vite scripts and frontend dependencies.
vite.config.ts                     # Enables React and Tailwind v4 plugins.
tsconfig.json                      # Strict TypeScript configuration for browser code.

src/
├── main.tsx                       # React entry point; HashRouter, error boundary, global CSS.
├── App.tsx                        # Main app: auth, setup, routes, screens, POS and store commits.
├── components/
│   ├── AppErrorBoundary.tsx        # Render-crash fallback with reload action.
│   └── ui.tsx                     # Shared buttons, inputs, modal, badges, empty states, toasts.
├── lib/
│   ├── data.ts                    # Domain types, starter products, store loading/saving/normalization.
│   └── utils.ts                   # Tailwind class merge, INR currency, date formatting helpers.
├── styles/
│   └── main.css                   # Tailwind import, color tokens, invoice and print styles.
└── types/
    └── genmb.d.ts                 # Type declarations for injected GenMB browser APIs.
```

## Key Features

### Authentication and first-time setup

- Unauthenticated visitors see the login experience.
- Supported sign-in methods are password, Google, and magic link.
- Signup requires email verification before access.
- Password reset requests are handled through `window.genmb.auth`.
- New users must complete shop setup before using business screens.
- Shop setup/settings persist:
  - Shop name, owner name, mobile, email, address, optional logo
  - Invoice prefix and next invoice number
  - Default discount and currency
  - Receipt paper size (`58mm` or `80mm`)
  - Auto-print receipt preference
  - Low-stock alerts and negative-stock permission

### Products and inventory

`Product` data:

```ts
type Product = {
  id: string;
  name: string;
  sku: string;
  category: string;
  purchasePrice: number;
  sellingPrice: number;
  stock: number;
  minimumStock: number;
  unit: string;
  image?: string;
};
```

- Products can be added, edited, searched, viewed, and deleted.
- Search behavior supports product name and SKU.
- Product stock status is based on `stock <= minimumStock`.
- Starter products are included only in a newly created local store via `starterProducts` in `src/lib/data.ts`.
- Completing a bill reduces each sold product’s stock.
- Sales above available stock are blocked unless `shop.allowNegativeStock` is enabled.
- Low-stock warnings depend on both product minimum stock and `shop.lowStockAlert`.

### Billing / POS

- Product search adds products to an in-progress bill.
- Each line preserves a billing-time snapshot of product name, SKU, price, quantity, and discount.
- Quantity can be increased/decreased or entered directly.
- Discount supports two semantics:

```ts
discountScope?: "line" | "per-unit";
```

- `per-unit` discount is multiplied by quantity.
- `line` discount applies once to the entire bill item.
- Bill totals are calculated as:
  - `subtotal = sum(price × quantity)`
  - `discount = sum(line discounts)`
  - `total = max(0, subtotal - discount)`

Current billing models do **not** include GST/tax fields. Do not add tax calculations to invoice totals without extending `Bill`, `Shop`, billing UI, and `normalizeStore()` together.

### Customers and dues

```ts
type Customer = {
  id: string;
  name: string;
  mobile: string;
  address: string;
  totalPurchases: number;
  due: number;
};
```

- Customers can be created and managed independently or selected while billing.
- Bills store customer name/mobile snapshots so historic receipts remain accurate after customer edits.
- Credit/Due bills increase customer outstanding due.
- Customer purchase totals and dues are updated on completed sales.

### Bills, receipts, and history

```ts
type Bill = {
  id: string;
  invoiceNumber: string;
  date: string;
  customerId?: string;
  customerName: string;
  customerMobile: string;
  items: BillItem[];
  subtotal: number;
  discount: number;
  total: number;
  paymentMethod: "Cash" | "UPI" | "Card" | "Credit / Due";
  amountReceived: number;
  change: number;
};
```

- Invoice numbers use `shop.invoicePrefix` plus `shop.nextInvoiceNumber`.
- Invoice number allocation and next-number update must occur in the same store commit as bill creation.
- Bill history supports invoice/customer lookup and date-based sales review.
- Invoice print markup uses the `#invoice-print` element and `.invoice-document` styles.
- Browser print is optimized for A4 and thermal receipt widths using:
  - `.receipt-80`
  - `.receipt-58`
- “Download PDF” behavior should use the browser print dialog’s PDF destination unless a real PDF generation library/service is introduced.

### Dashboard and reports

- Dashboard summarizes today’s sales, bill count, product count, low-stock products, recent bills, and quick actions.
- Reports are derived from persisted bills/products rather than a separate reporting API.
- Profit calculations must use `Product.purchasePrice` and should be treated as estimates for historical bills because bill items do not currently store purchase-price snapshots.

## Design Guidelines

- Use the existing clean, high-contrast shop-management visual system from `src/styles/main.css`.
- Primary colors:
  - Background: `#F5F8FB`
  - Navy text/navigation: `#102A43` / `#062B49`
  - Cyan primary action: `#08A9D6`
  - Bright focus ring: `#00C2E8`
  - Destructive: `#DC2626`
  - Success: `#16A34A`
  - Warning: `#F59E0B`
- Typography is **Inter** with system-font fallback.
- Keep controls large and touch-friendly; shared buttons have a minimum height of `2.5rem`.
- Use cards, rounded corners, subdued table headers, readable form labels, and clear empty states.
- Desktop uses a sidebar; mobile uses a compact mobile navigation/menu.
- Preserve print-specific rules in `src/styles/main.css`; do not let app shell backgrounds or hidden controls leak into printed invoices.

## App Flow

1. Open app → auth session restores through `AuthGate`.
2. Sign in, sign up/verify, use Google, magic link, or request password reset.
3. New user → complete shop setup.
4. Existing configured user → dashboard.
5. Create Bill:
   - Search/select products.
   - Set quantities and discounts.
   - Optionally attach/create a customer.
   - Choose Cash, UPI, Card, or Credit / Due.
   - Enter received amount where relevant.
   - Validate stock and complete the bill.
6. Completion:
   - Create bill and increment invoice number.
   - Reduce product stock.
   - Update customer total purchases/due where applicable.
   - Navigate to/show printable invoice.
7. Use Products, Customers, Bills, Reports, and Settings through hash routes.

Key edge cases:

- Never permit a completed empty bill.
- Clamp discounts so a line cannot become negative.
- Reject unavailable stock unless negative stock is enabled.
- Treat a cancelled auth popup as non-fatal and keep the login screen available.
- Cloud sync failures should not discard successfully saved local state.
- Historic bill totals must come from stored bill items, not current product prices.

## Conventions

- Use TypeScript types from `src/lib/data.ts`; do not duplicate domain model definitions in page components.
- Use `formatCurrency`, `formatDateTime`, and `toLocalDate` from `src/lib/utils.ts` for user-facing values.
- Use `cn()` for conditional Tailwind class composition.
- Prefer shared primitives from `src/components/ui.tsx` over ad hoc buttons, inputs, dialogs, and toast implementations.
- Route additions belong in `src/App.tsx` alongside the existing route configuration and navigation decisions.
- Add persisted fields in this order:
  1. Update the relevant type in `src/lib/data.ts`.
  2. Add defaults in `emptyShop` or `createInitialStore()`.
  3. Update `normalizeStore()`.
  4. Update setup/settings/forms and all affected calculations/views.
- Keep sensitive auth behavior inside `window.genmb.auth`; do not store credentials in the business `Store`.
- Preserve user-scoped storage key formats unless intentionally migrating all existing local/cloud data.

## Platform (GenMB)

This app is built and hosted on GenMB.

**Runtime:** Browser sandbox (iframe) or Cloud Run. No Node.js server — all code runs client-side unless `backend/` exists.

**Dependencies:** CDN-only (esm.sh, cdn.tailwindcss.com, unpkg). Use ES module imports with full CDN URLs. No `npm install` at runtime.

**Entry point:** `index.html` must include all CDN script tags. Tailwind via CDN with inline config.

**Built-in services (relative API paths only, never hardcode domains):**
- `/api/ai/completion` — AI proxy | `/api/data/{appId}/*` — PostgreSQL (DataConnect SDK)
- `/api/storage/{appId}/*` — File uploads (GCS) | `/api/auth/google/*` — Google OAuth
- `/api/contact/submit` — Contact form | SDKs: `window.genmb.db`, `.storage`, `.auth`

**File structure:** `index.html` (entry), `src/` (source), `styles/` (CSS), `backend/` (optional FastAPI), `CLAUDE.md` (this file).

**Cannot:** Install npm packages at runtime, access filesystem, make direct server-side calls from frontend, modify infra.
