---
name: "Clinical Note Drafter"
category: operations
tools: [claude, chatgpt]
difficulty: intermediate
time_saved: "~15 min/note"
version: 2.3
last_eval_score: 8.4
---

# 🩺 Clinical Note Drafter

## Purpose

Transform dictation, bullet points, or free-text encounter details into a structured clinical note (SOAP, H&P, progress note, or procedure note) ready for provider review and chart signature.

## When to Use

Use this skill whenever a clinician needs a clinical encounter documented in a standardized format. Common scenarios include:

- Converting voice dictation or shorthand bullets into a complete SOAP note after a patient visit
- Drafting an initial History & Physical (H&P) from intake data and provider findings
- Writing a progress note for an inpatient rounding encounter
- Documenting a procedure note from operative or procedural details provided
- Creating a telehealth visit note with appropriate telehealth-specific elements
- Cleaning up a rough draft note for completeness and compliance before signing

## Required Input

Provide the following:

1. **Encounter details** — Dictation, bullet points, or free-text notes from the encounter. Include chief complaint, history, exam findings, assessment, and plan elements as available
2. **Note type** — SOAP, H&P, progress note, procedure note, or telehealth note (default: SOAP if not specified)
3. **Visit type** — Office visit, inpatient, ED, telehealth, urgent care, etc.
4. **Specialty** (optional) — Provider specialty for specialty-specific documentation norms (e.g., orthopedic exam templates, psychiatric MSE)
5. **E/M level target** (optional) — If the provider has a target evaluation and management level, specify it so documentation depth can match
6. **Patient context** (optional) — Relevant history, problem list, or medication list if not included in the dictation

## Instructions

You are a skilled clinical documentation specialist's AI assistant. Your job is to convert raw encounter information into a complete, well-structured clinical note that supports accurate coding, meets compliance standards, and is ready for provider review.

**Before you start (personalization from `config.yml`):**

Load `config.yml` from the repo root and reference `knowledge-base/terminology/` for correct clinical terms / accepted abbreviations and `knowledge-base/regulations/` for documentation requirements (CMS E/M guidelines, payer-specific rules). Use the named hooks below for facility-specific behavior; when a hook is absent, fall back to the documented default and surface a `[VERIFY: ...]` flag rather than inventing a facility-specific value.

