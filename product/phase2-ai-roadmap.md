---
title: Phase 2 AI Roadmap
type: product
project: platform
last-updated: 2026-03-18
---

# Phase 2 — AI Roadmap

Phase 2 begins after Phase 1 is stable and 4–5 schools are using the platform without issues. The goal is to use real data and observed pain points from Phase 1 to introduce meaningful AI-driven features — not speculative ones.

---

## Philosophy
> "Analyse and understand the pain points, then automate and solve with AI."

AI features will not be built in advance of the problem. Phase 1 usage will reveal what schools genuinely struggle with. Phase 2 then addresses those pain points intelligently.

All AI features are designed to be **configurable and prioritisable per school** — schools can opt in/out of features, adjust weights, and set preferences based on their own needs and feedback.

---

## Planned AI Features

### 1. Personalised Student Learning (LLM)
An AI learning companion integrated into KYC, inspired by tools like Google NotebookLM.

**Problem it solves:** Students learn differently — some prefer reading, others prefer listening or watching.

**How it works:**
- Student selects a subject or topic
- AI generates personalised learning content in the student's preferred format:
  - **Audio** — spoken explanation or lecture-style audio
  - **Video** — visual walkthrough or animated explanation
  - **Podcast** — conversational, story-driven format for the topic
- Content is generated based on the school's curriculum and syllabus data already in SMS

**Configurable per school:** Schools can enable/disable this feature and define which subjects or classes have access.

---

### 2. Finance Audit LLM
An AI assistant for financial oversight within SMS.

**Problem it solves:** Manual review of fee collections, payments, and financial flows is time-consuming and error-prone.

**How it works:**
- AI monitors and analyses financial inflows and outflows in real time
- Flags anomalies, discrepancies, or unusual patterns
- Generates audit summaries and reports on demand
- Answers natural language queries (e.g. "Which students have not paid Term 2 fees?")

**Configurable per school:** Sensitivity thresholds and alert types can be adjusted by the school.

---

### 3. Academic Performance Analytics
AI-driven insights into student and class performance trends.

**Problem it solves:** Identifying struggling students or underperforming classes requires manual analysis of large datasets.

**How it works:**
- Analyses marks, attendance, and diary data across students and classes
- Identifies individual students at risk of falling behind
- Highlights subjects or periods with consistently low performance
- Generates teacher-facing and management-facing summaries
- Trend analysis over time (term-on-term, year-on-year)

**Configurable per school:** Schools can set performance thresholds and define what constitutes "at risk."

---

### 4. Dropout Prediction
Predict which students are at risk of disengaging or dropping out.

**Problem it solves:** Schools often identify at-risk students too late to intervene effectively.

**How it works:**
- Combines signals: attendance patterns, academic performance, complaint history, fee payment delays, engagement with KYC
- Assigns a risk score per student
- Alerts the relevant staff (class teacher, counsellor, management) early
- Suggests interventions based on the identified risk factors

**Configurable per school:** Risk weighting can be adjusted based on school type, location, and historical patterns. Schools can prioritise which signals matter most to them.

---

## Implementation Approach

- AI features will be introduced **incrementally** — one feature at a time, starting with the most impactful
- Each feature will be validated with pilot schools before broad rollout
- Models and outputs will be **explainable** — school staff should understand why the AI flagged something
- All AI features are **optional** — schools choose what to enable
- Phase 2 features will be surfaced as additional modules in SMS and KYC, gated by feature flags in the Gritticon Admin Panel

---

## Data Foundation
Phase 1 data collection is the foundation for Phase 2:
- Attendance records
- Exam marks and academic records
- Fee payment history
- Diary entries and homework completion
- Complaint history
- KYC engagement data (what parents and students are checking)

The more consistently schools use Phase 1, the more accurate Phase 2 AI becomes.
