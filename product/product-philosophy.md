---
title: Product Philosophy — Gritticon for Indian Schools
type: reference
project: platform
last-updated: 2026-03-29
---

# Product Philosophy — Gritticon for Indian Schools

This document defines the psychological and cultural principles that must guide every product and design decision across all Gritticon modules. Every feature, workflow, and UX pattern should be measured against these principles before implementation.

---

## The Core Insight

> This software is built for Indians. They are happy to do the work themselves — large population is a plus, not a problem. The software should ease their work, not replace it. If it blocks them or removes their role entirely, it feels like a luxury rather than a need.

---

## 8 Psychological Principles

### 1. Control is Identity

In Indian organisations, especially schools, a person's role IS their identity. The accountant who runs payroll, the HR admin who manages leaves — their value comes from being the person who does that work. Full automation doesn't feel like help. It feels like erasure.

**Design rule:** Never remove the human from the loop. Let the system prepare, the human decide.

---

### 2. Trust is Earned Slowly, Lost Instantly

Decades of manual processes have worked — not perfectly, but predictably. A system that silently does something wrong, discovered only on salary day, destroys trust permanently. One bad incident undoes months of adoption.

**Design rule:** Always surface what the system calculated BEFORE it acts. Show the work. Seek confirmation.

---

### 3. The Approval Slip is Cultural, Not Bureaucratic

Indians are deeply comfortable with the approval chain — the "chit" culture. Leave application → coordinator sign → principal stamp. This is not inefficiency. It is how accountability is distributed in a hierarchical culture. Removing it doesn't feel like simplification — it feels like bypassing authority.

**Design rule:** Keep approval flows even when they seem unnecessary. Make them faster, not absent.

---

### 4. Fear of Being Blamed, Not Fear of Work

The real anxiety is not "more work." It is "if something goes wrong, will it be my fault?" Manual systems have paper trails that protect people. If the software acts automatically and it goes wrong, who is responsible? The admin fears being blamed for something they didn't directly control.

**Design rule:** Audit logs are not just compliance — they are accountability protection. Make them visible and human-readable: "System calculated this. You approved it on [date]."

---

### 5. Semi-Automation IS the Destination for Phase 1

Most products treat human-in-loop as temporary — something to remove as users get comfortable. For this market, semi-automation is a feature, not a limitation. The goal is: system does the tedious calculation, human makes the call.

**Design rule:** Frame every automated calculation as a "suggestion pending your approval" — not a "decision the system made."

---

### 6. Fallback = Safety Net = Adoption Accelerator

The knowledge that "I can always fix it manually" is what allows people to trust the automated path. If the manual escape hatch is hidden or hard to find, people won't trust automation enough to use it. Paradoxically, the easier manual correction is, the more people use automation.

**Design rule:** Manual override must be one tap away, always visible, never buried. It is a feature, not a fallback.

---

### 7. Population = Parallel Roles, Not Bottlenecks

Large staff count in Indian schools is not a problem to optimise away — it is the operating model. The software should enable many people to do their part of the work in parallel, not centralise everything into one automated system.

**Design rule:** Role-based workflows where each person does their slice. Not a single admin doing everything.

---

### 8. Adoption Curve: Digital First → Semi-Automated → Opted-In Full Automation

```
Phase 1: Same process, now on screen        (familiar, low fear)
Phase 2: System calculates, human approves  (time saved, trust builds)
Phase 3: Human opts in to auto-approve      (power users only, explicit choice)

Never:   Force full automation on everyone
```

**Design rule:** Build Phase 1 and Phase 2. Design Phase 3 as an opt-in upgrade, never a default.

---

## How This Reflects in Every Module

| Principle | What it means in the product |
|---|---|
| Control is identity | Admin always has override. System never acts without trace. |
| Trust earned slowly | Show calculations before applying. No silent actions. |
| Approval is cultural | Keep approval flows. Make them fast, not absent. |
| Fear of blame | Audit log must show: system did X, person Y approved it. |
| Semi-auto is the destination | Every output is a suggestion → approval → applied. |
| Fallback = adoption | Manual correction is one tap, always visible. |
| Parallel roles | Workflows split across roles, not centralised in one admin. |
| Adoption curve | Phase 1 = digital process. Phase 2 = assisted. Phase 3 = opt-in auto. |

---

## The Design Test

Before finalising any feature, ask:

1. **Can the admin override this?** If no — fix it.
2. **Does the system act silently anywhere?** If yes — add a confirmation or review step.
3. **Is the audit trail human-readable?** If no — improve it.
4. **Does this remove someone's role entirely?** If yes — reconsider.
5. **Is the manual fallback one tap away?** If no — surface it.
6. **Does this feel like a suggestion or a command?** It should always feel like a suggestion.
