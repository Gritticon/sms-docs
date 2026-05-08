---
title: KYC Orders Quick Shop — Design Layout
type: spec
project: kyc
module: Online Orders
last-updated: 2026-03-23
---

> Auto-promoted to testing on 2026-03-23 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

# KYC Orders Quick Shop — Design Layout

Layout specs and wireframe references for the KYC Orders quick-shop (catalog + cart drawer + checkout). Use with the product spec: `kyc-orders-quick-shop-design.md`.

---

## Layout overview

| Layout | Purpose | Connection |
|--------|---------|------------|
| **Catalog page** | Browse products by category; add to cart; open cart via app bar icon. | Primary screen. Cart icon opens the **cart drawer** (slide-out from right). |
| **Cart drawer** | View/edit line items, subtotal, proceed to checkout. | Overlays catalog. "Checkout" switches to **checkout step** inside the same drawer (second panel/view). |
| **Checkout (in drawer)** | Order summary, delivery/collection, Place order. | Replaces or continues below cart list in the same drawer; no full-page navigation. |

Flow: **Catalog** → (tap cart) → **Cart drawer** → (tap Checkout) → **Checkout view in drawer** → Place order → success / close.

---

## Screen / layout specs

### Catalog page

| Region | Description | Specs |
|--------|-------------|--------|
| **App bar** | Title "Orders", cart icon (with optional badge). | Height min 48px (56px typical). Center title; cart icon right-aligned. Touch target for cart min 48×48px. |
| **Category strip** | Horizontal tabs/chips: Bags, Books, Uniform, Event Passes, Other. | Single row, horizontal scroll. Chip min height 48px; padding 0 16px; 8px gap between chips. Selected: primary fill; unselected: outline. |
| **Product grid** | Cards in a grid. | **Mobile:** 2 columns. Gap 16px. Padding 16px. Cards: image (1:1), name, price, "Add to cart" button. |
| **Product card** | Image, name, price, Add to cart. | Image aspect ratio 1:1. Name 2-line clamp. Price prominent (e.g. primary color). Button full-width, min height 48px, 8px radius. |

**Spacing (catalog):**

- App bar padding: 16px horizontal.
- Category strip: 8px vertical, 16px horizontal; 8px gap between chips.
- Grid: 16px padding; 16px gap between cards.

**Touch targets:** All interactive elements (cart icon, chips, Add to cart) ≥ 48×48px.

---

### Cart drawer

| Region | Description | Specs |
|--------|-------------|--------|
| **Container** | Slide-out from right. | Width: `min(400px, 85vw)`. Max width 400px. Full viewport height; overlay/backdrop behind. |
| **Header** | "Your cart", close button. | Min height 48px; padding 0 16px. Close button 48×48px. Border below. |
| **Body** | Scrollable line items. | Scrollable; padding 16px. Empty/loading/error states (see below). |
| **Line item** | Thumbnail, name, price, qty stepper, remove. | Thumbnail 64×64px, 8px radius. Name + unit price; row below: stepper (minus / value / plus) + Remove. Stepper buttons min 36px; consider 48px for primary tap. |
| **Footer** | Subtotal row + Checkout button. | Border above. Subtotal: label left, amount right. Button full-width, min height 48px. |

**Drawer states:**

| State | Content |
|-------|--------|
| **Empty** | Centered message "Your cart is empty"; optional "Continue shopping" (closes drawer). No Checkout CTA or disabled. |
| **Loading** | Centered spinner or skeleton list until cart API responds. |
| **Error** | Message + "Retry" button. |
| **With items** | Line items list + subtotal + Checkout CTA. |

---

### Checkout step (inside drawer)

- **Option A:** Same drawer; body content switches to checkout (order summary + delivery/collection + Place order).
- **Option B:** Expandable section below cart list (summary + delivery + Place order).

**Checkout content:**

- **Order summary:** Read-only list (name × qty, price per line); subtotal.
- **Delivery / collection:** Chips or options (e.g. School pickup, Home delivery); address/contact if needed; pre-fill for logged-in student.
- **Place order:** Primary button; loading state until API response; then success message and close or reset drawer.

---

## Wireframe / mockup reference

Visual wireframes are **standalone HTML files** in this folder (open in browser). They were created as layout references; the cursor-ide-browser MCP canvas tool was not available in this environment.

| File | Content |
|------|--------|
| `wireframes/kyc-orders-catalog-wireframe.html` | Catalog: app bar (title + cart icon with badge), category chips, 2-column product grid (image, name, price, Add to cart). Mobile-first (max-width 390px). |
| `wireframes/kyc-orders-cart-drawer-wireframe.html` | (1) Cart drawer with header, two line items (thumbnail, name, price, qty stepper, remove), subtotal, Checkout. (2) Checkout view: order summary list, delivery options, Place order. |

**Aesthetic:** Clean school/app style. Outfit + Instrument Sans; primary blue; 8px grid; 8/12px radius; clear hierarchy. Open the HTML files in a browser to inspect layout and spacing.

---

## Responsive notes

| Breakpoint | Catalog | Drawer |
|------------|--------|--------|
| **Mobile (default)** | 2-column grid; category strip horizontal scroll. | Width 85vw max 400px; full height. |
| **Tablet** | 3 columns for product grid; category strip can wrap or stay single row. | Same drawer width (e.g. max 400px); centered or right-aligned. |
| **Desktop** | 3–4 columns; more padding. | Drawer max-width 400px; overlay from right. |

Catalog and drawer do not change structure; only grid columns and optional padding scale.

---

## Accessibility

- **Touch targets:** All tappable elements (cart icon, category chips, Add to cart, drawer close, stepper, remove, Checkout, Place order) minimum 48×48px (or 48px on shortest side).
- **Semantic structure:** Use `<header>`, `<nav>`, `<main>`, `<footer>`; heading hierarchy (e.g. one h1 "Orders", drawer title as h2). List of line items as list or group with `role="list"` and item `role="listitem"` if not native list.
- **Focus order (drawer):** When drawer opens, focus moves to drawer (e.g. close button or first focusable). Focus trapped inside drawer until close. On close, focus returns to cart trigger. Ensure close button and Checkout/Place order are reachable by keyboard.
- **Labels:** Cart icon: `aria-label="Open cart (n items)"`. Close: `aria-label="Close cart"`. Stepper: `aria-label="Decrease quantity"` / `Increase quantity`. Buttons and links have visible or accessible names.
- **Loading / errors:** Announce loading and error states to screen readers (e.g. `aria-live` region or status text).

---

## Summary

- **Catalog:** App bar + category strip + 2-col product grid; mobile-first; 16px padding/gap; 48px touch targets.
- **Cart drawer:** 85vw / 400px max; header, scrollable body (line items), footer (subtotal + Checkout); empty/loading/error states.
- **Checkout:** Second view in same drawer; order summary + delivery/collection + Place order.
- **Wireframes:** `wireframes/kyc-orders-catalog-wireframe.html`, `wireframes/kyc-orders-cart-drawer-wireframe.html`.
- **Responsive:** Grid 2→3(→4) columns; drawer width fixed max 400px.
- **Accessibility:** 48px targets, semantics, focus trap and return, ARIA labels.
