---
name: "Nurse Shift Handoff (I-SBARR) Generator"
category: operations
tools: [claude, chatgpt]
difficulty: intermediate
time_saved: "~6 min/patient handoff"
version: 1.0
last_eval_score: null
---

# 🩺 Nurse Shift Handoff (I-SBARR) Generator

## Purpose

Turn a nurse's working patient data — recent vitals, assessments, drips, pending orders, labs, and open safety concerns — into a complete, standardized Introduction–Situation–Background–Assessment–Recommendation–Readback (I-SBARR) handoff that the outgoing RN can review, edit, and verbally deliver to the incoming RN in under two minutes per patient. Output is designed for bedside handoff, charge-nurse board rounds, ICU step-down transfers, and change-of-shift reports on med-surg, progressive care, ICU, ED, L&D, and behavioral-health units.

## When to Use

Use this skill any time a licensed nurse or charge nurse is preparing to hand off a patient or a patient assignment to a peer and needs the handoff to meet The Joint Commission's National Patient Safety Goal for closed-loop communication (I-SBARR supersedes classic SBAR in most 2024–2026 policies). Common scenarios include:

- Change-of-shift report on a 4–6 patient assignment, med-surg or telemetry
- ICU and step-down handoff where drips, lines, ventilator settings, and sedation scores need structured reporting
- ED-to-floor admission handoff where the receiving nurse needs a rapid orientation
- PACU-to-floor or L&D-to-postpartum transitions
- Pre-rapid-response or pre-code pulse check where the charge nurse needs a fast snapshot
- Travel/float nurse onboarding to a patient assignment mid-shift
- Transfer between levels of care (ICU → step-down, med-surg → telemetry, behavioral health → med-surg for medical clearance)
- SNF, LTAC, or home-health-to-ED transfer where the receiving team needs a structured summary
- Interprovider handoffs during a nurse's break coverage (buddy system)
- Student-nurse supervised handoff practice with preceptor review

This skill drafts the handoff. It does not replace the bedside verification step (incoming nurse lays eyes on the patient, IV sites, drains, and drips), does not autonomously chart the handoff in the EHR, and must never invent vitals, orders, or assessments that were not entered as input.

## Required Input

Provide the following per patient on the assignment. Fewer inputs produce a shorter, more conservative handoff and more `[VERIFY]` flags.

1. **Patient identifiers and code status** — Room/bed, age, sex at birth, code status (Full Code / DNR / DNI / comfort care), allergies (and reaction if known), isolation precautions, fall risk level, language and interpreter need, primary attending and admitting service. De-identify any patient-specific data before testing with external models.
2. **Admission context** — Admission date, admitting diagnosis or chief complaint, relevant past medical history (2–4 problems that matter this shift), recent surgeries or procedures, weight, and primary emergency contact or surrogate.
3. **Current clinical status** — Most recent vitals with timestamps, neuro/pain/mental-status score (RASS, GCS, CAM-ICU, pain 0–10), respiratory status (O2 device, saturation, ventilator settings if intubated), cardiac status (rhythm, drips with dose and titration range), GI/GU status (diet, bowel last movement, Foley, I&O), skin (pressure-injury stage if any, wounds, drains), mobility (activity order, fall interventions), and any lines/tubes with insertion date.
4. **Active orders and treatments** — Continuous infusions with current rate, high-alert meds due this shift (insulin, anticoagulants, opioids, vasoactive drips), PRNs given and effect, scheduled meds coming due, pending labs or imaging, pending consults, and any time-sensitive to-dos (e.g., "BMP due at 0400, redraw if K < 3.5").
5. **Events this shift** — New symptoms, rapid-response or code activation, falls, restraint initiation, medication errors, family conversations, change in mental status, transfusion reaction monitoring, or behavioral escalations. Include time-stamped narrative snippets if available.
6. **Plan and disposition** — Expected disposition (discharge today, transfer to step-down, surgery tomorrow), anticipated holdbacks (needs PT clearance, awaiting echocardiogram), social-work or case-management status, and any family/visitor considerations.
7. **Sender identity and unit** — Outgoing RN, incoming RN (or team), unit, and handoff format (bedside vs. board vs. phone). If a unit uses local acronyms or board columns, list them so the output matches what the incoming nurse expects to see.

