---
name: "SDOH Risk Assessment Summarizer"
category: operations
tools: [claude, chatgpt]
difficulty: intermediate
time_saved: "~10 min/encounter"
version: 1.0
last_eval_score: null
---

# 🏠 SDOH Risk Assessment Summarizer

## Purpose

Turn a completed Social Determinants of Health (SDOH) screening — the G0136 standardized risk assessment or equivalent — plus relevant chart context into a concise, actionable SDOH summary that a clinician, care manager, or community health worker can use at the point of care. The output groups findings by domain, tags billable risk positives, suggests referral categories and standardized Z codes, and drafts patient-facing language at an appropriate reading level.

## When to Use

Use this skill in the outpatient or transitional care setting whenever a SDOH assessment needs to be summarized, translated into a care plan, or prepared for billing. Common scenarios include:

- Annual wellness visit or initial preventive physical exam with an embedded SDOH screen
- Medicare beneficiary receiving the G0136 standardized SDOH risk assessment (5–15 minutes, no more often than every 6 months)
- Medicaid population with required SDOH screening or state-specific quality measure
- Hospital discharge planning where social risk factors will affect a safe transition
- Pre-visit chart prep for a complex primary care or chronic disease patient
- Care management outreach where the care manager needs a quick ground-truth summary before calling the patient
- FQHC and community health center workflows meeting CMS outpatient SDOH reporting requirements
- ACO, value-based care, or Medicare Advantage programs tracking Z-code capture and closed-loop referrals
- Behavioral health intake where housing, food, safety, and transportation risks commonly drive treatment planning

This skill summarizes and organizes. It does not replace the clinician's judgment about which risks require intervention, does not generate referrals without human review, and does not infer SDOH positives that the patient did not actually disclose.

## Required Input

Provide the following:

1. **Screening instrument results** — The completed SDOH screening responses (raw scores or domain-level positives). Common instruments: PRAPARE, AHC-HRSN, Accountable Health Communities Screening, Hunger Vital Sign, PHQ-2/9 if bundled, or an EHR-native screener. Include the instrument name, date, and who administered it.
2. **Patient context** — Age, preferred language, household composition, insurance type, relevant diagnoses (diabetes, CHF, CKD, pregnancy, behavioral health), current medications that may be affected by social risk (e.g., insulin, anticoagulants, cold-chain biologics), and the visit type.
3. **Chart-based signals** — Any unstructured notes that corroborate or contradict the screen (missed appointments, reports of utility shut-off, transportation complaints, ED visits related to medication nonadherence, prior Z codes billed).
4. **Available resources** — A short list of referral partners the practice uses or a statement that the community resource directory is unknown. The skill will recommend referral categories regardless, but will tailor suggestions when resources are known.
5. **Output target** — Clinician-facing summary, care-manager handoff, patient-facing language, or all three.

If the screening result is missing, return a request for the completed instrument rather than fabricating responses. If a patient declined to answer specific items, preserve that as "declined" — never impute a positive or negative.

## Instructions

Follow this six-section structure. Do not reproduce patient quotes verbatim beyond short, de-identifiable snippets.

### 1. 30-Second Clinical Snapshot

One paragraph, no more than four sentences. Name the highest-acuity SDOH risks (food, housing, utilities, transportation, safety, IPV, financial strain, employment, education, social connection) that were positive, the single most immediate action if any (e.g., "patient reports active IPV, safety planning indicated"), and whether any social risk is actively interfering with a medical plan (e.g., "insulin refrigeration at risk due to reported utility shut-off notice"). Do not list every domain here — focus on what changes the visit.

### 2. Domain-by-Domain Summary

For each screened domain, present:

```
Domain: [e.g., Food Insecurity]
Screen positive? Yes / No / Declined / Not screened
Severity signal: [risk-positive, at-risk, stable based on instrument thresholds] [VERIFY — confirm scoring rules]
Corroborating chart signals: [1–2 sentences, or "none noted"]
Suggested Z code(s): [ICD-10-CM Z59.41 Food insecurity, Z59.48 Other specified problem related to inadequate food and water, etc.] [VERIFY coder confirms current-year code set]
Suggested referral category: [food pantry, WIC/SNAP enrollment assistance, Meals-on-Wheels, medically tailored meals] [tailor to practice's known resources if provided]
Patient-stated preference: [if captured; otherwise "not captured"]
```