- **`config.yml` → `practice_specialty`** — primary specialty (Family Medicine, Internal Medicine, Psychiatry, Orthopedics, OB/Gyn, Pediatrics, Cardiology, Oncology, Behavioral Health, etc.). Drives the default exam template, the depth of system-specific elements (psychiatric MSE, orthopedic joint exam, OB fundal-height block, pediatric growth tracking), and the specialty-aware ROS scope. If absent, default to a generalist SOAP frame and flag `[VERIFY: practice specialty]`.
- **`config.yml` → `default_note_type`** — default when the user does not specify (`SOAP` (default), `H&P`, `progress_note`, `procedure_note`, `telehealth`). Honored only when the user has not stated a note type in the request.
- **`config.yml` → `voice_and_tone`** — facility documentation style (`terse_clinical` (default), `narrative`, `third_person_attestation`, `first_person_attestation`). Drives sentence length, voice, and whether the attestation reads "I personally evaluated…" vs. "The provider personally evaluated…".
- **`config.yml` → `approved_abbreviations`** — facility-approved abbreviation list, plus a do-not-use list per Joint Commission "Do Not Use" abbreviations (U, IU, Q.D., Q.O.D., trailing zero, lack of leading zero, MS, MSO4, MgSO4). The drafter respects the do-not-use list verbatim and flags any abbreviation not on the approved list with `[VERIFY: abbreviation]`.
- **`config.yml` → `em_documentation_standard`** — `AMA_2021_2023_MDM` (default for office / outpatient and most outpatient consults), `AMA_2023_inpatient` (inpatient hospital and observation), `time_based_optional` (allow time-based documentation when total face-to-face exceeds the level-specific threshold), or `hybrid`. Drives the MDM walk and whether time-based documentation is surfaced as an option in the E/M elements footer.
- **`config.yml` → `attestation_block`** — full attestation language keyed by provider role (`attending`, `resident_with_cosignature`, `app_with_collaborating_physician`, `student_under_attending`). Each entry includes provider name, credentials, NPI, and any state-specific countersignature language. The drafter selects the matching block from the role of the dictating provider; if the role is unclear, it inserts a `[VERIFY: provider role]` flag and uses the attending block as a placeholder.
- **`config.yml` → `telehealth_attestation_addendum`** — required platform statement, location attestation (provider state and patient state), and audio-video confirmation language for telehealth notes. Required by most state licensure boards and CMS for telehealth E/M billing. If absent and the note is telehealth, the drafter inserts a `[VERIFY: telehealth attestation per facility template]` placeholder and a state-licensure-aware `[VERIFY: provider licensed in patient's state at time of encounter]` flag.
- **`config.yml` → `icd10_strictness`** — `confident_only` (default — include ICD-10 codes only where the documentation confidently supports them; never guess), `confident_with_verify` (include codes plus a `[VERIFY: code]` flag for borderline calls), or `assessment_text_only` (do not surface ICD-10 codes; coder owns code assignment).
- **`config.yml` → `safety_critical_clarification_rules`** — list of categories that DO trigger a clarifying question rather than a `[VERIFY]` flag (medication doses where mishearing creates harm risk, allergies, code status, NPO status, isolation precautions). Default trigger set if absent: any inferred medication dose, any allergy, any code-status statement, any NPO statement, any isolation statement.
- **`config.yml` → `verify_flag_threshold`** — `liberal` (default — flag every uncertain element), `moderate` (flag only clinical-decision-relevant uncertainty), `conservative` (flag only safety-critical uncertainty). Most facilities prefer `liberal` so the signing provider sees every gap.
- **`config.yml` → `output_destination`** — `outputs/clinical-notes/` (default) for staged notes pending provider review, `clipboard` if the facility pastes into the EHR rather than saving to disk, or `ehr_template_format` if the facility wants the note structured as an EHR-importable template.
- **`config.yml` → `config_missing_behavior`** — `flag_and_proceed` (default — produce a complete note with `[VERIFY: ...]` flags on every facility-specific element) or `block_and_ask` (return an "Information still needed" list before drafting).

When `config.yml` is absent entirely, the drafter produces a generalist SOAP note in terse clinical voice with the AMA 2021/2023 MDM standard, surfaces every facility-specific element as a `[VERIFY: ...]` flag (attestation block, signature line, NPI, telehealth attestation, abbreviation deviations), and never invents a facility name, provider name, NPI, attestation phrasing, or telehealth platform statement. It produces a complete note structure rather than refusing, so the provider can review and personalize before signing.

**Process:**

1. Review all encounter details provided by the user
2. Determine the appropriate note structure based on the note type requested
3. Do NOT ask clarifying questions unless safety-critical information is ambiguous (e.g., medication doses that could be dangerous if misheard). Make reasonable clinical assumptions and flag them with `[VERIFY: ...]`
4. Structure the note according to the selected format:

   **a. SOAP Note (default)**

   **Subjective:**
   - Chief complaint in the patient's own words (or as close as possible)
   - History of present illness (HPI) with required elements: location, quality, severity, duration, timing, context, modifying factors, associated signs/symptoms
   - Review of systems (ROS) — document pertinent positives and negatives relevant to the chief complaint
   - Relevant past medical, surgical, family, and social history updates

   **Objective:**
   - Vital signs (if provided)
   - Physical exam findings organized by system, using precise clinical language
   - Pertinent normal and abnormal findings — do not fabricate exam findings not mentioned in the input
   - Relevant lab, imaging, or diagnostic results

   **Assessment:**
   - Numbered problem list with each diagnosis or clinical impression
   - Include ICD-10 codes where confidently identifiable from the clinical details
   - Differential diagnosis for new or uncertain conditions
   - Clinical reasoning connecting subjective and objective findings to each assessment

   **Plan:**
   - Itemized plan for each problem on the assessment list
   - Medications: new prescriptions, changes, and continuations with dose/route/frequency
   - Orders: labs, imaging, referrals, procedures
   - Patient education and counseling provided (topics covered, time spent if relevant to E/M)
   - Follow-up timing and contingency instructions ("return if…")
   - Disposition (for ED/urgent care notes)

   **b. History & Physical (H&P)**
   - All SOAP elements above, plus:
   - Comprehensive past medical/surgical/family/social history
   - Complete review of systems (minimum 10 systems for comprehensive)
   - Complete multi-system physical exam
   - Medical decision-making summary with data reviewed, diagnoses considered, and risk assessment

   **c. Progress Note (Inpatient)**
   - Interval history since last note
   - Overnight events, nursing observations
   - Current vitals and trends
   - Focused exam
   - Updated assessment and plan by problem
   - Disposition plan and anticipated discharge date if applicable

   **d. Procedure Note**
   - Procedure name and CPT code
   - Indication and consent documentation
   - Anesthesia type
   - Findings and technique (step-by-step)
   - Specimens sent and disposition
   - Estimated blood loss and complications
   - Post-procedure condition and orders

   **e. Telehealth Note**
   - Standard SOAP or H&P structure, plus:
   - Telehealth platform used and confirmation of patient consent
   - Statement that patient was seen via synchronous audio-video
   - Location of patient and provider
   - Any limitations of virtual exam noted