If a patient is unstable, in the middle of a rapid response, or has an active safety event at the moment of handoff, the skill returns a **`[HOLD HANDOFF — STABILIZE FIRST]`** flag with a one-line recommendation to defer structured handoff until the bedside event is resolved, rather than generating an incomplete report.

## Instructions

For every patient, produce the six-element I-SBARR output in the exact order below. Keep each element to the minimum length needed to be actionable. Never use a patient's full name in the header — use room/bed and initials.

### I — Introduction

One sentence: outgoing RN name and role, incoming RN name and role, unit, patient room/bed, patient initials, age, sex, admitting attending, and code status. Example format:

```
I: "This is [Outgoing RN], [shift] RN on [unit], handing off to [Incoming RN]. Room [xxx], bed [x], [initials], [age]yo [sex], Dr. [attending] on [service]. Code status: [Full / DNR / DNI / comfort]."
```

Add allergies, isolation, and fall-risk level inline only if non-default. Never fabricate the incoming nurse's name if not provided — use "incoming RN" as a placeholder.

### S — Situation

Two to four sentences. State the reason the patient is on this unit *right now*, including the admitting chief complaint, current acuity, and the single most important issue of this shift. If there has been a meaningful change in the last 4 hours (new hypoxia, new arrhythmia, new altered mental status, new pain, behavioral escalation), lead with that change. Do not restate the admission H&P — this is what matters *now*.

### B — Background

Four to eight bullet-equivalent lines covering:

- Admission date, admitting diagnosis, day of hospitalization
- Two to four relevant PMH items
- Procedures or surgeries this admission with post-op day
- Pertinent social/functional context (lives alone, ambulates with walker, primary decision-maker is daughter, non-English preferred language + interpreter)
- Isolation precautions and why (e.g., MRSA colonization, C. diff, airborne, enhanced contact)
- Lines, tubes, drains with insertion date (e.g., "R subclavian CVC day 3, chest tube R day 2 on water seal, Foley day 4 — re-eval for removal")
- Weight and allergies if clinically relevant

Avoid PMH dumps. If an item has no bearing on this shift, leave it out.

### A — Assessment

Present by body system, only for systems that are active or changed. Use the `[stable / improving / worsening / new]` tag against each line. Always include vital-sign trends with timestamps, not a single point value. Example structure:

```
Neuro: [status] — RASS [x], GCS [x], PERRLA, moves all extremities, pain [x]/10 on [drug] PRN, last dose [time]
Cardio: [status] — [rhythm], HR [range] over shift, BP [range], on [pressor] at [dose] — titrating to MAP > 65
Resp: [status] — [O2 device/flow], SpO2 [range], RR [range], breath sounds [finding], last ABG [time/values] if relevant
GI: [status] — diet, last BM, N/V, abdomen exam, NG/OG, tube-feed rate + goal
GU: [status] — Foley or voiding, urine output last 8 hr, BUN/Cr if trending
Skin: [status] — any pressure injury (stage and location), wounds, drains, turn schedule
Endo: [status] — BG trend, insulin protocol, last dose, hypoglycemia events
Heme/ID: [status] — WBC/Hgb/Plt trend, antibiotic day X of Y, culture pending, fever curve
Lines: [status] — [site, insertion date, last dressing change], due for change on [date]
Psych/Behavioral: [status] — orientation, agitation events, sitter status, restraint status (type, last release, skin check time)
Safety: [falls this shift, CIWA/COWS score if applicable, suicide precautions, elopement risk]
```

Tag any abnormal finding that has not been acknowledged by the provider as `[ACTION — NOTIFY PROVIDER]`. Tag any finding the incoming nurse must personally re-verify at the bedside as `[VERIFY AT BEDSIDE]` (e.g., IV site patency, drain output level, ventilator settings, pump rates, restraint placement).

### R — Recommendation

Three to six concrete, time-boxed asks for the incoming RN. Use verbs: "titrate," "draw," "reassess," "hold," "escalate," "educate," "follow up with." Include expected times and thresholds, not vague handoffs. Examples:

