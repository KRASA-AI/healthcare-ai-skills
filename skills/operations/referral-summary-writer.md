---
name: "Referral Summary Writer"
category: operations
tools: [claude, chatgpt]
difficulty: beginner
time_saved: "~10 min/referral"
version: 2.3
last_eval_score: 8.3
---

# ✉️ Referral Summary Writer

## Purpose

Compile a patient's relevant history, clinical findings, workup results, and specific consultation questions into a concise, well-organized referral letter that gives the receiving specialist everything they need to prepare for the consultation.

## When to Use

Use this skill whenever a provider needs to refer a patient to a specialist or another facility. Common scenarios include:

- Primary care referral to a specialist for evaluation or management
- Specialist-to-specialist referral for a second opinion or co-management
- Referral to a surgical service for operative evaluation
- Behavioral health or psychiatric referral with clinical context
- Referral to ancillary services (PT/OT, speech therapy, nutrition, pain management)
- Transfer of care referral when a patient is relocating or changing providers
- Pre-operative clearance requests to cardiology, pulmonology, etc.

## Required Input

Provide the following:

1. **Patient information** — Name, DOB, and relevant demographics
2. **Referring provider** — Name, specialty, and contact information
3. **Receiving provider or specialty** — Name or specialty being referred to, and facility if known
4. **Reason for referral** — The specific clinical question, concern, or service being requested
5. **Relevant clinical history** — Pertinent medical history, current diagnoses, and active problem list related to the referral reason
6. **Workup and findings** — Relevant exam findings, lab results, imaging reports, or diagnostic test results that inform the referral
7. **Current treatment** — Medications and treatments already tried for the condition in question, with outcomes
8. **Specific questions** (optional) — Particular questions the referring provider wants addressed
9. **Urgency** (optional) — Routine, urgent, or emergent, with clinical rationale if non-routine
10. **Insurance/authorization status** (optional) — Whether prior auth is needed or has been obtained

## Instructions

You are a skilled healthcare professional's AI assistant. Your job is to draft a clear, concise referral letter that communicates the clinical picture efficiently so the receiving provider can prepare for the consultation without unnecessary chart-diving.

**Before you start (personalization from `config.yml`):**

Read these named hooks once. If a hook is absent, fall back to the default and surface every facility-specific element as a `[VERIFY: ...]` flag — never invent a referral partner, NPI, intake fax, secure-EHR network, urgency threshold, payer prior-auth path, or boilerplate consult question.

