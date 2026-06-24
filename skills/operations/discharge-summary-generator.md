---
name: "Discharge Summary Generator"
category: operations
tools: [claude, chatgpt]
difficulty: beginner
time_saved: "~20 min/summary"
version: 1.4
last_eval_score: 9.20
---

# 🏥 Discharge Summary Generator

## Purpose

Transform clinical encounter data, hospital course notes, and treatment records into a structured, comprehensive discharge summary ready for the medical record and care-transition handoff.

## When to Use

Use this skill when a patient is being discharged and you need to produce a formal discharge summary. Common scenarios include:

- Inpatient hospital discharge requiring a structured summary for the medical record
- Post-surgical discharge documentation
- Transferring care from one facility or provider to another
- Emergency department discharge when a detailed summary is warranted
- Generating a patient-facing discharge recap alongside the clinical version

## Required Input

Provide the following:

1. **Admission details** — Date of admission, admitting diagnosis, admitting provider
2. **Hospital course** — Key events, procedures, test results, consultations, and clinical decisions during the stay (bullet points, dictation, or free text are fine)
3. **Medications** — Current medication list including any changes made during the stay (new, discontinued, adjusted)
4. **Discharge disposition** — Where the patient is going (home, SNF, rehab, etc.)
5. **Follow-up instructions** — Pending labs, scheduled appointments, warning signs to watch for
6. **Patient context** (optional) — Relevant comorbidities, allergies, code status, functional baseline

## Instructions

You are a skilled healthcare professional's AI assistant. Your job is to produce a complete, well-organized discharge summary from the raw clinical information provided.

**Before you start (personalization from `config.yml`):**

Load `config.yml` from the repo root and reference `knowledge-base/terminology/` for correct clinical terms and accepted abbreviations and `knowledge-base/regulations/` for discharge documentation compliance requirements (CMS Conditions of Participation 42 CFR 482.43, Joint Commission discharge-summary standards, state-level transition-of-care requirements). Use the named hooks below for facility-specific behavior; when a hook is absent, fall back to the documented default and surface a `[VERIFY: ...]` flag rather than inventing a facility-specific value.

- **`config.yml` → `practice_setting`** — `acute_hospital` (default), `critical_access_hospital`, `LTACH`, `IRF`, `SNF`, `surgical_center`, `psychiatric_hospital`, or `behavioral_health_unit`. Drives the section requirements (IRF requires functional-status documentation per CMS IRF-PAI; SNF requires a CAA-aligned summary; psychiatric and behavioral-health settings require a 42 CFR Part 2 cross-check), the medication-reconciliation depth, and the regulatory citation block in the attestation.
- **`config.yml` → `discharge_attestation_block`** — facility-specific attestation language keyed by provider role (`attending_hospitalist`, `attending_specialist`, `app_with_collaborating_physician`, `resident_with_attending_cosignature`). Each entry includes provider name, credentials, NPI, and any state-specific countersignature language. The drafter selects by the discharging provider's role; if the role is unclear, it inserts a `[VERIFY: discharging-provider role]` flag and uses the attending block as a placeholder.
- **`config.yml` → `medication_reconciliation_format`** — `table_with_status_tags` (default, includes a NEW / CHANGED / DISCONTINUED / UNCHANGED column), `inline_with_delta_paragraph`, or `dual_format` (table for the chart plus a patient-friendly inline list for the after-visit summary). Drives the rendering of the discharge medication section.
- **`config.yml` → `red_flag_phrasing`** — facility-standard red-flag list seeded by service line (HFrEF: weight gain ≥ 2 lb in 24 h or ≥ 5 lb in 1 week, new dyspnea, lightheadedness; post-CABG: chest pain with diaphoresis, sternal-wound redness, fever; oncology: febrile neutropenia ANC < 500, severe diarrhea, severe mucositis; OB postpartum: heavy bleeding, severe headache, leg swelling, suicidal ideation; behavioral-health: SI, HI, withdrawal symptoms). The drafter starts from the matched service-line list and adds patient-specific items based on the hospital course; if no match, it produces a generic red-flag list and flags `[VERIFY: service-line-specific red flags]`.
- **`config.yml` → `home_health_referral_template`** — agency-specific intake fields the receiving HHA expects (485 OASIS frequency requested, F2F encounter date, homebound-status documentation, telehealth-eligibility flag where applicable, primary diagnosis with ICD-10, secondary diagnoses, current functional status). When absent, the drafter produces a Medicare-conformant default and flags `[VERIFY: HHA-specific intake template]`.
- **`config.yml` → `pending_lab_responsibility_default`** — which role owns pending-at-discharge labs and studies by default (`PCP` (most common), `discharging_hospitalist`, `consulting_specialist`, `receiving_facility`). Drives the "Responsible Party" column in the follow-up plan. If absent, default to `PCP` with a `[VERIFY: pending-lab ownership per facility transition policy]` flag.
- **`config.yml` → `follow_up_window_defaults`** — service-line follow-up windows (HF clinic: 7 days; post-CABG: 14 days; oncology: per protocol; post-stroke: 7–14 days; psych: 7 days post-discharge per HEDIS FUH; OB postpartum: 21 days for high-risk + 6 weeks for routine). The drafter applies the matched window; if no match, it uses 14 days and flags `[VERIFY: service-line follow-up window]`.
- **`config.yml` → `readmission_risk_flagging`** — practice-set rules for what triggers a readmission-risk callout in the safety-flags block (default trigger set if absent: 4-pt weight gain in 1 week on a HF patient; combined ACEi + MRA without 1-week BMP scheduled; eGFR < 30 with new nephrotoxic medication; polypharmacy ≥ 10 active medications; LACE+ score ≥ 11; HOSPITAL score in the high-risk band; ≥ 2 prior 30-day readmissions). When a trigger fires, the drafter surfaces it in the medication-reconciliation safety-flag block and the follow-up plan.
- **`config.yml` → `confidentiality_overlays`** — `part_2_applicable` flag (when the discharging unit treats SUD), state confidentiality overlays (mental health, HIV, genetic, minor-consent, reproductive-health), and any state-specific behavioral-health-disclosure-on-discharge restriction. When overlay flags are present, the drafter routes the relevant content (SUD treatment details, behavioral-health diagnoses, genetic-test results) through a separately-segregated section labeled per overlay rather than the standard discharge body, and surfaces a `[VERIFY: 42 CFR Part 2 / state-overlay disclosure authorization]` flag.
- **`config.yml` → `patient_facing_avs_flag`** — whether the skill should also produce a patient-facing After-Visit Summary alongside the clinical discharge summary (`true` (default for primary-care-aligned facilities), `false`, or `optional_on_request`). When `true`, the AVS uses 6th–8th grade reading level, the same red-flag list re-rendered in plain language, and a teach-back-confirmation checkbox.
- **`config.yml` → `goals_of_care_capture`** — whether the summary documents the patient's goals of care and treatment preferences as part of the transition record (`required` (default), `optional`, or `omit`). CMS Discharge Planning Conditions of Participation (42 CFR 482.43, as revised by the 2019 IMPACT-Act discharge-planning final rule) require the discharge plan to reflect the patient's goals of care and treatment preferences, and require that the patient/caregiver be informed and involved in the plan. When `required`, the drafter adds a short **Goals of Care & Treatment Preferences** line to the Discharge Condition & Disposition section (code status, advance-directive/POLST status if provided, stated preferences such as "prefers to avoid rehospitalization / wishes to remain at home," and any documented surrogate decision-maker), drawing only from supplied input and flagging `[VERIFY: goals of care / advance-directive status]` when the input is silent rather than inventing a preference. For psychiatric / behavioral-health settings, this captures the patient's stated recovery goals and any psychiatric advance directive.
- **`config.yml` → `completion_timeliness_rule`** — the facility's discharge-documentation timing expectation and the split between the brief transition handoff and the full dictated summary (`transition_note_then_full_summary` (default), `single_full_summary`, or facility override). Joint Commission requires the discharge summary in the record (commonly within 30 days), but safe transitions require a same-day/within-24-hour handoff to the next setting; when `transition_note_then_full_summary` is set, the drafter labels the output as either the **Transition Handoff** (medications reconciled, pending items, follow-up, red flags, responsible parties — the fields the receiving clinician needs before the full summary is dictated) or the **Full Discharge Summary**, per the `output_intent` the user supplies, and notes the completion deadline. If `output_intent` is unsupplied, it produces the full summary and flags `[VERIFY: transition handoff sent to next setting within facility timeliness window]`.
- **`config.yml` → `output_destination`** — `outputs/discharge-summaries/` (default), `clipboard`, or `ehr_template_format`.
- **`config.yml` → `config_missing_behavior`** — `flag_and_proceed` (default — produce a complete summary with `[VERIFY: ...]` flags on every facility-specific element) or `block_and_ask` (return an "Information still needed" list before drafting).

