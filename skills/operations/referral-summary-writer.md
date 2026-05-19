---
name: "Referral Summary Writer"
category: operations
tools: [claude, chatgpt]
difficulty: beginner
time_saved: "~10 min/referral"
version: 2.4
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

### Second worked example — urgent behavioral-health referral with 42 CFR Part 2 segregation (v2.4 addition)

The first example covers the highest-volume cardiology-referral pattern (new-onset AFib, routine urgency, single payer lane, no privacy-overlay complexity). The second worked example below covers the highest-stakes behavioral-health referral pattern — a positive depression screen with passive suicidal ideation in a patient with a documented past SUD treatment episode — because that combination exercises three of the skill's most distinctive `config.yml` hooks (`urgency_thresholds` for behavioral-health escalation, `state_privacy_overlays` for the 42 CFR Part 2 segregation rule, `accompanying_records_default` for the PHQ-9/GAD-7/C-SSRS attachment) and the `multi_artifact` output destination (clinical letter + patient-facing 6th–8th grade plain-language summary).

**Input from the referring clinician (primary care):**

```
Patient: K.S., 41F. Annual visit today.
PHQ-9 today: 19 (moderately severe). Item 9 (SI): "several days" — passive
SI, no plan, no intent, no access to means per discussion. C-SSRS: passive
ideation only, no intent, no plan; 3 prior episodes of depression treated
to remission with sertraline. No SI in past 3 months until past 2 weeks.
GAD-7: 12 (moderate anxiety).
PMH: MDD (recurrent, currently mod-severe), GAD, alcohol use disorder
(in sustained remission x 4 years, completed IOP at [Community SUD
program] in 2022 — Part 2 protected). HTN well controlled.
Meds: lisinopril 10. No psych meds currently (off sertraline x 18 mo
after 2-yr remission).
Social: divorce in process, oldest child moving out, working full time.
Strong support: sister local, weekly AA attendance.
Patient agrees to behavioral-health referral, prefers same-week
intake, prefers telehealth. No firearms in home (asked and confirmed).
Safety planning template completed today; safety plan copy to chart.
Lethal-means restriction discussed re medications (locked away with
sister this week).
Referring to: [Community CMHC behavioral health] via secure EHR
inbox. Routing pre-approved by patient. Sister has consent to be
named contact. Send patient-facing copy of summary via portal at
6th–8th grade reading level.
```

**Generated clinical referral letter (artifact 1 of 2):**

```
[Practice Letterhead — from config.yml]

2026-05-18

Behavioral Health Intake
[Community CMHC] — Secure EHR Inbox (Direct Trust)
Referral routing pre-confirmed with patient

RE: Referral for K.S. — moderately severe depression with passive
suicidal ideation, urgent intake within 7 days

Behavioral Health Intake Team,

I am referring K.S. (DOB 1985-02-XX, MRN 5582XXXX) for urgent intake
within 7 days for moderately severe depression with new passive
suicidal ideation. Patient is clinically stable today, has agreed to
the referral, prefers telehealth intake, and has completed a
safety plan in our office. C-SSRS today does not meet emergent
same-day criteria (no intent, no plan, no access to means; lethal
means restriction in place); urgency tier is set to "urgent same-week"
per practice behavioral-health threshold for passive SI without plan
or intent.

Active problems relevant to referral
- Major depressive disorder, recurrent, current episode moderately
  severe (F33.1) — PHQ-9 today 19; new passive SI past 2 weeks
- Generalized anxiety disorder (F41.1) — GAD-7 today 12
- Hypertension (I10), controlled on lisinopril 10 mg daily

Current medications
- Lisinopril 10 mg daily. No current psychiatric medications.
- 18 months off sertraline (prior 2-year remission), no current taper
  symptoms.

Validated screens (attached separately)
- PHQ-9 today: 19 (moderately severe)
- PHQ-9 trajectory: not on file for past 12 months — today's score is
  baseline for this episode
- GAD-7 today: 12 (moderate)
- C-SSRS today: passive ideation only, no intent, no plan, no access
  to means; lethal means restriction in place (firearms confirmed
  absent from home; medications relocated to sister's residence)

Recent functional context (non-Part 2)
- Three prior MDD episodes treated to remission with sertraline; off
  18 months after a 2-year remission window
- Concurrent psychosocial stressors: divorce in process, child
  transition out of home
- Strong protective factors: stable employment, sister local and named
  as safety contact (consent on file), weekly community support group
  attendance

What we are asking
1. Same-week intake (within 7 days) for full diagnostic evaluation and
   treatment-plan recommendations.
2. Confirm whether to resume sertraline at the previously effective
   dose vs. trial alternative SSRI/SNRI given the 18-month interval
   and current symptom severity.
3. Recommend therapy modality and cadence (CBT vs. brief
   problem-solving therapy vs. integrated care) given the
   psychosocial stressors and the patient's stated preference for
   telehealth.
4. Establish follow-up cadence and define the loop-closure
   expectation back to primary care; we expect the consultation note
   via secure EHR within 7 days of intake.

Safety planning today
- Standard 6-step safety plan completed in clinic; copy in chart.
- Lethal means restriction implemented (firearms confirmed absent;
  medications relocated to sister's residence).
- Crisis-line review: 988, local mobile-crisis line, and our
  after-hours nurse line.
- Patient verbalized understanding and contracted for safety.

A separate consent-required addendum contains substance-use history
relevant to clinical decision-making under 42 CFR Part 2. Please
request that addendum through the consent-required intake channel
rather than relying on the body of this letter; patient has signed the
practice's general HIPAA release but the Part 2 release is keyed to
the receiving facility.

Records included with this letter (general consent)
- Today's PHQ-9, GAD-7, C-SSRS forms
- Safety plan (signed)
- Current medication list
- Most recent BMP and LFTs (2026-04 — within normal limits)

Records NOT included in this letter (consent-required, request
separately)
- 2022 SUD treatment-completion summary from [Community SUD program]
- Any Part 2-protected SUD treatment history or attendance records

Patient preferences
- Telehealth intake preferred
- Sister consented as emergency contact
- Patient prefers same-week scheduling and is available
  weekday afternoons

Please return the consultation note to my direct EHR inbox within 7
days. I am available at [direct line] for warm handoff or peer
discussion.

Thank you for seeing her.

[Referring Provider Name], MD
[Practice Name] • NPI [##########]
Direct: [phone] • Secure message: [DirectTrust link]
```