- `referral_partners` — keyed list of preferred specialists or groups the practice routes to by category (e.g., `cardiology: "Cardiology Associates of [City] — Dr. J. Park"`, `orthopedics_spine: "Regional Spine Group — intake fax [xxx]"`, `behavioral_health: "[Community CMHC] — secure EHR inbox"`, `dermatology_mohs`, `gi_screening`, `endocrinology`, `pulmonology`, `nephrology`, `pain_management`). If the user did not name a receiving provider, default to the mapped partner for the relevant category. If the category is not mapped, produce the letter addressed to the specialty generically and flag `[VERIFY: practice-specific referral network]`.
- `referral_channel_defaults` — per-partner routing: `secure_ehr` (DirectTrust / Epic Care Everywhere / EHR-to-EHR message), `fax`, `portal_upload`, `directtrust`, or `hand-carry`. Use the partner's default; otherwise default to secure EHR if the partner is on the same network and fax otherwise.
- `urgency_thresholds` — practice overrides for what counts as routine / urgent / emergent for common conditions (e.g., new-onset AFib without instability → routine-7-day default; new focal neuro deficit → emergent same-day; new breast lump → urgent within 7 days; positive FIT → urgent colonoscopy ≤ 90 days; positive depression screen with passive SI → urgent same-week behavioral health; positive depression screen with active SI/HI/plan → emergent same-day with safety planning). Respect practice override rules over defaults.
- `preferred_consult_questions` — specialty-specific boilerplate questions the practice asks on every referral of that type (e.g., for every cardiology AFib referral: "confirm rate vs. rhythm strategy, recommend anticoagulant given renal function, specify follow-up cadence"). Blend boilerplate with patient-specific questions so every letter includes both.
- `practice_specialty` — referring side (primary care / internal medicine / pediatrics / OB-GYN / ED / urgent care / specialty-to-specialty). Drives the depth of the clinical-summary block, the default ICD-10 set, and which boilerplate question library applies.
- `provider_signature_blocks` — keyed per signing clinician role (attending physician with NPI / direct line / DirectTrust; APP with collaborating-physician attestation; resident with attending co-sign per facility bylaws). Pulls NPI, direct line, secure-message address, fax, and credentials into the signature block.
- `payer_prior_auth_routing` — keyed payer routing for the receiving service: `payer_portal` (specific URL), `fax_to_payer`, `eviCore` / `Carelon` / `Cohere` for utilization-management vendors, `peer_to_peer_callback_window` defaults. Flags whether prior auth is the referring or receiving practice's responsibility per the practice's payer-side memo.
- `state_privacy_overlays` — TX HB 300, CA CMIA, NY SHIELD, IL BIPA, WA My Health My Data Act, CO HB24-1054, plus 42 CFR Part 2 (substance use), state mental-health, HIV, genetic, minor-consent, and reproductive-health overlays. Apply the strictest applicable; segregate Part 2 / state-overlay-protected content into a separate consent-required attachment rather than inline.
- `language_preference_routing` — patient's preferred language. If unsupported, produce English with translation-ready formatting and route through the certified-translation workflow rather than shipping a machine translation in the referral letter or accompanying patient-facing summary.
- `accompanying_records_default` — `attach_summary + ECG + relevant labs + recent imaging` keyed per specialty (e.g., always send the most recent A1c and renal panel with endocrinology referrals; always send the relevant imaging report and the actual study link with surgical referrals; always send the validated PHQ-9 / GAD-7 / C-SSRS with behavioral-health referrals). Skill names the records and flags `[VERIFY: records attached]`.
- `closing_loop_protocol` — practice's loop-closure expectation (e.g., "consult note expected back via secure EHR within 7 days; if not received, care-coordinator follows up with receiving practice"). Inserted into the closing block.
- `output_destination` — `outputs/`, `chart_attachment`, `secure_ehr_routing`, `fax_cover_packet`, `patient_portal_copy_to_patient`, or `multi_artifact` (clinical letter + plain-language patient summary at 6th–8th grade).
- `config_missing_behavior` — `flag_and_proceed` (default — ship a complete letter with `[VERIFY: ...]` flags on every facility-specific element) vs. `block_and_ask`.

When `config.yml` is absent entirely, produce a generalist primary-care-to-specialist referral letter with the AFib-cardiology-style numbered consult-question block, secure-EHR-default routing if the receiving partner is on the same network and fax otherwise, generic provider signature block, and `[VERIFY: ...]` flags on every facility-specific element (referral partner, intake fax, NPI, payer PA path, accompanying records). Never invent a referral partner, NPI, intake fax, payer routing, or peer-to-peer callback window.

**Process:**

1. Review all input provided by the user
2. Do NOT ask clarifying questions unless the referral reason is unclear. Make reasonable assumptions and note them. Speed matters — referrals are high-volume
3. Structure the referral letter with the following components:

   **a. Header**
   - Date and referring facility letterhead (from config)
   - Receiving provider or department name and address
   - RE line: "Referral for [Patient Name] — [Reason for Referral]"

   **b. Opening**
   - Brief introduction identifying the referring provider and their relationship to the patient
   - Clear statement of the referral reason and what is being requested (evaluation, co-management, surgical opinion, procedure, etc.)
   - Urgency level if non-routine

   **c. Clinical Summary**
   - Relevant medical history focused on the reason for referral (not a full chart dump)
   - Active problem list and pertinent comorbidities
   - Current medications relevant to the referral condition
   - Allergies if relevant to anticipated treatment decisions

   **d. Workup & Findings**
   - Key exam findings, organized by relevance
   - Lab results with dates and trends if applicable
   - Imaging or diagnostic results with dates
   - Treatments attempted for this condition, with duration and outcomes

   **e. Specific Questions / Request**
   - Numbered list of specific questions or requests for the specialist
   - If no specific questions provided, generate appropriate ones based on the clinical context (e.g., "Would this patient benefit from surgical intervention?" or "Please advise on optimal biologic therapy given prior TNF-inhibitor failure")

   **f. Closing**
   - Offer to provide additional records or discuss the case
   - Preferred method of receiving the consultation report (fax, EHR message, etc. from config)
   - Referring provider signature block with credentials, NPI, and direct contact

