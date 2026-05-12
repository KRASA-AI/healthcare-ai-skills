---
name: "Denial Appeal Letter Writer"
category: admin
tools: [claude, chatgpt]
difficulty: intermediate
time_saved: "~30 min/letter"
version: 1.3
last_eval_score: 8.90
---

# 📨 Denial Appeal Letter Writer

## Purpose

Draft a persuasive, evidence-based appeal letter in response to a payer claim denial, referencing clinical guidelines, medical necessity criteria, and patient-specific documentation to support overturning the denial.

## When to Use

Use this skill when a claim has been denied by a payer and you need to draft a formal appeal. Common scenarios include:

- Medical necessity denials for procedures, imaging, or specialist referrals
- Level-of-care denials (e.g., inpatient downgraded to observation)
- Pre-authorization retroactive denials
- Formulary or step-therapy denials for medications
- Out-of-network or bundled service denials
- Any payer adverse determination that requires a written clinical appeal

## Required Input

Provide the following:

1. **Denial details** — Denial letter or Explanation of Benefits (EOB) with the stated reason for denial, denial date, and reference/claim number
2. **Patient information** — Name, DOB, member ID, and relevant clinical history
3. **Clinical justification** — The medical reasoning for why the service was necessary (diagnosis, symptoms, prior treatments tried, clinical findings)
4. **Service details** — What was ordered or performed (CPT/HCPCS codes, dates of service, provider)
5. **Supporting evidence** (optional but recommended) — Relevant clinical guidelines (e.g., InterQual, Milliman, specialty society guidelines), peer-reviewed literature, or prior medical records that strengthen the case
6. **Payer information** — Insurance company name, plan type, and appeal submission address or fax if known

## Instructions

You are a skilled healthcare professional's AI assistant specializing in revenue cycle and payer relations. Your job is to draft a compelling, well-structured appeal letter that maximizes the chance of overturning a claim denial.

**Before you start:**
- Load `config.yml` from the repo root for facility details, provider credentials, and preferences
- Reference `knowledge-base/terminology/` for correct clinical and billing terminology
- Reference `knowledge-base/regulations/` for appeal timelines, payer-specific rules, and compliance requirements
- Use the facility's communication tone from `config.yml` → `voice`

**Use practice-specific appeal hooks from `config.yml` when present:**

- **`config.yml` → `payer_appeal_routing`** — keyed routing per payer: appeal address, appeals fax, secure portal, dedicated reviewer name if the practice has one. Example:
  ```yaml
  payer_appeal_routing:
    aetna_commercial:
      address: "Aetna Appeals, PO Box [###], [City, State, ZIP]"
      fax: "[###-###-####]"
      portal: "Availity → Appeals queue"
      timely_filing_days: 180
    medicare_advantage_humana:
      address: "Humana MA Appeals, [Address]"
      fax: "[###-###-####]"
      portal: "Humana provider portal"
      timely_filing_days: 65        # MA standard appeal
      expedited_hours: 72            # 42 CFR 422.590
  ```
  Honor the routing on file. If the payer is not mapped, address the letter to the generic appeals department on the denial notice and flag `[VERIFY: practice-specific appeals routing for this payer]`.
- **`config.yml` → `appeal_level_defaults`** — what level of appeal this skill drafts unless the user says otherwise (`first_level` / `second_level` / `external_review` / `IRO` / `MA_reconsideration`). Most practices want first-level by default; some specialty practices default to second-level after a known auto-deny pattern. Default to `first_level` if absent.
- **`config.yml` → `peer_to_peer_callback_window`** — the standing callback window the practice offers (e.g., `"1–4 pm Central, Monday–Thursday"`). Use this in the closing rather than inventing a window. Default to `"normal business hours, [contact]"` if absent.
- **`config.yml` → `denial_pattern_library`** — denial reasons the practice sees often, with the practice's preferred rebuttal anchors:
  ```yaml
  denial_pattern_library:
    inpatient_to_observation_reclassification:
      authority_order: ["CMS Two-Midnight Rule (42 CFR 412.3)",
                        "InterQual",
                        "ACC/AHA / specialty-society guideline if cardiac",
                        "payer's own coverage policy"]
      common_payer_anchors: ["risk of clinical deterioration",
                              "qualifying inpatient-only procedure",
                              "documented physician expectation of >2 midnights"]
    medical_necessity_advanced_imaging:
      authority_order: ["ACR Appropriateness Criteria",
                        "CMS NCD/LCD",
                        "specialty-society guideline",
                        "payer's medical policy"]
      common_payer_anchors: ["red-flag exam finding",
                              "documented conservative-care failure"]
    step_therapy_specialty_med:
      authority_order: ["FDA label",
                        "compendia (DrugDex / AHFS / Clinical Pharmacology)",
                        "specialty-society treatment guideline",
                        "payer's own step-therapy policy + override criteria"]
  ```
  When the denial reason matches a library entry, use the practice's preferred authority order in **Section d (Evidence & Guidelines)**. If the denial reason is novel, default to: regulation → official guidance → industry standard → payer's own policy.