- "Draw BMP at 0400; notify provider if K < 3.5 or Mg < 1.8 before replacing."
- "Reassess pain 30 min after 0300 hydromorphone dose; escalate to provider if pain > 6/10 despite PRNs."
- "Titrate norepinephrine to MAP > 65; current dose 0.08 mcg/kg/min, max per order 0.5."
- "Hold metoprolol in AM if HR < 55 or SBP < 100; call provider before giving."
- "Case management expects placement by 1200; be ready with updated transfer summary."
- "Family meeting at 1500; cardiology and palliative attending expected."

For pending studies, state how the result will change the plan (e.g., "CT abdomen pending — if positive for abscess, IR consult pre-ordered").

### R — Readback / Closed-Loop Confirmation

Produce three to five specific questions the incoming nurse should read back to confirm. These are the items most likely to cause a handoff error if not reconfirmed. Format as a closed-loop checklist:

```
Readback checklist (incoming RN to confirm):
[ ] Code status: [value] — incoming RN repeats
[ ] Active drips and doses: [list]
[ ] High-alert meds due this shift: [list with times]
[ ] Pending critical results that change the plan: [list]
[ ] Safety precautions active: [falls / restraints / suicide / isolation]
Outgoing RN signature + time: _______
Incoming RN signature + time: _______
```

### Unit-Level Rollup (if multiple patients)

When a full assignment is provided, prepend a one-paragraph shift snapshot before the per-patient I-SBARRs:

- Total patients, acuity mix (how many ICU-equivalent, how many step-down, how many stable), staffing ratio, any patient who is the highest priority for the incoming RN to see first, open admissions or discharges, pending codes or rapids, and any unit-level safety issues (pharmacy delay, missing controlled-substance count, isolation room availability).

Flag the single highest-priority patient with `[SEE FIRST]`.

## Safety & Compliance Guardrails

- The skill never fabricates vitals, medication doses, drip rates, lab values, or orders. Missing inputs become `[MISSING — confirm before handoff]` flags.
- Anything that suggests clinical deterioration (new hypoxia, new altered mental status, new tachyarrhythmia, hypotension trending, new focal deficit, new chest pain, active bleeding, new suicidal ideation) is surfaced at the top of Assessment and triggers `[ACTION — NOTIFY PROVIDER]`.
- Restraint use, fall events, medication errors, and transfusion reactions are always surfaced, never buried.
- The skill does not produce a handoff for a patient who is actively unstable; it returns `[HOLD HANDOFF — STABILIZE FIRST]` and recommends the outgoing RN complete bedside stabilization and provider notification first.
- High-alert drips (insulin, heparin, vasopressors, paralytics, sedation, TPA/tPA, PCA) are always listed with current rate, titration range, and time of last rate change.
- When the patient is pediatric, geriatric, pregnant, or on behavioral precautions, the skill includes a one-line population-specific safety note (e.g., "pediatric — weight-based dosing verified against current weight [VERIFY]"; "pregnant — fetal monitoring Q[interval], last fetal HR [value]").
- The skill assumes closed-loop verbal confirmation at the bedside. It will not mark an item "confirmed" — the Readback checklist is deliberately unchecked.

## Example Output