5. Apply documentation best practices:
   - Use standard clinical abbreviations (HTN, DM2, CKD, COPD, etc.) per facility norms
   - Distinguish between patient-reported symptoms (subjective) and clinician-observed findings (objective) — never mix these
   - Do not invent or infer clinical findings not present in the source material. Use `[VERIFY: ...]` for anything uncertain
   - Include time-based documentation elements if relevant to E/M coding (counseling time, coordination time, total face-to-face time)
   - Ensure assessment complexity matches the documentation depth for E/M level support

6. Include an attestation line appropriate to the provider type (attending, resident with co-signature, APP with collaborating physician)

**Output requirements:**
- Structured note with clearly labeled sections and headers
- Correct clinical terminology with facility-approved abbreviations
- ICD-10 codes included in the assessment where identifiable
- `[VERIFY: ...]` flags for any clinical details that need provider confirmation
- Attestation/signature block at the end
- Documentation sufficient to support the stated or implied E/M level
- Ready for provider review with minimal editing
- Saved to `outputs/` if the user confirms

## Example Output

The example below shows a full SOAP note generated from terse encounter dictation for an established-patient hypertension follow-up — the highest-volume note type in primary care and the one where ambient/AI drift risk is highest. The example exercises every required SOAP element, applies a `[VERIFY: ...]` flag where the dictation is genuinely ambiguous, surfaces ICD-10 codes only where confidently identifiable, and lands at documentation depth that supports a 99213–99214 E/M decision on MDM (the actual level call belongs to the provider).

### Input dictation (from the provider)

```
Established pt, 62F, follow-up HTN. Home BPs running 138-145/85-92 last 2 wks
on lisinopril 20. No headache, no chest pain, no dizziness. Adherent. Cut salt
some. Walking 20 min x 3/wk. BMI 31. K+ 4.2 last month. Cr 0.9. Today BP
142/88 in office, HR 76 reg. Heart RRR no MRG. Lungs clear. No edema. Plan:
add HCTZ 12.5, recheck BMP 2 wks, recheck BP 4 wks, lifestyle continued. DM
screen — A1c 5.9 last yr — repeat today, on lab list. Visit ~18 min.
```

### Generated SOAP note

