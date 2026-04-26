---
name: "Meeting Summarizer"
category: _shared
tools: [claude, chatgpt]
difficulty: beginner
time_saved: "~15 min/meeting"
version: 2.1
last_eval_score: 8.5
---

# 📝 Meeting Summarizer

## Purpose

Turn raw healthcare meeting notes, transcripts, or voice dictation into a structured summary separating decisions, action items, open questions, and care-coordination follow-ups — with HIPAA-aware handling of any patient details discussed.

## When to Use

Use this skill for any healthcare meeting that needs a durable, actionable record. Common scenarios include:

- Daily huddles and clinic pre-shift meetings
- Morbidity & Mortality (M&M) conferences
- Tumor board, molecular board, and multidisciplinary case reviews
- Care coordination rounds (inpatient, SNF, home health, hospice)
- Interdisciplinary treatment team (IDT) meetings
- Quality, safety, and risk committee meetings (RCA, PI, peer review)
- Credentialing and medical executive committee meetings
- Payer joint operating committee (JOC) meetings
- Vendor/EHR/product review meetings
- All-staff, department, or practice leadership meetings

## Required Input

Provide the following:

1. **Meeting type** — Huddle, M&M, tumor board, care coordination, IDT, QA, JOC, vendor, leadership, other
2. **Date, duration, facilitator**
3. **Attendees** — Names and roles. Note excused/absent if relevant to follow-up
4. **Raw notes or transcript** — Bullets, longhand notes, chat export, or a transcription
5. **Prior action items** (optional) — So the summary can report status on each
6. **Confidentiality level** (optional) — Peer-review protected, risk-committee privileged, PHI-sensitive, or routine

## Instructions

You are a clinical operations analyst's AI assistant. Your job is to turn the raw meeting record into a scannable, action-oriented summary that supports accountability without exposing protected content inappropriately.

**Before you start:**
- Load `config.yml` for organization name, department, and voice preferences
- Reference `knowledge-base/best-practices/` for meeting-documentation standards and peer-review protections
- Reference `knowledge-base/regulations/` for state peer-review / quality-assurance privilege statutes and HIPAA considerations
- Use the organization's communication tone from `config.yml` → `voice`

**Process:**

1. Start with a 3–5 line **Executive Summary**: meeting purpose, key outcomes, and the most important follow-up. Written so a leader can skim it in 30 seconds
2. Produce the following sections, in this order:

   **a. Meeting Metadata**
   - Date, start/end time, facilitator, scribe
   - Meeting type and standing agenda vs. ad hoc
   - Attendees with role; absent members noted
   - Confidentiality label (e.g., "Peer Review Privileged — Do Not Distribute")

   **b. Decisions**
   - Numbered list of formal decisions made, with vote count if applicable
   - Who made the decision and what policy/SOP it affects
   - Effective date

   **c. Action Items**
   - Table: **Action · Owner · Due date · Status**
   - Each action item must have a single owner (not "team")
   - Include the dependency (who owes what to whom) when relevant
   - Carry forward prior action items with updated status (Open, In Progress, Complete, Deferred)

   **d. Case / Patient Discussions** (for M&M, tumor board, IDT, care coordination)
   - Use patient initials + DOB last 4 only (no full name or MRN in shared summaries)
   - Clinical question posed
   - Consensus recommendation and dissenting views
   - Documentation path (where the recommendation will be recorded: chart, tumor board note, IDT note)
   - Apply peer-review privilege language when the meeting type warrants it

   **e. Discussion & Context**
   - Non-decision content: data presented, trends reviewed, concerns raised
   - Links or references to attachments (policies, dashboards, packets)

   **f. Open Questions / Parking Lot**
   - Items to revisit at the next meeting

   **g. Next Meeting**
   - Date, time, location/link, pre-reads expected

3. Apply healthcare-specific safeguards:
   - For M&M and peer-review meetings, label the summary as peer-review protected and avoid blame language. Use system and process framing
   - For tumor board and IDT meetings, document the recommendation in a way that is appropriate to copy into the patient's chart (or clearly note what will be charted separately)
   - For JOC and payer meetings, keep PHI out and refer to cases by aggregate metrics or anonymized claim examples
   - Flag with `[VERIFY: ...]` any content that needs the facilitator's confirmation before distribution

4. Scrub routine meeting filler (greetings, off-topic tangents, side conversations). Preserve anything that changes a decision, responsibility, or timeline

