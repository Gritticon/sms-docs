---
title: "Inventory & Orders — SMS-UI"
type: planning-stub
project: sms-ui
module: Inventory & Orders
last-updated: null
---

## Status

**Needs spec** — Feature is in the product roadmap but has no SMS-UI implementation specification. The KYC parent-side shop and order flow is already specced.

## What this feature is

The Inventory & Orders module lets school staff manage the school shop and fulfil orders placed by parents via the KYC app:
- Manage product catalogue (uniform, books, stationery, etc.)
- Set prices and stock levels
- Manage product categories
- View and fulfil incoming orders
- Mark orders as ready / dispatched / collected
- Track inventory levels and get low-stock alerts

The parent-side (browsing, cart, checkout) is already specced in KYC — this stub covers the **staff inventory management and order fulfilment UI**.

## What needs to be planned

- [ ] Product management screen (add/edit product: name, category, price, stock, image)
- [ ] Category management (create/edit product categories)
- [ ] Stock management (adjust stock levels, view history)
- [ ] Orders queue (list incoming orders: student, items, total, status)
- [ ] Order detail view (itemised view, mark status: Confirmed → Ready → Collected)
- [ ] Low-stock alerts (threshold per product, notification to admin)
- [ ] Inventory reports (stock levels, order history, revenue summary)
- [ ] Library module connection (library books tracked as a separate inventory type?)
- [ ] Permission model (who can manage products vs. fulfil orders vs. view-only)

## Related docs

- KYC side spec: `/doc/kyc/specs/built/online-orders-spec`
- KYC orders quick shop design: `/doc/kyc/specs/testing/kyc-orders-quick-shop-design`
- KYC orders design layout: `/doc/kyc/specs/testing/kyc-orders-design-layout`
- KYC library spec: `/doc/kyc/specs/built/library-spec`
- Feature matrix: `/doc/product/feature-matrix`
- API needed: `sms-api/modules/needs-spec/orders-inventory-api`

## Dependencies

- SMS-API Orders & Inventory endpoints (also needs-spec)
- KYC app reads product catalogue and submits orders to SMS-API
- Payment gateway (orders may be paid online via KYC)