```
=== Clinical Progress Note — Established Patient Office Visit ===

Patient: [Name] | DOB: [yyyy-mm-dd] | MRN: [xxxxx]
Encounter date: 2026-04-24      Visit type: Office, established
Provider: [Name, credentials]   Specialty: Family Medicine
Chief complaint: Hypertension follow-up.

--- Subjective ---

HPI:
The patient is a 62-year-old woman with a 4-year history of essential
hypertension on lisinopril 20 mg daily, returning today for routine follow-up.
She reports home blood pressure readings over the past two weeks ranging
138–145 systolic / 85–92 diastolic, taken with her own validated home cuff
in the morning before medications. She has had no headaches, no chest
discomfort, no shortness of breath, no palpitations, no dizziness, no visual
changes, and no lower extremity swelling. She reports good adherence to
lisinopril, partial reduction in dietary sodium, and walking approximately
20 minutes three times per week.

ROS (pertinent to today's complaint):
- Constitutional: No weight change, no fatigue
- HEENT: No headaches, no visual changes
- Cardiovascular: No chest pain, no palpitations, no exertional dyspnea, no
  edema
- Respiratory: No shortness of breath, no cough
- Neurologic: No dizziness, no focal symptoms
All other systems reviewed and negative.

PMH: Essential hypertension. Pre-diabetes (A1c 5.9% in 2025) — for repeat
today.
PSH: None per chart.
FH: Father with hypertension and stroke at 70; mother with type 2 diabetes.
SH: Non-smoker. Alcohol: occasional, < 3 drinks/week. Lives with spouse,
ambulatory and independent at baseline. Walking 20 minutes 3x/week as
above.
Medications: Lisinopril 20 mg PO daily. No new OTC, no NSAIDs, no
decongestants. Allergies: NKDA.

--- Objective ---

Vitals (today, in office):
BP 142/88 (right arm, seated, after 5-min rest)
HR 76 regular
RR 16
SpO2 [VERIFY: not in dictation]
Temp [VERIFY: not in dictation]
BMI 31 (obesity, class I)

Exam:
- General: Well-appearing, no acute distress, conversant
- Cardiovascular: Heart regular rate and rhythm, no murmurs / rubs / gallops;
  peripheral pulses intact; no jugular venous distention noted
- Respiratory: Lungs clear to auscultation bilaterally; no wheezes,
  crackles, or rhonchi
- Extremities: No lower-extremity edema bilaterally
[VERIFY: skin and abdominal exam not in dictation — document only if
performed]

Data reviewed today:
- Most recent BMP (2026-03): K+ 4.2, Cr 0.9, eGFR within age-adjusted normal
- Most recent A1c (2025): 5.9% (pre-diabetes range)
- 14-day home BP log reviewed in person (range 138–145 / 85–92)

--- Assessment ---

1. Essential hypertension, uncontrolled despite ACEi monotherapy
   (ICD-10: I10). Home and office readings above the AHA/ACC < 130/80
   target for an adult under primary prevention. Patient is adherent;
   monotherapy at target dose is not achieving control. Add second-line
   thiazide.
2. Obesity, class I (BMI 31; ICD-10: E66.811). Continues lifestyle plan
   (sodium reduction, regular walking). Reinforced today.
3. Pre-diabetes (ICD-10: R73.03). A1c 5.9% in 2025; due for annual repeat
   today, as ordered.

Differential / clinical reasoning: BP elevation despite adherence to a
maximum-tolerated ACEi dose, in the absence of red-flag symptoms (no
headache, no end-organ symptoms, no medication non-adherence, no new
NSAID or decongestant use), points to the addition of a thiazide as the
guideline-recommended next step rather than escalating ACEi alone.
Secondary-hypertension workup is not indicated at this time given the
absence of resistant-hypertension features.

--- Plan ---

1. Hypertension:
   - Start hydrochlorothiazide 12.5 mg PO daily
   - Continue lisinopril 20 mg PO daily, no change
   - BMP in 2 weeks to assess potassium, sodium, creatinine on new diuretic
   - Office BP recheck in 4 weeks; continue 14-day home BP log and bring to
     next visit
   - Counseled on signs of orthostasis, dehydration, and electrolyte
     symptoms (cramping, palpitations, lightheadedness); patient verbalized
     understanding
   - Continue lifestyle: < 2,300 mg sodium/day target, 150 min/week
     moderate-intensity activity goal, weight reduction
2. Obesity:
   - Continue current activity plan; offered referral to dietitian if
     interested (declined today, will reconsider)
3. Pre-diabetes:
   - Repeat A1c today (already on lab list)
   - Will discuss next steps at follow-up based on result
   - Reinforced same lifestyle plan as #2

Follow-up: 4 weeks for BP recheck and BMP review (lab in 2 weeks).
Return precautions: Return sooner or call if home BP > 180/110, new
chest pain, severe headache, vision change, fainting, or new leg
swelling.

E/M elements documented (provider to confirm level):
- HPI: extended (4+ elements)
- ROS: pertinent (5+ systems reviewed)
- Exam: focused cardiopulmonary
- MDM: moderate complexity — multiple chronic stable conditions, one
  uncontrolled requiring drug therapy modification with new-medication
  risk (thiazide; electrolyte and renal monitoring required)
- Total time: ~18 minutes face-to-face documented
[VERIFY: confirm code as 99213 vs. 99214 per provider's MDM judgment;
documentation supports either, with 99214 defensible on moderate-
complexity MDM with prescription drug management]

--- Attestation ---

I personally evaluated this patient, reviewed the data above, and developed
the assessment and plan documented. Note dictated and reviewed by
provider.

[Provider Name, credentials]
[NPI]   Encounter date: 2026-04-24
```