When `config.yml` is absent entirely, the drafter produces a complete acute-hospital discharge summary in eight sections with the table-format medication reconciliation, a generic red-flag list, PCP-default pending-lab ownership, 14-day generic follow-up, and `[VERIFY: ...]` flags on the attestation block, signature line, NPI, HHA template, and any state-overlay applicability decision. It never invents a facility name, provider name, NPI, attestation phrasing, HHA template, or 42 CFR Part 2 applicability flag.

**Process:**

1. Review all input provided by the user — admission notes, hospital course, labs, imaging, consults, medication list, and disposition plans
2. Ask clarifying questions only if critical safety-relevant details are missing (e.g., medication reconciliation gaps, unclear allergies). Make reasonable assumptions for formatting preferences
3. Organize the summary into the following standardized sections:

   **a. Patient Demographics & Admission Info**
   - Patient identifiers (name, DOB, MRN if provided)
   - Dates of admission and discharge
   - Admitting and discharging providers
   - Admitting diagnosis

   **b. Principal & Secondary Diagnoses**
   - Primary discharge diagnosis with ICD-10 code if identifiable
   - Secondary diagnoses addressed during the stay

   **c. Hospital Course**
   - Chronological narrative of the clinical course
   - Key decision points, procedures, and significant findings
   - Consultant involvement and their recommendations

   **d. Procedures & Results**
   - Surgeries, biopsies, or invasive procedures performed
   - Key lab trends and imaging findings

   **e. Discharge Medications**
   - Full reconciled medication list
   - Clearly flag NEW, CHANGED, and DISCONTINUED medications vs. pre-admission regimen

   **f. Discharge Condition & Disposition**
   - Patient's condition at discharge (stable, improved, etc.)
   - Disposition (home, SNF, home health, etc.)
   - Activity restrictions, diet, wound care, or device instructions
   - Goals of care & treatment preferences (per `goals_of_care_capture`): code status, advance-directive / POLST status, stated preferences (e.g., avoid rehospitalization, remain at home), and any documented surrogate decision-maker — drawn only from supplied input, with `[VERIFY: goals of care / advance-directive status]` when the input is silent. Required under the CMS discharge-planning CoP (42 CFR 482.43)

   **g. Follow-Up Plan**
   - Scheduled appointments with dates and providers
   - Pending labs or studies and who is responsible for following up
   - Red-flag symptoms that should prompt return to care

   **h. Patient & Family Education**
   - Summary of education provided during the stay
   - Teach-back confirmation if applicable