```
=== Shift Snapshot — 0700 handoff, 5 Med-Surg Telemetry ===
5 patients: 1 ICU step-down (room 412, post-op CABG D2), 3 med-surg stable, 1 behavioral-health medical-clearance. 
Highest priority for first round: Room 412 [SEE FIRST] — on esmolol drip, titrating to HR < 100.
Pending: 2 AM discharges (rooms 408, 410 — awaiting PT and transportation). One pending admission from ED (sepsis rule-out).

---

=== Room 412, Bed A — JM, 68yo M — [SEE FIRST] ===

I: Handoff from N. Park RN (night) to incoming day RN. Unit 4S, Room 412A, JM, 68yo M. Attending: Dr. Chen, cardiothoracic. Code status: Full Code. Allergies: PCN (rash). Fall risk: High.

S: POD2 from 3-vessel CABG, now step-down from CVICU since 1800 yesterday. Remains on esmolol drip for rate control; most acute issue tonight is intermittent a-fib with RVR requiring esmolol titration. No chest pain, no respiratory distress at handoff.

B:
- Admitted 04/12 for NSTEMI; CABG x3 on 04/13 (LIMA-LAD, SVG-OM1, SVG-RCA)
- PMH: HTN, HLD, T2DM on metformin, OSA on home CPAP
- Lives with spouse, ambulates independently at baseline; daughter is health-care proxy
- Isolation: none
- Lines/tubes: R IJ triple-lumen day 3 (due for dressing change today), two mediastinal chest tubes on water seal day 2, Foley day 2 (re-eval for removal), two peripheral IVs
- Weight 92 kg; height 178 cm

A:
Neuro: stable — A&O x3, pain 3/10 sternotomy on PO oxycodone 5 mg q4h PRN, last dose 0430, effective. RASS 0.
Cardio: improving but monitor — a-fib with RVR x2 overnight (0130, 0420), HR spiked to 140s both times, converted with esmolol titration up to 100 mcg/kg/min, now back to 60 mcg/kg/min. BP 108–135/60–78. MAP > 65 throughout. [VERIFY AT BEDSIDE: current esmolol rate and line patency.]
Resp: stable — 2 L NC, SpO2 94–97%, RR 16–20, diminished BLL bases, IS 1200 mL. CPAP on at night per patient.
GI: stable — cardiac diet, tolerated dinner, no N/V, last BM 04/13 pre-op, bowel regimen started.
GU: stable — Foley draining clear yellow, 45–80 mL/hr.
Skin: sternotomy incision — dry dressing, intact, no drainage; mediastinal CT sites clean, no crepitus; heels floated.
Endo: BG 140–180 overnight, covered per sliding scale; last accucheck 0600 = 162.
Lines: R IJ day 3 — dressing change due today by AM. CT output last 4 hr: R 20 mL, L 15 mL, no air leak, on water seal.
Safety: high fall risk, yellow gown + non-slip socks, bed alarm on, call light in reach.

R (Recommendation):
- Titrate esmolol to HR < 100 per order (range 50–200 mcg/kg/min); if rate does not hold at max, notify Dr. Chen — consider amiodarone load per CT surgery protocol
- Dressing change R IJ by 1000; swab site, document
- Reassess for CT removal with CT surgery team on rounds ~0900
- Reassess Foley removal — voided trial if removed, bladder scan in 6 hr
- AM BMP + Mg due at 0600 already drawn; replace K if < 4.0 (post-op target per order), replace Mg if < 2.0 [ACTION — NOTIFY PROVIDER if K < 3.5 before replacement]
- PT expected 1030 for first walk; ensure adequate pain control 30 min prior
- Daughter expected at 1100 for family update with CT surgery PA

Readback checklist (incoming RN to confirm):
[ ] Code status: Full Code
[ ] Active drip: esmolol, current rate ____ mcg/kg/min, titrating to HR < 100
[ ] High-alert meds due: AM K/Mg replacement per criteria; insulin SS per BG
[ ] Pending critical result: 0600 BMP + Mg — hold rate-related changes until reviewed
[ ] Safety: high fall risk, bed alarm on, 2x chest tubes on water seal — no air leak at handoff

Outgoing RN: N. Park — 0700
Incoming RN: _______ — _______
```

## Notes on Fit and Limits

- Intended for licensed-nurse use. Not a substitute for bedside verification.
- Designed around I-SBARR (not classic SBAR) to match 2024 Joint Commission NPSG.02.03.01 handoff expectations. If a unit still uses SBAR, the Introduction and Readback sections can be compressed but should not be omitted.
- Pediatric, obstetric, and behavioral-health units should have local policies that supersede this skill's default structure; the skill includes population-specific safety notes but is not a substitute for unit-specific handoff tools (e.g., SBAR-P, I-PASS for pediatrics).
- For ambulatory, procedural, or OR-to-PACU handoffs, use the team's local checklist-driven handoff tool (e.g., WHO Surgical Safety Checklist) rather than this skill.
- Output is drafted content only. The outgoing RN is responsible for accuracy before verbal handoff and EHR charting.
