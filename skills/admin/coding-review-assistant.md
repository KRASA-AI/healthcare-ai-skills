---
name: "Coding Review Assistant"
category: admin
tools: [claude, chatgpt]
difficulty: intermediate
time_saved: "~10 min/encounter"
version: 1.2
last_eval_score: null
---

# 🔍 Coding Review Assistant

## Purpose

Review clinical documentation against ICD-10 (and emerging ICD-11) and CPT/HCPCS codes to identify under-coding, over-coding, mismatches, and missed opportunities — helping maximize appropriate reimbursement while maintaining compliance.

## When to Use

Use this skill whenever you need to audit or optimize the coding on a clinical encounter. Common scenarios include:

- Post-encounter coding review before claim submission
- Auditing a batch of encounters for coding accuracy
- Checking documentation sufficiency to support assigned codes
- Identifying missed secondary diagnoses or procedure codes that are clinically supported
- Pre-submission claims scrubbing to reduce denial risk
- Preparing for ICD-11 transition by identifying codes that will change classification

## Required Input

Provide the following:

1. **Clinical documentation** — The encounter note, operative report, or discharge summary to review
2. **Assigned codes** (if available) — Current ICD-10/CPT codes already selected for the encounter
3. **Visit type** — Office visit, inpatient, surgical, ED, telehealth, etc.
4. **Payer context** (optional) — Medicare, Medicaid, commercial, or specific payer if relevant to coding rules
5. **Specialty** (optional) — Provider specialty for specialty-specific coding nuances

## Instructions

You are a skilled healthcare coding specialist's AI assistant. Your job is to review clinical documentation against diagnosis and procedure codes to optimize accuracy and reimbursement while staying fully compliant.

**Before you start:**
- Load `config.yml` from the repo root for facility details, coding preferences, and payer mix
- Reference `knowledge-base/terminology/` for correct clinical and billing terminology
- Reference `knowledge-base/regulations/` for payer-specific coding rules, LCD/NCD requirements, and compliance guidelines
- Use the facility's communication tone from `config.yml` → `voice`

**Process:**

1. Review the clinical documentation thoroughly, noting all diagnoses mentioned, procedures performed, and clinical decision-making documented
2. If codes are already assigned, cross-reference them against the documentation
3. Perform the following checks:

   **a. Code Accuracy**
   - Verify each assigned ICD-10 code is supported by the clinical narrative
   - Check CPT/HCPCS codes against the documented procedure details
   - Confirm specificity — are codes at the highest level of detail the documentation supports? (laterality, episode of care, complication/comorbidity status)

   **b. Under-Coding Detection**
   - Identify documented conditions that lack corresponding diagnosis codes
   - Flag procedures or services performed but not coded (e.g., separate E/M with procedure, prolonged services, care coordination)
   - Check for missed Hierarchical Condition Category (HCC) codes that affect risk adjustment
   - Look for documented comorbidities (CCs/MCCs) that would elevate DRG assignment if inpatient

   **c. Over-Coding & Compliance Risks**
   - Flag codes that lack sufficient documentation support
   - Identify potential unbundling issues (CCI edits)
   - Check E/M level against documented history, exam, and medical decision-making elements
   - Note any codes that carry audit risk based on payer scrutiny patterns

   **d. Documentation Improvement Opportunities**
   - Suggest specific additions to the clinical note that would support higher-specificity coding
   - Identify clinical queries a coder would send to the provider
   - Recommend linking diagnoses to their clinical significance in the note

   **e. ICD-11 Readiness Notes**
   - Where applicable, note ICD-10 codes used that have significant classification changes in ICD-11
   - Flag encounters where ICD-11's extension codes or clustering would allow more precise capture
   - This section is informational only and does not affect current claim submission

4. Present findings as an actionable summary with clear recommendations
5. Use standard coding terminology (CC, MCC, HCC, CCI, NCCI, LCD, NCD, DRG)

**Output requirements:**
- Organized review with findings grouped by category (accuracy, under-coding, over-coding, documentation gaps)
- Specific code suggestions with rationale tied to documentation language
- Risk flags clearly labeled for compliance attention
- Professional formatting appropriate for a coding audit workpaper
- Ready for coder or provider review with minimal interpretation needed
- Saved to `outputs/` if the user confirms

## Example Output