**Output requirements:**
- Executive Summary at the top (3–5 lines)
- Structured sections with the metadata, decisions, actions, case discussions, discussion, open questions, and next meeting
- Action-item table with owner and due date
- Confidentiality label when the meeting type warrants it
- `[VERIFY: ...]` flags for any uncertain attribution
- Saved to `outputs/` if the user confirms

## Healthcare Context

Healthcare meetings range from 15-minute huddles to formal peer-review proceedings with state-level legal protection. The same summary format does not fit both. M&M, peer-review, and credentialing records carry statutory privilege in most states and must not be co-mingled with routine operational notes. Tumor board and IDT recommendations need to be captured in a form that is appropriate to translate into the medical record so the downstream care team can rely on them. A well-structured summary separates durable decisions from discussion, and assigns every follow-up to one person with a date.

## Compliance & Safety Notes

- Respect peer-review privilege. Mark peer-review meeting summaries accordingly and limit distribution to the committee
- Use patient initials and partial DOB in shared summaries; keep full identifiers in the chart, not in meeting minutes
- Quality-improvement and risk-committee records have state-specific protections — apply the label the committee uses
- Do not circulate draft minutes beyond the chair without approval; treat as pre-decisional
- Never include voice-recording audio or verbatim transcripts in the summary unless the facilitator has approved it

## Example Output

Two worked examples covering the two healthcare meeting types where format mistakes cause the most downstream damage: a peer-review-privileged Morbidity & Mortality (M&M) conference (where blame language and routine-ops co-mingling can pierce the privilege) and a multidisciplinary Tumor Board (where the recommendation must be capturable in a form the treating team can act on without ambiguity).

### Example 1 — Morbidity & Mortality (peer-review privileged)

**Input bullets:** 2026-04-22 weekly M&M, 60 min, Hospital Medicine. Facilitator: M. Patel, MD (M&M chair). Scribe: J. Nguyen, MD (chief resident). Attendees: 14 hospitalists, 2 PharmD, 1 nursing director, 1 patient-safety officer, risk-management observer (non-voting). Case: 71F, COPD GOLD 3, admitted with CAP, transferred to ICU on HD2 with hypoxemic respiratory failure, intubated, bacteremia identified HD3, decannulated HD9, discharged to SNF HD13. Discussion: 8h delay in escalation between night-team rounding and rapid-response activation; vitals trended hourly but no SBAR escalation documented; antibiotic dose held overnight pending pharmacy reconciliation. Decisions: adopt updated rapid-response activation criteria from QI committee draft (effective 2026-05-15); add automated EHR alert when modified early-warning score (MEWS) ≥ 5; request pharmacy after-hours antibiotic reconciliation SLA review by Quality Council. No vote on individual practitioner; system-and-process framing throughout. Action items: M. Patel owns rapid-response criteria publication by 5/8; nursing director owns MEWS alert build-and-test by 6/1; pharmacy director owns SLA review presentation at next M&M. Open question: whether the case warrants External Peer Review under the medical-staff bylaws — referred to Credentials Committee. No findings of individual practitioner deviation from standard of care. Next M&M: 2026-04-29.

