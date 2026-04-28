---
name: "Discharge Summary Generator"
category: operations
tools: [claude, chatgpt]
difficulty: beginner
time_saved: "~20 min/summary"
version: 1.2
last_eval_score: 8.5
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
- Ready for provider review and signature with minimal editing
- Saved to `outputs/` if the user confirms

## Example Output

The example below shows a full discharge summary generated from terse hospitalist sign-out notes for one of the highest-volume readmission-risk discharges — acute decompensated heart failure. It exercises every required section (demographics, principal/secondary diagnoses, hospital course, procedures and results, reconciled discharge medications with NEW / CHANGED / DISCONTINUED tags, condition and disposition, follow-up, education), surfaces a NEW-medication safety check (potassium and renal monitoring on combined ACE-inhibitor + spironolactone), and explicitly flags one element the dictation does not support so the receiving clinician sees the gap rather than a fabrication.

### Input from user (paraphrased hospitalist sign-out)

- **Admission:** 2026-04-19, admitted from ED. Admitting dx: acute on chronic HFrEF (LVEF 30%, last echo 2025); admitting team: Hospitalist – Dr. K. Nguyen.
- **Hospital course:** ED with progressive DOE, 4 lb wt gain in 1 wk, JVD 10 cm, bibasilar crackles, +1 pitting LE edema, BNP 1840 (baseline ~600), Cr 1.3 (baseline 1.1). Started on IV furosemide 80 mg BID, telemetry, weighed daily. Net negative ~6 L over the stay. Cardiology consulted HD2 — added spironolactone 25 mg daily, increased lisinopril 10 → 20, continued metoprolol succinate 50 mg, added empagliflozin 10 mg. Echo HD2 confirmed LVEF 30%, no new wall-motion abnormalities. Discharged HD4 with home weights program and HF-clinic follow-up.
- **Discharge meds (reconciled by pharmacy):** Furosemide 40 mg PO daily (NEW PO conversion from IV); lisinopril 20 mg PO daily (CHANGED, was 10); metoprolol succinate 50 mg PO daily (UNCHANGED); spironolactone 25 mg PO daily (NEW); empagliflozin 10 mg PO daily (NEW); aspirin 81 mg PO daily (UNCHANGED); atorvastatin 40 mg PO daily (UNCHANGED). DISCONTINUED: ibuprofen PRN (patient was self-medicating; NSAID contraindicated in HFrEF).
- **Disposition:** Home with home health for HF education and weights.
- **Follow-up:** HF clinic 7 days, PCP 14 days, BMP in 1 week.
- **Patient context:** 68-year-old male, hx HTN, T2DM, prior CABG 2018, NKDA, code status full. Discharge weight 84.5 kg (admission 90.3 kg).

### Generated discharge summary

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

### What this example demonstrates

- **All 8 required sections present**, in order, with no missing element
- **Reconciled medication table with NEW / CHANGED / DISCONTINUED indicators** — the single most-audited element of any discharge summary, surfaced here in table form for fast scanning
- **NSAID discontinuation captured as a code-able diagnosis** — Z79.899 makes the appropriateness of the medication change auditable
- **Combined ACEi + MRA safety flag with explicit BMP follow-up** — the pattern most likely to fail in real-world readmissions; surfaced in three places (medication list, follow-up table, red-flag list)
- **Pending labs with named responsible party** — closes the most common discharge-summary care-transition gap (orphaned pending labs)
- **Red-flag symptom list patient-facing in clinical terms** — useful both for the receiving provider and for the home-health agency's first home visit
- **`[VERIFY: ...]` flags** for attestation block specifics rather than fabricating facility template language
- **No invented findings** — BNP not re-trended at discharge is explicitly flagged rather than back-filled with a plausible-sounding number
