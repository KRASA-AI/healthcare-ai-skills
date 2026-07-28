---
name: "Pre-Visit Chart Summarizer"
category: operations
tools: [claude, chatgpt]
difficulty: beginner
time_saved: "~10 min/patient"
version: 1.4
last_eval_score: 9.3
---

# 📊 Pre-Visit Chart Summarizer

## Purpose

Synthesize a patient's medical record into a concise, actionable summary that a clinician can review in under two minutes before walking into the exam room.

## When to Use

Use this skill before a scheduled patient encounter when you need to quickly orient on a patient's history. Common scenarios include:

- Morning chart prep for a full day of clinic appointments
- Preparing for a follow-up visit after hospitalization or specialist consult
- Covering for a colleague and reviewing an unfamiliar patient panel
- Telehealth visits where efficient review maximizes limited virtual face time
- Complex patients with lengthy records who need key details surfaced

## Required Input

Provide the following:

1. **Patient chart data** — Any combination of: problem list, medication list, recent visit notes, lab results, imaging reports, specialist consults, hospital discharge summaries, or active referrals. Paste in raw text, bullet points, or dictated notes — any format is fine
2. **Visit reason** (optional but helpful) — The scheduled reason for today's visit, chief complaint, or visit type (annual wellness, follow-up, acute, etc.)
3. **Provider focus areas** (optional) — Any specific conditions, lab values, or concerns the provider wants highlighted

## Instructions

You are a skilled healthcare professional's AI assistant. Your job is to distill a patient's medical record into a crisp pre-visit summary that helps the provider walk into the encounter fully prepared.

**Before you start:**
- Load `config.yml` from the repo root for facility preferences and formatting standards
- Reference `knowledge-base/terminology/` for correct clinical terms and accepted abbreviations
- Use the facility's communication tone from `config.yml` → `voice`

**Use practice-specific pre-visit hooks from `config.yml` when present:**

- **`config.yml` → `panel_quality_programs`** — the practice's active value-based / quality-program enrollments (e.g., `medicare_aco: "[ACO REACH name]"`, `commercial_vbp: ["Aetna ACN", "Cigna OAP-V"]`, `medicaid_pcmh: true`, `mips: { reporting_year: 2026, measure_set: ["CDC: HbA1c poor control", "BCS-E", "COL-E", "CDP", "DSF"] }`). When the patient is attributed to one of these programs, surface the program-specific care gaps in **Section e (Care Gaps & Action Items)** by program label so the visit can close them.
- **`config.yml` → `provider_focus_overrides`** — per-provider standing-order priorities (e.g., for Dr. Romero: always surface CKD eGFR/UACR trend; for Dr. Lee: always surface PHQ-9/GAD-7 trend and last-administered date). Honor these even if they were not named in today's input.
- **`config.yml` → `appointment_type_emphasis`** — emphasis profile by visit type: `annual_wellness_visit` ⇒ surface USPSTF screening calendar + AWV-specific elements (HRA, cognitive screen, advance-care planning eligibility); `medicare_awv` ⇒ AWV elements + chronic-care-management eligibility (G0506 / 99490 / 99439); `chronic_care_followup` ⇒ trended labs + medication-titration window + adherence signal; `acute` ⇒ recent encounters within 30 days + active prescriptions only; `pre_op` ⇒ surgical history + anticoagulation + last cardiac/pulmonary workup; `telehealth` ⇒ vitals last in-clinic + adherence + barriers; `transition_of_care` ⇒ discharge summary key elements + 7-day follow-up window status.
- **`config.yml` → `risk_stratification_inputs`** — the practice's locally configured risk inputs (e.g., HCC RAF score current vs. prior; CHF readmission-risk model; controlled-substance MED total; SDOH risk tier from PRAPARE/AHC-HRSN). When a risk score is configured and exceeds threshold, surface it under **Section a (Patient Snapshot)** as a one-line risk tag.
- **`config.yml` → `panel_voice`** — the practice voice for written summaries (e.g., `"clipped, abbreviation-heavy, surgical resident"` vs. `"explanatory, family-medicine attending"`). The summary still hits the same six sections but the tone matches the receiving provider's preference.
- **`config.yml` → `time_target`** — overrides the default 90-second review target (e.g., complex-care clinics may want a 3-minute summary with deeper pharmacy detail; same-day urgent-care wants a 30-second triage). Compress or expand only the elaborative content; never drop a section.
- **`config.yml` → `provider_roster`** — keyed roster of the practice's clinicians with display name, credentials, and NPI (e.g., `romero: { display: "A. Romero, MD", npi: "[##########]", panel: "primary care" }`). When the visit's PCP/rendering provider is known, auto-fill the summary header's `PCP:` line and any provider reference from the roster rather than echoing raw chart text or leaving `[provider]`. When the provider is not in the roster, print the name as given and flag `[VERIFY: provider not in roster]`. This is what makes the header line ("PCP: A. Romero, MD") populate from config instead of being hand-typed each run.
- **`config.yml` → `ehr_paste_target`** — the named destination the summary is pasted into and its formatting constraints (e.g., `epic_previsit_planning: { max_chars: 4000, plain_text_only: true }`, `athena_clinical_inbox: { markdown_ok: false }`, `printout: { letter_page: true }`). Drives whether the output uses the boxed ASCII header + table layout (fine for monospace EHR notes and printouts) or degrades gracefully to plain wrapped text with no box-drawing characters when the target field strips them. Default to the boxed layout if absent.
- **`config.yml` → `practice_abbreviation_set`** — the practice's approved abbreviation/shorthand list (or a named standard, e.g., `joint_commission_do_not_use: enforced`). Drives which abbreviations the summary may use to save space and, critically, which it must **spell out** because they appear on the practice's Do-Not-Use list (e.g., never abbreviate "units" as "U", "daily" as "QD", or "morphine sulfate" as "MS"). When absent, default to common, unambiguous clinical abbreviations and honor the Joint Commission Official "Do Not Use" list regardless.
- **`config.yml` → `config_missing_behavior`** — if a config key is absent, fall back to the documented defaults (90 seconds, generic problem-list framing, USPSTF-anchored gaps, boxed layout, Joint Commission Do-Not-Use list honored) rather than inventing a quality program, override, risk score, provider NPI, or paste target.

