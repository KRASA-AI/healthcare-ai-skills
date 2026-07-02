---
name: "Medication Reconciliation Assistant"
category: operations
tools: [claude, chatgpt]
difficulty: intermediate
time_saved: "~15 min/reconciliation"
version: 1.1
last_eval_score: 9.40
---

# 💊 Medication Reconciliation Assistant

## Purpose

Compare a patient's medication lists across care-transition points (admission, transfer, discharge, primary care follow-up, specialty visit) and produce a reconciled, structured medication list with changes, discrepancies, therapeutic duplications, interaction concerns, and high-priority clarifying questions — reducing the chance that an adverse drug event slips through a care transition.

## When to Use

Use this skill whenever a provider or pharmacy team needs to reconcile medications at a care transition. Common scenarios include:

- Hospital admission medication reconciliation against the patient's home regimen
- Discharge medication reconciliation from inpatient to home, SNF, rehab, or home health
- Primary care follow-up after a hospitalization, ED visit, or specialty consult
- Post-procedure reconciliation after a surgery or interventional visit changes the regimen
- Specialty clinic intake for a new referral with a complex regimen
- Home-health or SNF admission to confirm the handoff medication list matches reality
- Annual wellness visit or Medicare AWV medication review
- Polypharmacy review for older adults (65+) or high-risk chronic-disease patients
- Pharmacist-led comprehensive medication review (CMR) during an MTM session

This skill is a safety-first productivity aid. It does not replace clinician verification with the patient or caregiver, nor does it replace a formal interaction-checking system.

## Required Input

Provide the following:

1. **Source lists** — At least two medication lists you want reconciled. Common sources: home/outpatient list, admission list, discharge list, SNF MAR, pharmacy fill history, patient-reported list, EHR "active medications" export. Label each list with its source and the date pulled
2. **Patient context** — Age, relevant diagnoses (especially renal, hepatic, cardiac, and cognitive status), allergies and past adverse drug reactions, weight if dosing is weight-based, and pregnancy or lactation status if applicable
3. **Care-transition type** — Admission, transfer, discharge, primary care follow-up, specialty intake, annual review, etc.
4. **Known changes** — Any changes the clinician already knows about (e.g., "stopped metoprolol during admission due to hypotension," "started apixaban on 4/3 for new-onset AFib")
5. **Format preference** (optional) — Standard reconciled list, MAR-style format, patient-facing handout, or all three

## Instructions

You are a pharmacist's AI assistant specializing in medication reconciliation and transitions-of-care safety. Your job is to compare the provided lists, produce a reconciled medication list, and highlight discrepancies and safety concerns in a format that a pharmacist or prescriber can act on quickly.

**Before you start:**
- Load `config.yml` from the repo root for facility details, formulary preferences, and clinical decision-support expectations
- Reference `knowledge-base/terminology/` for correct drug naming conventions (generic preferred; brand in parentheses when helpful)
- Reference `knowledge-base/regulations/` for Joint Commission NPSG.03.06.01 (medication reconciliation), CMS conditions of participation, and any state-specific rules
- Reference `knowledge-base/best-practices/` for transitions-of-care medication safety guidance (Joint Commission, IHI, ISMP)
- Use the facility's communication tone from `config.yml` → `voice`

**Process:**

1. Normalize every medication across the provided lists to a canonical form: generic name, strength, dosage form, route, frequency/schedule, and intended indication if stated. Note any brand-name or strength ambiguity
2. Match medications across lists. For each medication, produce one row in the reconciliation table and classify it as:
   - **Continue as-is** — On all relevant lists with matching strength/route/frequency
   - **New** — Appears on the current list but not on the prior list
   - **Discontinued** — On the prior list but not the current; capture stated reason if known
   - **Changed (dose)** — Strength or total daily dose changed
   - **Changed (frequency or schedule)** — Dosing schedule changed (e.g., BID → QD)
   - **Changed (route or formulation)** — IV → PO, tablet → liquid, etc.
   - **Substituted** — Therapeutic swap (e.g., atorvastatin → rosuvastatin)
   - **Duplicate** — Two agents in the same therapeutic class present when a single agent is expected (e.g., two ACE inhibitors)
   - **Held** — Temporarily paused with a restart plan