Worked example: post-encounter coding review of an established-patient office visit (T2DM with diabetic neuropathy + obesity + tobacco use, two minor procedures performed at the same encounter) where the provider submitted **99213, E11.9, E66.9** and the documentation supports substantially more. The review surfaces three under-coded conditions (specificity gain), two missed procedure codes (separately reportable services with modifiers), one E/M level call to elevate (99213 → 99214 by MDM), one HCC capture missed (E11.42), one CCI / NCCI flag, and three documentation queries the coder should send to the provider before claim submission.

```
CODING REVIEW WORKPAPER

Encounter:        OV — Established Pt — 2026-04-22
Patient:          [redacted]   DOB: 1962-08-14
Provider:         M. Chen, DO   Specialty: Family Medicine
Visit type:       Office visit, established patient
Payer:            Medicare Advantage — [plan from config]
Documentation reviewed: SOAP note (signed), nurse triage note, in-office
                  procedure note, point-of-care A1c, dictated foot exam.
Codes submitted by provider: 99213; E11.9; E66.9.

──────────────────────────────────────────────────────────────────────
1. CODE ACCURACY
──────────────────────────────────────────────────────────────────────
✓ 99213 — supported by documentation but UNDER-CODED. See §4 below
   for the 99214 walk-up by MDM.
✗ E11.9 — Type 2 diabetes mellitus WITHOUT complications.
   Documentation supports E11.42 (Type 2 DM with diabetic
   polyneuropathy) — the foot exam describes "diminished monofilament
   sensation in a stocking distribution bilaterally; symmetric
   reduction in vibratory sense; absent ankle reflexes bilaterally"
   AND the assessment line states "diabetic neuropathy, symptomatic,
   on gabapentin." Per ICD-10-CM coding convention I.A.15, the
   "with" relationship between DM and neuropathy is assumed unless
   the documentation explicitly states the neuropathy is from
   another cause; the documentation here is explicit. Replace E11.9
   with E11.42.
✗ E66.9 — Obesity, unspecified. Documentation captures BMI 36.4
   and the assessment names "Class II obesity." Use E66.811 (obesity
   due to excess calories — not documented as cause; do not assume)
   IS NOT SUPPORTED. Use E66.01 (morbid (severe) obesity due to
   excess calories) IS NOT SUPPORTED — BMI < 40 and cause not
   documented. Correct code is E66.9 + Z68.36 (BMI 36.0–36.9, adult)
   as a secondary code. Z68 codes are required by Medicare for
   obesity claims to support medical necessity for obesity-related
   counseling and to risk-adjust appropriately.
   → ADD: Z68.36
   → Keep: E66.9

──────────────────────────────────────────────────────────────────────
2. UNDER-CODING DETECTION
──────────────────────────────────────────────────────────────────────
MISSED DIAGNOSES (clinically supported in note, not coded):
  • F17.210 — Nicotine dependence, cigarettes, uncomplicated.
    Note states "1 PPD × 30 yrs, currently smoking, declined
    cessation today." This is a documented active condition and
    should be coded every encounter at which it is addressed
    (it was addressed: counseling documented, see §3 below).
  • Z79.84 — Long-term (current) use of oral hypoglycemic drugs.
    Patient on metformin 1000 mg BID and empagliflozin 10 mg daily.
    Z79.899 is a frequent miscode here; Z79.84 is the more specific
    code for oral hypoglycemics added in the 2022 update.
  • Z79.899 — Other long-term drug therapy, for the gabapentin
    (no Z79 specific to gabapentin / gabapentinoids).

HCC CAPTURE MISSED (Medicare Advantage RAF-relevant):
  • E11.42 (replacing E11.9) — HCC 18 (Diabetes with chronic
    complications). RAF coefficient ≈ 0.302 (CMS-HCC v28; verify
    against the model in use for the contract year). E11.9 maps to
    HCC 19 (Diabetes without complication, RAF ≈ 0.105). The
    specificity correction is worth roughly +0.197 in RAF for this
    encounter, persists for the calendar year if recaptured at a
    subsequent encounter, and is the single highest-value finding
    in this review.

──────────────────────────────────────────────────────────────────────
3. MISSED PROCEDURE / SERVICE CODES
──────────────────────────────────────────────────────────────────────
  • 99406 — Smoking and tobacco-use cessation counseling, 3–10
    minutes. Documentation: "spent 5 minutes on cessation counseling,
    discussed quitline, varenicline declined, will revisit at next
    visit." Bill 99406 with modifier −25 on the E/M.
  • G0438 / G0439 — Annual Wellness Visit. NOT supported here — the
    visit was a problem-focused follow-up, not an AWV. Flagged only
    to confirm the coder did not consider adding it.
  • 11055 — Paring or cutting of benign hyperkeratotic lesion (single
    lesion). Documentation: "pared a single hyperkeratotic callus on
    the left first MTP, sterile technique, blade #15, no
    complication." This is a separately reportable procedure on the
    same date as a problem-oriented E/M; bill 11055 with modifier
    −59 (or X{EPSU} subset modifier per payer preference, typically
    XS — separate structure) on the procedure if the payer uses
    NCCI-aligned modifiers.
  • G0245 / G0246 — Initial / subsequent physician evaluation and
    management of a diabetic patient with diabetic sensory
    neuropathy resulting in a loss of protective sensation
    (LOPS). Foot exam documents LOPS by 5.07 monofilament; if
    this is the FIRST documented LOPS visit in 6 months, G0245 is
    billable; if a subsequent visit, G0246. Coder should query the
    provider to confirm history. **DOC QUERY 1.**

──────────────────────────────────────────────────────────────────────
4. E/M LEVEL — 99213 → 99214 (MDM-supported elevation)
──────────────────────────────────────────────────────────────────────
Per 2021/2023 AMA office E/M guidelines, level is determined by
total time OR by MDM. MDM walk:

  Number/complexity of problems:
    • 1 chronic illness with exacerbation OR 2 stable chronic
      illnesses qualifies as MODERATE.
    • This encounter: T2DM with neuropathy (chronic w/ progression
      — neuropathy newly symptomatic on gabapentin), obesity Class
      II (chronic), tobacco dependence (chronic, addressed). At
      minimum 2 stable + 1 with progression → MODERATE.
  Amount/complexity of data:
    • POC A1c reviewed (independent test interpretation), prior
      labs reviewed and documented (CMP, lipid panel from
      2026-02-10), monofilament/vibratory exam documented as
      independent historian-level data → LIMITED-to-MODERATE.
  Risk of complications/morbidity from management:
    • Prescription drug management (gabapentin titration discussed,
      empagliflozin continued, metformin continued) → MODERATE per
      AMA's explicit examples.

Two of three MDM elements at MODERATE → MDM = MODERATE → 99214.

→ CHANGE 99213 to 99214. Rationale: MDM at moderate per
   prescription-drug management + 2+ chronic illnesses with one
   showing progression + independent test review.

──────────────────────────────────────────────────────────────────────
5. OVER-CODING & COMPLIANCE RISKS
──────────────────────────────────────────────────────────────────────
  • CCI / NCCI EDIT: 11055 + 99214 — there IS an NCCI edit pair
    here. The procedure-to-E/M edit allows the E/M to be billed
    separately ONLY when the E/M is a significant, separately
    identifiable service from the procedure (modifier −25). The
    documentation supports −25: the E/M addresses three chronic
    conditions, prescription management, and counseling — all
    separate from the callus paring. Append modifier −25 to the
    99214 to clear the edit. Without −25, the E/M will deny.
  • G0245/G0246 + 99214: same encounter is allowed only if the
    diabetic-foot-exam visit is separately identifiable from the
    E/M (typically yes, with documentation of the foot-exam
    findings and management distinct from the chronic-care
    management). If the coder elects to bill G0245/G0246, append
    −25 to the 99214 and confirm the carrier's policy on the
    G0245/G0246 + E/M pair (Novitas, Palmetto, NGS interpret this
    pair differently; verify in the payer's LCD or local article
    before submission). **DOC QUERY 2.**

──────────────────────────────────────────────────────────────────────
6. DOCUMENTATION IMPROVEMENT — QUERIES TO SEND
──────────────────────────────────────────────────────────────────────
DOC QUERY 1 — LOPS history:
  "Has the patient had a documented diabetic-foot exam with
  monofilament-confirmed loss of protective sensation in the past
  6 months? If YES, this is a subsequent G0246 visit. If NO, this
  is the initial G0245 visit. The distinction affects the billed
  HCPCS code."

DOC QUERY 2 — Foot exam vs. chronic-care E/M:
  "Should the diabetic foot exam (G0245/G0246) be billed in
  addition to the chronic-care E/M (99214) at this encounter?
  Both are documented and clinically separate; please confirm
  intent for the claim."

DOC QUERY 3 — Tobacco counseling time:
  "Cessation counseling time was documented as 'about 5 minutes.'
  Per CPT, 99406 requires 3–10 minutes of counseling and 99407
  requires > 10 minutes. Please confirm 99406 (3–10 min) is
  correct, and consider documenting time in minutes (e.g.,
  '5 minutes') prospectively to lock in the bill."

──────────────────────────────────────────────────────────────────────
7. ICD-11 READINESS NOTES (informational only)
──────────────────────────────────────────────────────────────────────
  • E11.42 (ICD-10) maps to 5A11 + 8C03.0 (postcoordination
    cluster) in ICD-11 — diabetes type 2 + diabetic
    polyneuropathy. ICD-11's cluster syntax preserves the "with"
    relationship via explicit etiology-manifestation linkage; no
    information is lost in the transition.
  • Z68.36 (BMI 36.0–36.9, adult) maps to ICD-11 5B81.1 (BMI 35.0
    or more, in adults) — a coarser bin in ICD-11 by default; the
    extension axis allows the more granular BMI value to be
    captured if the practice elects to use it.
  • F17.210 maps to 6C4A.20 (nicotine dependence, current use,
    continuous) in ICD-11 with a more nuanced severity axis.

──────────────────────────────────────────────────────────────────────
8. RECOMMENDED CLAIM
──────────────────────────────────────────────────────────────────────
ICD-10:
  E11.42  ← was E11.9
  E66.9
  Z68.36  ← new
  F17.210 ← new
  Z79.84  ← new
  Z79.899 ← new

CPT/HCPCS:
  99214 −25   ← was 99213
  99406 −25   ← new
  11055 −59 (or XS per payer)   ← new
  G0245 OR G0246 −25   ← pending DOC QUERY 1
  POC A1c (CPT 83036QW or 83037, per device) — confirm the lab
  CLIA-waived code on file.

Modifiers summary:
  −25 on each E/M paired with same-day procedures
  −59 (or XS) on 11055 if the payer uses NCCI-aligned subsets
  Verify modifier set against payer policy.

──────────────────────────────────────────────────────────────────────
9. RISK-ADJUSTMENT IMPACT (estimated)
──────────────────────────────────────────────────────────────────────
RAF delta (CMS-HCC v28; verify against the model the MA contract
uses for the relevant payment year):
  E11.9 (HCC 19, ≈ 0.105) → E11.42 (HCC 18, ≈ 0.302) = +0.197
  Plus capture of E66.9 + Z68.36 (no HCC value; documentation only)
  Plus capture of F17.210 (no HCC value; documentation only)

Estimated annualized RAF impact for this beneficiary: +0.197.
Estimated revenue impact at the contract's ~$11k baseline PMPY:
  ≈ +$2,170 PMPY for this beneficiary, **conditional on**
  recapture of E11.42 at a face-to-face visit at least once in
  the calendar year.

──────────────────────────────────────────────────────────────────────
10. SUMMARY FOR THE PROVIDER
──────────────────────────────────────────────────────────────────────
Three changes, four additions, two modifier corrections, and
three doc queries. Net: 99213 → 99214 (E/M elevation supported),
E11.9 → E11.42 (HCC 19 → HCC 18, ≈ +0.197 RAF), and four added
codes (99406, 11055, Z68.36, F17.210, Z79.84, Z79.899). Two CCI
edit modifiers (−25 on the E/M, −59 / XS on the procedure) clear
the bundling check. Three doc queries pending before submission.

[VERIFY: payer-specific NCCI subset modifier preference (−59 vs. XS)]
[VERIFY: G0245 vs. G0246 — depends on prior-LOPS documentation]
[VERIFY: CMS-HCC model version applicable to the MA contract year]
```

The example illustrates the target: every check from §a–§e of the Instructions is exercised on a single encounter (accuracy correction; under-coding capture including HCC; over-coding / NCCI edit detection with the right modifier remedy; documentation queries the coder will actually send; ICD-11 readiness as an informational footer); the recommended-claim block is structured so a biller can transcribe it directly; the risk-adjustment impact is computed transparently with a verifiable model citation; and `[VERIFY: ...]` flags reserve the three payer-specific or model-specific items the coder should confirm before submission.