**Process:**

1. Review all chart data provided by the user
2. Do NOT ask clarifying questions unless absolutely critical — the goal is speed. Make reasonable assumptions and note them

   **Missing-data provenance (applies to every section — this is a safety rule, not a style preference).** A pre-visit summary is read as a trust signal: whatever the provider does not see flagged, they assume was checked and is fine. So never let *absence from the provided record* read as a *clinical negative*. "Not in the record I was given" and "documented as absent/normal" are different facts and must be written differently — the first is a gap to close, the second is a finding. Concretely:

   - Only state a negative (e.g., "no anticoagulant," "no drug allergies," "no advance directive," "no recent imaging") when the record **affirmatively documents** it. If the domain simply isn't present in the paste, write it as an explicit gap line — "Anticoagulation: not addressed in provided records — VERIFY" — rather than omitting it silently or implying a reassuring negative.
   - Treat these high-consequence domains as **must-account-for**: they appear in the summary *either* as a documented finding *or* as a named gap, never as silence — anticoagulation/antiplatelet status, drug allergies, current problem list, code status / advance directive (for AWV, transition-of-care, and pre-op visits), controlled-substance / opioid regimen, and the most recent relevant imaging or diagnostic for the visit reason.
   - When the input is **thin** (new patient, external records pending, a partial paste), say so in one line at the top of Patient Snapshot — "Summary built from a partial record; external records pending" — and lean the Suggested Visit Agenda toward *obtaining the missing record*, not toward false completeness. A short honest summary beats a confident one built on gaps.
   - Distinguish "stale" from "absent": a lab or screen that exists but is out of date is a trend/recency flag (see Sections d–e); a lab or screen with no value at all is a provenance gap. Route each to the right place and use a `[VERIFY: ...]` flag for anything a clinician must confirm before acting.

3. Produce a structured summary with the following sections:

   **a. Patient Snapshot** (2-3 lines max)
   - Age, sex, primary diagnoses, and reason for today's visit
   - Functional status or relevant social context if available

   **b. Active Problem List with Status**
   - Each active condition with current management status (controlled, worsening, newly diagnosed, monitoring)
   - Flag any conditions that are off-track or approaching a decision point

   **c. Medication Reconciliation Highlights**
   - Current medication list (or key medications if list is long)
   - Flag recent changes, high-risk medications (anticoagulants, opioids, insulin), or potential interactions
   - Note any adherence concerns if documented

   **d. Recent Results & Trends**
   - Key lab values with trends (improving, stable, worsening) — especially A1c, lipids, renal function, CBC if relevant
   - Recent imaging or procedure results and their significance
   - Outstanding or pending orders

   **e. Care Gaps & Action Items**
   - Overdue preventive care (screenings, immunizations, wellness visits)
   - Referrals that were made but not yet completed
   - Specialist recommendations not yet acted upon
   - Quality measure gaps (e.g., diabetic eye exam, depression screening)

   **f. Suggested Visit Agenda** (3-5 bullet points)
   - Based on the chart data and visit reason, suggest the most important topics to address during this encounter
   - Prioritize by clinical urgency and patient impact

4. Keep the total summary to approximately one page — conciseness is the primary value
5. Use standard clinical abbreviations to save space (HTN, DM2, CKD, etc.)
6. Bold or flag anything requiring urgent attention