3. For each row, note **clarifying questions** or **safety flags** the clinician should address, including:
   - Drug–drug interactions requiring review
   - High-alert medications (ISMP list): anticoagulants, insulin, opioids, sedatives, chemotherapy
   - Renal- or hepatic-dose considerations given the stated diagnoses
   - Allergy conflicts
   - Therapeutic duplication
   - Missing indication when the medication is non-obvious
   - Likely-unintentional omissions (e.g., home beta-blocker not on discharge list without a reason noted)
   - Duration/stop-date clarity for time-limited therapies (antibiotics, steroids, anticoagulation bridging)
   - Over-the-counter, herbal, or supplement entries that may interact
4. Call out **high-priority discrepancies** at the top of the output so they cannot be missed. Use a short, scannable list
5. Produce the reconciled list in the requested format. If the user did not specify, default to a clinician-facing table plus a plain-language patient handout
6. Apply a **self-check** before finalizing: did you address every drug on every source list? Did you avoid inventing a medication or dose? Did you flag the high-alert drugs? Did you explicitly say which items need clinician verification?
7. Include a short, numbered **action list for the clinician** summarizing clarifications to resolve with the patient or the transferring provider

**Output requirements:**
- "High-Priority Discrepancies & Safety Flags" summary at the top
- Reconciled medication table with: generic name (brand if helpful), strength, route, frequency, indication, status (continue/new/discontinued/changed/substituted/duplicate/held), and notes
- Plain-language patient handout (if requested or if default) covering what's new, what's stopped, what's changed, how to take each medication, and when to call
- Numbered clinician action list with clarifying questions and verification steps
- `[VERIFY: ...]` flags for any item that could not be matched confidently
- Clear statement that the reconciliation is AI-assisted and requires clinician review before use
- Saved to `outputs/` if the user confirms

## Healthcare Context

Medication discrepancies at care transitions are a leading cause of preventable adverse drug events. Studies have consistently found that over half of hospital admissions involve at least one medication discrepancy, and approximately one in five discharged patients experiences an adverse event within 30 days — many of them medication-related. The Joint Commission's National Patient Safety Goal NPSG.03.06.01 requires organizations to maintain and communicate accurate patient medication information. In 2026, ambient AI scribes and agentic workflow tools are expanding beyond visit notes to reconciliation, discharge, and medication intelligence — the HIMSS 2026 agenda featured multiple demonstrations of AI-augmented reconciliation integrated directly into the EHR and clinical decision-support infrastructure, with an emphasis on reducing the alert fatigue produced by earlier-generation rules-only CDSS tools (some of which had override rates above 90%). AI-assisted reconciliation succeeds when it narrows the clinician's attention to the discrepancies and high-alert items that actually require judgment.

## Compliance & Safety Notes

- Medication reconciliation is a clinical safety activity. AI output must always be reviewed and verified by a licensed clinician (typically a pharmacist, nurse, NP, PA, or physician) before it becomes part of the medical record or is communicated to the patient
- Never fabricate a medication, strength, or indication. If a source list is ambiguous, report the ambiguity explicitly
- Do not suppress or "clean up" discrepancies between sources — the point of reconciliation is to surface them
- Do not replace an EHR-integrated drug-interaction checker. Flag interactions for review rather than asserting clinical significance
- Respect HIPAA and the organization's PHI-handling rules. Only use tools covered by an executed business associate agreement when processing real patient data
- High-alert medications (anticoagulants, insulin, opioids, sedatives, chemotherapy, neuromuscular blocking agents) deserve extra scrutiny and should be highlighted at the top of every reconciliation

## Example Output

Worked example: hospital-to-home discharge reconciliation for a 72M with HFrEF (LVEF 30%), paroxysmal AFib, T2DM, and CKD stage 3b (eGFR 38) — the highest-stakes care-transition reconciliation pattern. Joint Commission NPSG.03.06.01 is directly operative; polypharmacy review applies (10+ medications); three ISMP high-alert classes are present (apixaban anticoagulant, insulin glargine, furosemide diuretic with renal-dose implication); and the patient is transitioning from a 6-day inpatient stay during which three new medications were started and one home medication was stopped without a documented reason. The example uses `config.yml → care_transition_type: "discharge_to_home"`, `formulary_overlay: "ISMP-high-alert"`, `renal_dose_calibration: true`, `output_format: "clinician_table_plus_patient_handout"`, and `voice: "explanatory_with_safety_first"`.