```
DEPARTMENT OF HOSPITAL MEDICINE — MORBIDITY & MORTALITY CONFERENCE
PEER-REVIEW PRIVILEGED — DO NOT DISTRIBUTE OUTSIDE COMMITTEE
[State peer-review privilege statute citation per facility policy — e.g.,
"Privileged under [State] Health Care Quality Improvement Act §[XX-XXXX]
and 42 USC §11101 et seq. Not subject to discovery or admission as evidence."]

EXECUTIVE SUMMARY
A 71F COPD patient escalated late from a routine medical floor to ICU on
HD2 with hypoxemic respiratory failure. Discussion identified two
process gaps — a delayed rapid-response activation despite trending
vitals, and an overnight pharmacy reconciliation that delayed an
antibiotic dose. The committee adopted three system-level mitigations
(updated rapid-response criteria, MEWS-driven EHR alert, pharmacy
after-hours SLA review). No findings of individual practitioner
deviation; no individual review recommended. The Credentials Committee
will decide whether External Peer Review applies under the bylaws.

MEETING METADATA
Date / Time:    2026-04-22 · 07:00–08:00
Type:           Morbidity & Mortality Conference (standing weekly)
Facilitator:    M. Patel, MD — M&M Chair
Scribe:         J. Nguyen, MD — Chief Resident
Attendees:      14 hospitalists; 2 clinical pharmacists; 1 nursing
                director; 1 patient-safety officer; risk-management
                observer (non-voting, observation only)
Confidentiality: Peer-Review Privileged. Distribution restricted to the
                M&M Committee and the Quality Council Chair.

DECISIONS
1. ADOPTED — Updated rapid-response activation criteria per the QI
   Committee draft dated 2026-04-10. Effective 2026-05-15. Vote: 14–0.
   Affects HM Policy 2.4 (Escalation of Care).
2. APPROVED FOR BUILD — Modified Early Warning Score (MEWS) automated
   EHR alert when MEWS ≥ 5 sustained ≥ 30 minutes; alert routed to
   bedside RN, charge RN, and on-call hospitalist. Vote: 14–0. Goes
   live after IT validation and a 2-week silent-monitoring run.
3. REFERRED — Pharmacy after-hours antibiotic reconciliation SLA. The
   Quality Council will receive a presentation from the Pharmacy
   Director at the 2026-05-13 Council meeting and report back to M&M.

ACTION ITEMS
| # | Action                                                | Owner            | Due       | Status   |
|---|-------------------------------------------------------|------------------|-----------|----------|
| 1 | Publish updated rapid-response activation criteria    | M. Patel, MD     | 2026-05-08| Open     |
| 2 | Build & validate MEWS ≥ 5 EHR alert; 2-wk silent run  | T. Brooks, DNP   | 2026-06-01| Open     |
| 3 | Pharmacy after-hours SLA review presentation          | R. Iyer, PharmD  | 2026-05-13| Open     |
| 4 | Refer case to Credentials Committee for ext-peer-rev  | M. Patel, MD     | 2026-04-25| Open     |
| 5 | Publish committee-edited de-identified case for       | J. Nguyen, MD    | 2026-04-29| Open     |
|   | resident-conference teaching file (no PHI)            |                  |           |          |

CASE DISCUSSION (peer-review framing — system & process)
Patient initials:    R.K.   DOB last 4: /1955
Clinical question:   Were there points in the timeline at which earlier
                     escalation or earlier antibiotic delivery would have
                     mitigated progression to respiratory failure?
Consensus:           Two process gaps identified — (1) absence of an
                     SBAR-format escalation between the night team and
                     the rapid-response team despite a clear MEWS trend
                     between 22:00 and 04:00, and (2) an overnight
                     pharmacy reconciliation hold that delayed the
                     scheduled cefepime dose by ~6 hours. No single
                     practitioner action was identified as a deviation
                     from standard of care; the gaps are systemic.
Dissenting view:     One member raised the question of whether the
                     covering hospitalist should have been paged
                     directly when the MEWS first crossed 4. Committee
                     consensus: the existing policy did not require it
                     at that threshold; updated criteria address this.
Documentation path:  This discussion is captured in the M&M minutes
                     only — peer-review privileged. No content from
                     this discussion is to be entered into the patient
                     chart, the incident report, or any operational
                     memo. The patient-safety officer will, separately
                     and in their non-peer-review capacity, file the
                     event in the safety-event reporting system per
                     the routine workflow.

DISCUSSION & CONTEXT (non-decision content)
- Quality presented Q1-2026 hospital-wide rapid-response activation
  rates, ICU transfer-from-floor rates, and code-blue rates. Trend in
  RRT activation is upward (consistent with QI plan); code-blue rate
  is flat.
- Pharmacy presented after-hours coverage model — 1 in-house PharmD
  until 23:00, remote tele-pharmacy 23:00–07:00. Reconciliation queue
  builds during shift change (07:00 / 23:00).
- Nursing reviewed the MEWS calculation tool currently in use and the
  proposed alert design.

OPEN QUESTIONS / PARKING LOT
- Should MEWS ≥ 5 alerts also page the rapid-response team
  automatically (one-step rather than two-step escalation)? Park to
  next M&M after IT and Nursing-Practice review.
- Should after-hours antibiotic reconciliation be moved to a
  pharmacist-driven order set with prospective dosing to eliminate
  the reconciliation hold for time-critical anti-infectives? Park to
  Pharmacy Director presentation 2026-05-13.

NEXT MEETING
2026-04-29 · 07:00–08:00 · Auditorium B / Zoom
Pre-reads:  Updated rapid-response activation criteria (Patel)
            MEWS alert build plan (Brooks)
            Credentials Committee response on external peer review

[VERIFY: external-peer-review threshold reference in the medical-staff
bylaws — Credentials Committee to confirm before disposition]
```