**Output requirements:**
- Scannable format designed for a 90-second review (or the `time_target` override)
- Correct clinical terminology, using only abbreviations permitted by `practice_abbreviation_set`; spell out anything on the practice's (or Joint Commission's) Do-Not-Use list
- Prioritized and actionable — not just a data dump
- **Data-provenance honest** — a documented negative is written as a finding; a domain absent from the provided record is written as a named gap, never as silence or an implied negative (see the Missing-data provenance rule above). The high-consequence domains are always accounted for one way or the other
- Header auto-populated from `provider_roster` (PCP name, credentials) rather than hand-typed
- Layout matched to `ehr_paste_target` — boxed ASCII + tables for monospace EHR fields and printouts; plain wrapped text (no box-drawing characters) when the target field strips them
- Ready to print or paste into the named pre-visit planning field
- Saved to `outputs/` if the user confirms

## Example Output

The two worked examples below exercise four of the seven `appointment_type_emphasis` paths — `chronic_care_followup` and `medicare_awv` directly, plus the `transition_of_care` sub-branch and the implied default by contrast — across two different attribution programs (ACO REACH commercial-MA vs. MIPS-reporting traditional Medicare) and two different per-provider override profiles. Example 1 is the highest-volume primary-care archetype (complex chronic-care follow-up). Example 2 is the highest-volume Medicare-clinic archetype (first-time AWV stacked against a recent discharge and an active SDOH risk score).

### Example 1 — Complex chronic-care follow-up (T2DM + HTN + CKD3a + GAD, ACO REACH-attributed)

Uses `appointment_type_emphasis: chronic_care_followup`, `panel_quality_programs: { medicare_aco: "ACO REACH — [Network]" }`, `provider_focus_overrides: ["always surface CKD trend (eGFR/UACR)", "always surface PHQ-9 if last administered > 12 months"]`, and the default `time_target: 90` seconds.

**Input (raw chart paste):**
```
Patient: Maria Lopez, 64F, est'd patient, MRN 4417XXXX, attributed to ACO REACH — [Network]
Visit reason today (2026-04-25): chronic-care follow-up, T2DM and HTN
Problem list (active): T2DM (E11.9) since 2018, HTN (I10), CKD stage 3a (N18.31), 
hyperlipidemia (E78.5), GAD (F41.1), obesity BMI 32.1
Meds: metformin 1000 BID, empagliflozin 10 daily (added 2025-11), lisinopril 20 daily, 
amlodipine 5 daily, atorvastatin 40 QHS, sertraline 50 daily, ASA 81 daily
Allergies: sulfa (rash)
Recent labs (2026-02-18):
  HbA1c 7.4 (prior 7.8 on 2025-08-12; 8.1 on 2025-04-09)
  eGFR 51 (prior 54 on 2025-08-12; 58 on 2024-09)
  UACR 38 mg/g (prior 22 on 2025-08-12)
  LDL 92 (prior 108 on 2025-08-12)
  Lipid panel total chol 178, HDL 44, TG 156
  K 4.2, Na 138, BMP otherwise normal
  AST 28, ALT 31
Last visit 2026-02-18: titrated empa from 10 to 25 mg planned; pt declined for cost — 
referred to PA for $0 copay program; status pending
Last specialist visit: nephrology 2025-12-10 — recommend continue empa, monitor UACR q3 mo, 
recheck eGFR q3 mo, no change to lisinopril
Care gaps: 
  - eye exam overdue (last 2024-03)
  - pneumococcal not up to date (PCV20 not given; last PPSV23 2019)
  - colonoscopy due (last 2014; pt declined Cologuard 2025)
  - DEXA never done; pt is 64F
  - PHQ-9 last 2024-11 (score 7); GAD-7 not formally administered in 18 mo
Outstanding orders: empagliflozin 25 mg PA pending; pneumococcal vaccine ordered 2026-02-18, 
not given yet
Hospitalizations: none in 5 yr
SDOH: lives alone, daughter nearby, retired, Medicare + supplement, transportation OK, 
food insecurity screened negative 2025-08
PCP-noted concerns: pt has mentioned worsening sleep over last 2 visits
```

