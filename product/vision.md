---
title: Product Vision
type: product
project: platform
last-updated: 2026-03-18
---

# Product Vision

## Company
**Gritticon** is both the company and the brand behind this product. There is no separate product brand name at this time — Gritticon is what is being built into a brand.

---

## What We Are Building
An end-to-end, white-labeled, multi-tenant **School ERP SaaS platform** for K-12 schools. The platform consists of two interconnected products:

- **SMS (School Management System)** — used by school management and all staff (teaching and non-teaching) to manage, plan, and organise all school operations.
- **KYC (Know Your Child)** — used by parents and students to track data, stay connected with the school, and receive updates faster.

Both products are web applications, white-labeled per school, and accessible via iframe embedded into the school's own website.

---

## Why We Are Building It
Schools today manage operations manually or with fragmented tools. The goal is to:

1. Consolidate all school operations into one unified platform
2. Give parents and students real-time visibility into school life
3. Understand real pain points from actual school usage (Phase 1)
4. Use those insights to introduce AI-driven automation and intelligence (Phase 2)

The approach is deliberately **manual-first** — build the process correctly first, then accelerate with automation. This applies to everything: school onboarding, mobile app builds, deployments, and integrations.

---

## Who Uses It

| User Type | Product | Access |
|-----------|---------|--------|
| School admin & management | SMS | Web (iframe on school website) |
| Teaching staff | SMS | Web (iframe on school website) |
| Non-teaching staff | SMS | Web (iframe on school website) |
| Parents | KYC | Web (iframe) + Mobile app (school-branded) |
| Students | KYC | Web (iframe) + Mobile app (school-branded) |

---

## Core Design Principles

- **Flexibility first** — everything is dynamic and configurable: roles, departments, classes, subjects, timetables, holidays, exams, and KYC settings. Schools structure things the way they want.
- **White-labeled** — every school gets the platform under their own identity (logo, branding).
- **Manual before automated** — processes are proven manually before automation is introduced.
- **Interconnected** — SMS and KYC are separate applications sharing one backend. Actions in SMS reflect in KYC and vice versa.

---

## Phases

### Phase 1 — Core ERP
Build and stabilise the full SMS and KYC platform.

**Phase 1 is considered complete when 4–5 schools are using the software without issues.**

Pending modules to complete before Phase 1 ends:
- Dashboard
- Communication Hub
- Transport Management
- Complaints
- School Management (profile, settings, KYC configuration)
- Fee Management
- Inventory Management
- Order Management
- Audit Logging (dedicated module)
- Teacher Diary (linked to attendance and timetable)

### Phase 2 — AI Integration
Based on real pain points and usage patterns collected during Phase 1, introduce AI features. See `phase2-ai-roadmap.md` for details.

---

## Business Model
To be defined. Currently in development phase — no schools are live yet.

---

## Go-to-Market Approach
- Schools embed SMS and KYC into their own website via iframe
- Each school gets a custom-branded KYC mobile app (built and published manually by Gritticon)
- School onboarding is currently manual; a Gritticon admin panel is planned post-MVP
- Automation of app builds and deployments is planned after processes are proven