- **`config.yml` → `signature_block_appeals`** — the practice's appeal-letter signature owner: ordering provider, medical director, revenue-integrity lead, or the practice's contracted appeal-writer. Default to the ordering provider when absent.
- **`config.yml` → `enclosure_pack_defaults`** — the standing enclosure list the practice attaches to every appeal of a given type (e.g., for inpatient-to-obs reclass: full H&P, all ED-to-inpatient progress notes, telemetry strips, troponin trend, ECGs, payer's own denial letter, Two-Midnight attestation note). Render this as a numbered list at the bottom of the letter. If absent, generate a deal-specific list and flag `[VERIFY: enclosure pack against practice standard]`.
- **`config.yml` → `expedited_review_triggers`** — practice-set rules for when to file an appeal as expedited rather than standard (e.g., active hospitalization, post-discharge with active medication-management need, oncology in active treatment, transplant work-up). When a trigger is met, surface the expedited timeline (e.g., 42 CFR 422.590(d) 72-hour expedited reconsideration for MA) at the top of the letter.
- **`config.yml` → `config_missing_behavior`** — if a config key is absent, fall back to documented defaults and insert a `[VERIFY: ...]` flag rather than inventing a payer address, deadline, or callback window.

**Process:**

1. Review the denial reason carefully and identify the specific clinical or administrative basis for the denial
2. Ask clarifying questions only if the denial reason is unclear or critical clinical evidence is missing. Make reasonable assumptions for formatting and style preferences
3. Structure the appeal letter with the following components:

   **a. Header & Identification**
   - Date, payer address, and appeal submission line
   - Patient name, DOB, member ID, claim/reference number
   - Date(s) of service and provider information
   - Clear subject line: "Appeal of Denial — [Claim #] — [Patient Name]"

   **b. Opening Statement**
   - State that this is a formal appeal of the denial decision
   - Reference the specific denial reason quoted from the payer's correspondence
   - Assert that the denied service meets medical necessity criteria

   **c. Clinical Narrative**
   - Present the patient's relevant medical history and current condition
   - Explain the clinical decision-making process that led to the service being ordered
   - Document prior conservative treatments attempted and their outcomes (step-therapy history)
   - Describe what would happen without the requested service (risk of harm, deterioration)

   **d. Evidence & Guidelines**
   - Cite specific clinical guidelines that support the service (InterQual criteria, specialty society guidelines, CMS National Coverage Determinations)
   - Reference peer-reviewed literature if applicable
   - Quote the payer's own coverage policy language if it supports the case
   - Include relevant ICD-10 and CPT codes with clinical correlation

   **e. Rebuttal of Denial Rationale**
   - Address each specific point raised in the denial
   - Counter with clinical evidence and documentation
   - Highlight any information the payer may have overlooked

   **f. Closing & Request**
   - Clearly request reversal of the denial and authorization/payment of the service
   - Note the appeal deadline and request timely response
   - Offer to provide additional documentation or participate in peer-to-peer review
   - Include provider signature block with credentials

4. Maintain a professional, factual, and assertive tone — not adversarial
5. Flag any potential weaknesses in the appeal case for the provider to address before submission

**Output requirements:**
- Formal business letter format on facility letterhead (from config)
- Precise clinical terminology with ICD-10 and CPT codes
- Evidence-based arguments with specific guideline citations
- Professional, persuasive tone appropriate for payer correspondence
- Ready for provider review and signature with minimal editing
- Saved to `outputs/` if the user confirms

## Example Output

The example below shows a full first-level appeal letter for one of the most-denied service categories — an inpatient-versus-observation level-of-care denial — drafted from a payer adverse-determination letter and the patient's hospital course. It exercises every required section (header, opening, clinical narrative, evidence, point-by-point rebuttal, closing), cites the specific InterQual criteria the payer applies, references the Medicare Two-Midnight Rule, and includes the deadline language and peer-to-peer offer that materially raise overturn rates on this denial type.

### Input from user (paraphrased)

- **Denial:** Anthem Blue Cross commercial PPO denied inpatient admission for a 71M who presented with NSTEMI; payer reclassified to observation. Denial letter dated 2026-04-09; claim # `BCAA-2026-0409-12345`. Denial reason quoted: *"Care could have been delivered at observation level; documentation did not establish medical necessity for inpatient admission."*
- **Patient:** 71-year-old male, member ID `XAA000123456`, hx HTN, T2DM, prior PCI 2022, eGFR 48.
- **Clinical:** Presented to ED with substernal chest pressure, troponin I 0.18 → 0.62 → 0.41 ng/mL (above lab's 99th percentile), new T-wave inversions V3–V5, ED BP 168/96. Started heparin gtt, dual antiplatelets, high-intensity statin, IV nitroglycerin. Cardiology consulted; cath performed HD#2 showed 90% mid-LAD stenosis → DES placed. Discharged HD#3 stable.
- **Service billed:** Inpatient admission DRG 247 (PCI without MCC), CPT 92928 (PCI with stent), DOS 2026-04-04 to 2026-04-06.
- **Supporting evidence available:** ED note, troponin trend, ECG comparisons, cath report, cardiology consult note, ACC/AHA NSTEMI guidelines (2014, updated 2025), CMS Two-Midnight Rule (42 CFR § 412.3), InterQual Acute Adult Cardiovascular criteria.
- **Payer info:** Anthem Blue Cross of California, commercial PPO. Appeal deadline: 180 days from denial.

### Generated appeal letter

```
[Practice Letterhead — pulled from config.yml]

April 24, 2026

Anthem Blue Cross of California
Attn: Appeals Department
P.O. Box 60007
Los Angeles, CA 90060
Fax: 1-855-XXX-XXXX

Re: First-Level Appeal of Inpatient-to-Observation Reclassification
    Patient: [Patient Name]
    Date of Birth: [yyyy-mm-dd]
    Member ID: XAA000123456
    Claim / Reference #: BCAA-2026-0409-12345
    Dates of Service: 2026-04-04 through 2026-04-06
    Admitting Provider: [Name, MD]   NPI: [xxxxxxxxxx]
    Subject: Appeal of Denial — BCAA-2026-0409-12345 — [Patient Name]

Dear Appeals Reviewer:

This letter constitutes a formal first-level appeal of Anthem's adverse
determination dated April 9, 2026, which reclassified the above-named
patient's April 4–6, 2026 inpatient admission to observation status. The
denial letter states: "Care could have been delivered at observation
level; documentation did not establish medical necessity for inpatient
admission."

We respectfully disagree. The patient was admitted with a Type 1 non-ST-
elevation myocardial infarction (NSTEMI) requiring more than 24 hours of
hospital-based monitoring, urgent cardiac catheterization, and percutaneous
coronary intervention with drug-eluting stent placement. Both the
clinical course and the chart documentation meet inpatient-level criteria
under InterQual Acute Adult Cardiovascular: Acute Coronary Syndrome and
under the CMS Two-Midnight Rule (42 CFR § 412.3).

--- Clinical Narrative ---

The patient is a 71-year-old man with a documented history of essential
hypertension, type 2 diabetes mellitus with an HbA1c of 7.4%, prior
percutaneous coronary intervention with stent placement to the right
coronary artery in 2022, and chronic kidney disease stage 3a (baseline
eGFR 48). He presented to the emergency department on April 4, 2026 at
14:42 with new-onset substernal chest pressure radiating to the left jaw,
present at rest for approximately 90 minutes prior to arrival. Initial
ED vital signs showed BP 168/96, HR 92, SpO2 96% on room air. ECG
demonstrated new T-wave inversions in leads V3 through V5, with ST-segment
depression of approximately 1 mm in V4–V5, compared to the patient's
2024 baseline ECG (no ischemic findings). Initial troponin I was 0.18 ng/mL,
above the laboratory's 99th-percentile upper reference limit of 0.04 ng/mL.
Repeat troponin I at 3 hours rose to 0.62 ng/mL — a delta of more than
threefold and consistent with acute myocardial injury per the Fourth
Universal Definition of Myocardial Infarction. A third troponin at 6
hours was 0.41 ng/mL.

The clinical picture — ischemic chest pain at rest, dynamic ECG changes,
and a rising-then-falling troponin pattern in a patient with multiple
ASCVD risk factors and prior PCI — established a Type 1 NSTEMI per ACC/AHA
2014 NSTEMI guidelines (updated 2025). The patient was started in the ED
on a heparin infusion, dual antiplatelet therapy (aspirin 325 mg loading
dose followed by 81 mg daily plus ticagrelor 180 mg loading dose),
high-intensity atorvastatin 80 mg, and intravenous nitroglycerin titrated
for ongoing chest pain.

Cardiology was consulted in the ED at 16:55 on April 4. Per cardiology's
recommendation, the patient was admitted to inpatient telemetry for
continuous cardiac monitoring, anticoagulation management, and early-
invasive coronary angiography within 24 hours per Class I ACC/AHA
recommendation for high-risk NSTEMI (TIMI risk score 5; GRACE score 142,
both predictive of substantial in-hospital mortality risk).

On hospital day 2 (April 5), the patient underwent coronary angiography,
which revealed a 90% mid-LAD stenosis with TIMI 2 flow. A 2.75 × 18 mm
drug-eluting stent was deployed with restoration of TIMI 3 flow. The
patient was monitored post-procedure on telemetry overnight for access-
site complications and arrhythmia, with serial troponins and renal
function checks (creatinine peaked at 1.4 from baseline 1.1 — contrast-
associated acute kidney injury, requiring inpatient monitoring of
hydration and renal trajectory). On hospital day 3 (April 6), creatinine
returned to 1.2, the patient remained chest-pain free on telemetry, and
he was discharged home on guideline-directed medical therapy with cardiac
rehabilitation referral.

The total inpatient stay was 50 hours, 18 minutes (April 4 at 17:30
admission order to April 6 at 19:48 discharge), spanning two midnights.

--- Evidence and Guidelines ---

1. **CMS Two-Midnight Rule (42 CFR § 412.3)** — Inpatient admission is
   appropriate when the admitting practitioner reasonably expects the
   patient to require hospital care that crosses two midnights, supported
   by the medical record. The patient's NSTEMI diagnosis on presentation,
   immediate need for early-invasive cardiac catheterization, and
   anticipated post-PCI monitoring (≥ 24 hours per ACC/AHA recommendation)
   gave the admitting clinician an objectively reasonable expectation of
   a stay crossing two midnights. The patient in fact crossed two
   midnights, and the medical record contemporaneously documents the
   clinical reasoning.

2. **InterQual Acute Adult Cardiovascular criteria** — Acute Coronary
   Syndrome admission criteria are met: (a) elevated troponin above the
   99th-percentile upper reference limit with rise-and-fall pattern;
   (b) dynamic ECG changes in two contiguous leads (V3–V5); (c) ongoing
   ischemic symptoms at rest in the ED; (d) initiation of anticoagulation
   and IV antianginal therapy. InterQual review intensity criteria are
   independently met by the urgent cardiac catheterization with PCI on
   hospital day 2.

3. **2014 ACC/AHA NSTEMI Guidelines (Updated 2025) — Class I
   Recommendation** — "An early invasive strategy (diagnostic angiography
   with intent to perform revascularization) is indicated in NSTE-ACS
   patients who have refractory angina or hemodynamic or electrical
   instability, and is reasonable within 24 hours in stable patients
   with high-risk features." This patient's TIMI 5 / GRACE 142 risk
   profile constitutes high-risk features per the guideline.

4. **Anthem's own commercial medical policy CG-MED-XX (Acute Coronary
   Syndrome Management)** — recognizes inpatient admission for Type 1
   MI with troponin elevation and need for revascularization. The denial
   appears to apply observation-level criteria appropriate to chest pain
   rule-out without an established MI, not to a confirmed NSTEMI with
   PCI.

--- Rebuttal of the Stated Denial Rationale ---

The denial states care could have been delivered at observation level
and that documentation did not establish medical necessity for inpatient
admission. We respectfully address each point:

- **"Care could have been delivered at observation level."** Observation
  status is appropriate for chest pain under evaluation when myocardial
  injury has not yet been established and the expected length of stay is
  under 48 hours. This patient's diagnosis of Type 1 NSTEMI was
  established in the ED prior to admission by the rising troponin trend
  and dynamic ECG changes. Observation status would not have permitted
  the inpatient-level monitoring, anticoagulation management, and 24-hour
  early-invasive cardiac catheterization that the ACC/AHA Class I
  recommendation requires. The patient additionally developed contrast-
  associated AKI requiring inpatient renal monitoring.

- **"Documentation did not establish medical necessity for inpatient
  admission."** The chart contains, contemporaneously: (a) the ED
  physician's documented clinical impression of NSTEMI with rationale
  citing the troponin trend, ECG changes, and ongoing symptoms; (b) the
  cardiology consult note recommending inpatient admission with early-
  invasive strategy and explicitly citing TIMI 5 / GRACE 142; (c) the
  admitting provider's H&P documenting the two-midnight expectation;
  and (d) a procedural note from the catheterization with the 90% mid-
  LAD culprit lesion identified and treated. We are providing copies of
  each document as enclosures.

We additionally note that observation reclassification of a patient who
underwent PCI with a confirmed NSTEMI diagnosis would expose Anthem
members to significantly higher cost-sharing under the plan's outpatient
benefit, and is inconsistent with Anthem's own coverage policy CG-MED-XX
on Acute Coronary Syndrome Management.

--- Closing and Specific Request ---

For the foregoing reasons, we respectfully request that Anthem reverse
the April 9, 2026 adverse determination and process the claim for the
April 4–6, 2026 admission at inpatient level of care under DRG 247. We
appreciate Anthem's timely review of this appeal within the 30-day window
applicable to first-level commercial-PPO appeals in California.

We are available to participate in a peer-to-peer review with Anthem's
medical director at any time during the appeal-review window.
Dr. [Provider Name] is available for a peer-to-peer call Monday through
Friday, 7:00 AM to 5:00 PM Pacific Time. Please contact our office at
[phone] to schedule.

If additional clinical information would be helpful, please contact
[Appeals Coordinator Name] at [phone] or [email]. Enclosed with this
letter, please find: (1) the full ED record; (2) serial troponin and
ECG documentation with the 2024 baseline ECG for comparison; (3) the
cardiology consult note; (4) the catheterization report; (5) the
discharge summary; (6) the relevant pages of ACC/AHA 2014 NSTEMI
Guidelines (updated 2025) cited above; and (7) the relevant pages of
Anthem CG-MED-XX cited above.

We expect Anthem to overturn this determination on the merits.

Sincerely,

[Provider Name, MD, FACC]
[NPI]
Department of Cardiology
[Practice / Hospital Name]
[Address] | [Phone] | [Email]

cc: [Patient]
    [Hospital Revenue Integrity / Compliance]
```

### What this example demonstrates

- **Header completeness** — patient identifiers, claim and reference numbers, exact date range, and a clean subject line that lets the payer route the appeal to the correct queue
- **Quoted denial rationale** — the payer's exact language is reproduced and addressed, not paraphrased away
- **Multi-source guideline citation** — CMS regulation, InterQual, ACC/AHA, and the payer's own coverage policy, in that order of authority
- **Point-by-point rebuttal** — each sentence of the denial reason is answered with chart-level evidence, not generic appeals language
- **Two-Midnight Rule treatment** — the regulatory citation, the expectation language, and the actual stay length are all on the page; this is the single most-cited basis for overturning inpatient-to-observation reclassifications
- **Peer-to-peer offer with windows** — a specific callback window raises overturn rates and gets the file out of pure paper review
- **Enclosure list** — a numbered enclosure inventory makes it harder for the reviewer to claim "documentation did not establish medical necessity" a second time
- **Tone** — professional and assertive, never adversarial; the closing sentence makes the request explicit without insulting the prior reviewer

---

### Second Example — Step-Therapy Exception Appeal for Specialty Biologic (Crohn's Disease)

The example below demonstrates the second major denial category in outpatient specialty practice: a step-therapy non-compliance denial for a biologic agent. This scenario exercises the `denial_pattern_library` → `step_therapy_specialty_med` path, the FDA-label-first authority chain, and the documented-failure framing that is the single most-cited basis for overturning step-therapy denials on first appeal.

#### Input from user (paraphrased)

- **Denial:** UnitedHealthcare commercial PPO denied adalimumab (Humira) for a 38F with confirmed moderate-to-severe Crohn's disease. Denial letter dated 2026-04-30; claim/reference # `UHC-2026-0430-88210`. Denial reason quoted: *"Requested medication does not meet step-therapy criteria. The member has not documented an adequate trial of a preferred formulary agent as required under UnitedHealthcare's Step Therapy Protocol for Biologics."*
- **Patient:** 38-year-old female, member ID `UHC00456789`, established with gastroenterology for Crohn's disease confirmed on ileocolonoscopy with biopsy (2025-09-12). Harvey-Bradshaw Index 12 (moderate-to-severe). CDAI 310 at last visit.
- **Clinical:** Failed mesalamine 4.8 g/day × 4 months (2025-09-15 to 2026-01-15) — inadequate response, persistent rectal bleeding and abdominal pain. Failed azathioprine 150 mg/day × 6 months (2026-01-18 to 2026-07-18 anticipated, but discontinued 2026-04-01 at 2.5 months for azathioprine-induced pancreatitis confirmed on lipase 4× ULN — documented adverse event). Methotrexate contraindicated (planning pregnancy). Gastroenterologist recommending adalimumab per AGA and ACG 2021 guidelines.
- **Service billed:** Adalimumab (Humira) 160 mg SQ loading dose then 80 mg at week 2, then 40 mg every other week. CPT J0135 / NDC on claim.
- **Supporting evidence available:** ileocolonoscopy report, pathology, Harvey-Bradshaw/CDAI scores, mesalamine failure note, azathioprine pancreatitis ED note + lipase trend, gastroenterology office notes, AGA Clinical Practice Guideline on Crohn's disease (Gastroenterology, 2021), ACG Clinical Guideline (Am J Gastroenterol, 2018), UHC's own Biologic Step Therapy Protocol, FDA Humira label (Crohn's disease first-line indication after conventional therapy failure).
- **Payer info:** UnitedHealthcare commercial PPO. Appeal deadline: 180 days from denial. Step-therapy protocol version on file: UHC Biologics Step Therapy v4.2 (2025).

#### Generated appeal letter

```
[Practice Letterhead — pulled from config.yml]

May 6, 2026

UnitedHealthcare Appeals Department
Attn: Clinical Appeals — Pharmacy / Medical Benefit
P.O. Box 30432
Salt Lake City, UT 84130
Fax: 1-866-XXX-XXXX

Re: First-Level Step-Therapy Exception Appeal — Adalimumab (Humira)
    Patient: [Patient Name]
    Date of Birth: [yyyy-mm-dd]
    Member ID: UHC00456789
    Claim / Reference #: UHC-2026-0430-88210
    Ordering Provider: [Name, MD, FACG]   NPI: [xxxxxxxxxx]
    Subject: Appeal of Denial — UHC-2026-0430-88210 — [Patient Name]

Dear Clinical Appeals Reviewer:

This letter constitutes a formal first-level step-therapy exception appeal
of UnitedHealthcare's adverse determination dated April 30, 2026, which
denied coverage for adalimumab (Humira) on the basis that the member has
not documented an adequate trial of a preferred formulary agent.

We respectfully submit that the member has, in fact, documented adequate
trials of two conventional agents — mesalamine and azathioprine — with
clinically documented failure of mesalamine and a serious adverse event
(acute pancreatitis) precluding continuation of azathioprine. Moreover,
the FDA-approved indication for adalimumab includes patients with
moderate-to-severe Crohn's disease who have had an inadequate response to
conventional therapy, and UHC's own Biologic Step Therapy Protocol v4.2
(2025), Section 3.2, recognizes documented adverse drug reaction to a
step-therapy agent as a qualifying exception criterion.

--- Clinical Narrative ---

The patient is a 38-year-old woman with a diagnosis of Crohn's disease
involving the terminal ileum and proximal colon, confirmed by
ileocolonoscopy with biopsy on September 12, 2025 (active chronic
ileitis with granulomatous inflammation; pathology report enclosed).
At diagnosis, the Harvey-Bradshaw Index was 14 (severe); at the most
recent visit (April 28, 2026), the HBI is 12 and the CDAI is 310 —
both in the moderate-to-severe range despite two trials of conventional
therapy (see below). The patient experiences persistent abdominal
cramping, four to six loose or liquid stools per day, rectal bleeding,
and significant functional impairment limiting her work as a physical
therapist.

The patient and her gastroenterologist have documented the following
sequential conventional-therapy trials:

**Trial 1 — Mesalamine (5-ASA), September 2025 – January 2026**
Mesalamine 4.8 g/day (Asacol HD 1.6 g TID) was initiated September 15,
2025. After 4 months of therapy (through January 15, 2026), the patient
demonstrated inadequate response: persistent rectal bleeding, HBI
unchanged at 13, no endoscopic improvement on flexible sigmoidoscopy
(December 2025). Mesalamine was discontinued January 18, 2026. Failure
documentation: gastroenterology office note 2026-01-15 (enclosed).

**Trial 2 — Azathioprine, January 2026 – April 2026 (discontinued for
adverse event)**
Azathioprine 150 mg/day was initiated January 18, 2026. On April 1,
2026, the patient presented to the emergency department with acute
epigastric pain, nausea, and vomiting. Laboratory evaluation revealed
lipase 1,240 U/L (4× the upper reference limit) with CT abdomen/pelvis
showing pancreatic inflammation consistent with drug-induced pancreatitis.
Azathioprine was immediately and permanently discontinued. The ED
attending and the gastroenterologist concur that the pancreatitis is
azathioprine-induced based on temporal relationship and exclusion of
biliary and alcoholic etiologies (normal gallbladder on CT, non-drinker).
Adverse event documentation: ED note 2026-04-01 with lipase trend +
CT impression (enclosed).

**Contraindication to alternative conventional agents:**
Mercaptopurine (6-MP) is contraindicated as a thiopurine cross-reaction
with azathioprine — use of 6-MP after azathioprine-induced pancreatitis
carries an approximately 40% risk of recurrent pancreatitis (ACG 2018
guideline, Section 4.3). Methotrexate, while guideline-endorsed for
maintenance in Crohn's disease, is contraindicated in this patient:
she is actively planning pregnancy and has declined methotrexate on
this basis after counseling (documented in gastroenterology note
2026-04-28, enclosed).

--- Evidence and Guidelines ---

1. **FDA-Approved Indication — Adalimumab (Humira) label, Current**
   The FDA-approved indication for adalimumab in adults specifically
   includes: "reducing signs and symptoms and inducing and maintaining
   clinical remission in adult patients with moderately to severely
   active Crohn's disease who have had an inadequate response to
   conventional therapy." The member's documented failure of two
   conventional agents — one for inadequate response, one for a
   serious adverse event — satisfies the FDA's own indication language.

2. **AGA Clinical Practice Guideline — Crohn's Disease (Gastroenterology,
   2021, Singh et al.)**
   The AGA guideline provides a strong recommendation (GRADE: Strong,
   Moderate Evidence) for anti-TNF therapy (including adalimumab) in
   patients with moderate-to-severe Crohn's disease who fail or are
   intolerant of conventional immunosuppressive therapy. The guideline
   explicitly includes intolerance due to adverse events as equivalent
   to treatment failure for the purpose of escalating to biologic therapy.

3. **ACG Clinical Guideline — Management of Crohn's Disease in Adults
   (Am J Gastroenterol, 2018, Lichtenstein et al.)**
   The ACG guideline identifies azathioprine-induced pancreatitis as a
   thiopurine-class adverse event that precludes re-challenge with either
   azathioprine or mercaptopurine. In this context, the guideline supports
   direct escalation to biologic therapy without a mandatory 6-MP trial.

4. **UnitedHealthcare Biologic Step Therapy Protocol v4.2 (2025),
   Section 3.2 — Exception Criteria**
   UHC's own step-therapy protocol states that coverage of a
   non-preferred biologic is appropriate when the member has documented
   "a clinically significant adverse event to a preferred step-therapy
   agent that, in the judgment of the treating physician, precludes
   continued or alternative conventional therapy." Azathioprine-induced
   pancreatitis with lipase 4× ULN and radiographic evidence of pancreatic
   inflammation satisfies this exception criterion as stated in the plan's
   own policy. We request that UHC apply its own Section 3.2 exception
   pathway to this case.

--- Rebuttal of the Stated Denial Rationale ---

The denial states the member has not documented an adequate trial of a
preferred formulary agent. We address each element:

- **"Has not documented an adequate trial."** Two conventional agents
  were trialed: mesalamine for 4 months with documented inadequate
  response, and azathioprine for 2.5 months discontinued for a serious
  adverse event (pancreatitis). Both trials are fully documented in
  enclosed office notes, ED records, and laboratory trends. The
  adequacy of the mesalamine trial (4 months at full therapeutic dose)
  is consistent with the AGA guideline's definition of an adequate
  trial for 5-ASA agents in luminal Crohn's disease. The azathioprine
  trial was terminated not by patient choice but by a serious adverse
  event — which UHC's own Protocol Section 3.2 recognizes as a
  qualifying exception.

- **"Of a preferred formulary agent."** UHC's step-therapy protocol
  lists azathioprine and 6-MP as preferred step-therapy agents.
  Azathioprine was trialed and caused pancreatitis; 6-MP is
  contraindicated by the ACG guideline following azathioprine-induced
  pancreatitis; methotrexate is contraindicated by the patient's
  pregnancy planning. The preferred formulary agents are exhausted or
  contraindicated. The only remaining guideline-endorsed conventional
  agent (methotrexate) carries a teratogenic contraindication the
  patient has affirmatively declined after informed counseling.
  Requiring further conventional-agent trials in this context would
  expose the member to known harm with no guideline support.

--- Closing and Specific Request ---

For the foregoing reasons, we respectfully request that UnitedHealthcare
approve adalimumab (Humira) under the step-therapy exception pathway in
Section 3.2 of the plan's Biologic Step Therapy Protocol v4.2 (2025).
We request a determination within the standard 30-day commercial appeal
window.

Dr. [Provider Name, MD, FACG] is available for a peer-to-peer review
with UHC's medical director. Please contact our office at [phone]
to schedule. We are available Monday through Friday, 8:00 AM to 5:00 PM
Central Time.

Enclosed with this letter: (1) ileocolonoscopy report and pathology
(2025-09-12); (2) gastroenterology office notes documenting HBI/CDAI,
mesalamine failure (2026-01-15), and azathioprine initiation
(2026-01-18); (3) ED note with lipase trend and CT impression
documenting azathioprine-induced pancreatitis (2026-04-01); (4)
gastroenterology office note documenting methotrexate contraindication
counseling (2026-04-28); (5) relevant pages of the AGA 2021 Crohn's
Disease guideline; (6) relevant pages of the ACG 2018 Crohn's Disease
guideline; (7) the adalimumab (Humira) FDA prescribing information,
Crohn's disease indication; (8) UHC Biologic Step Therapy Protocol
v4.2 (2025), Section 3.2, annotated to the relevant exception language.

Sincerely,

[Provider Name, MD, FACG]
[NPI]
Division of Gastroenterology
[Practice / Hospital Name]
[Address] | [Phone] | [Email]

cc: [Patient]
    [Pharmacy Benefit Manager — if applicable]
    [Practice Revenue Integrity]
```

#### What this second example demonstrates

- **Step-therapy exception framework** — the denial_pattern_library `step_therapy_specialty_med` path: authority order is FDA label → compendia / specialty-society guideline → payer's own step-therapy policy + override criteria. The payer's own exception criteria are cited last and cited precisely — Section 3.2 by version number — which is the single highest-leverage rebuttal move on a step-therapy denial.
- **Two distinct failure modes** — inadequate response (mesalamine) and adverse event (azathioprine pancreatitis) are documented separately, each with dated records, because they satisfy different exception criteria in the payer's protocol.
- **Contraindication chain** — azathioprine → 6-MP contraindicated (thiopurine cross-reaction per ACG guideline) → methotrexate contraindicated (teratogenicity / pregnancy planning). The chain shows the payer that every preferred formulary alternative is exhausted or medically precluded, not merely inconvenient.
- **Payer's own policy used as the primary rebuttal anchor** — quoting the plan's own exception-criterion language (Section 3.2, version-numbered) is the fastest path to overturn on a step-therapy denial: the reviewer cannot claim the exception doesn't exist.
- **Enclosure inventory** — eight numbered items, each tied directly to a claim made in the letter body, making it difficult for a second reviewer to assert documentation gaps.
- **Tone** — identical to the first example: professional, factual, assertive. The closing specifically offers peer-to-peer, which raises overturn rates on specialty-biologic step-therapy denials where the payer's physician reviewer may not be a gastroenterologist.