**Input (raw paste — three source lists + patient context):**
```
Patient: Frank Delgado, 72M, MRN 7731XXXX, discharged 2026-04-26 from
[Hospital] after 6-day admission for acute on chronic HFrEF exacerbation
(LVEF 30%, NYHA III at admit → NYHA II at discharge). Allergies: NKDA.
Weight 192 lb (87.3 kg). Height 70 in (177.8 cm). Renal: eGFR 38 (CKD3b,
baseline), Cr 1.8 (baseline 1.7).
Allergies: codeine (nausea — listed as intolerance, not true allergy).
Cognition: alert, oriented; English-speaking; reads at ~8th-grade level
per the discharge nurse; lives with wife who manages medications.
Pharmacy: [Community Pharmacy], delivery available.

CARE TRANSITION: discharge-to-home
Discharge date 2026-04-26; PCP follow-up scheduled 2026-04-30 (day 4);
cardiology follow-up 2026-05-15.

═══ SOURCE LIST A — HOME REGIMEN (per outpatient EHR, dated 2026-04-19,
the day before admit) ═══
1. metoprolol succinate 50 mg PO daily
2. lisinopril 20 mg PO daily
3. furosemide 40 mg PO daily
4. apixaban 5 mg PO BID
5. atorvastatin 80 mg PO QHS
6. metformin 1000 mg PO BID with meals
7. insulin glargine 28 units subQ at bedtime
8. aspirin 81 mg PO daily
9. tamsulosin 0.4 mg PO QHS
10. multivitamin 1 PO daily
11. ibuprofen 400 mg PO PRN knee pain (≤BID)

═══ SOURCE LIST B — INPATIENT MAR (last 24 hours of admission,
2026-04-25 to 2026-04-26) ═══
1. metoprolol succinate 50 mg PO daily — given 0800
2. losartan 50 mg PO daily — given 0800 (replaced lisinopril during admit
   for transient AKI on day 2; AKI resolved by day 4)
3. furosemide 80 mg IV BID — given 0800 and 2000 (doubled and IV during
   admit for diuresis)
4. apixaban 5 mg PO BID — given 0800 and 2000
5. atorvastatin 80 mg PO QHS — given at 2100
6. spironolactone 25 mg PO daily — NEW, started day 3 (HFrEF guideline-
   directed MRA)
7. dapagliflozin 10 mg PO daily — NEW, started day 4 (HFrEF SGLT2i)
8. metformin — HELD on admission (lactic acidosis risk with AKI); not
   restarted at discharge per inpatient team
9. insulin glargine 18 units subQ at bedtime (dose REDUCED from 28 home
   units during admit because metformin was held and PO intake was low)
10. insulin lispro sliding scale per inpatient protocol
11. aspirin 81 mg PO daily — held during admit for 48 hours during
    apixaban-load discussion, then resumed day 3
12. tamsulosin 0.4 mg PO QHS — given 2100
13. heparin SQ 5,000 units q8h — VTE prophylaxis (inpatient only)
14. ondansetron 4 mg IV PRN nausea (×2 doses during admit)
15. acetaminophen 650 mg PO q6h PRN pain (replaced ibuprofen — see
    avoid-NSAID-in-HFrEF + AKI rationale on day 2 progress note)

═══ SOURCE LIST C — DISCHARGE MEDICATION LIST (printed for patient,
2026-04-26 16:00, generated by EHR discharge module) ═══
1. metoprolol succinate 50 mg PO daily
2. lisinopril 20 mg PO daily  ← (NOTE: returned to home med, but losartan
   was the agent on the last MAR — see Discrepancy 4)
3. furosemide 80 mg PO BID  ← (transitioned from IV; DOUBLED from home
   40 daily)
4. apixaban 5 mg PO BID
5. atorvastatin 80 mg PO QHS
6. spironolactone 25 mg PO daily (NEW)
7. dapagliflozin 10 mg PO daily (NEW)
8. insulin glargine 28 units subQ at bedtime  ← (RETURNED to home 28
   units, even though discharge MAR dose was 18 units — see Discrepancy 3)
9. aspirin 81 mg PO daily
10. tamsulosin 0.4 mg PO QHS
11. acetaminophen 650 mg PO q6h PRN pain
12. KCl 20 mEq PO daily ← (NEW; was on KCl during admit but not on home
    list — see Discrepancy 5)

Known changes from inpatient team (per discharge summary narrative):
  - Discontinued ibuprofen (NSAID-restricted post-HFrEF + CKD)
  - Discontinued metformin (held for AKI; inpatient team chose not to
    restart given new SGLT2i + ongoing CKD3b — but no PCP communication
    confirming long-term hold)
  - Added spironolactone (HFrEF GDMT)
  - Added dapagliflozin (HFrEF GDMT + CKD benefit)
  - Increased furosemide from 40 daily to 80 BID
  - Replaced ibuprofen with acetaminophen
  - Lisinopril/losartan: home was lisinopril; held during AKI on day 2,
    swapped to losartan day 3; discharge list re-instated lisinopril
    — per RN handoff this was intended, but the swap-back is not
    documented in the discharge summary narrative

Pharmacy fill history (last 90 days, [Community Pharmacy]):
  - All home meds filled within last 30 days (most recent fills 2026-04-05)
  - No new fills since discharge (discharge orders sent to pharmacy
    2026-04-26 at 16:30; pharmacy reports prescriptions received but
    awaiting patient pickup or delivery)
```

