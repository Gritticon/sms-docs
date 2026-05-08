---
title: KYC Orders Quick Shop Design
type: spec
project: kyc
module: Online Orders
last-updated: 2026-03-23
---

> Auto-promoted to testing on 2026-03-23 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

# KYC Orders — Quick Shop Experience — Specification

Design document for the KYC orders experience as a quick, minimal-friction shopping site for students (bags, books, uniform, event passes, other). Cart and checkout live in a slide-out drawer; no full-page cart/checkout.

---

## Feature Overview & User Flows

### Goal
Quick shopping: browse by category → add to cart → open cart drawer → adjust/remove → checkout from drawer (summary, confirm/place order).

### User flow (high level)
1. **Browse** — Student lands on Orders (quick shop). Sees category filter/tabs (e.g. Bags, Books, Uniform, Event Passes, Other) and a product grid/list.
2. **Add to cart** — Each product card shows image, basic info, and "Add to cart". Tapping adds the item (default qty 1) and may show brief feedback (e.g. snackbar or cart badge update).
3. **Open cart** — FAB, app bar icon, or persistent cart entry opens the **cart drawer** (slide-out from right). No navigation to a full cart page.
4. **Cart drawer** — List of line items (thumbnail, name, price, quantity controls, remove), subtotal, and "Checkout" CTA. States: empty cart message, loading, error.
5. **Checkout (in drawer)** — Either as a second panel/section within the same drawer or as a continuation step: order summary, delivery/collection details (if needed), "Place order" button. On success: confirmation and drawer closes or shows success state.

### Out of scope for this doc
Payment gateway integration, order history list, and order tracking are MVP-out; place order creates an order record and shows confirmation only.

---

## 1. Summary

We are building the **KYC Orders quick-shop**: a category-based product catalog (Bags, Books, Uniform, Event Passes, Other) with product cards (image, info, Add to cart), and a **cart + checkout drawer** (slide-out). Students browse, add to cart, open the drawer to review/edit quantities and remove items, then complete checkout inside the drawer (summary, delivery/collection details, place order). The experience is mobile-first, minimal-friction, and accessible; all business logic (catalog, cart validation, order placement) lives in the FastAPI backend.

---

## 2. Atomic Requirements

### 2.1 Functional

| ID | Requirement | Priority |
|----|-------------|----------|
| F1 | System displays a product catalog organized by categories (Bags, Books, Uniform, Event Passes, Other). | P0 |
| F2 | User can filter or switch the product list by category (tabs, chips, or dropdown). | P0 |
| F3 | Each product is displayed with at least: image, name, price, and an "Add to cart" control. | P0 |
| F4 | User can add a product to the cart (default quantity 1) from the product card. | P0 |
| F5 | Cart is accessible via a slide-out drawer (not a full page). | P0 |
| F6 | Cart drawer shows line items with: image thumbnail, name, price, quantity controls (increment/decrement), and remove. | P0 |
| F7 | Cart drawer shows subtotal and a primary "Checkout" / "Place order" CTA. | P0 |
| F8 | User can update quantity per line item and remove items from the cart within the drawer. | P0 |
| F9 | User can proceed to checkout from the drawer: view order summary and provide/confirm delivery or collection details. | P0 |
| F10 | User can place the order from the drawer; system creates the order and shows confirmation (success or error). | P0 |
| F11 | System enforces quantity limits per product (min/max) when adding or updating in cart. | P1 |
| F12 | System handles out-of-stock products (e.g. hide Add to cart or show "Out of stock"; prevent adding). | P1 |
| F13 | Guest vs logged-in student: cart/checkout behavior defined (e.g. guest may browse only; checkout requires login). | P1 |

### 2.2 Non-Functional

