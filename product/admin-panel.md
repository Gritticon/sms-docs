---
title: Gritticon Admin Panel
type: product
project: platform
last-updated: 2026-03-18
---

# Gritticon Admin Panel

The Gritticon Admin Panel is an internal tool used exclusively by the Gritticon team to manage all schools on the platform.

**Status:** In use (Vite + React admin-panel + sms-api internal employee auth).

---

## Purpose
Currently all school onboarding and management is done manually. The admin panel will replace and streamline these manual processes as the number of schools scales.

---

## Scope

### 0. Internal Employee Account Security
Manage internal employee accounts and credentials.

**Capabilities:**
- Internal employee CRUD (Super Admins)
- Change own password (requires current password)
- Admin override: employee id `1` can reset other internal employees' passwords
- Logout reliably returns to login page

### 1. School Onboarding
Create and configure a new school on the platform.

**Capabilities:**
- Create a new school record (generates `school_id`, initialises dynamic tables)
- Set school status: `TRIAL`, `ACTIVE`, `INACTIVE`, `SUSPENDED`
- Upload school branding (logo)
- Set initial admin staff account credentials
- Configure initial settings (academic year, modules enabled)
- Transition school through lifecycle: `TRIAL → ACTIVE → INACTIVE / SUSPENDED`

---

### 2. Billing
Track and manage financial relationship with each school.

**Capabilities:**
- View billing status per school
- Record payments and subscription details
- Set billing period (monthly / yearly)
- Flag overdue accounts
- Trigger status changes based on billing (e.g. suspend on non-payment)

*Note: Specific pricing model is TBD.*

---

### 3. Usage Monitoring
Understand how schools are using the platform.

**Capabilities:**
- Active users per school
- Login activity and session data
- Module usage (which modules are being used and how frequently)
- Data storage per school
- API usage metrics

*Useful for identifying underused features, performance issues, and high-growth schools.*

---

### 4. Feature Flags
Control which features are available to each school.

**Capabilities:**
- Enable or disable specific SMS/KYC modules per school
- Roll out new features to selected schools first (beta testing)
- Override school-level settings when needed for support purposes
- Enable optional features (e.g. online payments) per school

---

### 5. Support Tickets
Handle support requests from schools.

**Capabilities:**
- View incoming support tickets
- Assign tickets to team members
- Track ticket status (open, in progress, resolved)
- Communication trail per ticket
- Link tickets to school accounts for context

---

## Access
- Accessible only to Gritticon internal employees
- Uses `internal_employee` login type (cross-school access)
- Protected by the same JWT-based auth system

---

## Development Approach
Consistent with the overall Gritticon philosophy:
- Manual processes will be documented and proven first
- The admin panel will then automate and streamline what is already working
- Start with the most critical workflow (onboarding) and expand from there