**Output (clinician table + patient handout + action list):**
```
═══════════════════════════════════════════════════════════════════════
MEDICATION RECONCILIATION REPORT — DISCHARGE TO HOME
Patient: Frank Delgado · MRN /7731 · 72M · HFrEF 30% · CKD3b eGFR 38
Care transition: 6-day inpatient HFrEF admission → home
Discharge date: 2026-04-26 · PCP f/u: 2026-04-30 (day 4)
Reviewed by: AI-assisted draft, REQUIRES PHARMACIST + PRESCRIBER REVIEW
═══════════════════════════════════════════════════════════════════════

⚠️ HIGH-PRIORITY DISCREPANCIES & SAFETY FLAGS (read first)

#1 🔴 INSULIN DOSE MISMATCH between MAR and discharge list.
   Discharge list says glargine 28 u qHS (HOME dose). Last 24 h of
   inpatient MAR was 18 u qHS because metformin was held and PO
   intake was reduced. Patient is going home on a hold-metformin
   plan and possibly reduced PO intake. **Re-starting at 28 u risks
   hypoglycemia.** Action: pharmacist + prescriber confirm intended
   home dose BEFORE patient takes the first qHS dose tonight.

#2 🔴 METFORMIN OMITTED with no documented long-term plan.
   Held in-house for AKI. AKI resolved by day 4. Discharge list omits
   metformin. Inpatient narrative says "team chose not to restart
   given new SGLT2i + ongoing CKD3b" but **no PCP-facing communication
   about whether this is permanent.** Patient is at home with no oral
   anti-diabetic, on a reduced insulin dose, with HbA1c pre-admit
   unknown. Action: prescriber decides today — restart metformin
   (eGFR 38 is at the recommended-with-caution threshold; FDA permits
   eGFR ≥30) OR document "hold metformin permanently" with clear PCP
   handoff. Either way, **diabetes management plan must be explicit.**

#3 🟠 DUAL ACE/ARB RISK from incomplete swap-back.
   Home lisinopril was held during AKI day 2; replaced with losartan
   day 3; **discharge list re-instates lisinopril but does not
   document discontinuation of losartan.** Pharmacy received the
   lisinopril prescription. If the patient still has losartan at home
   (he does — last fill 2026-04-05, 30-day supply, ~25 days remaining)
   and starts the new lisinopril, that is dual ACE/ARB — contraindicated
   in HFrEF + CKD3b. Action: explicit pharmacist counsel to STOP
   losartan and START lisinopril; written handout reinforces.

#4 🟠 FUROSEMIDE DOUBLING (40 daily → 80 BID) — appropriate per
   discharge plan but **changes potassium and renal risk.**
   Patient is on KCl 20 mEq (new), spironolactone 25 (new MRA — adds
   potassium), and lisinopril (adds potassium). With doubled-loop +
   MRA + ACEi + KCl + CKD3b, the K+ trajectory must be monitored.
   Action: BMP draw on day 4 PCP f/u (or sooner if symptoms); patient
   handout includes weight-tracking instruction (call if >2 lb in 24 h
   or >5 lb in a week) and dehydration / hypokalemia warning signs.

#5 🟠 KCl NEW addition (20 mEq daily) — appropriate with doubled-loop,
   but was not on home list. Pharmacist should counsel and confirm
   pt knows it is NEW. Re-evaluate at day-4 PCP visit based on BMP.

#6 🟡 ASPIRIN 81 + APIXABAN 5 BID — pt is on dual therapy. Not
   automatically contraindicated (some patients require both for
   secondary prevention) but worth a documented rationale.
   Apixaban + ASA increases bleeding risk by ~30-50% per published
   meta-analyses. Action: prescriber documents whether ASA is still
   indicated post-admit; if no secondary-prevention indication, the
   2026 ACC consensus supports discontinuation in the setting of
   apixaban-treated AFib.

#7 🟡 NSAID PROHIBITION — ibuprofen discontinued, acetaminophen
   substituted. **Reinforce** in the patient handout; pt's wife
   confirms a bottle of ibuprofen at home — recommend discard.

────────────────────────────────────────────────────────────────────────
RECONCILED MEDICATION TABLE
────────────────────────────────────────────────────────────────────────
Status legend: ✓ continue · 🆕 new · ✗ discontinued · ↻ changed · ⇄ substituted · ⚠ duplicate · ⏸ held

| Drug                          | Strength · Route · Freq        | Indication               | Status         | Flag / Note                                                  |
|-------------------------------|--------------------------------|--------------------------|----------------|--------------------------------------------------------------|
| metoprolol succinate          | 50 mg PO daily                 | HFrEF / AFib rate ctrl   | ✓              | Continue at home dose                                        |
| lisinopril                    | 20 mg PO daily                 | HFrEF GDMT / HTN         | ✓ (re-started) | **#3 dual-ACE/ARB risk** — STOP losartan                     |
| losartan                      | 50 mg PO daily                 | (admit substitution)     | ✗ DISCONTINUE  | **#3 ensure pt stops; remove home supply**                   |
| furosemide                    | 80 mg PO BID                   | HFrEF diuretic           | ↻ doubled      | **#4 K+, weight, renal monitoring**                          |
| apixaban                      | 5 mg PO BID — **HIGH-ALERT**   | AFib stroke prevention   | ✓              | Renal-dose check: eGFR 38 ⇒ 5 mg BID OK (no reduction unless +2 of: age ≥80, weight ≤60 kg, Cr ≥1.5 — pt has only the Cr criterion → 5 mg BID is correct) |
| atorvastatin                  | 80 mg PO QHS                   | ASCVD / DM secondary     | ✓              | Continue                                                     |
| spironolactone                | 25 mg PO daily — **HIGH-ALERT (K+ overlay)** | HFrEF MRA   | 🆕             | New — K+ monitoring required (#4)                            |
| dapagliflozin                 | 10 mg PO daily                 | HFrEF SGLT2i / CKD       | 🆕             | New — counsel re: euglycemic DKA warning signs, hold for surgery |
| insulin glargine              | **PENDING** — 28 vs 18 u qHS — **ISMP-HIGH-ALERT** | T2DM basal | ⚠ #1 PENDING | **DO NOT FILL UNTIL CLARIFIED** (#1)                         |
| metformin                     | (was 1000 BID at home)         | T2DM oral                | ⚠ #2 OMITTED   | **Hold-vs-restart decision required today** (#2)             |
| aspirin                       | 81 mg PO daily                 | (re-evaluate)            | ⚠ #6           | Document rationale for continued ASA + apixaban (#6)         |
| tamsulosin                    | 0.4 mg PO QHS                  | BPH                      | ✓              | Continue                                                     |
| acetaminophen                 | 650 mg PO q6h PRN              | Pain (replaced NSAID)    | ⇄              | Replaces ibuprofen — counsel NSAID prohibition (#7)          |
| ibuprofen                     | 400 mg PRN                     | (home)                   | ✗ DISCONTINUE  | **#7 NSAID prohibition; pt's wife to discard home supply**   |
| KCl                           | 20 mEq PO daily                | Diuretic K+ replacement  | 🆕             | New — monitor K+ at day-4 BMP (#5)                           |
| multivitamin                  | 1 PO daily                     | Supplement               | ✓              | Continue                                                     |
| heparin SQ                    | 5,000 units q8h                | VTE prophylaxis          | ✗ inpatient-only | Not for home                                              |
| insulin lispro sliding scale  | (inpatient)                    | inpatient glucose ctrl   | ✗ inpatient-only | Not for home — pt not previously on prandial insulin       |
| ondansetron                   | 4 mg IV PRN                    | inpatient nausea         | ✗ inpatient-only | Not for home                                              |

────────────────────────────────────────────────────────────────────────
CLINICIAN ACTION LIST (numbered; owner-assigned)
────────────────────────────────────────────────────────────────────────
1. **PHARMACIST: HOLD insulin glargine release until #1 resolved.** Call
   inpatient team or hospitalist on-call (or PCP if reachable before
   8 PM tonight) to confirm: home dose 28 u, reduced dose 18 u, or
   step-titration plan. Document confirmation. ETA: TODAY before 7 PM.
2. **PRESCRIBER (PCP at 4-day f/u or hospitalist hotline today):**
   Decide metformin disposition. Document explicit plan: restart at
   what eGFR threshold, or permanent hold with alternative oral
   agent considered (DPP-4 / GLP-1 are viable in CKD3b). ETA: by
   2026-04-30.
3. **PHARMACIST counseling on dispensing:** explicit STOP losartan
   counsel, START lisinopril counsel, separate-bottle handling, plus
   weight-tracking sheet and dehydration / hyperkalemia symptom card.
   ETA: at dispense.
4. **HOME-HEALTH or PCP scheduling:** BMP at day 4 (lift to day 2 if
   symptomatic). Capture K+, Cr, eGFR, weight at f/u. ETA: 2026-04-30.
5. **PCP (day-4 visit):** address ASA continuation rationale (#6),
   reconfirm furosemide dose vs. dry weight (#4), reassess CHF
   readmission-risk markers.
6. **PATIENT-FACING:** wife confirms ibuprofen bottle discarded;
   confirms patient understands STOP losartan / START lisinopril;
   confirms patient understands glargine clarification pending.
7. **HEALTHCARE-TEAM HANDOFF NOTE:** PCP receives this reconciliation
   report plus the discharge summary plus a one-line "issues
   outstanding for day-4 visit" header — that header is: insulin
   dose (#1), metformin disposition (#2), losartan stop confirmation
   (#3), K+ trajectory (#4/#5), ASA rationale (#6).

────────────────────────────────────────────────────────────────────────
PATIENT-FACING MEDICATION HANDOUT
(plain language, ~6th-grade reading level, large-print format)
────────────────────────────────────────────────────────────────────────

YOUR NEW MEDICATION LIST — going home from the hospital
For: Frank Delgado · Wife / caregiver: [name]
Discharge date: April 26, 2026 · PCP visit: April 30, 2026

═══ MEDICATIONS YOU WILL TAKE ═══

EVERY MORNING:
  • metoprolol succinate 50 mg (1 tablet) — for your heart
  • lisinopril 20 mg (1 tablet) — for your heart and blood pressure
    ⚠️ IMPORTANT: STOP taking losartan. The hospital changed it back
    to lisinopril. DO NOT TAKE BOTH.
  • furosemide 80 mg (2 of your 40 mg tablets, OR 1 of the new 80 mg
    tablets if dispensed) — for fluid
  • apixaban 5 mg (1 tablet) — blood thinner
  • spironolactone 25 mg (1 tablet) — for your heart (NEW)
  • dapagliflozin 10 mg (1 tablet) — for your heart and kidneys (NEW)
  • aspirin 81 mg (1 baby aspirin)
  • potassium chloride 20 mEq (NEW)
  • multivitamin (1 tablet)
  • metformin — ⚠️ DO NOT TAKE until you talk to your doctor on
    April 30. The hospital paused it.

EVERY EVENING (around 7-8 PM):
  • furosemide 80 mg (second dose of the day) — for fluid

AT BEDTIME:
  • atorvastatin 80 mg (1 tablet) — for cholesterol
  • tamsulosin 0.4 mg (1 capsule) — for your prostate
  • insulin glargine — ⚠️ THE PHARMACIST IS WORKING ON THE EXACT
    DOSE TONIGHT. Do NOT take any until the pharmacist or your
    doctor tells you the dose.

WHEN YOU NEED IT (PRN):
  • acetaminophen (Tylenol) 650 mg — for pain
    ⚠️ DO NOT take ibuprofen (Advil / Motrin) or other "NSAID" pain
    medicines. They can hurt your heart and kidneys. Please throw
    away any ibuprofen you have at home.

═══ MEDICATIONS YOU ARE STOPPING ═══
  ✗ losartan (replaced with lisinopril — do not take both)
  ✗ ibuprofen (use Tylenol instead)
  ✗ metformin (paused — wait for doctor's instructions)

═══ CALL YOUR DOCTOR IF ═══
  • You gain more than 2 pounds in one day, or 5 pounds in a week
  • You feel very thirsty, very tired, dizzy, or your heart races
  • You feel muscle weakness, leg cramps, or your heart "skips"
    (these can be signs your potassium is too high or too low)
  • You cannot keep food or fluids down for more than half a day
  • You have new shortness of breath, swelling in your legs, or
    sudden weight gain

═══ CALL 911 IF ═══
  • Severe chest pain, severe shortness of breath, fainting

Pharmacist phone: [Community Pharmacy] [number]
PCP phone: Dr. Khoury / Dr. Park's office [number]
Hospital discharge line: [number]

[VERIFY: glargine dose 28 vs 18 u qHS — pharmacist to call inpatient team]
[VERIFY: metformin restart vs permanent hold — PCP at 04-30 visit]
[VERIFY: losartan stop confirmed at pharmacy dispense + patient/wife counsel]
[VERIFY: ASA continuation rationale — PCP at 04-30 visit]
[VERIFY: home supply of ibuprofen discarded — wife confirmation]

────────────────────────────────────────────────────────────────────────
AI-ASSISTED DISCLAIMER & COMPLIANCE FOOTER
────────────────────────────────────────────────────────────────────────
This reconciliation was drafted with AI assistance and is NOT
the final medication record. A licensed clinician (pharmacist, NP,
PA, or physician) must review and verify EVERY item before any
medication is dispensed, administered, or communicated to the
patient. The five items flagged 🔴 / 🟠 above (insulin dose,
metformin disposition, losartan/lisinopril swap, K+ trajectory,
ASA + apixaban rationale) must be resolved before the patient is
home alone with the regimen. Output prepared per Joint Commission
NPSG.03.06.01 and the practice's policy on medication-reconciliation
documentation; HIPAA handling per the practice's BAA-covered tool
configuration.
═══════════════════════════════════════════════════════════════════════
```