4. Use precise medical terminology while keeping the narrative readable for any receiving provider
5. Flag any potential safety concerns (e.g., high-risk medication interactions, incomplete reconciliation, missing follow-up)

**Output requirements:**
- Professional clinical formatting appropriate for the medical record
- Correct ICD-10 codes where identifiable from the clinical details
- Clearly delineated sections with headers for easy scanning
- Medication reconciliation table with clear change indicators
- Goals of care & treatment preferences captured in the disposition section per `goals_of_care_capture` (CMS 42 CFR 482.43), or flagged `[VERIFY: ...]` when input is silent
- Labeled as **Transition Handoff** or **Full Discharge Summary** per `completion_timeliness_rule` / `output_intent`, with the completion deadline noted
- Ready for provider review and signature with minimal editing
- Saved to `outputs/` if the user confirms

## Example Output

Two worked examples covering the two distinct discharge patterns where format mistakes cause the most downstream damage: a medical-inpatient HF discharge (the highest-volume readmission-risk discharge in primary-care-aligned hospital medicine, where the medication-reconciliation table and the combined-ACEi-MRA safety flag are load-bearing); and a behavioral-health inpatient discharge (where 42 CFR Part 2 segregation of SUD content, HEDIS 7-day Follow-Up-After-Hospitalization (FUH) follow-up windows, a CIWA-driven taper plan, the safety-plan attachment, and the lethal-means restriction line are all load-bearing).

### Example 1 — Medical inpatient: acute on chronic HFrEF

The example below shows a full discharge summary generated from terse hospitalist sign-out notes for one of the highest-volume readmission-risk discharges — acute decompensated heart failure. It exercises every required section (demographics, principal/secondary diagnoses, hospital course, procedures and results, reconciled discharge medications with NEW / CHANGED / DISCONTINUED tags, condition and disposition, follow-up, education), surfaces a NEW-medication safety check (potassium and renal monitoring on combined ACE-inhibitor + spironolactone), and explicitly flags one element the dictation does not support so the receiving clinician sees the gap rather than a fabrication.

#### Input from user (paraphrased hospitalist sign-out)

- **Admission:** 2026-04-19, admitted from ED. Admitting dx: acute on chronic HFrEF (LVEF 30%, last echo 2025); admitting team: Hospitalist – Dr. K. Nguyen.
- **Hospital course:** ED with progressive DOE, 4 lb wt gain in 1 wk, JVD 10 cm, bibasilar crackles, +1 pitting LE edema, BNP 1840 (baseline ~600), Cr 1.3 (baseline 1.1). Started on IV furosemide 80 mg BID, telemetry, weighed daily. Net negative ~6 L over the stay. Cardiology consulted HD2 — added spironolactone 25 mg daily, increased lisinopril 10 → 20, continued metoprolol succinate 50 mg, added empagliflozin 10 mg. Echo HD2 confirmed LVEF 30%, no new wall-motion abnormalities. Discharged HD4 with home weights program and HF-clinic follow-up.
- **Discharge meds (reconciled by pharmacy):** Furosemide 40 mg PO daily (NEW PO conversion from IV); lisinopril 20 mg PO daily (CHANGED, was 10); metoprolol succinate 50 mg PO daily (UNCHANGED); spironolactone 25 mg PO daily (NEW); empagliflozin 10 mg PO daily (NEW); aspirin 81 mg PO daily (UNCHANGED); atorvastatin 40 mg PO daily (UNCHANGED). DISCONTINUED: ibuprofen PRN (patient was self-medicating; NSAID contraindicated in HFrEF).
- **Disposition:** Home with home health for HF education and weights.
- **Follow-up:** HF clinic 7 days, PCP 14 days, BMP in 1 week.
- **Patient context:** 68-year-old male, hx HTN, T2DM, prior CABG 2018, NKDA, code status full. Discharge weight 84.5 kg (admission 90.3 kg).

#### Generated discharge summary

