---
title: KYC Module Specifications
type: product
project: platform
last-updated: 2026-03-18
---

# KYC — Module Specifications

KYC (Know Your Child) is the parent and student-facing application. It is a responsive Flutter web application, accessible via iframe on the school's website, and wrapped inside a school-branded mobile app.

Both parents and students use the same application. Parents can manage multiple children within a single account.

---

## Module Status Overview

| Module | UI Status | API Status |
|--------|-----------|------------|
| Authentication | Built | Pending |
| Home Feed | Built | Pending |
| Attendance | Built | Pending |
| Marks / Exams | Built | Pending |
| Timetable | Built | Pending |
| Diary | Built | Pending |
| Fees | Built | Pending |
| Orders | Built | Pending |
| Transport | Stub | Pending |
| Notifications | Stub | Pending |
| Messages | Stub | Pending |
| Complaints | Stub | Pending |
| Documents | Stub | Pending |
| Library | Stub | Pending |
| Reminders | Built | Pending |
| Student Profile | Stub | Pending |
| Account | Built | Pending |

---

## Navigation Structure

```
/school                    → School Selection
/:schoolId/login           → Login
/:schoolId/
├── /home                  → Home Feed
├── /attendance            → Attendance & Analytics
├── /marks                 → Exam Marks & Results
│   └── /marks/detail      → Individual Exam Detail
├── /timetable             → Class Timetable
├── /diary                 → Class Diary & Homework
├── /fees                  → Fees & Payments
│   ├── /fees/invoice      → Invoice Detail
│   └── /fees/result       → Payment Result
├── /orders                → School Shop / Orders
├── /transport             → Transport / Bus Tracking
├── /notifications         → Notifications
├── /messages              → Messages
├── /complaints            → Complaints
├── /documents             → Documents
├── /library               → Library
├── /reminders             → Reminders
├── /profile               → Student Profile
└── /account               → Account & Settings
```

---

## Authentication

### School Selection
- User enters their school ID to identify which school they belong to
- Loads school branding (logo, color)
- Animated entry screen

### Login
- Student ID + Password authentication
- Role-based: Student or Parent
- Error handling and loading states

### Session & Multi-Child Support
- Session stores: `schoolId`, `role`, `children list`, `activeChildIndex`
- Parents can switch between their children from the AppBar or sidebar
- All modules reload data for the selected child on switch

---

## Home Feed
Posts and events from the school's Communication Hub in SMS.

- Instagram-style feed of school posts (pictures, key notes, event updates)
- Posts are created and managed by staff via SMS Communication Hub
- Responsive layout: single column on mobile, wider layout on desktop/web

---

## Attendance
**Source:** Teacher-marked attendance from SMS (Teacher Diary module)

**Features:**
- Monthly / period selector
- Summary analytics block: present count, absent count, percentage, school target percentage
- Calendar view of attendance by date
- Drill down into a specific day: subject-wise attendance breakdown
- Alert for low attendance (below school target)
- Error handling and retry

---

## Marks / Exams
**Source:** Academic records from SMS

**Features:**
- Term selector dropdown
- List of exams with overall performance indicator and publish status
- Drill into an exam: subject-wise marks (obtained / max / grade / remark)
- Pending results handled gracefully
- Empty state when no data

---

## Timetable
**Source:** Timetable from SMS

**Features:**
- Day view — shows periods and breaks for a selected day
- Week view — full weekly schedule
- Month view — calendar with holidays
- Period details: subject, teacher, room
- Day picker for navigation
- View mode toggle in AppBar (Day / Week / Month)

---

## Diary
**Source:** Teacher Diary entries from SMS (homework and progress notes)

**Features:**
- Day picker for date selection
- Period-wise tabs
- Per period: 3 sections
  1. Prerequisites / Planned content
  2. Completed content / Progress notes
  3. Homework assignments
- Homework items show: title, description, due date, attachments
- Attachment links (files uploaded by teacher in SMS)

---

## Fees
**Source:** Fee Management module in SMS

**Features:**
- **Dues tab:** Outstanding fees with status (upcoming, overdue, partially paid)
- Fee summary: total due, total overdue
- Multi-select fees for payment
- Sticky payment action bar
- **Invoices & History tab:** Past payments with receipts
- Invoice detail: amount, date, items covered, transaction reference

### Online Payments
- Optional — schools can enable or disable via School Management settings in SMS
- Payment gateway: TBD
- Payment result screen: success / failed / pending with transaction reference
- Receipt download

---

## Orders
**Source:** Inventory Management module in SMS

A school shop where students/parents can purchase school items.

**Features:**
- Category filter chips: Bags, Books, Uniform, Event Passes, Other
- Responsive product grid (2–6 columns based on screen width)
- Product cards: image, name, price, availability status
- Add to cart
- Cart drawer with overlay
- Quantity stepper and item removal
- Delivery option: School Pickup or Home Delivery
- Checkout flow
- Order confirmation

**Backend flow:** Orders placed in KYC are managed and fulfilled via the Order Management module in SMS. Inventory is deducted on fulfilment.

---

## Transport
**Source:** Transport Management module in SMS

- Shows student's assigned bus/route
- Transport tracking (planned for future)
- Empty state when no transport assigned

---

## Notifications
**Source:** Communication Hub in SMS

- School announcements and broadcast messages
- Card-based list layout

---

## Messages
**Source:** Communication Hub in SMS

- Messages from teachers and administration
- Sender identification and subject display

---

## Complaints
- Text input for complaint description
- Submit to SMS Complaints module
- Status tracking (pending, in progress, resolved)

---

## Documents
**Source:** Student Documents module in SMS

- List of school documents (report cards, circulars, etc.)
- Document type display
- Download button

---

## Library
**Source:** Inventory Management (Library Books) in SMS

- Borrowed books list
- Due date display
- Overdue indicator (color coded)
- Book title and type display

---

## Reminders
- Fee payment reminders (upcoming and overdue)
- Event reminders
- Transport fee reminders

---

## Student Profile
**Source:** Student Management module in SMS

- Read-only student information
- Name, class, section, admission details

---

## Account
- Active child display and switcher (multi-child parents)
- Logout
- Account settings

---

## Responsive Design

| Breakpoint | Layout |
|-----------|--------|
| < 600px (Mobile) | Bottom navigation, single column, full-width cards |
| ≥ 600px (Tablet / Web) | Claymorphic sidebar rail (260px), multi-column grids |

Navigation adapts automatically — bottom tabs on mobile, sidebar on wider screens.

---

## Design System
- **Style:** Claymorphism with Material Design 3
- **Primary Color:** Configured per school via KYC Settings in SMS
- **Font:** Plus Jakarta Sans (Google Fonts)
- **Icons:** Phosphor Flutter
- **Animations:** Flutter Animate (fade, slide, scale)
- **Loading States:** Shimmer skeletons and circular progress indicators
