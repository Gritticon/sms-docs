---
title: SMS KYC Integration Points
type: architecture
project: platform
last-updated: 2026-03-18
---

# SMS ↔ KYC Integration Points

SMS and KYC are separate frontend applications but share the same backend (sms-api). Data flows between them through the shared API and database.

---

## Integration Map

| SMS Module | KYC Module | Data Flow | Direction |
|------------|------------|-----------|-----------|
| Teacher Diary | Diary | Homework and progress notes posted by teachers | SMS → KYC |
| Staff Attendance / Teacher Diary | Attendance | Student attendance marked by teachers | SMS → KYC |
| Timetable | Timetable | Published class timetables | SMS → KYC |
| Academic Records | Marks | Student exam marks and grades | SMS → KYC |
| Communication Hub | Home Feed | School posts and event pictures | SMS → KYC |
| Communication Hub | Notifications | Broadcast announcements | SMS → KYC |
| Communication Hub | Messages | Staff-to-parent/student messages | SMS → KYC |
| Fee Management | Fees | Fee structure, dues, payment status | SMS → KYC |
| Fee Management | Fees | Online payments made in KYC | KYC → SMS |
| Inventory Management | Library | Library book borrow/return records | SMS → KYC |
| Inventory Management | Orders | Product catalogue for school shop | SMS → KYC |
| Order Management | Orders | Order fulfilment and status updates | SMS ↔ KYC |
| Transport Management | Transport | Route and bus assignment | SMS → KYC |
| Complaints | Complaints | Complaints submitted by parents/students | KYC → SMS |
| Student Management | Student Profile | Student information | SMS → KYC |
| Student Documents | Documents | Document availability and downloads | SMS → KYC |
| School Management | All KYC modules | Module visibility settings, school color | SMS → KYC |

---

## Detailed Integration Flows

### 1. Teacher Diary → KYC Diary
**Trigger:** Teacher marks attendance and adds homework/notes via the Teacher Diary in SMS

**Flow:**
```
Teacher opens profile in SMS
→ Today's timetable displayed automatically
→ Teacher clicks a subject period
→ If attendance enabled in school settings:
    → Teacher marks student attendance (class/section auto-fetched)
→ Teacher adds homework and progress notes
→ Data saved via sms-api
→ KYC Diary displays homework and notes for that date and period
→ Parents and students can view it in KYC
```

**Key design decision:** Class, section, and period are never manually selected by the teacher — they are auto-derived from the timetable.

---

### 2. Student Attendance → KYC Attendance
**Trigger:** Student attendance marked by teacher via Teacher Diary in SMS

**Flow:**
```
Teacher marks attendance per period/subject
→ Saved to school's attendance records via sms-api
→ KYC Attendance module displays:
    - Monthly calendar view
    - Subject-wise breakdown per day
    - Overall attendance percentage
    - Alert if below school target
→ School management can track attendance for every class and section via SMS
```

---

### 3. Communication Hub → KYC Home Feed
**Trigger:** Staff/admin creates a post (picture, event update, key note) in Communication Hub

**Flow:**
```
Staff creates post in SMS Communication Hub
→ Post saved with school_id
→ KYC Home Feed fetches and displays posts
→ Students and parents see school events and updates
```

---

### 4. Fee Management → KYC Fees
**Bidirectional**

**SMS → KYC:**
```
Admin creates fee structure and assigns fees to students in SMS
→ Due dates and amounts visible in KYC Fees module
→ Overdue fees highlighted
```

**KYC → SMS:**
```
Parent selects fees and initiates payment in KYC
→ Payment processed via payment gateway (if school has enabled online payments)
→ Payment result recorded in sms-api
→ Invoice generated and visible in both KYC and SMS
→ Fee status updated in SMS Fee Management
```

---

### 5. Inventory → KYC Orders / Library
**Orders flow:**
```
Admin adds products to inventory in SMS
→ Products visible in KYC Orders (school shop)
→ Student/parent adds items to cart and places order
→ Order appears in SMS Order Management module
→ Admin processes and fulfils the order
→ Inventory stock deducted on fulfilment
→ Order status updated in KYC
```

**Library flow:**
```
Library books tracked in SMS Inventory
→ Borrow/return managed in SMS
→ KYC Library module shows student's currently borrowed books, due dates, and overdue items
```

---

### 6. School Management → KYC Configuration
**Trigger:** Admin updates KYC settings in SMS School Management module

**Settings that flow into KYC:**
- **School color** — applied as KYC's primary theme color
- **Module visibility** — admin can hide any KYC module the school doesn't use (e.g. if school doesn't use transport, transport tab is hidden)
- **Attendance setting** — if enabled, teacher diary triggers attendance marking before homework
- **Payment setting** — if enabled, KYC shows the payment flow in Fees module

---

### 7. Complaints — KYC → SMS
```
Parent/student submits complaint via KYC
→ Saved via sms-api with school_id and submitter details
→ Appears in SMS Complaints module
→ Admin assigns and resolves
→ Status visible back in KYC
```

---

## Shared Backend
All integration flows go through the same `sms-api`:
- No direct communication between SMS and KYC frontends
- Both read and write to the same school-scoped tables
- Real-time updates reflected by fetching latest data from the API
