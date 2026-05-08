---
title: "Orders & Inventory API"
type: planning-stub
project: sms-api
module: Inventory & Orders
last-updated: 2026-04-08
---

> Auto-promoted to testing on 2026-04-08 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

## Status

**Needs spec** — No orders or inventory endpoints exist in the current SMS-API. SMS-UI (staff manages catalogue and fulfils orders) and KYC (parent browses and orders) both depend on this.

## What this API needs to cover

### Product Catalogue
- `POST /inventory/products` — Create product (name, category, price, stock, image, school)
- `GET /inventory/products` — List products (filter by category, in-stock, school)
- `GET /inventory/products/{id}` — Product detail
- `PUT /inventory/products/{id}` — Update product
- `DELETE /inventory/products/{id}` — Remove product

### Categories
- `POST /inventory/categories` — Create category
- `GET /inventory/categories` — List categories for school
- `PUT /inventory/categories/{id}` — Update
- `DELETE /inventory/categories/{id}` — Delete

### Stock Management
- `PUT /inventory/products/{id}/stock` — Adjust stock (add / reduce with reason)
- `GET /inventory/products/{id}/stock/history` — Stock movement history

### Cart (KYC session-based)
- `POST /orders/cart` — Create/update cart (items: [{productId, qty}])
- `GET /orders/cart/{sessionId}` — Get cart
- `DELETE /orders/cart/{sessionId}` — Clear cart

### Orders
- `POST /orders` — Place order (from cart, studentId, paymentMethod)
- `GET /orders` — List orders (staff: all for school; parent: their orders)
- `GET /orders/{id}` — Order detail (items, status, payment, student)
- `PUT /orders/{id}/status` — Update fulfilment status (CONFIRMED → READY → COLLECTED / CANCELLED)

### Reports
- `GET /inventory/reports/stock` — Current stock levels per product
- `GET /orders/reports/summary` — Orders by status, revenue, period

## Key decisions to make before writing spec

- Is the school library tracked through this same inventory system, or separate?
- Can a parent pay for orders online (same payment gateway as fees), or cash-on-collection only?
- Stock reservation on order placement (or first-come-first-served at collection)?

## Related docs

- SMS-UI stub: `sms-ui/modules/needs-spec/inventory-orders`
- KYC spec: `/doc/kyc/specs/built/online-orders-spec`
- KYC library spec: `/doc/kyc/specs/built/library-spec`
- Fee Management API stub (payment gateway shared): `sms-api/modules/testing/fee-management-api`
