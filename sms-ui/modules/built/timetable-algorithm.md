---
title: Timetable Generation Algorithm
type: module
project: sms-ui
module: Schedule Management
last-updated: 2026-03-18
---

# Timetable generation algorithm

How the CSP-based timetable generator works, how it maximises staff usage, and use cases.

---

## 1. Overview

The generator assigns **periods** (subject + teacher) to **(day, period)** slots for one class–section per run. It:

- Respects **template** (working days, periods per day, full/half days).
- Pre-fills **fixed-slot constraints** (e.g. "Sports on Thu P5"), then fills the rest with a **backtracking CSP** and heuristics.
- Avoids **cross-section double-booking** (same teacher, same day/period in another section).
- Prefers **compact teacher days** (gap-filling) to maximise staff usage.
- Retries up to **5 times** with **shuffled placement order** when the first attempt fails.

---

## 2. Inputs and data flow

| Input | Source | Use |
|-------|--------|-----|
| Class, section, template | Request | Slots = working days × periods per day (full/half). |
| Subject periods | UI `subject_periods` or backend subject hours | Periods per week per subject. |
| Teacher allotment | Class–section–subject–teacher | One teacher per subject (or option). |
| Fixed-slot constraints | Request (optional) | Pinned (day, period) → subject; expanded then pre-filled. |
| Occupied teacher slots | Sibling timetables (draft + published) | Set of (teacher_id, day, period) already used in other sections. |

**Flow:**

1. Load template → eligible slots (list of (day, period)).
2. Load variables: one per subject (or subject-option), with `periods`, `teacher_id`.
3. Expand fixed constraints → fixed assignment + conflicts (if any). Deduct fixed periods from variables; remove used slots.
4. Build placement units (see §4). Optionally shuffle units.
5. Backtrack: for each unit, try slots in heuristic order; if all placed, return fixed + CSP result.
6. On failure, retry from step 4 with new shuffle (up to 5 attempts).
7. Return (all slot dicts, fixed_constraint_conflicts).

---

## 3. Hard constraints

- **One assignment per (day, period)** in this section.
- **One teacher per (day, period)** in this section (no double booking within section).
- **Cross-section:** Teacher must not be in `occupied_teacher_slots` for (day, period). So the same teacher is never placed in two sections at the same (day, period).
- **Max periods per day per subject** (default 3) to avoid one subject dominating a day.
- **Optional groups:** All subjects in the same optional group share a single slot (one period among them).

Invalid input (e.g. subject not in class, day not working) still raises before generation.

---

## 4. Placement order (hierarchy)

Variables are ordered before placement:

1. **Mandatory subjects first** (subject_type = mandatory).
2. **By periods descending** (high-load subjects before low-load). This gives more slot choices to subjects that need many periods.
3. **Optional groups** last; each group is one unit (one slot shared by the group).

Order is implemented in `_order_variables` and `_build_placement_units`. This order is then optionally **shuffled** when `shuffle_units=True` (used in the retry loop) so that a different subject order is tried when the first order fails.

---

## 5. Slot ordering (spread + teacher gap-filling)

For each subject (or unit), when choosing which **(day, period)** to try next, slots are **scored and sorted** in `_slots_for_spread` / `_slots_for_variable`:

| Priority | Criterion | Effect |
|----------|-----------|--------|
| 0 | **Gap-filling** | Prefer slots where the **teacher** already has an adjacent period on the **same day** (period−1 or period+1). This clusters a teacher's classes on fewer days and maximises staff usage. |
| 1 | **No consecutive same subject** | Avoid placing the same subject in two consecutive periods on the same day (soft preference). |
| 2 | **Spread across days** | Prefer days that have not yet been used for this subject in this run (underused days first). |
| 3 | **Fewer unique subjects on day** | Prefer days with fewer distinct subjects already placed (balance load). |
| 4 | **Earlier period** | Prefer lower period number within the day. |

Gap-filling uses a **teacher_occupied** set: (day, period) where this teacher is already placed (in fixed + current assignment) or in `occupied_teacher_slots`. A slot (d, p) "fills a gap" if (d, p−1) or (d, p+1) is in that set.

---

## 6. Cross-section teacher capacity

- **Source:** Before generate/regenerate, the backend builds `occupied_teacher_slots` from **sibling sections** (same class, other sections): all (teacher_id, day, period) from their draft and published timetables.
- **Use:** In `can_place_single` / `can_place_group`, a candidate (day, period) for a teacher is rejected if (teacher_id, day, period) is in `base_occupied` or `local_occupied`.
- **Effect:** No teacher is double-booked across sections at the same (day, period). Regenerating one section still respects the others.

---

## 7. Shuffle and retry

- **Units** = list of placement units (each unit is one subject or one optional group).
- **Shuffle:** Before backtracking, `random.shuffle(units)` when `shuffle_units=True`.
- **Retry:** The main `generate()` loop runs up to **5** attempts. Each attempt calls `_assign_slots(..., shuffle_units=True)`. Different shuffle can make an otherwise infeasible instance feasible (e.g. when one subject order blocks another).
- **Failure:** If all attempts fail, a 422 is raised with a message (e.g. teacher has no free period, or could not place all subjects).