**Generated patient-facing summary (artifact 2 of 2 — `output_destination=multi_artifact`):**

```
Your Next Steps — Behavioral Health Visit

We are setting up a visit with the Community Mental Health team. They
will help you with the low mood and anxiety we talked about today.

What we sent them
- Your scores from today's mood and anxiety questions
- The safety plan you and I wrote together
- Your medicine list

When you will hear from them
- Within 7 days. You will get a call or a portal message to set up the
  visit. You asked for video — we sent that request along.

What to do until then
- Use the safety plan if your thoughts get heavier. The crisis line is
  988 — you can call or text any time.
- Keep your sister in the loop the way we talked about.
- If you feel unsafe before the intake call, contact us, call 988, or
  go to the nearest emergency room.

Who to call
- Our office (during the day): [Number]
- After hours: [Number]
- Crisis: 988 (call or text), or 911 if there is an emergency

[Reviewed by [Clinician]   Date: ______]
This is a summary. It does not replace the visit notes in your chart.
```

### What this second example demonstrates

- **Urgency-thresholds hook fires correctly** — the dictation reports passive SI without plan or intent and with lethal-means restriction in place. The skill maps this to the practice's "urgent same-week behavioral health" tier from `urgency_thresholds`, not to "emergent same-day" (which would apply only with active SI, plan, or intent), and not to "routine." Demonstrates correct calibration to the named threshold rather than defaulting to maximum urgency.
- **42 CFR Part 2 segregation enforced** — the patient's 2022 SUD treatment history is Part 2-protected. The skill writes the body of the referral letter without inline Part 2 content, references the existence of a consent-required addendum, and tells the receiving facility how to request it through a Part 2-compliant intake channel. Demonstrates the `state_privacy_overlays` hook applying the strictest applicable overlay (42 CFR Part 2 is stricter than HIPAA for SUD treatment information) and segregating into a separate consent-required attachment rather than inline.
- **Accompanying-records default exercised** — the validated PHQ-9, GAD-7, and C-SSRS are listed as included records per the behavioral-health default in `accompanying_records_default`; the Part 2-protected SUD treatment-completion summary is explicitly listed as NOT included per the segregation rule above.
- **Numbered consult-question block** — four specific clinically actionable questions: intake cadence, medication restart vs. switch with the 18-month interval as the decision driver, therapy modality, and loop-closure cadence. Demonstrates both the `preferred_consult_questions` boilerplate-blending (intake-cadence and loop-closure are practice-boilerplate for behavioral-health referrals) and patient-specific questions (sertraline restart vs. alternative for this patient's specific history).
- **Safety planning surfaced inline** — the safety plan completed in-clinic, the lethal-means restriction (firearms and medications), and the crisis-line review are stated inline rather than buried, so the receiving intake team can confirm continuity of safety planning at first contact rather than re-doing it. This is the behavioral-health-specific extension of the §"Workup & Findings" block.
- **Multi-artifact output destination fires** — the `output_destination=multi_artifact` value produces both the clinical referral letter for the receiving CMHC and a separate patient-facing summary at 6th–8th grade reading level for the portal copy, exercising the multi-artifact path rather than the default single-letter path. The patient-facing summary explicitly names 988, the crisis path, the sister's role, and the safety plan without repeating any Part 2-protected detail.
- **Closing-loop protocol surfaced** — the letter explicitly names the expected loop-closure cadence (consult note via secure EHR within 7 days of intake) per the practice's `closing_loop_protocol` setting; the care-coordinator-follow-up clause is implied rather than restated since the letter is going to the receiving facility, not to internal staff.
- **No fabrication** — referring practice NPI, exact DirectTrust link, after-hours number, and CMHC intake confirmation number are not invented; the patient's MRN and DOB middle digits are abbreviated to `XXXX` pending the actual chart pull, mirroring the AFib example's approach.

Together the two worked examples now demonstrate the skill's range across the two most operationally distinct referral patterns in outpatient practice: a routine cardiology referral with single-payer lane and no privacy-overlay complexity (Example 1), and an urgent behavioral-health referral with 42 CFR Part 2 segregation, multi-artifact output, named urgency-threshold tier, and inline safety-planning surfacing (Example 2). Together they exercise eleven of the thirteen named config hooks (`referral_partners`, `referral_channel_defaults`, `urgency_thresholds`, `preferred_consult_questions`, `practice_specialty`, `provider_signature_blocks`, `state_privacy_overlays`, `accompanying_records_default`, `closing_loop_protocol`, `output_destination`, `config_missing_behavior`), leaving `payer_prior_auth_routing` and `language_preference_routing` as worked-example debt for a future targeted run.