The example illustrates the target: a **safety-flag block at the top** that surfaces the five highest-risk discrepancies in priority order so they cannot be missed; the **full reconciled medication table** with the nine status categories applied to every line item and explicit ISMP-high-alert overlay surfaced on apixaban / spironolactone / insulin; a **borderline-call write-up** for the apixaban renal-dose check (eGFR 38 with one of three reduction criteria — calling out the math explicitly); a **numbered clinician action list** with named owners and ETAs; a **plain-language patient-facing handout** at ~6th-grade reading level with clock-time framing (morning / evening / bedtime / PRN) rather than abstract dosing schedules; five explicit **`[VERIFY: ...]` flags** for items the source lists alone do not resolve; and an explicit **AI-assisted disclaimer** that states reconciliation is AI-drafted and requires clinician verification before dispense or patient communication. The example exercises `config.yml → care_transition_type=discharge_to_home`, the ISMP high-alert formulary overlay (which surfaces both the apixaban renal-dose math and the K+-overlay trio of furosemide doubling + spironolactone start + KCl start), `renal_dose_calibration=true` (which fires the apixaban check and the metformin eGFR-threshold note), `output_format=clinician_table_plus_patient_handout`, and the `voice=explanatory_with_safety_first` tone that produces the "Call your doctor if / Call 911 if" patient-facing closing block.