| ID | Requirement | Priority |
|----|-------------|----------|
| NF1 | All catalog, cart, and order logic (validation, totals, stock checks) are implemented in the FastAPI backend; Flutter only performs API calls and presentation. | P0 |
| NF2 | UI is mobile-first and responsive (drawer usable on small screens; product grid adapts). | P0 |
| NF3 | List/catalog and cart drawer show a loading indicator for every API request until response is received. | P0 |
| NF4 | Interactive elements meet minimum touch target 48x48px and have semantic labels for accessibility. | P1 |
| NF5 | Cart drawer and product list perform acceptably (e.g. smooth open/close; no blocking UI during fetch). | P1 |

---

## 3. User Stories & Acceptance Criteria

### Story 1: Browse products by category

**As a** student, **I want** to see products grouped by category and switch between categories **so that** I can quickly find bags, books, uniform, or other items.

**Acceptance criteria:**

- [ ] On opening Orders (quick shop), the product list is shown (all or default category).
- [ ] User can switch category via tabs, chips, or a single-select control; the list updates to show only that category's products.
- [ ] A loading indicator is shown while the catalog is fetched; it is dismissed on success or error.
- [ ] Empty state: when a category has no products, show a clear message (e.g. "No products in this category").

**Requirement IDs:** F1, F2, NF1, NF3

---

### Story 2: Product card and Add to cart

**As a** student, **I want** to see each product's image, name, price, and add it to the cart **so that** I can build my order quickly.

**Acceptance criteria:**

- [ ] Each product card shows: at least one image, name, price, and an "Add to cart" button (or equivalent control).
- [ ] Tapping "Add to cart" adds one unit of that product to the cart (or respects backend default).
- [ ] User receives brief feedback that the item was added (e.g. snackbar or cart badge update).
- [ ] If the product is out of stock, "Add to cart" is disabled or replaced with "Out of stock"; user cannot add.
- [ ] If quantity limit is reached (e.g. max per product), adding again is disabled or user sees a message.

**Requirement IDs:** F3, F4, F11, F12, NF1, NF3

---

### Story 3: Open and view cart in drawer

**As a** student, **I want** to open the cart in a slide-out drawer and see my items **so that** I can review and edit without leaving the catalog.

**Acceptance criteria:**

- [ ] A control (e.g. app bar icon, FAB) opens a drawer from the right (or configured side) overlaying the catalog.
- [ ] Drawer shows the list of line items: image thumbnail, name, unit price, quantity, line total (or equivalent).
- [ ] Drawer shows subtotal and a prominent "Checkout" (or "Place order") CTA.
- [ ] Empty cart: drawer shows a clear message (e.g. "Your cart is empty") and no checkout CTA or disabled.
- [ ] Loading: while cart is fetched, drawer shows a loading state; on error, show an error message with retry if applicable.

**Requirement IDs:** F5, F6, F7, NF1, NF3

---

### Story 4: Update quantities and remove items in cart

**As a** student, **I want** to change quantities and remove items in the cart drawer **so that** I can correct my order before checkout.

**Acceptance criteria:**

- [ ] Each line item has quantity controls (e.g. minus/plus or stepper) and a remove control.
- [ ] Increasing quantity is limited by product max (if any); decreasing to 0 or tapping remove removes the line.
- [ ] Subtotal (and any per-line totals) update when quantity changes or items are removed.
- [ ] Changes are persisted (backend or local) so that closing and reopening the drawer reflects the same cart.

**Requirement IDs:** F6, F8, F11, NF1, NF3

---

### Story 5: Checkout and place order from drawer

**As a** student, **I want** to complete checkout inside the drawer (summary, delivery/collection, place order) **so that** I can confirm and submit my order without a full-page flow.

**Acceptance criteria:**

- [ ] From the cart drawer, user can tap "Checkout" to see order summary (items, quantities, subtotal).
- [ ] User can enter or confirm delivery/collection details (or these are pre-filled for logged-in student).
- [ ] "Place order" submits the order to the backend; a loading indicator is shown until response.
- [ ] On success: show confirmation (e.g. "Order placed" with order ref); drawer can close or show success state.
- [ ] On failure: show error message (e.g. via AppMessage); user can retry or correct and resubmit.
- [ ] If user is not logged in, checkout prompts login (or redirects) before allowing place order.