---

## 8. Fixed-slot constraints and conflicts

- Fixed constraints are **expanded** to concrete (day, period) slots (with "last period" resolved per day).
- **Duplicate slot:** If two constraints request the same (day, period), the **first** wins; the second is **skipped** and a conflict is recorded (`fixed_constraint_duplicate`).
- **Exceed periods:** If a subject has more fixed slots than its period allowance, only the first N (by iteration) are placed; the rest are **skipped** and conflicts recorded (`fixed_constraint_exceed_periods`).
- Skipped slots remain **available** for the CSP. Generation always proceeds; conflicts are returned in `conflict_summary` for the UI.

---

## 9. How staff usage is maximised

1. **Gap-filling (compact days)**  
   By preferring slots where the teacher already has an adjacent period on the same day, the algorithm clusters that teacher's classes on fewer days. That leaves other days free for the same teacher in **other sections**, so more sections can be timetabled without increasing staff.

2. **Cross-section awareness**  
   Using `occupied_teacher_slots` ensures no double-booking across sections. The generator only uses (day, period) where the teacher is free **globally** (for that class). So staff capacity is used consistently across sections.

3. **Retry with shuffled order**  
   Different subject order can yield a feasible assignment when the first order does not. So the same staff can often be fully utilised instead of failing with "teacher has no free period".

4. **High-load first**  
   Placing high-period subjects first gives them more freedom to choose slots; low-period subjects fill the gaps. That tends to reduce fragmentation and make better use of each teacher's free slots.

---

## 10. Use cases

### 10.1 Single section, no fixed constraints

- **Input:** Class X, Section A, template (e.g. Mon–Sat, 8 periods/day), subject_periods from UI or subject hours.
- **Flow:** All slots are free. Units are ordered (mandatory first, then by periods descending, then optional). Each unit is placed in heuristic slot order (gap-filling, spread, no consecutive same subject). Cross-section set is from other sections of Class X (if any).
- **Outcome:** A conflict-free timetable for Section A; teachers avoid double-booking with other sections.

### 10.2 Multiple sections, same class (sibling sections)

- **Input:** Generate Section A first; then generate Section B (same class).
- **Flow:** For Section B, `occupied_teacher_slots` includes all (teacher, day, period) from Section A's timetable. Placement for B rejects any slot where the teacher is already in A.
- **Outcome:** No teacher teaches two sections at the same (day, period). Gap-filling in B still prefers compact days for each teacher given their existing load in A.

### 10.3 Fixed constraints with duplicate slot

- **Input:** Two rows: "Sports on Mon, Wed, Fri, last period" and "Drawing on Wed, last period". Wednesday last period is requested twice.
- **Flow:** First constraint (Sports) gets Wed last period. Second (Drawing) sees slot used → skipped, conflict `fixed_constraint_duplicate` recorded. Drawing's other slots (if any) are placed. CSP fills remaining slots.
- **Outcome:** Timetable is generated; conflict summary shows the duplicate so the user can fix constraints.

### 10.4 Fixed constraints exceed subject periods

- **Input:** Subject "Art" has 2 periods/week; user adds fixed constraints that request Art in 3 slots.
- **Flow:** First two fixed slots get Art; the third is skipped and conflict `fixed_constraint_exceed_periods` recorded. CSP places remaining subjects.
- **Outcome:** Timetable generated; user sees that one fixed Art slot was dropped.

### 10.5 First attempt fails; retry succeeds

- **Input:** Many subjects, tight template; with order [Math, Science, …] the backtracker cannot place Science (teacher has no free period after Math takes "good" slots).
- **Flow:** Attempt 1 fails. Attempt 2 shuffles units to e.g. [Science, …, Math, …]. Science places first and uses some slots; Math fits in the rest. Success.
- **Outcome:** Timetable generated on retry; staff usage improved by different placement order.

### 10.6 Regenerate after teacher allotment change

- **Input:** User changes teacher for a subject, then clicks Regenerate.
- **Flow:** Same as generate: stored options (subject_periods, fixed_slot_constraints) + current allotment; `occupied_teacher_slots` from sibling sections. New slots produced; draft updated.
- **Outcome:** Timetable reflects new teacher without cross-section conflicts.

---

## 11. Summary

| Mechanism | Purpose |
|-----------|---------|
| Placement order (mandatory → high-load → optional) | Place constrained / high-load subjects first so they get better slot choices. |
| Slot scoring (gap-fill, spread, no consecutive) | Prefer compact teacher days and spread subjects across the week. |
| Cross-section occupied slots | Prevent double-booking; maximise usable teacher capacity across sections. |
| Shuffle + retry (5 attempts) | Find a feasible assignment when one exists under a different subject order. |
| Fixed-constraint conflicts | Proceed with generation and report conflicts so the user can correct constraints. |

Together, these make the algorithm **respect hard constraints**, **prefer compact teacher schedules** (gap-filling), **use cross-section information** to maximise staff usage, and **improve feasibility** via retries.