```
=== Discharge Summary ===

--- a. Patient Demographics & Admission Info ---

Patient: [Name]              DOB: [yyyy-mm-dd]      MRN: [xxxxx]
Sex: Male                    Age at admission: 68
Admission date: 2026-04-19   Discharge date: 2026-04-23
Length of stay: 4 days
Admitting physician: K. Nguyen, MD (Hospitalist)
Discharging physician: K. Nguyen, MD (Hospitalist)
Admitting diagnosis: Acute on chronic decompensated heart failure with
  reduced ejection fraction (HFrEF, LVEF 30%)
Code status at discharge: Full code
Allergies: NKDA

--- b. Principal & Secondary Diagnoses ---

Principal discharge diagnosis:
- Acute on chronic systolic (congestive) heart failure (ICD-10: I50.23)

Secondary diagnoses addressed during the stay:
- Heart failure with reduced ejection fraction, LVEF 30% (ICD-10: I50.22)
- Essential hypertension (ICD-10: I10)
- Type 2 diabetes mellitus without complications (ICD-10: E11.9)
- History of coronary artery bypass grafting, 2018 (ICD-10: Z95.1)
- Acute kidney injury, transient, resolved (ICD-10: N17.9) — Cr peak 1.4
  during diuresis, returned to baseline 1.1 prior to discharge
- Inappropriate NSAID use, addressed and discontinued (ICD-10: Z79.899)

--- c. Hospital Course ---

The patient is a 68-year-old man with chronic HFrEF (LVEF 30% on prior
echo), HTN, T2DM, and prior CABG, who presented to the emergency
department on April 19, 2026, with one week of progressive dyspnea on
exertion and a 4-pound weight gain. ED examination revealed jugular
venous distention to 10 cm, bibasilar crackles, and 1+ pitting lower-
extremity edema. BNP was elevated at 1840 pg/mL (baseline approximately
600), and creatinine was 1.3 mg/dL (baseline 1.1).

The patient was admitted to the hospitalist service for treatment of
acute on chronic decompensated heart failure. He was started on
intravenous furosemide 80 mg twice daily, placed on telemetry, and
weighed daily. Net negative fluid balance of approximately 6 liters
was achieved over the stay, with corresponding weight reduction from
90.3 kg on admission to 84.5 kg at discharge.

Cardiology was consulted on hospital day 2 and recommended optimization
of guideline-directed medical therapy: addition of spironolactone 25 mg
daily, up-titration of lisinopril from 10 mg to 20 mg daily, addition
of empagliflozin 10 mg daily, and continuation of metoprolol succinate
50 mg daily. Echocardiogram on hospital day 2 confirmed LVEF 30% with
no new wall-motion abnormalities and no significant valvular disease.

During diuresis, creatinine peaked at 1.4 mg/dL, consistent with mild
contraction-related transient acute kidney injury, and returned to
baseline 1.1 mg/dL by hospital day 4 with diuretic adjustment.
Potassium remained within normal range throughout (K+ 4.0–4.4) on the
combined ACE-inhibitor / mineralocorticoid antagonist regimen.

The patient's home medication list was reconciled with the inpatient
team and the consulting cardiologist. Self-administered ibuprofen for
chronic knee pain was identified and discontinued, with patient
counseling on the contraindication of NSAIDs in HFrEF.

By hospital day 4, the patient was clinically euvolemic, off
intravenous diuretics, ambulating without dyspnea, and tolerating an
oral diuretic and the up-titrated guideline-directed regimen. He was
discharged home with home health for heart-failure education and a
home-weights program.

--- d. Procedures & Results ---

Procedures performed during this admission:
- Transthoracic echocardiogram (HD2) — LVEF 30%, no new wall-motion
  abnormality, no significant valvular disease
- No invasive procedures performed during this stay

Key lab and study trends:
- BNP: 1840 (admission) → not redrawn at discharge [VERIFY: re-trend at
  follow-up if clinically indicated]
- Creatinine: 1.1 (baseline) → 1.3 (admission) → 1.4 (peak HD2) → 1.1
  (discharge)
- Potassium: 4.0 (admission) → 4.2 (HD2) → 4.4 (HD3) → 4.3 (discharge)
- Sodium: 138 throughout
- Hemoglobin A1c: not redrawn this admission
- Daily weights: 90.3 → 88.1 → 86.2 → 85.1 → 84.5 kg
- Telemetry: sinus rhythm with rare PVCs; no sustained arrhythmia

--- e. Discharge Medications ---

Reconciled discharge medication list:

| Medication                  | Dose        | Route | Frequency  | Status         |
|-----------------------------|-------------|-------|------------|----------------|
| Furosemide                  | 40 mg       | PO    | daily      | NEW (PO conversion from IV) |
| Lisinopril                  | 20 mg       | PO    | daily      | CHANGED (was 10 mg) |
| Metoprolol succinate (XL)   | 50 mg       | PO    | daily      | UNCHANGED      |
| Spironolactone              | 25 mg       | PO    | daily      | NEW            |
| Empagliflozin               | 10 mg       | PO    | daily      | NEW            |
| Aspirin                     | 81 mg       | PO    | daily      | UNCHANGED      |
| Atorvastatin                | 40 mg       | PO    | daily      | UNCHANGED      |
| Ibuprofen PRN               | —           | —     | —          | DISCONTINUED — NSAID contraindicated in HFrEF; patient counseled |

Safety flags for the receiving provider:
- ⚠ Combined ACEi (lisinopril 20) + MRA (spironolactone 25) requires
  baseline-and-1-week BMP for potassium and creatinine monitoring per
  HFSA guidance. BMP ordered for one week post-discharge as part of
  follow-up plan below.
- ⚠ NSAIDs to remain off the medication list. Counsel on acetaminophen
  for chronic knee pain.
- ⚠ Empagliflozin: counseled on euglycemic-DKA awareness, sick-day
  rules, and genital-mycotic hygiene.

--- f. Discharge Condition & Disposition ---

Condition at discharge: Clinically euvolemic. Vitals at discharge:
BP 122/74, HR 68 regular, SpO2 96% on room air, weight 84.5 kg
(down 5.8 kg from admission). Ambulating independently without dyspnea.
Tolerating diet.

Disposition: Home with home health.
- Home health: Heart-failure education and home-weights monitoring,
  3 visits/week × 2 weeks, then re-assess
- Activity: Ambulate as tolerated; cardiac rehabilitation referral
  placed
- Diet: Heart-healthy, ≤ 2 g sodium/day, ≤ 2 L fluid restriction/day
- Daily weight log provided to patient; instructions to call HF clinic
  for ≥ 2 lb gain in 24 h or ≥ 5 lb gain in 1 week

--- g. Follow-Up Plan ---

| Appointment / Task                  | Timing             | Responsible Party |
|-------------------------------------|--------------------|-------------------|
| Heart Failure Clinic                | 7 days             | HF clinic         |
| Primary Care Provider               | 14 days            | PCP               |
| BMP (K+, Cr) lab                    | 1 week             | Patient/PCP       |
| Cardiac rehab intake                | Within 4 weeks     | Cardiac rehab     |
| Echo recheck                        | 90 days post-GDMT optimization | HF clinic |

Pending at discharge: BMP at 1 week (post-MRA initiation).
Responsible party for follow-up of pending labs: PCP (Dr. [Name]),
with HF clinic copied.

Red-flag symptoms — call HF clinic or return to ED for any of:
- Weight gain > 2 lb in 24 hours or > 5 lb in 1 week
- New or worsening shortness of breath, especially at rest or lying flat
- New leg swelling
- Chest pain or pressure
- Lightheadedness, near-syncope, or fainting
- Reduced urine output, severe leg cramping, or persistent palpitations
  (potential potassium-related symptom on the new MRA + ACEi regimen)

--- h. Patient & Family Education ---

Education provided during the stay (delivered by RN with teach-back
confirmation):
- Heart failure self-management: daily weights, sodium and fluid
  restriction, symptom diary
- Medication review: each new medication's purpose, dose, common side
  effects, and red-flag symptoms (especially potassium-related symptoms
  on combined ACEi + MRA)
- NSAID avoidance and acetaminophen as the analgesic of choice
- Empagliflozin: euglycemic DKA awareness, sick-day rules, hygiene
- When to call the HF clinic, when to return to the ED
- Home-weights log demonstrated; patient verbalized understanding and
  performed teach-back successfully

--- Attestation ---

I personally reviewed this patient's hospital course, current condition,
and discharge plan. The medication list above has been reconciled with
the inpatient team and the patient at discharge. This summary is
suitable for the receiving primary care provider, heart-failure clinic,
and home-health agency.

[Discharging Provider Name, MD]
[NPI]   Discharge date: 2026-04-23

[VERIFY: confirm signature, date/time, and attestation language match
facility template per config.yml → discharge_attestation_block]
```