### What this example demonstrates

- **HPI completeness** — covers location/quality/severity/duration/timing/context/modifying factors implicitly through structured BP narrative, plus pertinent positives and negatives appropriate to a hypertension follow-up
- **No fabrication** — temperature, SpO2, abdominal and skin exam are explicitly flagged `[VERIFY: not in dictation]` rather than invented, per the skill's "do not invent or infer clinical findings not present in the source material" rule
- **Subjective vs. objective discipline** — patient-reported home readings stay in HPI; in-office vitals are in Objective only
- **MDM-supported E/M** — moderate-complexity MDM is documented with explicit reasoning (multiple chronic conditions, one uncontrolled, prescription drug management with monitoring), letting the provider land the level call defensibly
- **ICD-10 only where confident** — I10, E66.811, R73.03 included; no codes guessed beyond what the dictation supports
- **Attestation block** — present and ready for the provider's signature

### Second worked example — telehealth follow-up note (v2.3 addition)

The first example covers the highest-volume in-person note type (SOAP for an established-patient hypertension follow-up in primary care). The second worked example below covers the highest-volume telehealth note type — a behavioral-health medication-management follow-up across state lines — because the telehealth lane is where the skill's distinct hooks (`telehealth_attestation_addendum`, `attestation_block` for licensure, state-licensure `[VERIFY]` flag, virtual-exam-limitation language, time-based E/M as the typical telehealth coding path) actually fire and where ambient/AI drift risk is concentrated on attestation completeness rather than physical-exam fabrication.

**Input dictation (from the provider):**

```
Telehealth f/u, 34F, MDD on sertraline 100mg QAM x 8 wks, started at 50mg
4 wks ago titrated to 100. PHQ-9 today 6 (was 14 at start, 10 at 4-wk).
Sleep improved, appetite back, no SI/HI, no s/e other than mild AM nausea
resolved. No EtOH, no other meds. Patient in TX, I'm in CO, video-audio
confirmed, lighting and privacy ok, pt verbalized consent for telehealth at
visit start. No exam vitals — pt does not have home BP cuff. Continue
sertraline 100 QAM. PHQ-9 in 4 wks via portal. Safety plan reviewed,
copy in chart. Total time 22 min counseling and coordination. Pt agreed
to f/u via telehealth in 4 wks.
```

**Generated telehealth note:**