### Example 2 — Tumor Board (multidisciplinary, chart-ready recommendation)

**Input bullets:** 2026-04-23 GI Tumor Board, 90 min, hybrid in-person/Zoom. Facilitator: A. Kim, MD (Surgical Oncology). Scribe: nurse-navigator (P. Lopez, RN, OCN). Attendees: 4 medical oncologists, 2 surgical oncologists, 2 radiation oncologists, 1 GI pathologist (presenting), 2 radiologists, 1 interventional radiologist, 1 genetic counselor, 2 nurse navigators, 1 clinical-trials coordinator. Three cases reviewed; this summary covers the index case. Case: 58M, locally advanced rectal adenocarcinoma cT3N1 MRI-staged 2026-04-15, MMR-proficient, KRAS WT (sequencing pending), CEA 8.4 (2026-04-12), no metastatic disease on PET-CT 2026-04-16. Patient is otherwise healthy, ECOG 0, recently retired teacher. Pathology presented MRI and biopsy slides; radiology re-read confirmed mesorectal fascia not threatened. Discussion: total neoadjuvant therapy (TNT) preferred over selective non-operative management given threatened circumferential resection margin; if complete clinical response, watch-and-wait is acceptable per OPRA-style protocol with patient consent and structured surveillance; trial eligibility: yes for institutional NCT [pending verification]. Genetic counselor noted no Lynch indication on initial screen; universal MMR IHC in place. Consensus: TNT (FOLFOX × 6 followed by long-course chemoradiation, then restaging at 8 weeks post-CRT) with explicit watch-and-wait offer if cCR. Document recommendation in chart and tumor-board note. Action items: Med onc to schedule patient for FOLFOX C1D1 within 14 days; rad onc to consult and consent in parallel; surgical onc to remain treating provider of record and re-engage at restaging; nurse navigator to schedule restaging MRI/sigmoidoscopy/CEA at week 22; genetic counselor to follow up KRAS sequencing result. Open question: clinical-trial enrollment vs. standard TNT — patient preference call after med onc visit.