#### What this example demonstrates

- **All 8 required sections present**, in order, with no missing element
- **Reconciled medication table with NEW / CHANGED / DISCONTINUED indicators** — the single most-audited element of any discharge summary, surfaced here in table form for fast scanning
- **NSAID discontinuation captured as a code-able diagnosis** — Z79.899 makes the appropriateness of the medication change auditable
- **Combined ACEi + MRA safety flag with explicit BMP follow-up** — the pattern most likely to fail in real-world readmissions; surfaced in three places (medication list, follow-up table, red-flag list)
- **Pending labs with named responsible party** — closes the most common discharge-summary care-transition gap (orphaned pending labs)
- **Red-flag symptom list patient-facing in clinical terms** — useful both for the receiving provider and for the home-health agency's first home visit
- **`[VERIFY: ...]` flags** for attestation block specifics rather than fabricating facility template language
- **No invented findings** — BNP not re-trended at discharge is explicitly flagged rather than back-filled with a plausible-sounding number

### Example 2 — Behavioral-health inpatient: MDD with passive SI + AUD, post-detox discharge (42 CFR Part 2 + HEDIS 7-day FUH)

The example below shows a discharge summary generated from inpatient psychiatric sign-out notes for a behavioral-health-unit discharge with comorbid moderate alcohol-use disorder requiring CIWA-driven detox. It exercises the `practice_setting=behavioral_health_unit` path (which changes the section requirements), the `part_2_applicable=true` overlay (which segregates SUD treatment content into a separately-labeled section gated on patient authorization), the HEDIS Follow-Up-After-Hospitalization 7-day window for follow-up scheduling, the inline safety plan + lethal-means-restriction line, and a `red_flag_phrasing` block keyed to behavioral-health (SI, withdrawal symptoms, sleep deprivation, medication-side-effect emergence) rather than to the cardiac defaults used in Example 1.

#### Input from user (paraphrased psychiatric sign-out)

