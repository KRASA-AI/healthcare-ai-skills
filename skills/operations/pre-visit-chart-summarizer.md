---
name: "Pre-Visit Chart Summarizer"
category: operations
tools: [claude, chatgpt]
difficulty: beginner
time_saved: "~10 min/patient"
version: 1.1
last_eval_score: 8.5
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
- **`config.yml` → `config_missing_behavior`** — if a config key is absent, fall back to the documented defaults (90 seconds, generic problem-list framing, USPSTF-anchored gaps) rather than inventing a quality program, override, or risk score.

**Process:**

1. Review all chart data provided by the user
2. Do NOT ask clarifying questions unless absolutely critical — the goal is speed. Make reasonable assumptions and note them
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
- Scannable format designed for a 90-second review
- Correct clinical terminology with standard abbreviations
- Prioritized and actionable — not just a data dump
- Ready to print or paste into a pre-visit planning field
- Saved to `outputs/` if the user confirms

## Example Output

Worked example: 90-second pre-visit summary for a complex chronic-care follow-up — the highest-volume primary-care archetype and the one where chart-diving on the way to the room costs the most time. The example uses `appointment_type_emphasis: chronic_care_followup`, `panel_quality_programs: { medicare_aco: "ACO REACH — [Network]" }`, `provider_focus_overrides: ["always surface CKD trend (eGFR/UACR)", "always surface PHQ-9 if last administered > 12 months"]`, and the default `time_target: 90` seconds.

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