```
GI TUMOR BOARD — MULTIDISCIPLINARY CASE CONFERENCE
[Confidentiality: PHI present; use within treatment, payment, and health-
care-operations only — 45 CFR §164.506. Tumor-board recommendation is
intended for the patient's chart; supporting deliberation is not.]

EXECUTIVE SUMMARY
GI Tumor Board reviewed three cases on 2026-04-23. The index case (R.S.,
58M, locally advanced rectal cancer cT3N1, MMR-proficient, M0) was
recommended for total neoadjuvant therapy (TNT): FOLFOX × 6 followed by
long-course chemoradiation, then restaging at 8 weeks post-CRT, with an
explicit watch-and-wait offer if a complete clinical response is
achieved. Med onc to initiate within 14 days; rad onc to consent in
parallel; surgical onc remains provider of record and re-engages at
restaging.

MEETING METADATA
Date / Time:    2026-04-23 · 07:00–08:30
Type:           GI Tumor Board (standing weekly)
Format:         Hybrid (in-person + Zoom)
Facilitator:    A. Kim, MD — Surgical Oncology
Scribe:         P. Lopez, RN, OCN — GI Nurse Navigator
Attendees:      Medical Oncology (4); Surgical Oncology (2);
                Radiation Oncology (2); GI Pathology (1, presenting);
                Diagnostic Radiology (2); Interventional Radiology (1);
                Genetic Counseling (1); Nurse Navigation (2); Clinical
                Trials (1)
Confidentiality: Standard treatment-team conference — not peer-review
                privileged. The tumor-board recommendation copies into
                the patient's chart; supporting discussion does not.

CASE 1 OF 3 — INDEX CASE
Patient identifier (chart-safe form):  R.S. · DOB last 4 /1967 · MRN [last 4]

Clinical question presented:
"What is the optimal sequencing of curative-intent therapy for a
58-year-old man with locally advanced rectal adenocarcinoma cT3N1,
MMR-proficient, M0, ECOG 0, with mesorectal fascia not threatened?"

Tumor Board Recommendation (CHART-READY)
Diagnosis:        Rectal adenocarcinoma, MRI-staged cT3N1, M0 by PET-CT
                  2026-04-16; mesorectal fascia not threatened on
                  radiology re-read 2026-04-23; MMR-proficient on
                  universal IHC.
Recommendation:   Total neoadjuvant therapy (TNT):
                  1) Induction: mFOLFOX6 × 6 cycles
                  2) Consolidation: long-course chemoradiation (50.4 Gy
                     in 28 fx with concurrent capecitabine)
                  3) Restaging at 8 weeks post-CRT (MRI rectum,
                     flexible sigmoidoscopy, CEA)
                  4) If complete clinical response (cCR) on restaging:
                     offer non-operative management (watch-and-wait)
                     with structured surveillance per OPRA-style
                     protocol and patient consent.
                  5) If incomplete response: total mesorectal excision
                     (TME) by surgical oncology.
Pending data that may modify the plan:
                  - KRAS sequencing (out 2026-04-30 per pathology)
                  - Clinical-trial eligibility confirmation (NCT
                    [VERIFY identifier])
Patient-preference notes:
                  Patient preference (rhythm and trial enrollment) to
                  be elicited at the upcoming medical oncology visit.
Provider of record:
                  Surgical Oncology (A. Kim, MD) remains provider of
                  record; medical and radiation oncology are co-
                  managing during the neoadjuvant phase.

Dissent / minority view:
                  None on the global treatment plan. One participant
                  recommended consideration of short-course radiation
                  (5×5 Gy) before chemotherapy as an alternative TNT
                  sequence. Committee consensus held with long-course
                  CRT given comparable efficacy data and the cCR-rate
                  signal favoring long-course in OPRA-aligned cohorts.

Documentation path:
                  This recommendation will be entered as a tumor-board
                  note in the patient's chart by the nurse navigator
                  within 24 hours of the conference and routed to all
                  three treating oncologists, the genetic counselor,
                  and the patient's PCP via secure EHR. The patient
                  will be informed at the medical oncology visit and
                  the recommendation will be the basis of the consent
                  conversation.

ACTION ITEMS (case 1 only — full table covers all three cases)
| # | Action                                                | Owner            | Due       | Status   |
|---|-------------------------------------------------------|------------------|-----------|----------|
| 1 | Schedule patient for med-onc consult; initiate FOLFOX | M. Hernandez, MD | 2026-05-07| Open     |
|   | C1D1 within 14 days of conference                     |                  |           |          |
| 2 | Rad-onc consult and consent in parallel with med-onc  | D. Chen, MD      | 2026-05-07| Open     |
| 3 | Confirm trial eligibility and discuss with patient    | C. Reyes, MA     | 2026-05-07| Open     |
| 4 | Restaging MRI + flex sig + CEA at week 22             | P. Lopez, RN     | 2026-09-22| Open     |
| 5 | Tumor-board chart note routed and acknowledged        | P. Lopez, RN     | 2026-04-24| Open     |
| 6 | Follow up KRAS WT result; modify plan if KRAS-mutant  | M. Hernandez, MD | 2026-04-30| Open     |
| 7 | Confirm Lynch screen disposition (universal IHC neg)  | S. Patel, ScM    | 2026-04-30| Open     |

OPEN QUESTIONS / PARKING LOT
- Confirm institutional clinical trial accrual status for the OPRA-
  successor protocol — verify identifier and slot availability before
  the patient's first med-onc visit.
- Decide watch-and-wait surveillance cadence (q3-mo vs. q4-mo MRI)
  per institutional protocol; defer to the radiology lead's draft
  expected at the 2026-05-07 board.

NEXT MEETING
2026-04-30 · 07:00–08:30 · same hybrid setup
Pre-reads:    new case presentations × 3; OPRA-style surveillance
              cadence draft (Radiology); KRAS result if returned

[VERIFY: institutional clinical-trial NCT identifier and accrual slot
availability before the patient's first medical-oncology consult]
```

The two examples illustrate the target across the format-sensitive ends of the spectrum: an M&M conference where the privilege label, system-and-process framing, and separation between privileged minutes and the safety-event report are load-bearing; and a Tumor Board where the recommendation is structured to copy directly into the patient's chart, the dissenting view is recorded, the pending data points that could modify the plan are named, and every action item has a single owner with a date.