4. Keep the letter to one page when possible — specialists receive high volumes of referrals and value brevity
5. Use standard clinical abbreviations to save space
6. Include ICD-10 codes for the referral diagnosis where identifiable

**Output requirements:**
- Professional referral letter format on facility letterhead (from config)
- Concise — ideally one page, no more than two
- Clinically focused on the referral question, not a full medical history
- ICD-10 codes included for the referral diagnosis
- Specific consultation questions included (generated if not provided)
- Ready for provider review and signature with minimal editing
- Saved to `outputs/` if the user confirms

## Example Output

```
[Practice Letterhead — from config.yml]

2026-04-14

John Park, MD
Cardiology Associates of [City]
[Address]
Fax: [fax] • Secure EHR: [network]

RE: Referral for Maria L. — new-onset atrial fibrillation, rate control and
anticoagulation strategy

Dr. Park,

I am referring Maria L. (DOB 1961-03-22, MRN 4417XXXX) for cardiology
evaluation of new-onset atrial fibrillation diagnosed on outpatient ECG on
2026-04-08. Routine urgency. Rate currently controlled; patient is clinically
stable.

Active problems relevant to referral
- New-onset atrial fibrillation (I48.91), first documented 2026-04-08
- HTN (I10), controlled on lisinopril 20 mg daily
- T2DM (E11.9), HbA1c 6.8% (2026-02-18)
- CKD stage 3a, eGFR 51 (2026-02-18)
- No prior CVA/TIA, no known structural heart disease, non-smoker, no prior bleeding

Current medications
- Lisinopril 20 mg daily • Metformin 1000 mg BID • Atorvastatin 40 mg QHS
- No anticoagulant or antiplatelet at this time

Workup to date
- ECG 2026-04-08: atrial fibrillation, rate 96, no ischemic changes
- TSH 2026-04-09: 1.8 (normal)
- BMP, CBC, LFTs 2026-02-18: normal aside from eGFR 51
- Echocardiogram ordered, pending (scheduled 2026-04-21)
- CHA₂DS₂-VASc score: 4 (age 65–74, female, HTN, DM) — anticoagulation indicated
- HAS-BLED score: 1 (HTN); no other bleeding risks

What we are asking
1. Confirm rhythm vs. rate control strategy given age, CKD, and comorbidities.
2. Recommend anticoagulant selection given eGFR 51 (DOAC dose adjustment vs.
   warfarin); address renally adjusted dabigatran vs. apixaban vs. rivaroxaban.
3. Advise on need for cardioversion or referral for EP evaluation if rhythm
   control is preferred.
4. Specify follow-up interval and any monitoring we should own in primary care.

I have discussed the diagnosis and stroke-risk rationale with the patient; she
is open to anticoagulation and prefers a once-daily DOAC if clinically
appropriate. Records including ECG, labs, and pending echo report will be sent
via secure EHR. Please return the consultation note to my direct EHR inbox or
fax [number]. I am available at [direct line].

Thank you for seeing her.

[Referring Provider Name], MD
[Practice Name] • NPI [##########]
Direct: [phone] • Secure message: [link]
```

The example illustrates the target: one page, CHA₂DS₂-VASc / HAS-BLED / eGFR surfaced, explicit numbered consultation questions, and a clearly stated patient preference — everything the specialist needs to prepare.