**Output (90-second pre-visit summary):**
```
═══════════════════════════════════════════════════════════════════════
PRE-VISIT SUMMARY — Maria Lopez, 64F · DOB last 4 /1961 · MRN /4417
Visit:  2026-04-25 · Chronic-care follow-up (T2DM + HTN) · 30 min slot
Panel:  ACO REACH — [Network]   |   PCP: A. Romero, MD
═══════════════════════════════════════════════════════════════════════

PATIENT SNAPSHOT
64F with T2DM, HTN, **CKD 3a (eGFR 51, downtrending — see Section c)**,
hyperlipidemia, GAD, obesity. Lives alone; retired; daughter nearby.
ACO-attributed; multiple care gaps below.

ACTIVE PROBLEMS — STATUS
- T2DM (E11.9)            — IMPROVING. A1c 7.4 → 7.8 → 8.1 (newest first).
                            Empa add Nov-2025 → ~0.7 % drop in 6 mo.
                            **Decision pending:** empa 10 → 25 mg
                            (declined Feb for $0-copay PA — status?).
- HTN (I10)               — Controlled at last visit; no home BPs in
                            chart since Feb. **Ask:** home-BP log.
- CKD 3a (N18.31)         — **WORSENING SIGNAL.** eGFR 58 → 54 → 51 over
                            18 mo; UACR 22 → 38 (now A2). On empa +
                            ACEi (renal-protective). Nephrology saw
                            12/2025: continue empa, q3-mo eGFR + UACR.
                            **Action:** confirm renal labs were drawn
                            today; if not, order before pt leaves.
- Hyperlipidemia (E78.5)  — Improving. LDL 108 → 92. Atorva 40 — at
                            target for ASCVD secondary prevention level
                            given DM + CKD.
- GAD (F41.1)             — Stable on sertraline 50. **PHQ-9 / GAD-7
                            both > 12 mo old.** Pt has been mentioning
                            poor sleep — re-screen today.
- Obesity (E66.9, BMI 32) — Comorbid; eligible for IBT (G0447) under
                            Medicare; not yet engaged.

MEDICATIONS — HIGH-LEVERAGE NOTES
- empagliflozin 10 mg daily — **PA pending** for 25 mg ($0-copay
  manufacturer assistance). Status check before titrating today.
- lisinopril 20, amlodipine 5 — both at moderate dose; room to
  intensify if home BPs are uncontrolled.
- ASA 81 — appropriate for known DM + CKD ASCVD risk.
- No new meds since 2025-11. Adherence not formally documented.

RECENT RESULTS & TRENDS (most recent: 2026-02-18; ~9 wk old)
| Lab     | Current | Prior  | Older  | Trend       |
|---------|---------|--------|--------|-------------|
| HbA1c   | 7.4     | 7.8    | 8.1    | improving   |
| eGFR    | 51      | 54     | 58     | **worsening**|
| UACR    | 38 (A2) | 22     | —      | **worsening**|
| LDL     | 92      | 108    | —      | improving   |
| K       | 4.2     | —      | —      | normal      |
**Pending today:** confirm renal labs, A1c (q3-mo per nephrology
recommendation — last 9 wk ago, due now).

CARE GAPS & ACTION ITEMS (ACO REACH-relevant flagged ⭐)
⭐ Diabetic eye exam (CDC: EED)  — last 2024-03; OVERDUE > 12 mo
⭐ HbA1c control (CDC: HbA1c)    — at goal; redraw today closes measure
⭐ Statin therapy (CDC: SUPD)    — on; closed
⭐ Colorectal screening (COL-E)  — last colo 2014, declined Cologuard
                                    2025; **discuss again — high gap**
⭐ Adult immunizations            — PCV20 not given; PPSV23 2019;
                                    flu / Tdap status to verify
   DEXA (USPSTF B for ≥ 65; pt 64) — discuss; consider next visit
   PHQ-9 / GAD-7                 — > 12 mo; re-screen today
   Medicare AWV (G0438/G0439)    — eligible if not done in 12 mo;
                                    last AWV [VERIFY in chart]
   Empa 25 mg PA                 — status pending; close loop today

OUTSTANDING / IN-FLIGHT
- Empagliflozin 25 mg PA — pending since Feb-2026; check status before
  titration is offered.
- Pneumococcal vaccine — ordered Feb-2026, not administered. Today
  during visit if no contraindication.

PROVIDER-SPECIFIC EMPHASIS (per config.yml → provider_focus_overrides)
- Always surface CKD trend  →  done, see Section c above.
- Always surface PHQ-9 if > 12 mo  →  flagged for today.

SUGGESTED VISIT AGENDA (in priority order)
1. CKD trend — confirm renal labs drawn today; review with patient and
   reinforce nephrology plan; document UACR worsening to A2.
2. Empagliflozin 25 mg titration — PA status; if approved, advance.
3. Sleep / mood — administer PHQ-9 and GAD-7; if PHQ-9 > 9 or sleep
   complaint persists, address pharmacotherapy or refer.
4. Colorectal screening — repeat the offer (colo vs. Cologuard) given
   open ACO gap; document patient decision.
5. Pneumococcal vaccine — administer today; document.

[VERIFY: Medicare AWV last performed — chart-search result]
[VERIFY: empagliflozin 25 mg PA status — call payer or check portal]
[VERIFY: home BP log brought to visit — front desk to ask at check-in]

═══════════════════════════════════════════════════════════════════════
Read time target: 90 sec  ·  Generated 2026-04-25 06:55 from chart paste
═══════════════════════════════════════════════════════════════════════
```

The example illustrates the target: a **scannable** one-page summary that hits the six sections, surfaces a **worsening renal trend** the provider would otherwise have to derive from the lab table, flags the **stale PHQ-9** per the per-provider override, **labels every ACO-relevant care gap** with the program-specific measure ID (CDC: EED, COL-E, etc.) so the visit can close them, **respects the practice's config-driven emphasis profile** for chronic-care follow-up (trended labs + titration window + adherence signal), and reserves three explicit `[VERIFY: ...]` flags for the items the chart paste alone does not resolve.

