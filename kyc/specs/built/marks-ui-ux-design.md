---
title: Marks UI UX Design
type: spec
project: kyc
module: Marks
last-updated: 2026-03-18
---

> **Built snapshot** — Marks UI implemented. See **marks-ui-build-outline.md** in this folder.

# Marks UI — UX Design Note

Design-time UX guidance for the KYC Marks (markbooks) screen. Audience: **students and parents** (view-only; data for active student/selected child). Aligns with [marks-markbooks-spec.md](marks-markbooks-spec.md), [marks-ui-build-outline.md](marks-ui-build-outline.md), and KYC UI expectations (period picker, AppCard, theme colors, 48px touch targets, loading from request start until response).

---

## 1. Structure

**Order of blocks (top to bottom) — selector then list then detail:**

1. **Period selector** — Term/academic period at top (same pattern as Diary/Attendance: `[←] [Period label] [→]`; tap opens period picker). Default: current academic period. Min 48px touch targets.
2. **Optional summary block** — When API provides period-level aggregate (e.g. "Overall this term"), one compact AppCard below selector. Omit if API does not provide.
3. **Exam list** — One row/card per exam: name + overall indicator (e.g. percentage or grade). User sees "what exams do I have?" before drilling into one.
4. **Exam detail (drill-down)** — Tap exam → subject-wise marks: overall summary (when provided) then subject list (subject, obtained, max, grade/remark). "Marks not yet published" when API indicates so.
5. **Areas of concern** — When API flags failing or concerning performance, show as label/icon + text in exam detail (or badge on list row); not color-only.

**Rationale:** Period first keeps pattern consistent with Diary/Attendance. Exam list before detail lets users choose which exam to open. Summary only when API supports it avoids fake logic in Flutter.

**Wide layouts:** Same order; content can use max width (~600–720px) for readability.

---

## 2. Exam list & optional visualization

All content from API; no business logic in Flutter.

### 2.1 Exam list (primary)

- **Pattern:** List of AppCards or list tiles. Each row: exam name + overall indicator (e.g. "82%" or "Grade A") when provided by API.
- **Interaction:** Tap row → exam detail. Min 48px touch target per row.
- **Accessibility:** Each row has semantic label (e.g. "Unit Test 1, 82 percent"). Not color-only; use text/labels for meaning.

### 2.2 Optional: bar or simple chart (if product wants)

- **When:** Only if backend provides multiple exams with comparable overall values and product explicitly wants a visual comparison.
- **Pattern:** Compact bar chart (e.g. one bar per exam, height = percentage or grade index). Axes or bars must have text labels (e.g. exam name under each bar) so meaning is not color-only. Prefer list-first; chart as optional supplement above or beside list.
- **Placement:** Above exam list in same scroll, or in optional summary block. Use theme colors; semanticLabel for chart (e.g. "Exams comparison, Unit Test 1 82 percent, Mid Term 76 percent").

### 2.3 Areas of concern in list

- **When:** API flags an exam or subject as concerning (e.g. failing).
- **Pattern:** Small badge or icon + short text (e.g. "Needs attention") on the exam row or in detail. Use theme warning/error surface; ensure meaning is clear without color (icon + text).

---

## 3. Exam detail (subject-wise marks)

- **Entry:** From exam list tap.
- **Layout:** AppBar with back; title = exam name. Below: overall summary line when API provides (total, %, grade). Then subject-wise list or table: columns/labels "Subject", "Marks obtained", "Maximum", "Grade" (and "Remarks" if API provides).
- **Not published:** If backend says marks not yet published, show single message (e.g. "Marks not yet published") and no table. No empty table.
- **Partial data:** When some subjects are graded and others pending, show pending with label (e.g. "—" or "Pending") so users are not misled.
- **Accessibility:** Table or list has semantic labels; 48px min touch if rows are tappable; headers associated with cells. Summary line announced (e.g. "Overall 82 percent").

---

## 4. States

### Loading

- **Scope:** Content area (period selector can stay interactive). Show **skeleton or overlay** from request start until response (per project rules).
- **Exam list:** Skeleton rows matching list layout; no fake percentages or grades.
- **Exam detail:** Skeleton for summary + subject rows. Do not show partial or stale marks.

### Empty

- **No exams for period:** "No exams for this period" or "No marks available yet" in content area. No misleading indicators (e.g. no 0% bar).
- **New student / pre-exam:** Same empty message; optional short context (e.g. "Marks will appear here once published by the school").

### Not published

- **When:** User opens an exam whose marks are not yet published (per API).
- **Behavior:** Single clear message: "Marks not yet published". No empty table; no fake data.

### Error

- **Pattern:** AppMessage + **Retry**. Keep period selector and nav visible; content area shows error state. No progress or marks with fake data.

---

## 5. Copy & labels (student/parent-facing)

- Screen title: **Marks** or **Marks & Markbooks**.
- Period: "Term 1", "Semester 1", or API-supplied period label.
- Exam list: Exam name; "Overall: 82%" or "Grade A" (as from API).
- Detail: "Subject", "Marks obtained", "Maximum", "Grade", "Remarks"; "Overall" for summary.
- Not published: "Marks not yet published".
- Empty: "No marks available yet", "No exams for this period".
- Concern: Short text (e.g. "Needs attention", "Below passing") with icon; not color-only.

---

## 6. Accessibility

- **Focus order:** Period selector → optional summary → exam list (row by row) → exam detail (summary then subject rows). Logical and predictable.
- **Period selector:** "Previous period", "Current period Term 1", "Next period"; min 48px targets.
- **Exam list:** Each row: exam name + value announced (e.g. "Unit Test 1, 82 percent"). No meaning by color alone.
- **Charts (if any):** semanticLabel describing key values (e.g. "Exams: Unit Test 1 82 percent, Mid Term 76 percent"). Bar labels for screen readers.
- **Exam detail:** Summary line and table/list have proper semantics; headers associated with data cells.
- **Areas of concern:** Icon has semanticLabel; text visible so "Needs attention" is clear without color.
- **Touch:** Min 48×48px for period arrows, period label, exam rows, back, retry.
- **Loading/empty/error:** "Loading marks", "No marks available yet", "Something went wrong. Try again." announced.

---

## Summary checklist (for implementation)

- [x] Structure: Period selector → optional summary → exam list → exam detail (subject-wise).
- [x] Period selector: same pattern as Diary/Attendance; default current period; 48px targets.
- [x] Exam list: name + overall indicator per exam; tap → detail; no fake data during load.
- [x] Exam detail: overall summary (when API provides) + subject table/list; "Marks not yet published" when applicable.
- [ ] Areas of concern: label/icon + text; not color-only (P1).
- [x] Loading: skeleton/overlay for content; no partial or stale marks/percentages.
- [x] Empty: "No marks available yet" / "No exams for this period".
- [x] Error: message + retry.
- [x] Accessibility: focus order, semantic labels, 48px targets, contrast, non–color-only meaning.

This structure is testable against the acceptance criteria in [marks-ui-pm-addendum.md](marks-ui-pm-addendum.md).