Repeat for every domain in the instrument, including domains where the patient declined. Typical domains to include when present in the screener:

- Food insecurity
- Housing stability and housing quality (mold, heat, pests)
- Utilities (power, heat, water shut-off)
- Transportation (to medical appointments and to pharmacy/grocery)
- Intimate partner violence and personal safety
- Financial strain and material hardship
- Employment and income disruption
- Education and health literacy (if captured)
- Social isolation / loneliness
- Legal needs (immigration, eviction, custody) if captured
- Substance use risk if bundled (e.g., TAPS, AUDIT-C)
- Behavioral health risk if bundled (e.g., PHQ-2/9, GAD-7)
- Caregiver strain (for medically complex patients or pediatrics)

### 3. Risk-to-Medical-Plan Crosswalk

For each positive SDOH risk, show how it intersects the patient's medical plan. Example rows:

```
Risk                    | Medical Plan Touchpoint         | Recommended Adjustment
------------------------|----------------------------------|---------------------------------------
Utility shut-off risk   | Insulin glargine refrigeration   | Switch to room-temp-stable option if clinically appropriate; expedite social work referral
Transportation barrier  | Monthly HbA1c / CMP draws        | Offer phlebotomy home visit or coordinate transportation benefit
Food insecurity         | Newly diagnosed T2DM counseling  | Refer to medically tailored meals + RD; adjust teaching to low-cost, non-perishable plan
IPV disclosure          | Medication storage, follow-up    | Private callback number, safety plan, avoid mailing materials to home
```

If the plan is complex, preserve one row per material interaction. If no interaction exists, state "no direct medical-plan intersection at this time."

### 4. Coding & Billing Hand-Off

- List every Z code that is supported by documented screening positives, with the specific documentation snippet that justifies it.
- Flag the G0136 eligibility status: was the screen administered, was it a qualifying standardized instrument, was it within the 6-month frequency limit, is the 5–15 minute time element documented? `[VERIFY — biller confirms current-year coverage and modifiers]`
- If the encounter is part of a value-based contract with SDOH quality measures (e.g., NCQA SDOH-E, state Medicaid measures), list which measure this encounter helps close.
- Never bill a Z code on the basis of suspected risk that was not disclosed. Flag as "consider re-screening" instead.

### 5. Patient-Facing Summary and Referral Language

Produce a short, plain-language summary that a clinician or care manager can share verbally or in the after-visit summary, targeted to a 6th–8th grade reading level, in the patient's preferred language where feasible. Example structure:

```
Today you and Dr. ___ talked about some things outside of medicine that can make it harder
to stay healthy. Here is what we heard and what we can do together.

What we heard:
- [plain-language reflection of each positive domain, one bullet each]

What we will do:
- [each referral or next step, with "who will call you" and "by when" if known]

What you can do if things change:
- [one sentence on how to reach the team, crisis lines where relevant]
```

For IPV disclosures, safety-plan the written output: avoid language that discloses the conversation in a shared home, offer to send nothing in writing, and confirm the safest phone/email contact path.

### 6. Closed-Loop Referral Tracking Block

Output a structured block the practice can drop into the EHR so the referral loop can be tracked:

```
Referrals placed today (pending closure):
- [partner/program], [domain], [who will call patient], [target contact date], [method: phone/secure message/warm handoff]

Referrals patient declined:
- [domain], [reason if stated]

Re-screen in: [e.g., 6 months per G0136 frequency; or sooner if clinically indicated]
```

### Safety & Privacy Flags

Automatically surface these flags:

- `[SAFETY — IMMEDIATE]` for active IPV, suicidal ideation, homicidal ideation, child safety, or elder abuse disclosures — with a note that the clinician, not this skill, must initiate the safety plan and any mandated reporting.
- `[SENSITIVE CHANNEL]` for any risk that changes safe contact method (IPV, stigma around substance use or HIV, immigration concerns). Suggest private callback, encrypted portal, or in-person only follow-up.
- `[PHI MINIMIZATION]` on any downstream draft (patient-facing language, referral text) to ensure the minimum necessary information is shared with the community partner.
- `[NO IMPUTATION]` — never infer a SDOH positive from race, ethnicity, ZIP code, or insurance class alone.

