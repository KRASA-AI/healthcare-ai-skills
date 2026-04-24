---
name: "Ambient Scribe Note & Coding Audit"
category: admin
tools: [claude, chatgpt]
difficulty: advanced
time_saved: "~12 min/audit"
version: 1.1
last_eval_score: null
---

# 🔍 Ambient Scribe Note & Coding Audit

## Purpose

Audit an ambient-AI-drafted clinician note against an auditable source (structured encounter data, clinician-corrected final note, or — where policy allows — the transcript/audio log) to flag note inflation, phantom diagnoses, over-stated HPI or ROS elements, E/M-level drift, HCC upcoding, and assessment/plan content that was not discussed in the encounter. Output is designed for compliance, coding, and clinical-documentation-integrity (CDI) teams reviewing ambient-scribe output under a 2026 policy environment where both payers and provider organizations are responding to measurable coding-intensity drift from ambient tools.

## When to Use

Use this skill whenever a human reviewer needs a structured second-pass check on an ambient-AI-drafted note — the "auto-accept" moment is the single largest source of drift. Common scenarios include:

- Random compliance audits (e.g., 5–10% sample of ambient-authored notes per provider per quarter)
- Pre-bill coding review on high-dollar E/M levels (99214, 99215, 99204, 99205, high-complexity inpatient, or any 2-tier level jump from the provider's baseline)
- CDI review when HCC/risk-adjustment diagnoses appear on the problem list that were not the documented reason for the encounter
- Provider-level pattern review when an ambient tool is new to a practice (30/60/90 day cadence)
- Targeted review after a payer downcoding event or a payer audit request
- Post-implementation monitoring when a health system first deploys an ambient scribe at enterprise scale
- RAC, MAC, OIG, or commercial-payer audit preparation and rebuttal packet assembly
- Residency, fellowship, or teaching-service review where attending attestations may conflict with ambient-drafted assessments
- Internal coding training: surfacing drift patterns for feedback to individual clinicians
- Payer contract renegotiation where coding-intensity shifts need to be explained with evidence

This skill is a reviewer's tool. It does not modify the note, does not re-code the claim, does not create a corrective amendment, and does not replace the clinician's clinical judgment. It surfaces risk-ranked findings; a human makes the final call.

## Required Input

Provide, at minimum, inputs 1 and 2. Inputs 3–5 dramatically improve audit quality.

1. **The ambient-AI-drafted note (or the signed final note if that is what is being audited)** — Full text with clear section headers (CC, HPI, ROS, PMH, FH, SH, Exam, Assessment & Plan, Patient Instructions). Indicate whether this is the raw draft, the clinician-edited draft, or the signed note. De-identify before testing with external models.
2. **Ground-truth encounter signal** — At least one of: (a) the provider's pre-visit agenda / chief complaint; (b) the structured visit data (vitals, orders placed, labs resulted in the encounter, meds reconciled, problem-list changes); (c) the transcript or summary log from the ambient tool if your policy permits reviewer access; (d) the patient's own pre-visit intake questionnaire. If none of these are available, the audit output is clearly marked "limited-input audit" and risk scores are reduced accordingly.
3. **Billing submission** — Submitted CPT (including E/M level), ICD-10 codes on the claim, any modifiers, HCC-relevant diagnoses captured, and whether a time-based or MDM-based E/M was billed. If a risk-adjustment model (CMS-HCC V24/V28, HHS-HCC) is in play, note which.
4. **Clinician baseline and policy** — The individual provider's historical E/M distribution (for drift comparison), the specialty, the practice's ambient-scribe policy (auto-accept allowed? required review field? dual-signature?), and any payer-specific guardrails (e.g., Medicare Advantage plan's HCC attestation policy, local MAC LCDs).
5. **Scope and purpose of the audit** — Random sample, targeted high-risk, payer-requested, pre-bill, post-bill, post-denial, or teaching-case. Include whether this is a standalone audit or part of a trend analysis across N notes.

If the ambient note was already hand-edited by the clinician without a diff available, the skill notes that the audit is against the final signed document only and cannot distinguish AI-authored content from clinician edits.

## Instructions

Produce a structured audit report in the following six sections. Keep risk ratings calibrated — not every discrepancy is a compliance event. Use the rating rubric below exactly.

**Risk rating rubric (use verbatim):**
- `🟢 LOW` — Documentation style preference or minor discrepancy; no coding or compliance impact.
- `🟡 MEDIUM` — Documentation element not supported by encounter signal; would not change the billed code but should be corrected via amendment or addendum.
- `🟠 HIGH` — Documentation element affects an E/M element, a billable diagnosis, or HCC capture; requires clinician review and either correction or explicit attestation before the claim is finalized.
- `🔴 CRITICAL` — Fabricated exam, fabricated diagnosis, fabricated procedure, fabricated history element, or documentation of a service not provided. Stop the claim; engage compliance. Also includes any assessment content that contradicts the patient's stated concerns or the structured chart.

### 1. Audit Header

- Audit date, auditor (or "automated pre-review"), note encounter date, clinician, specialty, encounter type (new/established/inpatient/telehealth), ambient tool name if known, audit scope, and which inputs were provided. Flag `LIMITED-INPUT AUDIT` if inputs 3–5 are missing.

### 2. Overall Drift Summary

Two to four sentences. Describe at a high level whether the ambient-drafted note appears aligned with the encounter signal, partially aligned, or materially drifted. End with a single-line verdict:

```
VERDICT: [ALIGNED / MINOR DRIFT / MATERIAL DRIFT / FABRICATION RISK]
```

If the verdict is `MATERIAL DRIFT` or `FABRICATION RISK`, the next sections focus on the specific findings; if it is `ALIGNED`, the findings section is still completed but short.

### 3. Element-by-Element Findings

For each major note element, compare drafted content against ground-truth signal. Use this structure:

```
[Element] — HPI / ROS / PMH-FH-SH / Exam / A&P / Orders / Patient Instructions
  Drafted: [one-line excerpt or paraphrase — never reproduce > 15 words verbatim from the note]
  Signal: [what the structured data / transcript / agenda supports]
  Finding: [supported / partially supported / unsupported / contradicted]
  Risk: [🟢 / 🟡 / 🟠 / 🔴]
  Reason: [specific — e.g., "ROS lists 10 systems negative; encounter agenda is focused MSK visit for knee pain; 10-system ROS is one of the elements supporting the billed 99214."]
  Suggested action: [clinician amendment / addendum / down-level / remove diagnosis / attestation / compliance escalation]
```

Run this for every element where there is a non-trivial discrepancy. Do not fill space with green-light lines for every sentence.

Specific patterns to watch for (each maps to a known 2026 ambient-scribe drift risk):

- **ROS inflation** — 10-system negative ROS appended to a focused encounter. Common ambient behavior; HIGH risk if it supports an E/M level.
- **Phantom exam elements** — Documentation of an exam component (e.g., fundoscopic, neuro, full skin, genitourinary) that the visit's agenda, duration, and orders make implausible. CRITICAL if it supports a higher E/M.
- **Phantom diagnoses on the A&P** — Diagnoses listed in the assessment that appear nowhere in the structured problem list, patient's stated concerns, orders, or labs. CRITICAL if billed; HIGH if captured but not billed (still a chart-integrity issue).
- **HCC drift** — A risk-adjustment-relevant diagnosis (diabetes with complications, CHF, CKD stage 3+, major depression, vascular disease, amputation status) added to the A&P without an MEAT element (Monitored, Evaluated, Assessed, Treated). HIGH risk; stop-the-claim if the HCC changes the risk score and no MEAT is documented.
- **E/M level drift** — MDM-based level that the encounter complexity does not support, or time-based level where the documented time is implausible given the encounter. HIGH to CRITICAL depending on magnitude.
- **Time documentation** — A stated time that exceeds plausible encounter duration, or a time that conflicts with adjacent encounters on the same day.
- **Procedure documentation** — A billable procedure (e.g., joint injection, skin biopsy, ECG interpretation, destruction of lesion) documented without corresponding order, consent, or product-use record. CRITICAL unless an order or supply charge corroborates.
- **"Copy-forward" drift from prior notes** — HPI or A&P language that matches a prior encounter verbatim and contradicts today's chief complaint.
- **Assessment contradicting the patient's concern** — The patient's stated problem does not appear in the A&P, or the A&P introduces a problem the patient did not raise.
- **Patient education / AVS fabrication** — "Discussed ____, patient verbalized understanding" for topics not on the encounter agenda.
- **Counseling/Coordination of Care inflation** — "50% of visit spent in counseling about ____" language that the encounter signal does not support (historical pre-2021 E/M language that still surfaces in drafts).

### 4. Coding & Billing Impact

Map the findings to the billed CPT, E/M level, ICD-10/HCC capture, and any modifiers. Produce:

- Billed E/M level and whether it is supported by documentation *that is itself supported* by ground-truth signal
- Recommended E/M level range if drift is present (lowest defensible, highest defensible), with rationale
- ICD-10 codes billed that are supported vs. unsupported; flag HCC-relevant items with 🟠 or 🔴 as appropriate
- Modifier checks (e.g., modifier 25 where the additional E/M is not substantiated by separate A&P content)
- Time-based vs. MDM-based billing — if mixed signals, recommend which is defensible
- Estimated financial exposure range if the highest-risk findings were subject to a payer takeback, expressed as a per-encounter range, clearly flagged as a *compliance estimate, not a legal opinion*

### 5. Recommended Remediation

Produce up to five concrete actions, in priority order:

- Pre-bill correction items (what the clinician should amend or addend before the claim drops)
- Dual-signature or attestation requests (who signs what)
- Whether to hold the claim pending review
- Whether the finding rises to the level of routine-feedback vs. compliance-escalation (e.g., OIG-level) per the organization's policy
- Whether this single note is part of a trend that warrants a broader sample (e.g., "pull next 10 99215s from this provider for pattern check")

End with two to four lines of *provider-friendly feedback language* the CDI team can paste into a feedback message — written as coaching, not blame. Never write language that impugns the provider's intent; ambient-scribe drift is a tool-and-workflow problem first.

### 6. Audit Trail & Export

Produce a short audit-trail block the team can paste into the EHR audit log or an external QA tracker:

```
Note ID: [xxxx]  Encounter date: [yyyy-mm-dd]  Provider: [NPI or internal ID]
Ambient tool: [name]  Input completeness: [full / partial / limited]
Verdict: [ALIGNED / MINOR DRIFT / MATERIAL DRIFT / FABRICATION RISK]
Highest finding risk: [🟢 / 🟡 / 🟠 / 🔴]
Recommended claim action: [release / hold / amend-then-release / escalate to compliance]
Reviewer: [name]  Review date: [yyyy-mm-dd]
Follow-up trigger: [none / 30-day recheck / broader sample / compliance review]
```

## Safety, Policy, and Fairness Guardrails

- Never reproduce > 15 words verbatim from the note; paraphrase for excerpts. Full-text quotations belong in a compliance-protected audit tool, not in the report body.
- The skill does not guess a provider's intent. All findings are framed as documentation-vs-signal gaps, not accusations.
- If the ambient transcript was used, the skill never reproduces patient voice verbatim beyond short de-identifiable snippets and honors any local policy on transcript use.
- The skill flags its own limits: when inputs 3–5 are missing, risk ratings are conservative (MEDIUM at most for coding impact) and the verdict ceiling is `MINOR DRIFT` unless there is direct evidence of fabrication.
- Pediatric, obstetric, behavioral-health, and oncology encounters often have legitimate high-complexity MDM that looks like "drift" to a naive audit. The skill applies specialty-adjusted thresholds and will state the adjustment in the audit header.
- The skill does not judge any individual provider's pattern from one note. Provider-level trend claims require N ≥ 10 notes and are explicitly opted in.
- The financial-exposure estimate is a range, not a liability determination. Legal exposure is out of scope.
- Output is a compliance work-product. It should be routed through the organization's attorney-client-privileged QA channel where applicable.

## Example Output (abridged)

```
=== Ambient Scribe Note Audit ===
Audit date: 2026-04-15   Auditor: CDI – J. Lopez
Encounter date: 2026-04-11   Clinician: Dr. R. Patel, Family Medicine
Ambient tool: [vendor]   Note type: Established patient, level 4 (99214)
Input completeness: full (signed note + structured chart + agenda + transcript excerpt)
Audit scope: Pre-bill random sample, 2026-Q2, 10% sample

Overall drift summary:
The ambient-drafted note is partially aligned with the encounter signal. The chief complaint (back pain) and orders (lumbar MRI, cyclobenzaprine) are supported. The ROS and exam content are not. The billed 99214 is defensible at 99213 on MDM, and 99214 only if the ROS and exam inflation are corrected to match the encounter.

VERDICT: MINOR DRIFT

--- Element-by-element findings ---

Element: ROS
Drafted: "Ten-system ROS, all negative except MSK."
Signal: Agenda and transcript focus on back pain x 6 weeks; no other systems discussed.
Finding: Unsupported.
Risk: 🟠 HIGH (contributes to billed E/M level)
Reason: The 10-system ROS is one of the data elements supporting 99214 complexity; the encounter signal does not support a 10-system review.
Suggested action: Amend ROS to pertinent-positive MSK + relevant pertinent negatives only; reassess E/M level accordingly.

Element: Exam
Drafted: "Full neurologic exam including cranial nerves II–XII, sensory, motor, reflexes, gait, Romberg."
Signal: Transcript captures strength and straight-leg raise; no evidence of cranial-nerve or Romberg assessment.
Finding: Partially supported (strength/SLR yes; CN exam no; Romberg no).
Risk: 🟠 HIGH
Reason: Documentation of exam components not performed is a fabrication risk and may affect MDM complexity.
Suggested action: Amend exam to only what was performed: focused MSK + distal neurovascular exam of the lower extremities.

Element: Assessment & Plan
Drafted: "1) Lumbar radiculopathy, likely L5; 2) Obesity, class I; 3) Pre-diabetes."
Signal: Lumbar radiculopathy is supported. Obesity class I is plausible (BMI in chart). Pre-diabetes: HbA1c not drawn today, no counseling documented in transcript beyond a passing reference by the patient.
Finding: Items 1 and 2 supported; item 3 is an HCC-relevant diagnosis added without documented MEAT.
Risk: 🟠 HIGH
Reason: HCC/risk-adjustment capture without MEAT is a known 2026 drift pattern and a payer audit target. If the claim is submitted with this code and no supporting element, the organization carries avoidable exposure.
Suggested action: Either (a) remove pre-diabetes from today's A&P, or (b) amend with a 1-sentence MEAT statement reflecting today's assessment and plan if clinically accurate.

--- Coding & billing impact ---

Billed: 99214  ICD-10: M54.16, E66.01, R73.03 (pre-diabetes)
Documentation supports: 99213 on MDM after ROS/exam corrections; 99214 only if corrected documentation still supports moderate-complexity MDM.
Recommended range: 99213 (lowest defensible) – 99214 (highest defensible after amendment).
HCC exposure: E66.01 is non-HCC; R73.03 is non-HCC. Low risk-adjustment exposure on this note.
Modifier 25: not used. OK.
Estimated financial exposure if uncorrected: ~$30–$45 per encounter between 99213 and 99214 at the practice's 2026 Medicare rates (compliance estimate, not legal opinion).

--- Recommended remediation ---

1. Amend ROS to pertinent-positive/pertinent-negative only; drop the 10-system line.
2. Amend exam to the components actually performed.
3. Either remove R73.03 from the A&P or attach a MEAT statement from the encounter.
4. Re-evaluate whether 99214 remains defensible on MDM after corrections; if not, submit at 99213.
5. Do not release claim until items 1–4 are resolved.

Provider-friendly feedback (coaching):
"Today's back-pain encounter was appropriately coded on complexity, but the ambient-drafted ROS and exam included elements that weren't actually performed. A quick amendment to reflect only what you did will keep the note compliant and protect the E/M level. Also — the ambient tool added pre-diabetes to the A&P; because it's a risk-adjustment–adjacent code, please either remove it or add a one-line MEAT statement if you're addressing it today."

--- Audit trail ---

Note ID: 2026-0411-xxxx   Encounter: 2026-04-11   Provider: [NPI]
Ambient tool: [vendor]   Input completeness: full
Verdict: MINOR DRIFT   Highest finding: 🟠 HIGH
Recommended claim action: hold; amend; release at 99213 or 99214 per post-amendment MDM
Reviewer: J. Lopez, CDI   Review date: 2026-04-15
Follow-up trigger: pattern check if 3+ similar ROS/exam findings in next sample
```

## Nursing-Specific Bias & HITL Audit Extension (v1.1)

Added April 24, 2026. Use this extension when the ambient-drafted note under audit is a nursing note (RN, LPN, APRN), a nurse-led handoff, or a nurse-authored patient-education / discharge-education artifact produced with an ambient tool. The extension applies the February 25, 2026 American Academy of Nursing (AAN) AI Position Statement's Human-in-the-Loop (HITL) and bias-evaluation expectations to the audit — nurses signing AI-assisted documentation retain full legal and ethical responsibility for each entry, so the audit surfaces drift patterns that are specifically nursing-side and specifically bias-shaped.

Trigger this extension whenever any of the following is true:
- The author of record is a nurse (RN, LPN, APRN) or a nursing-led team (charge nurse handoff, care-manager documentation, school nurse, home-health visit note)
- The note is the nurse-handoff (I-SBARR) artifact already covered by `nurse-shift-handoff-isbarr.md` and an audit is being run against the ambient output
- The encounter involves a patient from a population historically harmed by healthcare-AI performance gaps (Black, Indigenous, Hispanic/Latine, LEP, deaf/hard-of-hearing, older adult, pediatric, disabled)
- The ambient tool has been reported (internally or in published literature) to transcribe, summarize, or prioritize differently by dialect, accent, or phrasing — the "Invisible Scribes" pattern

When this extension is triggered, add Section 7 (Nursing-specific findings) and Section 8 (Bias & HITL attestation) to the standard six-section report.

### Section 7: Nursing-specific findings

For each of the following, score independently of the E/M coding focus in Sections 3–4:

1. **Nursing scope & standardized language** — Drafted content uses nursing-specific language (NANDA-I, NIC, NOC, nursing process) where the structured chart expects it; ambient note does not substitute physician-style A&P language for nursing-process documentation. Risk 🟡–🟠 if a scope-of-practice mismatch could affect a licensing board review.
2. **Handoff integrity (I-SBARR)** — If the note is a handoff, the Introduction–Situation–Background–Assessment–Recommendation–Readback elements are each represented and supported by the encounter signal. Readback / confirmation element is captured. Risk 🟠 if a safety-critical element is missing or fabricated.
3. **Pain-assessment, behavioral, and patient-voice fidelity** — The ambient note's paraphrase of patient-reported pain, distress, or symptom language does not systematically under-score or re-frame a patient's own words. Red flag: patient states "worst pain of my life" and ambient draft records "moderate discomfort." Risk 🟠–🔴.
4. **Skin / wound / fall-risk / pressure-injury documentation** — Findings match bedside-assessment signal (photo log, pressure-injury scale, fall-risk tool). Ambient tool did not default to "skin intact" when the bedside assessment captured otherwise. Risk 🟠 if a CMS-reportable hospital-acquired condition could be mis-documented.
5. **Medication administration & titration** — Times, doses, and routes match MAR / smart-pump log; ambient note did not collapse multiple administrations into one line. Risk 🔴 if a controlled-substance or high-alert med event is mis-documented.
6. **Education & teach-back** — "Patient verbalized understanding" statements are only present where the encounter signal actually captured a teach-back turn. Risk 🟠 if fabricated teach-back supports a discharge-readiness decision.

### Section 8: Bias & HITL attestation

A short block (6–12 lines) that reports:

- **Demographic context** — Patient's self-reported race, ethnicity, preferred language, and any accommodation needs, pulled from the structured chart (do not infer from voice or free text).
- **Transcript-fidelity signal** — Whether the ambient tool's transcript shows signs of the known bias-shaped failure modes ("Invisible Scribes" patterns): dropped turns in non-English or accented English, summarization that omits the patient's own phrasing, prioritization that promotes clinician voice over patient voice. If no transcript is available, state that explicitly — *absence of testability is itself a finding.*
- **HITL attestation** — Confirms that the signing nurse reviewed the ambient draft, verified it against bedside assessment, and accepts full professional responsibility per AAN 2026. The audit does not accept "auto-accept" as HITL. If the note was auto-signed without a documented review turn, risk is 🟠 by default.
- **Governance trigger** — Whether the finding should be added to the deployer's AI Tool Registry performance log and whether a trend escalation to the nursing-led AI oversight committee is warranted (yes if ≥ 2 bias-shaped findings in a rolling 10-note sample).

Output of Sections 7–8 feeds back into the Section 2 verdict: if any nursing-specific finding is 🔴, the overall verdict is `FABRICATION RISK` regardless of coding impact; if ≥ 2 bias-shaped findings surface in Section 8, the overall verdict is at minimum `MATERIAL DRIFT` and the audit is flagged for governance review.

## Notes on Fit and Limits

- Intended for CDI, coding, compliance, revenue-cycle, and (v1.1) nursing-led AI oversight teams. Not a clinical decision-support tool.
- Operates on documents, not audio. Where transcript access is permitted by policy and vendor agreement, audit quality improves substantially; otherwise the skill operates on structured encounter data plus the signed note only.
- Complements, rather than replaces, the existing Coding Review Assistant skill (which focuses on ICD-10 / HCC / DRG code selection). This skill focuses on the *documentation-to-coding* integrity path specifically in an ambient-authored note.
- Complements, rather than replaces, payer-specific denial rebuttal work. When a payer downcodes a claim based on ambient-drafted content, use this skill's output as an input to the Denial Appeal Letter Writer skill.
- v1.1 extension complements, rather than replaces, the `nurse-shift-handoff-isbarr.md` skill — handoff drafting is a different workflow than handoff auditing. Use I-SBARR handoff skill to draft; use this skill (Sections 7–8) to audit.
- Anti-plagiarism: output is original paraphrasing and structured risk rating. Never reproduces vendor templates, payer policy language, clinical note content, or AAN position-statement text verbatim.