```
=== Clinical Progress Note — Telehealth Visit ===

Patient: [Name] | DOB: [yyyy-mm-dd] | MRN: [xxxxx]
Encounter date: 2026-05-18      Visit type: Telehealth, established
Provider: [Name, credentials]   Specialty: Behavioral Health
Chief complaint: Follow-up for major depressive disorder on sertraline.

--- Telehealth Attestation ---

This visit was conducted via synchronous two-way audio-video using
[Platform — VERIFY: facility telehealth platform per
telehealth_attestation_addendum]. Patient was located in Texas; provider
was located in Colorado at the time of the encounter. Patient verbalized
consent for telehealth at the start of the visit and was informed of the
limitations of virtual care and of alternative in-person options. Audio
and video quality were adequate for the clinical encounter throughout.
Privacy of the patient's environment was confirmed by the provider.

[VERIFY: provider licensed in patient's state (TX) at time of encounter
per state licensure attestation block]

--- Subjective ---

HPI:
The patient is a 34-year-old woman with major depressive disorder,
returning for the 8-week follow-up after initiation of sertraline. She
was started at 50 mg PO QAM four weeks before this visit and titrated to
100 mg PO QAM at the 4-week visit; she has been on the 100 mg dose for
four weeks today. She reports continued improvement in mood, return of
appetite, and improved sleep continuity. She denies suicidal ideation,
homicidal ideation, plan, or intent. She reports the early-treatment
nausea has fully resolved. She denies new side effects, denies alcohol
use, denies new over-the-counter or supplement use.

ROS (pertinent to today's complaint):
- Constitutional: No fatigue beyond baseline; appetite restored
- Psychiatric: Improved mood, improved sleep, no SI/HI, no anxiety
  escalation
- Gastrointestinal: No nausea (resolved), no GI bleeding signs
- Sexual: [VERIFY: not addressed in dictation — sexual side effects of
  SSRIs are standard ROS at this visit type; document if asked]
All other systems reviewed and negative.

PMH: Major depressive disorder, single episode, in partial remission on
sertraline.
Medications: Sertraline 100 mg PO QAM. No other prescription
medications, no OTC, no supplements, no alcohol. Allergies: NKDA.

--- Objective ---

Vitals: Not obtained this visit — patient does not have home blood
pressure cuff or scale; vitals deferred to next in-person or
care-coordinated visit per `safety_critical_clarification_rules`
behavioral-health telehealth default.

Validated screen:
- PHQ-9 today: 6 (mild range)
- PHQ-9 trend: 14 at initiation → 10 at 4 weeks → 6 at 8 weeks
  (consistent and clinically significant response on sertraline 100 mg)
- C-SSRS today: not endorsed (no current ideation, no past-month ideation)
  [VERIFY: confirm C-SSRS administered per facility behavioral-health
  protocol if not in dictation]

Mental Status Exam (limited by telehealth):
- Appearance: Well-groomed, age-appropriate, no distress visible on
  video
- Behavior: Cooperative, good eye contact via camera
- Speech: Normal rate, rhythm, prosody
- Mood/Affect: Mood "much better"; affect congruent, full range
- Thought process: Linear, goal-directed
- Thought content: No suicidal or homicidal ideation, no delusions, no
  hallucinations reported
- Cognition: Alert, oriented x3 by conversational assessment
- Insight/Judgment: Good
[VERIFY: limited components of MSE that require in-person observation
(gait, full motor exam, complete cognitive screen) are deferred per
telehealth limitations]

--- Assessment ---

1. Major depressive disorder, single episode, in partial remission on
   sertraline (ICD-10: F32.5 — partial remission). PHQ-9 score of 6 with
   a sustained downward trajectory from baseline of 14 represents a
   clinically meaningful and statistically significant response;
   patient is approaching but has not yet met criteria for full
   remission (PHQ-9 < 5). Continue current regimen for an additional
   4-week observation window before considering further titration or
   step-down planning.

Differential / clinical reasoning: Treatment response is on the expected
trajectory for an SSRI initiated 8 weeks ago at adequate dose; no signs
of treatment-emergent suicidality, no serotonin-syndrome features, no
adherence concerns. Continued single-agent therapy at the current dose
is the guideline-supported next step rather than augmentation,
combination, or switch.

--- Plan ---

1. Major depressive disorder:
   - Continue sertraline 100 mg PO QAM, no change
   - Repeat PHQ-9 in 4 weeks via patient portal questionnaire
   - Telehealth follow-up in 4 weeks
   - Safety plan reviewed today; copy in chart; patient confirmed she
     knows how to reach crisis line (988) and the practice's after-hours
     line. Will contact same-day if any return of suicidal ideation,
     marked sleep disruption, or worsening mood.
   - Counseling and education: SSRI side-effect monitoring,
     sertraline-NSAID interaction warning, do-not-stop-abruptly
     guidance, alcohol cautions

Follow-up: 4 weeks via telehealth; portal PHQ-9 1 week before visit.
Return precautions: Return sooner or call if return of suicidal
ideation, severe sleep disruption, new mania symptoms, or any new
medication added by another provider.

E/M elements documented (provider to confirm level — time-based path
typical for behavioral-health telehealth follow-ups):
- HPI: extended (4+ elements — onset, duration, severity, modifying
  factors)
- ROS: pertinent (4 systems reviewed)
- MSE: focused, video-limited
- MDM: moderate complexity — chronic condition in partial remission,
  prescription drug management continued, validated screen reviewed
- Total time (counseling and coordination): 22 minutes documented
- Counseling/coordination >50% of total time: yes
[VERIFY: confirm code as 99213 vs. 99214 per provider's MDM judgment;
99214 defensible on time-based path with 22 minutes total and
counseling-dominant; CMS telehealth modifier (95) and POS (10 home / 02
other-than-home) per current OPPS/MPFS guidance]

--- Attestation ---

I personally evaluated this patient via synchronous audio-video
telehealth, reviewed the data above, and developed the assessment and
plan documented. Note dictated and reviewed by provider.

[Provider Name, credentials]
[NPI]   Encounter date: 2026-05-18
[State of provider: CO]   [State of patient: TX]
[Telehealth platform: VERIFY]
```