### Anti-Error Guardrails

- Never generate a Z code that is not tied to a documented screening positive.
- Never auto-populate a referral to a specific organization unless the user provided that resource in input; offer a generic category with "practice-specific resource needed here" otherwise.
- Never suggest mandated reporting language unilaterally — surface the clinical concern and defer the decision to the clinician.
- If the screener includes a suicide-risk item and the patient endorsed it, output only the `[SAFETY — IMMEDIATE]` flag plus a reminder to follow the practice's suicide-risk protocol. Do not generate patient-facing language for this domain.
- Keep patient-facing language at or below the practice-specified reading level (default 6th–8th grade). Avoid clinical jargon, acronyms, and Z codes in patient-facing text.

## Example Output

```
30-Second Snapshot
Mr. R. is a 67-year-old Spanish-preferred Traditional Medicare patient with T2DM and CHF who
screened positive for food insecurity and transportation barriers and reports a pending utility
shut-off notice. The utility risk threatens insulin refrigeration; social work consult placed
today. No IPV, suicidality, or active safety risk disclosed.

Domain-by-Domain
- Food insecurity: positive. Hunger Vital Sign 2/2. Z59.41. Refer to medically tailored meals
  via [practice partner]. Patient accepted.
- Housing stability: negative.
- Utilities: positive. Reports APS shut-off notice within 14 days. Z59.12 [VERIFY current code].
  Social work to assist with LIHEAP application today.
- Transportation: positive. Missed 2 of last 5 appointments due to transportation. Z59.82.
  Enrolled in plan-provided non-emergency medical transport starting 2026-04-18.
- IPV / safety: declined.
- Financial strain: positive. Z59.86. RX cost review scheduled with pharmacist.
- Social isolation: negative.

Risk-to-Medical-Plan Crosswalk
Utility shut-off risk | Insulin glargine cold chain | If shut-off occurs, switch to insulin
degludec pen (room-temp-stable) — [VERIFY with prescriber before substitution]
Transportation       | Q3-month HbA1c / BMP        | Arrange lab home-draw or align with transport schedule
Food insecurity      | T2DM counseling             | Refer to MTM meals; update teaching to low-cost, low-carb plan

Coding Hand-Off
- G0136 administered by social worker, 12 min, standardized Hunger Vital Sign + AHC-HRSN short form [VERIFY billing rules]
- Z codes: Z59.41, Z59.12, Z59.82, Z59.86 — each supported by documented screen response
- Value-based contract: counts toward plan SDOH-E Z-code capture measure for 2026

Patient-Facing Summary (Spanish preferred — translate before sharing)
[plain-language text per template]

Closed-Loop Referrals
- Medically tailored meals (food) | CM J. Alvarez calls by 2026-04-16 | warm handoff
- LIHEAP utility assistance | social work calls by 2026-04-15 | phone
- NEMT enrollment (transportation) | health plan rep calls by 2026-04-18 | phone
Re-screen in 6 months (G0136 frequency limit).
```

## Notes on Policy & Instruments

- CMS defines G0136 as the standardized, evidence-based SDOH Risk Assessment, 5–15 minutes, no more often than every 6 months. Any billing and reimbursement assertions must be validated against the current-year fee schedule by a credentialed coder.
- CMS outpatient SDOH reporting expectations expanded in 2026; practices should confirm their specific measure set (hospital inpatient, Medicare Advantage Star, Medicaid MCO contracts, ACO REACH) since each has distinct thresholds and specs.
- NLP-based extraction of SDOH from unstructured notes is common in 2026 toolchains, but this skill never reclassifies a "not screened" domain as positive based on notes alone — any such signal is surfaced as "consider re-screening."
- The AHC-HRSN, PRAPARE, and similar screeners are public-domain; the skill works with their result fields without reproducing the instrument verbatim.
- Nothing in this skill replaces clinician judgment, mandated reporting workflows, or the practice's compliance program.