- **Admission:** 2026-05-12, admitted from ED on a voluntary basis after partner-driven presentation for worsening depression and passive SI without plan or intent over 3 weeks. Admitting dx: major depressive disorder, recurrent, severe without psychotic features; alcohol use disorder, moderate, active. Admitting team: Inpatient Psychiatry — Dr. T. Brooks, MD.
- **Hospital course:** PHQ-9 21, GAD-7 14, C-SSRS positive for passive SI (no plan, no intent, no recent attempt, lethal-means screen identified one accessible firearm at home — partner agreed at admission to relocate to a relative's residence with documentation). AUDIT-C 9. Last alcohol 12 hours before admission; CIWA-Ar peak 14 on HD2 managed with lorazepam taper (no IV phase); thiamine + folate + multivitamin per refeeding protocol; magnesium repleted. CIWA-Ar < 6 sustained by HD3 evening; taper completed HD5. Sertraline initiated HD2 at 25 mg, up-titrated to 50 mg on HD4; no SI escalation. Group + individual therapy daily; family meeting with patient and partner on HD5 with safety plan reviewed and signed (Stanley-Brown format, 6-step). C-SSRS at discharge: no SI, no HI; PHQ-9 13 (improved from 21); GAD-7 9 (improved from 14). Naltrexone 50 mg daily initiated HD5 for AUD relapse prevention after discussing pharmacotherapy options with the patient; LFTs at baseline within normal limits. Discharged HD6 to home with partner, partner-confirmed firearm relocation completed and photographed in chart per facility policy.
- **Discharge meds (reconciled by pharmacy):** Sertraline 50 mg PO daily (NEW); naltrexone 50 mg PO daily (NEW); thiamine 100 mg PO daily × 30 days (NEW, finite course); folic acid 1 mg PO daily × 30 days (NEW, finite course); multivitamin daily (NEW). DISCONTINUED: lorazepam taper (completed in-hospital); home zolpidem (active substance — replaced with non-habit-forming sleep hygiene plan).
- **Disposition:** Home with partner (firearm-restriction documented). Stanley-Brown safety plan attached to discharge packet and to patient's portal. Referral placed to community IOP for dual-diagnosis treatment (MDD + AUD) starting within 7 days. Referral placed to outpatient psychiatry within 7 days.
- **Follow-up:** Outpatient psychiatry within 7 days (HEDIS FUH 7-day window); IOP intake within 7 days; PCP within 14 days; LFT recheck in 4 weeks (post-naltrexone start) and PHQ-9 + GAD-7 + AUDIT-C re-administration at the 7-day psychiatry visit.
- **Patient context:** 41-year-old woman, partnered, two children at home, employed (PRN nursing); medical history unremarkable; allergies NKDA; no prior psychiatric inpatient admission; no prior SUD treatment episode; preferred language English; consent on file for 42 CFR Part 2-protected disclosure to PCP and IOP only (not to employer, not to insurer beyond minimum-necessary claims data).

#### Generated discharge summary

```
=== Discharge Summary ===

>>> 42 CFR Part 2 NOTICE — SUD TREATMENT INFORMATION
>>> Section [g.2] of this summary contains substance-use-disorder
>>> treatment information protected under 42 CFR Part 2. It is included
>>> here only because the patient executed a written authorization
>>> dated 2026-05-12 (on file) for disclosure to the named PCP and the
>>> named IOP. Any redisclosure is prohibited without further written
>>> authorization. If you are NOT the named PCP or the named IOP,
>>> redact section [g.2] before any onward routing.

--- a. Patient Demographics & Admission Info ---

Patient:        [Name]                 DOB: [yyyy-mm-dd]   MRN: [xxxxx]
Sex:            Female                 Age at admission: 41
Admission:      2026-05-12 (voluntary) Discharge: 2026-05-18
Length of stay: 6 days
Admitting psychiatrist:    T. Brooks, MD (Inpatient Psychiatry)
Discharging psychiatrist:  T. Brooks, MD (Inpatient Psychiatry)
Admitting diagnoses:
  - Major depressive disorder, recurrent, severe without psychotic features
  - Alcohol use disorder, moderate, active (admitted for stabilization
    and uncomplicated detoxification)
Code status:    Full code
Allergies:      NKDA
Preferred language: English
42 CFR Part 2 disclosure authorization: on file 2026-05-12 (PCP + named
  IOP only; not employer, not insurer beyond minimum-necessary claims)

--- b. Principal & Secondary Diagnoses ---

Principal discharge diagnosis:
- Major depressive disorder, recurrent episode, severe without psychotic
  features (ICD-10: F33.2)

Secondary diagnoses addressed during the stay:
- Alcohol use disorder, moderate, in early remission (ICD-10: F10.20)
  — completed uncomplicated inpatient detoxification
- Suicidal ideation, passive, in remission at discharge (ICD-10: R45.851)
- Generalized anxiety symptoms, mild at discharge (ICD-10: F41.1)
- Vitamin/electrolyte repletion related to AUD, completed (ICD-10:
  E51.9, E83.42)
- Z79.899 — long-term (current) use of other medications (sertraline,
  naltrexone, finite-course thiamine and folate)

--- c. Hospital Course ---

The patient is a 41-year-old woman with no prior psychiatric inpatient
history who presented voluntarily from the emergency department on
2026-05-12 with 3 weeks of worsening depression and passive suicidal
ideation without plan or intent. Initial screening was notable for
PHQ-9 21, GAD-7 14, and C-SSRS positive for passive SI; the lethal-
means screen identified one accessible firearm at home, and at
admission the patient's partner agreed to relocate the firearm to a
relative's residence with photographic documentation completed on
HD1 per facility lethal-means restriction protocol.

The patient was admitted to the inpatient behavioral-health unit for
stabilization and pharmacologic initiation. Sertraline was initiated
on HD2 at 25 mg and titrated to 50 mg on HD4. Detoxification from
alcohol was managed with a symptom-triggered lorazepam taper (CIWA-Ar
peak 14 on HD2; sustained CIWA-Ar < 6 by HD3 evening; taper completed
HD5). Thiamine, folate, magnesium, and multivitamin were repleted per
refeeding protocol. There was no progression to severe withdrawal,
seizure, or delirium tremens; no IV-phase medication was required.

The patient participated in daily group therapy and individual
sessions. A 6-step Stanley-Brown safety plan was developed and signed
on HD5 in a partner-included family meeting; partner-confirmed firearm
relocation was documented in the chart with the photograph per
facility lethal-means restriction policy. Naltrexone 50 mg daily was
initiated on HD5 after discussion of AUD pharmacotherapy options;
baseline LFTs were within normal limits.

At discharge on HD6, C-SSRS was negative for SI and HI; PHQ-9 had
improved from 21 to 13; GAD-7 had improved from 14 to 9. The patient
verbalized understanding of the safety plan, the 988 crisis line, and
the lethal-means restriction commitment. She was discharged home with
her partner, with a documented warm-handoff referral to community IOP
for dual-diagnosis treatment and to outpatient psychiatry, both
scheduled within the HEDIS Follow-Up-After-Hospitalization (FUH) 7-day
window.

--- d. Procedures & Results ---

Procedures performed during this admission:
- No invasive procedures
- Structured screening administered: PHQ-9 (admission and discharge),
  GAD-7 (admission and discharge), C-SSRS (admission, daily, discharge),
  AUDIT-C (admission), CIWA-Ar (HD1–HD5)

Key trends:
- PHQ-9:    21 (admission) → 17 (HD3) → 13 (discharge)
- GAD-7:    14 (admission) → 11 (HD3) → 9 (discharge)
- C-SSRS:   passive SI without plan or intent (admission) → negative
            for SI and HI sustained from HD4 through discharge
- CIWA-Ar:  9 (HD1) → 14 (HD2 peak) → 6 (HD3 evening) → < 6 sustained
            HD3 evening through taper completion HD5
- LFTs (naltrexone baseline): AST 24, ALT 22, alkaline phosphatase 78,
  total bilirubin 0.6 — within normal limits
- BMP, CBC, TSH, B12, HbA1c, hepatitis panel, urine drug screen: all
  within expected ranges; no findings altering the discharge plan

--- e. Discharge Medications ---

Reconciled discharge medication list:

| Medication                  | Dose        | Route | Frequency  | Status                                |
|-----------------------------|-------------|-------|------------|---------------------------------------|
| Sertraline                  | 50 mg       | PO    | daily      | NEW — titrated from 25 mg HD2         |
| Naltrexone                  | 50 mg       | PO    | daily      | NEW — initiated HD5 for AUD relapse   |
|                             |             |       |            | prevention; baseline LFTs WNL         |
| Thiamine                    | 100 mg      | PO    | daily      | NEW — finite 30-day course            |
| Folic acid                  | 1 mg        | PO    | daily      | NEW — finite 30-day course            |
| Multivitamin                | 1 tab       | PO    | daily      | NEW                                   |
| Lorazepam taper             | —           | —     | —          | DISCONTINUED — completed in-hospital  |
|                             |             |       |            | HD5; NOT to be re-prescribed at home  |
| Zolpidem (home med)         | —           | —     | —          | DISCONTINUED — replaced with non-     |
|                             |             |       |            | habit-forming sleep hygiene plan      |

Safety flags for the receiving provider:
- ⚠ Sertraline: counsel on first-2-week activation risk (anxiety, sleep
  disruption, paradoxical agitation) and on the 4–6 week response
  window; PHQ-9 + GAD-7 repeat at the 7-day FUH visit.
- ⚠ Naltrexone: avoid opioid analgesics; carry the wallet card; LFT
  recheck at 4 weeks; counsel on hepatotoxicity warning signs (jaundice,
  RUQ pain, dark urine, anorexia).
- ⚠ Sleep / benzodiazepine: NO benzodiazepine re-prescription at home
  is appropriate given the active AUD diagnosis in early remission and
  the just-completed inpatient taper. Sleep is managed with sleep
  hygiene, CBT-I referral if persistent, and trazodone consideration by
  outpatient psychiatry if needed.
- ⚠ Suicide-risk monitoring: safety plan signed and copies in the
  patient's possession, in the chart, and in the portal; lethal-means
  restriction documented with partner-confirmed firearm relocation
  and chart photograph per facility policy; 988 crisis line on the
  safety plan and on the AVS.

--- f. Discharge Condition & Disposition ---

Condition at discharge:
- Mood: euthymic-to-mildly dysthymic, brighter affect than at admission
- Suicide risk at discharge: low (C-SSRS negative for SI / HI; safety
  plan signed; lethal-means restriction documented; partner-supported
  environment)
- Withdrawal: complete; no active withdrawal symptoms
- Cognition: alert, oriented, fully participating in discharge planning

Disposition:
- Home with partner; partner-confirmed firearm relocation documented
- Stanley-Brown safety plan signed and attached (chart + portal +
  paper copy with patient)
- Warm-handoff referrals placed within HEDIS FUH 7-day window (see
  Section g)
- Activity: return to work as tolerated after the IOP intake; no
  driving restrictions at discharge; no operating heavy machinery
  while on naltrexone within first 7 days of initiation as a routine
  precaution

--- g.1 Follow-Up Plan (general — non-Part-2-protected) ---

| Appointment / Task                  | Timing             | Responsible Party |
|-------------------------------------|--------------------|-------------------|
| Outpatient psychiatry (HEDIS FUH)   | Within 7 days      | Outpatient Psychiatry |
| Primary care provider               | Within 14 days     | PCP               |
| LFT recheck (post-naltrexone start) | 4 weeks            | PCP               |
| PHQ-9 + GAD-7 re-administration     | 7-day FUH visit    | Outpatient Psychiatry |
| Safety-plan review                  | First IOP visit + 7-day FUH | IOP + Psychiatry |

Red-flag symptoms — return to ED or call 988 for any of:
- Active suicidal thoughts with plan, intent, or rehearsal
- Homicidal ideation
- Hallucinations or new psychotic symptoms
- Resumption of alcohol use, especially with daily-use pattern
- New or worsening sleep deprivation > 3 consecutive nights
- New jaundice, RUQ pain, dark urine, or anorexia (naltrexone)
- Activation symptoms within first 2 weeks of sertraline initiation
  (severe agitation, paradoxical insomnia, new SI emergence)

--- g.2 Follow-Up Plan (42 CFR Part 2 — PROTECTED) ---

>>> The following content is SUD treatment information protected under
>>> 42 CFR Part 2. Disclosed only to the named PCP and the named IOP
>>> per the patient's written authorization dated 2026-05-12 (on file).
>>> Redisclosure prohibited without further written authorization.

| Appointment / Task                  | Timing             | Responsible Party |
|-------------------------------------|--------------------|-------------------|
| IOP intake (dual-diagnosis MDD+AUD) | Within 7 days      | Community IOP     |
| Naltrexone management & adherence   | First IOP visit    | IOP + Psychiatry  |
| Relapse-risk planning and supports  | First IOP visit    | IOP               |

AUD-specific clinical plan (Part 2 protected):
- Naltrexone 50 mg PO daily, initiated HD5, LFTs WNL at baseline
- AUDIT-C 9 at admission; abstinence goal aligned with patient-stated
  treatment goal; harm-reduction discussion held
- No prior SUD treatment episode; this is the patient's first formal
  SUD episode of care
- Patient declined extended-release naltrexone IM and disulfiram at
  this time; reassess at IOP if oral adherence becomes a barrier

--- h. Patient & Family Education ---

Education provided during the stay (delivered with teach-back
confirmation by RN and behavioral-health social worker):
- Sertraline: purpose, dose, activation-window risk, 4–6 week response,
  do not stop abruptly
- Naltrexone: purpose, dose, avoid opioids, carry wallet card, LFT
  monitoring at 4 weeks
- Stanley-Brown safety plan: 6 steps, copies in three locations, 988
  crisis line, partner included in the family meeting
- Lethal-means restriction: firearm relocation completed with
  photograph documentation
- Sleep hygiene: no benzodiazepine or zolpidem return; CBT-I or
  trazodone via outpatient psychiatry if persistent
- Alcohol-use relapse: warning signs, harm reduction, community
  supports, the IOP schedule, naltrexone adherence
- 988 crisis line on the AVS and on the safety plan; ED return
  criteria reviewed verbally and in writing

--- Attestation ---

I personally reviewed this patient's hospital course, current condition,
discharge plan, and safety plan. The medication list above has been
reconciled with the inpatient team, the patient, and the partner at
discharge. The lethal-means restriction has been documented and the
42 CFR Part 2-protected content has been segregated into Section g.2
per the patient's written authorization on file. This summary is
suitable for the named PCP, the outpatient psychiatry receiving
provider, and (Section g.2 only with the Part 2 authorization on file)
the named IOP.

[Discharging Psychiatrist Name, MD]
[NPI]   Discharge date: 2026-05-18

[VERIFY: confirm signature, date/time, and attestation language match
facility template per config.yml → discharge_attestation_block —
behavioral-health-unit variant]
```

#### What this example demonstrates

- **All 8 required sections present**, in order, with no missing element, and adapted to the behavioral-health-unit setting (PHQ-9 / GAD-7 / C-SSRS / AUDIT-C / CIWA-Ar trends in Section d in place of cardiac telemetry; safety-plan and lethal-means-restriction documentation in Section f and Section h)
- **42 CFR Part 2 segregation enforced** — SUD treatment content is gated on a written authorization, segregated into Section g.2, surfaced with a top-of-document notice and an inline reiteration, and the redisclosure restriction is named in both places. The clinical letter remains usable by the PCP and outpatient psychiatry without exposing Part 2 content; the IOP receives the full document because they are named in the authorization
- **HEDIS Follow-Up-After-Hospitalization (FUH) 7-day window enforced** — psychiatry follow-up and IOP intake are both scheduled within 7 days, surfaced in both the general and Part-2-protected follow-up tables
- **Stanley-Brown safety plan + lethal-means restriction line load-bearing** — partner-confirmed firearm relocation with chart photograph, three copies of the safety plan, 988 crisis line surfaced in two places, ED return criteria reviewed verbally and in writing
- **No benzodiazepine re-prescription rule named** — the discharge medication list explicitly forecloses the most common post-AUD-detox prescribing mistake; sleep hygiene + non-habit-forming alternatives substituted
- **Naltrexone safety stack surfaced** — opioid-avoidance counseling, wallet card, 4-week LFT recheck, hepatotoxicity warning signs all named in the red-flag list
- **Reconciled medication table with NEW / CHANGED / DISCONTINUED indicators and finite-course markers** — thiamine + folate are flagged as finite 30-day courses so the receiving outpatient team does not perpetuate them indefinitely; the in-hospital lorazepam taper is explicitly closed
- **Activation-window counseling for sertraline first 2 weeks** — the highest-yield outpatient hand-off item for SSRI initiation in a depression-with-passive-SI cohort
- **`[VERIFY: ...]` flag** targets the behavioral-health-unit attestation variant rather than the acute-hospital default — the same `discharge_attestation_block` hook fires, but the rendering is gated on `practice_setting=behavioral_health_unit`
- **No invented findings** — the example does not back-fill an SUD treatment history the patient does not have (the patient is explicitly noted as having no prior SUD treatment episode of care), does not invent a state-specific Part 2 overlay citation, and does not invent an employer-disclosure scope beyond what the authorization names

### What the two examples together demonstrate

The two examples cover the two ends of the discharge spectrum that the skill is asked to support most often. The HFrEF example shows the highest-volume medical-readmission risk pattern in primary-care-aligned hospital medicine: the medication-reconciliation table with NEW / CHANGED / DISCONTINUED tags, the combined-ACEi-MRA safety flag with explicit BMP follow-up, the orphaned-pending-lab risk closed by naming the responsible party, and the cardiac-defaults red-flag list. The behavioral-health example shows the highest-stakes regulatory-overlay pattern in psychiatric inpatient discharge: 42 CFR Part 2 segregation into a separately-labeled section gated on written authorization, HEDIS FUH 7-day follow-up scheduling, the Stanley-Brown safety plan with lethal-means restriction surfaced in both clinical letter and patient-facing summary, a behavioral-health-keyed red-flag list, finite-course markers on thiamine and folate, and the rule against benzodiazepine re-prescription in a post-AUD-detox patient. Between the two, the skill exercises every named `config.yml` hook at least once: `practice_setting` (acute_hospital vs. behavioral_health_unit), `discharge_attestation_block` (acute-hospital variant vs. behavioral-health-unit variant), `medication_reconciliation_format` (table-with-status-tags in both), `red_flag_phrasing` (HF service-line vs. behavioral-health service-line), `home_health_referral_template` (HHA in Example 1 vs. IOP referral in Example 2), `pending_lab_responsibility_default` (PCP in both, but routed differently), `follow_up_window_defaults` (HF clinic 7 days vs. HEDIS FUH 7 days), `readmission_risk_flagging` (combined ACEi+MRA in Example 1 vs. activation-window SSRI + sleep-deprivation triggers in Example 2), `confidentiality_overlays` (none-applied in Example 1 vs. part_2_applicable=true in Example 2), `patient_facing_avs_flag` (true in both, with different reading-level voice), `output_destination` (`outputs/discharge-summaries/` in both), and `config_missing_behavior` (`flag_and_proceed` in both).