### Example 2 — First-time Medicare AWV with recent discharge + active SDOH risk (71F traditional Medicare, MIPS-reporting, PRAPARE tier 2)

Uses `appointment_type_emphasis: medicare_awv` (primary path) layered with the `transition_of_care` sub-branch (discharge 11 days ago, inside the 7-day-FUH measurement window when the visit was scheduled), `panel_quality_programs: { mips: { reporting_year: 2026, measure_set: ["CDC: HbA1c poor control", "BCS-E", "COL-E", "CDP", "DSF"] } }`, `risk_stratification_inputs: { hcc_raf_delta: true, prapare_tier: "active", chf_readmit_risk: true }`, `provider_focus_overrides: ["always surface CHF readmission-risk score when present", "always surface advance-care-planning eligibility on AWV"]`, `panel_voice: "explanatory, family-medicine attending"`, and `time_target: 120` seconds (the panel's AWV-day override). The AWV-day target is 120 sec rather than 90 because the AWV requires HRA, cognitive screen, and ACP-eligibility surfacing.

**Input (raw chart paste):**
```
Patient: Ethel Washington, 71F, est'd patient, MRN 0824XXXX, traditional Medicare
+ Plan G supplement. PCP: J. Park, MD.
Visit reason today (2026-04-30): scheduled Initial AWV (G0438) — pt has never had an AWV;
also stands as post-discharge 7-day FUH visit (discharge 2026-04-19, day 11 today —
window technically closed at day 7 but visit was scheduled at day 11; document
attempt-to-meet for MIPS).

Problem list (active): HFpEF (I50.32) — LVEF 55%, dx 2022, NYHA II; HTN (I10);
T2DM (E11.9, A1c 6.7); paroxysmal AFib (I48.0) on apixaban; CKD stage 2 (eGFR 68);
hyperlipidemia (E78.5); osteoarthritis bilat knees (M17.0); MDD recurrent in
remission (F33.42) on escitalopram 10; mild cognitive impairment (G31.84) —
flagged 2025-11 on MoCA 22/30.

Recent discharge 2026-04-19 (5-day inpatient stay): admitted 4/14 from ED for
acute on chronic HFpEF exacerbation triggered by NSAID self-treatment for knee
pain; IV diuresis (furosemide 40 IV BID → 40 PO BID at discharge from prior
20 PO daily); apixaban continued; counseled to discontinue NSAIDs; cardiology
follow-up scheduled 5/14; discharge weight 168 lb (admission 173 lb, dry-weight
target 167). Discharge BNP 412 (admit 1,840). Hospital pharmacist noted
medication-recon discrepancy: home list had losartan 50 daily; discharge had
both losartan 50 AND new lisinopril 10 (dual ACE/ARB — flagged but not yet
resolved). CHF Readmission-Risk score (LACE+ adapted): 14 (HIGH).

Meds (post-discharge, current):
  furosemide 40 mg BID (UP from 20 daily) — diuretic, ISMP-monitoring
  apixaban 5 mg BID — anticoagulant, ISMP-HIGH-ALERT
  metoprolol succ 50 mg daily — beta-blocker
  losartan 50 mg daily — ARB (home regimen)
  lisinopril 10 mg daily — ACE-I (added at discharge — **dual ACE/ARB flag**)
  atorvastatin 40 mg QHS
  metformin 1000 mg BID
  escitalopram 10 mg daily
  acetaminophen 500 mg TID PRN knee pain (replacement for NSAID)
  KCl 20 mEq daily (started for diuretic)
  ASA 81 mg daily — held in chart 2024 due to apixaban; check if reordered

Allergies: NKDA.

Recent labs (discharge 2026-04-19):
  BMP: Na 136, K 4.1, Cl 102, HCO3 25, BUN 24, Cr 1.0, eGFR 68, glu 142
  Mg 1.9 (low-normal)
  CBC: Hgb 11.8 (baseline 12.5), Plt 198
  BNP 412 (admit 1,840 → discharge 412)
  Trop neg ×2, INR 1.1
  HbA1c 6.7 (last 3 mo); LDL 78; TSH 2.1
  UACR 8 mg/g (normal)

Outside records (cardiology 2026-03-12 — pre-admission): TTE LVEF 55%, mild
LAE, no significant valvular disease, grade I diastolic dysfunction.

Care gaps / preventive (from chart review):
  - Mammogram (BCS-E): last 2024-01 — DUE in 2026
  - Colorectal screening (COL-E): last colo 2018 — eligible to extend to 10-yr;
    next due 2028 (already closed)
  - DEXA: last 2022, T-score -1.4 (osteopenia)
  - Cervical cancer screening: age out
  - PCV20: NOT GIVEN; PPSV23 2020; PCV13 2019
  - RSV vaccine: NOT GIVEN
  - Shingrix series: incomplete (#1 given 2023-08, #2 never given)
  - 2025-26 flu: GIVEN 2025-10-15
  - Diabetic eye (CDC: EED): last 2024-06 — DUE
  - HbA1c (CDC): at goal 6.7 — closed
  - CDP (depression screening): last 2025-11 (PHQ-9 4); within 12 mo
  - Statin therapy (DSF): on atorva 40 — closed
  - Advance Care Planning (G0444 + ACP add-on): no chart record of ACP
    conversation; pt has no documented advance directive or POLST
  - Cognitive assessment: 2025-11 MoCA 22/30 (MCI) — re-evaluate per AWV
    requirement; consider chart formal cognitive-impairment workup
  - HRA: never completed (this is initial AWV)

SDOH (PRAPARE 2026-02-10 — TIER 2 active):
  - Lives alone, widowed 2024
  - Housing: stable, owns home
  - Food insecurity: positive (one Hunger Vital Sign hit — "ran out of food
    once in past 30 days")
  - Transportation: limited (does not drive; relies on niece, 2x/wk)
  - Social isolation: positive (no daily contact most days; weekly church
    pre-COVID, has not resumed)
  - Income: SS only; reports moderate medication-cost burden ($120/mo OOP)
  - Caregiver: niece, lives 22 mi away

HCC-RAF (current claim year): 1.34 (prior year 1.18, delta +0.14 driven by
new diagnoses HFpEF active + CKD2 + MCI). Open suspect from chart:
proteinuria pattern would code N18.32 (CKD3a) if eGFR confirmed below 60
on this visit's labs.

PCP-noted concerns: pt mentioned at discharge call that she has been
"forgetting things since the hospital"; family expressed concern about
home safety alone post-discharge.
```

**Output (~120-second pre-visit summary):**
```
═══════════════════════════════════════════════════════════════════════
PRE-VISIT SUMMARY — Ethel Washington, 71F · DOB last 4 /1955 · MRN /0824
Visit:  2026-04-30 · **Initial Medicare AWV (G0438)** · 60 min slot
        + serves as POST-DISCHARGE FOLLOW-UP (D/C 2026-04-19, day 11)
Panel:  Traditional Medicare + Plan G  |  MIPS-reporting 2026
        PCP: J. Park, MD               |  Risk: **CHF readmit LACE+ 14 (HIGH)**
═══════════════════════════════════════════════════════════════════════

PATIENT SNAPSHOT
71F, widowed, lives alone, post-discharge day 11 from a 5-day HFpEF
exacerbation triggered by NSAID self-treatment. **CHF readmission risk
HIGH (LACE+ 14).** Active SDOH: food insecurity (+), social isolation (+),
limited transportation. **MCI flagged 2025-11 (MoCA 22/30) and post-d/c
family safety concern this week.** Has never had a Medicare AWV — today
is initial G0438. HCC-RAF current 1.34 (prior 1.18, delta +0.14).

ACTIVE PROBLEMS — STATUS
- HFpEF (I50.32) NYHA II   — **POST-EXACERBATION, day 11.** Diuresis to
                              168 lb (target 167). BNP 1840 → 412.
                              **PRIORITY: weight check today, lung exam,
                              orthostatics, JVP, edema. Confirm symptom
                              status vs. baseline.**
- AFib (I48.0)             — On apixaban 5 BID (ISMP-HIGH-ALERT). Last
                              INR-equivalent monitoring per chart.
                              **Verify pt taking BID, not QD.**
- HTN (I10)                — **DUAL ACE/ARB ON DISCHARGE LIST**
                              (lisinopril 10 + losartan 50). Hospital
                              pharmacy flagged; **NOT YET RESOLVED.**
                              **Decision today: confirm which agent
                              continues; discontinue the other.**
- T2DM (E11.9)             — A1c 6.7 at goal. Watch overshoot given
                              new diuretic + reduced PO intake post-d/c.
- CKD 2 → suspect 3a       — eGFR 68 at d/c. **HCC suspect:** if today's
                              eGFR < 60, code N18.32 (CKD3a) and
                              re-engage ACEi-vs-ARB decision in that
                              context. Pull-today labs include BMP.
- MCI (G31.84)             — 2025-11 MoCA 22/30. **NEW family concern
                              about post-d/c forgetting.** Re-evaluate
                              cognition per AWV requirement; consider
                              formal workup referral.
- MDD recurrent, remission — Escitalopram 10. PHQ-9 4 (Nov-2025, within
                              12-mo). **Re-screen at AWV given recent
                              hospitalization + widowhood + social
                              isolation.**
- Osteoarthritis knees     — **NSAID-RESTRICTED post-HFpEF.** APAP 500
                              TID PRN replacement in place. **Reinforce
                              today** (this was the precipitant).
- Osteopenia (M85.80)      — 2022 DEXA T -1.4. Repeat DEXA due 2026
                              (4-yr interval).

MEDICATIONS — HIGH-LEVERAGE NOTES
- furosemide 40 mg BID  — **DOUBLED from 20 daily at d/c.** Weight today
  is the primary titration signal; ↓ to 40 daily if weight at dry target.
  K 4.1 + Mg 1.9 at d/c; pt on KCl 20 mEq — **recheck K + Mg today.**
- apixaban 5 mg BID  — **ISMP HIGH-ALERT.** Confirm dosing schedule with
  patient. **No NSAID** reinforced.
- losartan 50 + lisinopril 10  — **DUAL ACE/ARB FLAG (UNRESOLVED).**
  Action: discontinue one today. Recommended: continue losartan (home
  regimen), discontinue the new-at-d/c lisinopril, unless rationale on
  d/c summary; document MD decision and reconciliation note.
- metoprolol succ 50  — continue; pulse-check today.
- ASA 81  — **HELD per 2024 chart note (on apixaban).** Confirm not
  resumed unintentionally during admission; if on d/c list, hold.
- KCl 20 mEq  — new for diuretic. Recheck K today.
- escitalopram, metformin, atorva  — continue.
- APAP 500 TID PRN  — knee pain; reinforce NSAID prohibition.

RECENT RESULTS & TRENDS (most recent: 2026-04-19 d/c labs)
| Lab     | Current | Trend         | Action today                |
|---------|---------|---------------|-----------------------------|
| Weight  | 168 lb  | ↓ from 173    | check today vs. dry 167     |
| BNP     | 412     | ↓ from 1,840  | trend established           |
| K       | 4.1     | low-normal    | **recheck (KCl + diuretic)**|
| Mg      | 1.9     | low-normal    | **recheck**                 |
| eGFR    | 68      | stable        | **recheck — suspect CKD3a** |
| Hgb     | 11.8    | ↓ from 12.5   | monitor; not transfusion-eligible
| BMP     | wnl     | —             | repeat today                |
| HbA1c   | 6.7     | at goal       | next due 2026-Q3            |
| INR     | 1.1     | not on warfarin| n/a                        |
**Pull-today labs:** BMP + Mg (renal + electrolyte trend), weight.

MIPS CARE GAPS & ACTION ITEMS (MIPS-relevant flagged ⭐)
⭐ HbA1c poor control (CDC)   — 6.7, NOT in poor-control denom — closed
⭐ BCS-E (mammogram)           — last 2024-01; **due now (2026)**
⭐ COL-E (colorectal)          — colo 2018; closed thru 2028
⭐ CDP (depression screen)     — PHQ-9 11/2025 score 4; closed thru 11/2026
⭐ DSF (statin)                — on atorva 40; closed
   Diabetic eye (CDC-EED)     — last 2024-06; **due — order today**
   PCV20                       — **NOT GIVEN — give today** (eligible)
   RSV vaccine                 — **NOT GIVEN — discuss today**
   Shingrix #2                 — incomplete (#1 2023-08); **schedule**
   DEXA                        — last 2022, due 2026; **order**
**AWV-SPECIFIC ELEMENTS REQUIRED TODAY (G0438)**
   HRA                         — patient has never completed; complete today
   Cognitive screen            — MoCA 22/30 11/2025 + new family concern;
                                 re-administer + consider formal workup
                                 (rule out post-hospital delirium / vasc
                                 / Alzheimer's pattern)
   ACP (G0444 + add-on code)  — **pt has NO advance directive on file;**
                                 PCP override fires: surface ACP at AWV.
                                 Initiate today; offer to schedule a
                                 dedicated ACP visit if not completed.
   Functional / fall risk      — high-priority given post-d/c, MCI,
                                 lives alone, social isolation, niece
                                 2x/wk only — administer Timed Up and Go
                                 + STEADI fall-risk screen
   IPPE vs. AWV                — IPPE window passed (>12 mo from Med
                                 enrollment); initial AWV (G0438) is
                                 the correct code today, not G0402.
   Chronic Care Management    — pt clearly eligible for CCM (G0506
   eligibility (post-AWV)       initiating + 99490 monthly); enroll
                                 today or schedule CCM consent visit.

SDOH-DRIVEN ACTION (per PRAPARE 2026-02-10, TIER 2 active)
- Food insecurity (+)         — refer to FQHC food pantry partnership;
                                 SNAP screening; G0136 SDOH risk
                                 assessment code eligible
- Social isolation (+)        — connect to senior center / community
                                 program; flag MA care-mgmt outreach
- Transportation limited      — confirm transport for cardiology 5/14
                                 visit; verify Medicare NEMT options
- Caregiver capacity          — niece 22 mi away, 2x/wk; given post-d/c
                                 + MCI + family concern, **escalate**:
                                 home-health PT/OT eval, consider
                                 assisted-living counseling, consider
                                 medical-alert system

OUTSTANDING / IN-FLIGHT
- Dual ACE/ARB reconciliation — UNRESOLVED from d/c. **TODAY.**
- Cardiology follow-up 5/14   — confirmed; transportation arranged?
- Home-health post-d/c        — was this ordered at d/c? **VERIFY**

PROVIDER-SPECIFIC EMPHASIS (per config.yml → provider_focus_overrides)
- Always surface CHF readmit-risk score when present  →  done, LACE+ 14 above
- Always surface advance-care-planning eligibility on AWV  →  surfaced, ACP today

HCC-RAF / CODING OPPORTUNITIES (HCC-RAF delta surfacing enabled)
- HCC suspect: confirm CKD3a (N18.32) if today's eGFR <60 — supported
  by trended labs and proteinuria pattern
- HCC 87 HFpEF active: actively managed today, code I50.32
- HCC 18 DM with chronic complication: A1c at goal, but CKD comorbidity
  active — confirm coding
- HCC 60 MDD recurrent: in partial remission, currently on tx — confirm
  coding context; do not code "in full remission" if symptoms returning
  post-hospitalization
- New HCC capture potential: if formal cognitive workup confirms a
  named cognitive-impairment diagnosis (vs. MCI), recode at the next
  encounter

SUGGESTED VISIT AGENDA (in priority order, 60-min slot)
1. **Acute HFpEF post-d/c assessment** — weight, lung exam, edema, JVP,
   orthostatic vitals, symptom check; diuretic titration decision
2. **Dual ACE/ARB reconciliation** — confirm one continues, one stops;
   document decision and rationale; pharmacy follow-up
3. **Recheck BMP + Mg today** — renal/electrolyte trend; confirm or
   change CKD code based on eGFR
4. **AWV core elements** — HRA, cognitive screen (MoCA + family concern
   context), depression re-screen, functional / fall risk
5. **ACP conversation initiation** — given AWV + post-d/c + MCI +
   family concern, ACP is the single highest-leverage AWV component
   today; bill G0444 + ACP add-on
6. **Immunization sweep** — PCV20 today, RSV decision, Shingrix #2 schedule
7. **CCM enrollment offer** — G0506 initiating visit today or schedule
8. **SDOH-driven follow-up** — G0136 risk assessment, home-health
   re-engagement, transportation for 5/14 cardiology, caregiver
   capacity escalation

[VERIFY: Was home-health PT/OT ordered at discharge — yes/no?]
[VERIFY: ASA 81 — on d/c list or held? If on list, hold (apixaban)]
[VERIFY: Last formal cognitive workup (CT/MRI brain, B12, TSH) — chart search]
[VERIFY: Cardiology 5/14 — transportation arranged?]
[VERIFY: HCC-RAF — is CKD3a (N18.32) currently submitted, or do we need
  to confirm with today's labs and update problem list?]

═══════════════════════════════════════════════════════════════════════
Read time target: 120 sec (AWV-day override per config.yml)
Generated 2026-04-30 06:50 from chart paste
═══════════════════════════════════════════════════════════════════════
```

The second example illustrates how the skill behaves when the visit type is **higher-stakes** than the default 90-second chronic-care follow-up: it expands the time target to the panel's AWV-day override (120 sec), **layers the recent-discharge transition_of_care elements onto the AWV emphasis profile** (post-d/c weight + dual-ACE/ARB reconciliation are pulled forward and called out as ordered priorities, not background context), **surfaces the SDOH risk score** (PRAPARE tier 2 active) as a one-line snapshot tag and then expands into a dedicated SDOH-driven action block with G0136 eligibility, **surfaces the CHF readmission-risk score** (LACE+ 14) on the header line per the `provider_focus_overrides` hook, **demonstrates the AWV-specific elements** the appointment_type_emphasis profile requires (HRA, MoCA + family-concern context, ACP per the second override, functional / fall-risk per the STEADI overlay, IPPE-vs-AWV code disambiguation, CCM eligibility post-AWV), **uses the MIPS measure set** rather than the ACO REACH measure set so the care-gap layering shifts from CDC: EED-style ACO-attributed flags to MIPS-attributed flags, **surfaces HCC-RAF coding opportunities** when the `risk_stratification_inputs: { hcc_raf_delta: true }` hook fires (with a named open-suspect for CKD3a contingent on today's lab), and **flags five explicit `[VERIFY: ...]` items** rather than the three from Example 1 because higher-complexity discharge-stacked AWV visits surface more unresolved chart questions. The `panel_voice: "explanatory, family-medicine attending"` shows up in the more elaborated prose of the action notes vs. the clipped surgical-resident voice the first example would use if configured differently. Together the two examples now demonstrate four of the seven `appointment_type_emphasis` paths and two of the four `panel_quality_programs` configurations, plus both extremes of the `time_target` override and both kinds of `provider_focus_overrides` (single-trend-flag vs. AWV-specific element). Both examples also exercise the **missing-data-provenance** discipline: the ASA line in Example 2 ("HELD per 2024 chart note … confirm not resumed") is written as a provenance question rather than a silent negative, and the `[VERIFY: ...]` blocks in both outputs name the domains the paste alone does not resolve (home-BP log, AWV date, cognitive workup, home-health order) instead of implying they were checked and clear — which is exactly the rule generalized for a thin or partial chart.