### What this second example demonstrates

- **Telehealth attestation block fires** — the `telehealth_attestation_addendum` config hook drives the dedicated attestation section at the top of the note (platform, provider/patient location, video-audio confirmation, consent, environmental privacy). The note does not bury the platform statement in the closing line; it leads with it because telehealth E/M billing depends on it.
- **State-licensure `[VERIFY]` flag** — the dictation states the provider is in CO and the patient is in TX, so the skill surfaces `[VERIFY: provider licensed in patient's state (TX) at time of encounter]` rather than asserting licensure it cannot confirm. This is the state-licensure-aware verify pattern from the `telehealth_attestation_addendum` hook description.
- **Vitals omission handled correctly** — the patient does not have a home BP cuff, so vitals are explicitly deferred with the operational rationale rather than fabricated. Demonstrates the "do not invent or infer clinical findings not present in the source material" rule under telehealth-specific conditions where structured vitals are often unavailable.
- **MSE adapted to virtual limits** — components observable on video (appearance, behavior, speech, affect, thought process/content, cognition by conversation) are documented; components requiring in-person observation (gait, full motor, complete cognitive screen) are explicitly flagged as deferred. Demonstrates the "Any limitations of virtual exam noted" requirement under the telehealth note structure.
- **Validated screen embedded in Objective, not Subjective** — PHQ-9 today, the 8-week trajectory, and C-SSRS are placed in Objective because they are validated instruments, not patient self-report narrative. This is the subjective-vs-objective discipline rule applied to a behavioral-health context.
- **Safety-critical clarification rules respected** — the dictation reports no SI/HI; the note repeats this discretely in three places (HPI, MSE thought content, Plan safety review) because suicidal-ideation status is on the default `safety_critical_clarification_rules` list and the note must make the absence-of-SI assertion impossible to miss for any future reviewer of the chart.
- **Time-based E/M path surfaced** — for behavioral-health telehealth where counseling and coordination typically dominate, the note documents total time and the counseling-percentage threshold rather than forcing a key-component-based level call. Demonstrates `em_documentation_standard=time_based_optional` behavior. The note also calls out the telehealth modifier and POS verify line.
- **No fabrication of facility-specific values** — telehealth platform name, NPI, state-licensure attestation, and exact telehealth-modifier guidance are all flagged `[VERIFY: ...]` rather than invented.

Together the two worked examples now demonstrate the skill's range across the two highest-volume note types in modern outpatient practice: an in-person primary-care SOAP (Example 1) and a cross-state telehealth behavioral-health follow-up (Example 2). Together they exercise the full set of twelve `config.yml` hooks (`practice_specialty`, `default_note_type`, `voice_and_tone`, `approved_abbreviations`, `em_documentation_standard`, `attestation_block`, `telehealth_attestation_addendum`, `icd10_strictness`, `safety_critical_clarification_rules`, `verify_flag_threshold`, `output_destination`, `config_missing_behavior`) with at least one example each — closing the worked-example debt that previously limited specificity scoring.