**Requirement IDs:** F7, F9, F10, F13, NF1, NF3

---

## 4. Dependencies

- **Catalog API** — Backend exposes product list/filter by category (e.g. `GET /products?category=...`). Owner: backend.
- **Cart API** — Backend (or client session) supports add/update/remove cart and return current cart (e.g. `GET /cart`, `POST /cart/items`, `PATCH/DELETE` line items). Owner: backend.
- **Order API** — Backend accepts order placement with cart snapshot and delivery/collection info (e.g. `POST /orders`). Owner: backend.
- **Auth** — If checkout requires login, KYC auth/session must be available; guest flow (browse only or limited cart) must be defined. Owner: product/backend.

---

## 5. Risks & Mitigations

| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| Cart/checkout API not ready | High | Med | Define API contract first; Flutter can use mock or minimal backend until ready. |
| Drawer UX too cramped on small screens | Med | Med | Design drawer for narrow width first; test with real content and long product names. |
| Stock/quantity rules unclear | Med | Med | Codify rules in backend (max per product, out-of-stock); document in API spec. |

---

## 6. Edge Cases & Error Handling

- **Empty cart:** Drawer shows message "Your cart is empty" (or similar); no checkout CTA or CTA disabled. No line items.
- **Out of stock:** Product card shows "Out of stock" and Add to cart is disabled; if already in cart, backend may reject update or order placement; show clear error.
- **Quantity limits:** When adding or increasing quantity, enforce min/max per product; show message or disable control when at max.
- **Guest vs logged-in:** Define once: e.g. guest can browse and add to cart; checkout requires login. If guest, show "Sign in to checkout" and redirect to login, then return to cart.
- **Network/API failure:** Show error message (AppMessage or inline); offer retry for cart fetch and for place order.
- **Loading:** Every catalog, cart, and place-order request shows loading from start until response (success or error).

---

## 7. MVP Scope

### In scope (v1)
- Category-based product catalog (Bags, Books, Uniform, Event Passes, Other).
- Product card: image, name, price, Add to cart.
- Cart drawer: line items (thumbnail, name, price, quantity controls, remove), subtotal, Checkout CTA.
- Checkout step in drawer: order summary, delivery/collection details, Place order.
- Empty cart, loading, and error states in drawer.
- Basic quantity limits and out-of-stock handling (disable add or show message).
- Logged-in student can place order; guest behavior (e.g. must log in to checkout) as defined.

### Out of scope (backlog)
- Payment gateway integration (e.g. online payment); MVP = place order without payment or "pay later" only.
- Order history list and order tracking screens.
- Wishlist, saved carts, or cross-device cart sync (unless explicitly in scope later).
- Advanced filters (price range, search) — can be added post-MVP.

### Success criteria for MVP
- Student can browse by category, add items to cart, open drawer, edit cart, and place an order from the drawer.
- All API requests show loading until response; errors are shown and retry where appropriate.
- Drawer is usable on mobile and meets 48px touch targets and semantic labels.

---

## 8. UI/UX Spec: Cart & Checkout Drawer

### 8.1 Drawer content (cart view)

- **Header:** Title "Cart" (or "Your cart") and close button.
- **Body:**
  - **Cart list:** Scrollable list of line items.
  - **Line item:** Image thumbnail (small, fixed aspect ratio), product name, unit price, quantity controls (minus / number / plus), optional line total, remove (icon or text).
  - **Footer:** Subtotal (single line), primary button "Checkout" (or "Proceed to checkout").
- **Empty state:** Centered message e.g. "Your cart is empty" and optional "Continue shopping" (dismisses drawer).
- **Loading:** Centered spinner or skeleton list while cart is loading.
- **Error:** Inline message and "Retry" (or "Try again") button.

### 8.2 Checkout step (inside drawer or continuation)

