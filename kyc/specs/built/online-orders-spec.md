---
title: Online Orders Spec
type: spec
project: kyc
module: Online Orders
last-updated: 2026-03-18
---

> **Built snapshot** — 2025-03-03 — UI implemented; API integration may be pending.

## Online Orders (Books, Uniform, etc.) — Specification

## 1. Summary
The Online Orders module allows parents (and optionally senior students, if permitted by the school) to order school-related items such as textbooks, uniforms, and accessories. It provides a simple catalog browsing and ordering experience while delegating inventory, pricing, and order fulfillment logic to the SMS backend.

The goal is to streamline common school purchases and reduce manual order forms and in-person visits.

## 2. Atomic Requirements

### 2.1 Functional
| ID | Requirement | Priority |
|----|-------------|----------|
| F1 | System shall display a catalog of available items categorized by type (e.g., books, uniforms, others) for the selected school and, where relevant, for the active child’s class/section. | P0 |
| F2 | System shall allow users to view item details (name, description, price, size/variant options if any, and any restrictions) as provided by the backend. | P0 |
| F3 | System shall allow users to add items to a simple cart or selection list, specifying quantities and required variants (e.g., size). | P0 |
| F4 | System shall allow users to review their cart and see a summary of items, quantities, and total cost before placing an order. | P0 |
| F5 | System shall place the order via SMS API and display confirmation of order status (e.g., placed, pending, failed) based on backend response. | P0 |
| F6 | System shall show an order history list for the active parent/child context, including item summaries, dates, and statuses (e.g., pending, ready for pickup, delivered). | P1 |
| F7 | System shall support multi-child parents by associating orders with a specific child where relevant (e.g., class-specific books, uniforms). | P1 |

### 2.2 Non-Functional
| ID | Requirement | Priority |
|----|-------------|----------|
| NF1 | Catalog and cart UIs shall be responsive and usable on mobile, tablet, and desktop. | P0 |
| NF2 | Item lists shall load quickly; large catalogs shall be paginated or lazily loaded to avoid performance issues. | P1 |
| NF3 | Pricing and totals shall be displayed clearly, avoiding hidden charges; any additional fees (e.g., delivery) must be explicitly included in backend responses. | P0 |

## 3. User Stories & Acceptance Criteria

### Story 1: Browse school catalog
**As a** parent, **I want** to browse items available from the school **so that** I can purchase required books and uniforms.

**Acceptance criteria:**
- [ ] Given I am logged in and my school has enabled the Online Orders module, when I open the Orders section, then I see a list or grid of available items categorized meaningfully.
- [ ] Given the catalog is large, when I scroll or navigate pages, then items load progressively without freezing the UI.
- [ ] Given some items are restricted to specific classes/sections, when I am viewing the catalog for my child, then I only see items applicable to that child (as enforced by backend rules).

**Requirement IDs:** F1, F2, NF1, NF2

---

### Story 2: Place an order
**As a** parent, **I want** to select items and place an order online **so that** I can avoid manual purchase processes.

**Acceptance criteria:**
- [ ] Given I am viewing an item, when I choose quantity and any required options and add it to the cart, then my cart reflects the updated items and totals.
- [ ] Given I have items in my cart, when I open the cart and review, then I see items, quantities, unit prices, and total price clearly before confirming.
- [ ] Given I confirm the order, when SMS API responds with success, then I see a confirmation screen with an order reference and summary.
- [ ] Given the order fails, when I return from the backend response, then I see a clear failure message and my cart remains editable.

**Requirement IDs:** F2, F3, F4, F5, NF1, NF3

---

### Story 3: Review past orders
**As a** parent, **I want** to see my previous orders **so that** I can track what I have already purchased and order again if needed.

**Acceptance criteria:**
- [ ] Given I have past orders, when I open the Order History view, then I see a list of orders with date, total, and high-level status.
- [ ] Given I select a specific order, when I open its details, then I see items, quantities, and status (e.g., pending, ready for pickup, completed) as provided by the backend.

**Requirement IDs:** F6, F7, NF1

## 4. Dependencies
- **SMS API catalog and order endpoints**: Provide items, pricing, availability, and handle order creation, status updates, and history.
- **Fees/Payments module**: If online payments are required for orders, the Orders module may delegate payment to the same payment flow used by Fees.
- **Class/section configuration**: Determines which items are relevant to which students.

## 5. Risks & Mitigations
| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| Complex inventory or variant logic may be hard to support in a simple UI. | Medium | Medium | Start with basic item and variant support; keep complex rules in SMS and expose only necessary controls to KYC. |
| Ambiguity around order fulfillment (pickup vs delivery). | Medium | Medium | Display fulfillment method and instructions as part of order details using backend-provided descriptions. |

## 6. Edge Cases & Error Handling
- **Empty/zero state:** If the school has not configured any items, show a message such as “No items available for online ordering” instead of an empty screen.
- **Invalid input:** Validate quantity and option selections in the UI and rely on backend for final validation (e.g., stock or eligibility).
- **Failure/offline:** If catalog or order APIs fail, show error messages with retry options and avoid losing cart contents unnecessarily.

## 7. MVP Scope

### In scope
- Basic catalog browsing by category.
- Simple cart and order placement flow.
- Basic order history listing.

### Out of scope (backlog)
- Advanced search and filtering (beyond simple category or text search).
- Real-time inventory indicators in KYC (e.g., live stock counts).
- Complex promotions, coupons, or combined fee+order checkouts.

### Success criteria for MVP
- Parents in pilot schools can successfully order required textbooks/uniforms via KYC with minimal support.
- Reduction in manual order forms or in-person queueing for common school items.