- **Option A — Same drawer, second panel:** Cart list collapses or is replaced by checkout content when user taps "Checkout".
- **Option B — Continuation in same drawer:** Below cart list, expandable "Checkout" section: order summary (read-only list + subtotal), delivery/collection fields (or summary if pre-filled), "Place order" button.
- **Order summary:** List of items (name, qty, price) and subtotal; no need for full product images in summary.
- **Delivery/collection:** Fields or chips (e.g. "School pickup", "Home delivery") and address/contact if required; pre-fill for logged-in student when available.
- **Place order:** Primary button; on tap show loading until API response; then success message and close drawer or show confirmation in drawer.

### 8.3 States summary

| State | Content |
|-------|---------|
| Empty cart | Message "Your cart is empty"; no checkout CTA or disabled. |
| Loading | Spinner or skeleton in drawer body. |
| Error | Error text + Retry. |
| Cart with items | Line items + subtotal + Checkout CTA. |
| Checkout | Summary + delivery/collection + Place order. |
| Placing order | Loading on Place order button or overlay. |
| Order placed | Success message (and optionally order ref); close or reset drawer. |

---

## 9. Component Inventory & Storybook Stories

Use Storybook (or Flutter equivalent) where applicable. Recommended components and stories:

| Component | Purpose | Suggested stories |
|------------|---------|-------------------|
| **ProductCard** | Single product: image, name, price, Add to cart | Default, Out of stock, Loading image, Long name/price |
| **CategoryFilter** | Tabs/chips/dropdown for category switch | Default, Selected state, Many categories |
| **CartDrawer** | Slide-out container and layout | Empty, With items, Loading, Error |
| **CartLineItem** | One row in cart: thumbnail, name, price, qty, remove | Default, Min qty, Max qty, Long name |
| **CheckoutSummary** | Order summary block (items + subtotal) | Default, Many items |
| **QuantityStepper** (or inline controls) | +/- and value | Default, Disabled minus at 1, Disabled plus at max |

Document in Storybook (or in this doc): which props drive "empty", "loading", "error" so both Flutter and design can align.

---

## 10. Categories & Product Model (High-Level)

### 10.1 Suggested categories

- **Bags** — School bags, backpacks, etc.
- **Books** — Textbooks, notebooks, stationery.
- **Uniform** — School uniform, PE kit, etc.
- **Event Passes** — Events, trips, tickets.
- **Other** — Anything that doesn't fit above.

Category is a single select per product (e.g. `category_id` or `category_slug`).

### 10.2 Minimal product fields (for spec only)

- `id` — Unique product id.
- `name` — Display name.
- `category` — Category id or slug (Bags, Books, Uniform, Event Passes, Other).
- `image_url` — Main image URL (optional; multiple images can be added later).
- `price` — Unit price (decimal or integer in smallest currency unit).
- `currency` — Optional; if single currency, can be system-wide.
- `stock_quantity` — Optional; if used, 0 or null = out of stock.
- `max_quantity_per_order` — Optional; cap per line item.
- `is_available` — Optional; if false, treat as out of stock.

Cart line item (minimal): `product_id`, `quantity`, and optionally `unit_price` snapshot at add time.

---

## 11. Design Document Summary

This document is the single reference for the KYC Orders quick-shop feature. It contains:

- **Feature overview and user flows** — Browse → add to cart → open cart drawer → checkout (Section above and Section 1).
- **Structured product spec** — Atomic requirements (Section 2), user stories with acceptance criteria (Section 3), dependencies (Section 4), risks (Section 5), edge cases (Section 6), MVP scope (Section 7).
- **UI/UX spec for the drawer** — Drawer content, checkout step, and states (Section 8).
- **Component inventory and Storybook** — Recommended components and stories (Section 9).
- **Categories and product model** — High-level list and minimal fields (Section 10).

Implementation: Flutter for UI only (presentation, drawer, forms, API calls); FastAPI for catalog, cart, validation, and order placement. Keep requirements atomic and testable for both frontend and backend.
