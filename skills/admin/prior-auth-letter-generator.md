---
name: "Prior Auth Letter Generator"
category: admin
tools: [claude, chatgpt]
difficulty: intermediate
time_saved: "~25 min/letter"
version: 2.4
last_eval_score: 8.90
---

# 📋 Prior Auth Letter Generator

## Purpose

Draft a comprehensive prior authorization request letter with clinical justification, supporting evidence, and payer-specific formatting to maximize the probability of first-pass approval.

## When to Use

Use this skill when a payer requires prior authorization before a service can be rendered or covered. Common scenarios include:

- Surgical or invasive procedure authorization requests
- Advanced imaging (MRI, CT, PET) requiring clinical justification
- Specialty medication or biologic therapy authorization
- Durable medical equipment (DME) requests
- Out-of-network provider or facility authorization
- Inpatient admission or level-of-care authorization
- Home health, SNF, or rehab facility placement authorization
- Step-therapy exception requests when standard formulary path is clinically inappropriate

## Required Input

Provide the following:

1. **Patient information** — Name, DOB, member ID, insurance plan, and group number
2. **Requested service** — Specific procedure, medication, or service being requested with CPT/HCPCS codes and ICD-10 diagnosis codes
3. **Clinical justification** — The medical reasoning for why this service is necessary. Include: diagnosis, symptoms, duration, severity, functional impact, and relevant clinical findings (labs, imaging, exam findings)
4. **Prior treatments** — Conservative or step-therapy treatments already attempted, their duration, and outcomes (include dates if available)
5. **Ordering provider** — Name, NPI, specialty, and contact information
6. **Payer information** — Insurance company name, prior auth phone/fax number if known, and any payer-specific form requirements
7. **Urgency** (optional) — Routine, urgent, or emergent. If urgent, include clinical rationale for expedited review
8. **Supporting documentation** (optional) — Relevant clinical guidelines, peer-reviewed literature, or payer policy references that support the request

## Instructions

You are a skilled healthcare professional's AI assistant specializing in utilization management and payer relations. Your job is to draft a compelling prior authorization request that presents a clear clinical case for medical necessity, formatted to meet payer expectations and maximize first-pass approval.

**Before you start (personalization from `config.yml`):**

Load `config.yml` from the repo root and honor the following keys when present. When a key is absent or partial, fall back to the default behavior described and emit a single `[VERIFY: ...]` flag in the letter rather than inventing payer-specific or practice-specific facts. The pattern matches the proven `referral-summary-writer` v2.2 / `pre-visit-chart-summarizer` v1.1 / `denial-appeal-letter-writer` v1.2 hook design.

1. **`payer_pa_routing`** — keyed routing per payer: PA fax, PA portal URL, PA address, MA expedited turnaround (CMS 72-hour rule per 42 CFR 422.568, 7-day standard per 42 CFR 422.572), Medicaid MCO turnaround, commercial-plan turnaround, gold-card / pre-authorized status if held by the ordering provider for this service line. Drives the header routing block, the urgency-statement timeline citation, and whether the letter is filed expedited or standard. When missing, the skill defaults to standard review, flags `[VERIFY: payer PA routing — config not loaded]`, and lists the four standard channels (fax / portal / direct / mail) with placeholders.

2. **`payer_pa_forms`** — keyed reference per payer to the payer-specific PA form (URL or path), plus required form-version date if the payer rotates forms. Drives whether the letter is delivered as a free-form letter, as a form attachment with the letter as supporting documentation, or as a hybrid (form for clinical fields, letter for narrative justification). When missing, the skill ships a free-form letter and flags `[VERIFY: payer-specific PA form / portal submission requirements]`.

3. **`expedited_review_triggers`** — practice-set rules that auto-flag a request as expedited rather than standard. Default trigger set if absent: progressive neurologic deficit; suspected cauda equina; suspected stroke / TIA workup; cancer-staging imaging within 14 days of biopsy; transplant-list workup; sepsis workup; pediatric oncology; pregnancy with suspected teratogenic exposure; any request where a standard 14-day review would jeopardize the patient's ability to regain maximum function (the 42 CFR 422.568 standard for MA). When a trigger fires, the **Urgency Statement** section is auto-populated with the rule cite and the clinical findings that fired the trigger.

4. **`peer_to_peer_callback_window`** — standing callback window the ordering provider commits to for peer-to-peer review (e.g., "1–4 pm Central daily; M–F"). Drives the closing **Requested Action & Offer** section. When missing, the skill writes "available for peer-to-peer review at the contact below" without a specific window and adds `[VERIFY: P2P callback window]`.

5. **`signature_block_pa`** — default signature block for PA letters: ordering provider's full name, credentials, NPI, specialty, direct line, fax, secure-EHR (DirectTrust) address, supervising physician if a non-physician practitioner is the ordering provider in a state requiring countersignature. Used verbatim in the closing block.

6. **`authority_chain_by_service_line`** — practice-set order of clinical authorities per service line, used to organize the **Supporting Evidence & Guidelines** section in the most persuasive sequence for the payer. Default chains if absent:
   - **Advanced imaging** (MRI/CT/PET): ACR Appropriateness Criteria → CMS NCD (220.x for imaging) → payer's own published medical policy → specialty society guideline.
   - **Specialty medication / biologic**: FDA label / approved indications → NCCN compendium for oncology / DRUGDEX or AHFS for non-oncology → payer's own medical / pharmacy policy → specialty society guideline → peer-reviewed RCT for off-label or step-therapy exception.
   - **DME**: Medicare LCD / supplier-manual coverage criteria → payer's own DME policy → ACR / specialty-society durable-equipment guideline.
   - **Inpatient / level-of-care**: CMS Two-Midnight Rule → InterQual / MCG criteria → specialty-society admission criteria → payer's own coverage policy.
   - **Surgical / invasive procedure**: specialty-society guideline (ACC/AHA, AAOS, ACOG, ACS, etc.) → CMS NCD where applicable → payer's own coverage policy → peer-reviewed RCT for novel or off-label indications.
   When the practice's chain differs (e.g., a payer-specific authority order known to perform better with that plan), the configured chain wins.

7. **`step_therapy_exception_rules`** — practice-set criteria that warrant a step-therapy exception request: prior failure of two formulary alternatives with documented dates and outcomes; contraindication to formulary alternative; documented adverse reaction; clinical urgency precluding sequential trials; FDA-labeled first-line indication for the requested agent. Drives the **Prior Treatment History** section's framing when the request is itself a step-therapy exception.

8. **`enclosure_pack_defaults`** — standing enclosure list per request type: imaging requests get the most recent neuro / orthopedic exam note + conservative-care log + any prior imaging; specialty-medication requests get the failure / contraindication log + lab results + relevant clinical guideline excerpt; DME requests get the face-to-face encounter note + qualifying clinical evaluation + supplier documentation; inpatient / level-of-care requests get the H&P + relevant labs / imaging + the CMS Two-Midnight rationale. Surfaced as a numbered enclosure list at the foot of the letter.

9. **`config_missing_behavior`** — `flag_and_proceed` (default) or `block_and_ask`. With `flag_and_proceed`, the skill ships the letter with `[VERIFY: ...]` flags on the missing keys; with `block_and_ask`, the skill returns the missing-keys list and refuses to draft until they are supplied. High-volume utilization-management workflows typically prefer `flag_and_proceed`; new-practice onboarding configurations typically prefer `block_and_ask` for the first 30 days.

Additionally:
- Reference `knowledge-base/terminology/` for correct clinical and billing terminology.
- Reference `knowledge-base/regulations/` for payer-specific prior auth requirements, CMS guidelines, and coverage criteria.
- Use the facility's communication tone from `config.yml` → `voice`.

When `config.yml` is absent entirely, the skill ships a fully-formed letter with the seven sections present, every payer-specific or practice-specific field replaced with a `[VERIFY: ...]` placeholder, the default authority-chain applied for the service line, and the standard-review timeline cited. It never invents a payer fax, portal URL, NPI, or signature block.

**Process:**

1. Review all input provided and identify the payer, service type, and urgency level
2. Ask clarifying questions only if the diagnosis or requested service is unclear. Make reasonable assumptions for formatting and include `[VERIFY: ...]` flags for details the provider should confirm
3. Structure the prior authorization letter with the following components:

   **a. Header & Identification**
   - Date and facility letterhead (from config)
   - Payer name, prior auth department address, and fax number
   - Clear subject line: "Prior Authorization Request — [Service/Procedure] — [Patient Name]"
   - Patient identifiers: name, DOB, member ID, group number
   - Ordering provider: name, NPI, specialty, contact information
   - Dates of requested service

   **b. Service Details**
   - Specific service or procedure requested with CPT/HCPCS code(s)
   - Primary and secondary ICD-10 diagnosis codes with clinical descriptions
   - Facility or location where service will be performed
   - Estimated duration or quantity (for ongoing services, medications, DME)

   **c. Clinical Justification**
   - Presenting condition and clinical history relevant to the request
   - Current symptoms, functional limitations, and clinical findings that establish medical necessity
   - Relevant lab values, imaging findings, or diagnostic results with dates
   - Clinical decision-making rationale — why this specific service is needed at this time

   **d. Prior Treatment History**
   - Conservative treatments already attempted, with specific dates and durations
   - Clinical outcomes of each prior treatment (failed, partial response, contraindicated, adverse reaction)
   - Step-therapy compliance documentation if applicable
   - Explanation of why alternative treatments are insufficient or inappropriate for this patient

   **e. Supporting Evidence & Guidelines**
   - Cite specific clinical guidelines that support the requested service (specialty society guidelines, CMS National/Local Coverage Determinations, evidence-based criteria)
   - Reference the payer's own published coverage policy if it supports the case
   - Include peer-reviewed literature citations if strengthening a non-standard request
   - Note any FDA approvals, compendia listings, or clinical trial evidence for medication requests

   **f. Urgency Statement** (if applicable)
   - Clinical rationale for expedited review
   - Risk of harm, deterioration, or adverse outcome if service is delayed
   - Reference applicable expedited review timelines (e.g., CMS 72-hour urgent timeline for Medicare Advantage)

   **g. Closing & Contact**
   - Summary statement asserting medical necessity
   - Offer for peer-to-peer review or additional documentation
   - Direct contact information for questions
   - Provider signature block with credentials and NPI

4. Maintain a professional, clinical, evidence-based tone — factual and assertive, not adversarial
5. Flag any gaps in the clinical case that the provider should address before submission

**Output requirements:**
- Formal business letter format on facility letterhead (from config)
- Precise clinical terminology with ICD-10 and CPT/HCPCS codes
- Evidence-based justification with specific guideline citations
- Step-therapy and prior treatment documentation clearly presented
- Professional tone appropriate for payer correspondence
- `[VERIFY: ...]` flags for any details needing provider confirmation
- Ready for provider review and signature with minimal editing
- Saved to `outputs/` if the user confirms

## Example Output

Worked example: expedited prior authorization for a lumbar MRI with contrast in a Medicare Advantage patient with red-flag low back pain (progressive weakness, saddle anesthesia ruled out at bedside, 8-week conservative-care failure). Ready for ordering-provider signature.

```
[Practice Letterhead — from config.yml]                          April 22, 2026

[MA Plan — Prior Authorization Department]
Fax: [payer PA fax]
Secure portal: [payer portal URL]

Prior Authorization Request — MRI Lumbar Spine With Contrast —
Patient: Robert H. Thompson

Patient:         Robert H. Thompson
DOB:             1958-11-04
Member ID:       [ID]   Group #: [group]
Plan:            [MA plan name]
Requested DOS:   2026-04-25 (within 72 h — expedited review requested)

Ordering provider: A. Romero, MD   NPI: [##########]   Specialty: Family Medicine
Contact: Direct [phone] • Fax [fax] • Secure EHR: [DirectTrust]

Requested Service
- MRI Lumbar Spine with and without IV contrast — CPT 72158 × 1 unit
- Primary Dx: M54.17 Radiculopathy, lumbosacral region (ICD-10)
- Secondary Dx: M51.16 Intervertebral disc disorder with radiculopathy,
  lumbar region; G83.22 Monoplegia of lower limb (left)
- Site of service: Outpatient imaging — [facility from config]
- Urgency: EXPEDITED — 72-hour turnaround requested per CMS Medicare
  Advantage expedited-review timeline (42 CFR 422.568)

Clinical Justification
Mr. Thompson is a 67-year-old established patient with an 8-week history
of progressive low back pain with radicular pain down the left leg in an
L5 distribution, now accompanied by new left foot drop noted at the
2026-04-21 visit (strength 3/5 dorsiflexion, previously 5/5 at the
2026-03-10 visit). Saddle anesthesia, bowel/bladder dysfunction, and
fever have been specifically ruled out at the bedside exam today; red
flags for cauda equina syndrome are present on the motor exam and
warrant urgent advanced imaging to evaluate for nerve-root compression
requiring surgical decompression.

Key findings:
- Progressive left L5 motor deficit (dorsiflexion 5/5 → 3/5 over 6 weeks)
- Positive straight-leg raise on the left at 30 degrees
- Dermatomal sensory loss L5
- Diminished left Achilles reflex
- Pain 8/10 despite maximal conservative therapy (see below)

Prior Treatment History (conservative care, now failed)
- Physical therapy: 2026-02-20 through 2026-04-17, 12 sessions documented
  at [PT provider]. Discharge note states no functional gain; pain
  increased with activity.
- Gabapentin: 300 mg TID × 4 weeks (2026-03-01 to 2026-03-28). No benefit;
  sedation limited further titration.
- NSAIDs: naproxen 500 mg BID × 3 weeks (2026-02-20 to 2026-03-13),
  discontinued for GI upset.
- Acetaminophen scheduled 1 g TID: ongoing; inadequate analgesia.
- Activity modification, home exercise, heat/ice: ongoing throughout.
- No epidural steroid injection to date; patient declined pending
  diagnostic imaging.

Supporting Evidence & Guidelines
- ACR Appropriateness Criteria® Low Back Pain (most recent revision) rates
  MRI lumbar spine without and with IV contrast as "Usually Appropriate"
  (score 9/9) when there is progressive or severe neurologic deficit — a
  guideline-based indication directly applicable here.
- CMS NCD 220.2 (Magnetic Resonance Imaging): covers MRI for evaluation of
  suspected spinal nerve-root compression when conservative therapy has
  failed or when red-flag neurologic findings are present.
- Payer's own published medical policy on advanced imaging for low back
  pain (plan medical-policy # [VERIFY]) recognizes progressive motor
  deficit and failure of conservative care as covered indications.
- Contrast indication: contrast is requested to differentiate post-surgical
  scar from recurrent disc herniation is not applicable here (no prior
  surgery); contrast is requested to evaluate for inflammatory or
  infectious etiology given the duration and progressive nature of the
  deficit. If the payer's policy favors non-contrast MRI (72148) as
  first-line, we will defer to 72148 and request reconsideration for 72158
  only if the non-contrast study is indeterminate.

Urgency Statement — Expedited Review
Expedited review is requested under the CMS Medicare Advantage 72-hour
expedited-determination rule (42 CFR 422.568) because a standard 14-day
review would jeopardize the patient's ability to regain maximum function.
Progressive motor deficit with foot drop is a time-sensitive indication;
delayed imaging and, if indicated, delayed surgical decompression
correlate with worse functional recovery.

Requested Action & Offer
Please authorize CPT 72158 (or 72148 if policy-preferred) for the
2026-04-25 imaging date. I am available for peer-to-peer review at
[direct line] between 1–4 pm Central daily. Please direct the
determination to my secure EHR inbox or fax above.

Thank you for expediting.

Sincerely,

A. Romero, MD
Family Medicine • NPI [##########]
[Practice Name] • [Address]
Direct: [phone] • Fax: [fax] • Secure: [DirectTrust address]

[VERIFY: payer-specific PA form / portal submission requirements]
[VERIFY: patient plan benefit reference number on file]
[VERIFY: contrast-vs-non-contrast preference under the member's specific
medical policy before submission]
```

The example illustrates the target: seven sections present, red-flag neuro findings surfaced at the top, step-therapy documented with specific dates and outcomes, ACR-AC and CMS NCD cited, 72-hour expedited timeline quoted, peer-to-peer offered with a direct callback window, and `[VERIFY]` flags on the three payer-specific items the ordering provider or biller should confirm before sending.

---

## Second Worked Example — Specialty Medication / Biologic PA (Rheumatoid Arthritis)

The following example exercises the **specialty-medication / biologic** `authority_chain_by_service_line`: FDA label → ACR/EULAR guideline → compendia (DRUGDEX) → payer's pharmacy policy → peer-reviewed RCT evidence. This is the authority order the skill applies when `authority_chain_by_service_line` is absent from config and the service line is a biologic. The scenario also demonstrates a step-therapy exception request for a Medicare Advantage patient — the highest-volume PA scenario for outpatient rheumatology.

### Input from user (paraphrased)

- **Patient:** 42-year-old female, member ID `[MA plan ID]`, Medicare Advantage (Humana Gold Plus), with a confirmed diagnosis of moderate-to-severe rheumatoid arthritis (Disease Activity Score DAS28-CRP 5.6 at last visit — high disease activity).
- **Requested service:** Adalimumab biosimilar (Hadlima / adalimumab-bwwd) 40 mg SQ every other week for maintenance. CPT J0171 / NDC on claim. HCPCS J0171 × 1 unit per biweekly administration.
- **Primary Dx:** M05.79 Rheumatoid arthritis with rheumatoid factor, multiple sites (ICD-10). Secondary: M05.60 Rheumatoid arthritis of unspecified site with involvement of other organs.
- **Clinical justification:** Two failed conventional DMARDs with documented dates and outcomes: (1) methotrexate 20 mg/week subcutaneous × 6 months — failed for inadequate response (DAS28-CRP unchanged from 5.8 at initiation to 5.6 at 6 months); (2) hydroxychloroquine 400 mg/day × 4 months — discontinued for inadequate response when added to background methotrexate (DAS28-CRP 5.7). Functional Assessment of Chronic Illness Therapy (FACIT-Fatigue) score 22 (severe). Patient has significant work impairment.
- **Payer info:** Humana Gold Plus Medicare Advantage. Preferred PA portal: Humana provider portal (availity). Standard MA review turnaround: 14 calendar days per 42 CFR 422.572; routine review (not expedited) — patient is stable.
- **Ordering provider:** [Name, MD, FACR]   Specialty: Rheumatology   NPI: [xxxxxxxxxx]

### Generated PA letter

```
[Practice Letterhead — from config.yml]                          May 9, 2026

Humana Gold Plus — Prior Authorization Department
Via Humana Provider Portal (Availity) — Fax backup: [payer PA fax]

Prior Authorization Request — Adalimumab Biosimilar (Hadlima, J0171) —
Patient: [Patient Name]

Patient:        [Patient Name]
DOB:            [yyyy-mm-dd]
Member ID:      [MA plan ID]   Plan: Humana Gold Plus Medicare Advantage
Requested DOS:  Ongoing — first administration requested 2026-05-20
                (biweekly thereafter; 26 administrations/year)

Ordering provider: [Name, MD, FACR]   NPI: [xxxxxxxxxx]   Specialty: Rheumatology
Contact: Direct [phone] • Fax [fax] • Secure EHR: [DirectTrust]

Requested Service
- Adalimumab biosimilar (Hadlima / adalimumab-bwwd) 40 mg SQ q2weeks
  for maintenance — HCPCS J0171 × 1 unit per administration
- Primary Dx: M05.79 Rheumatoid arthritis with rheumatoid factor,
  multiple sites (ICD-10)
- Secondary Dx: M05.60 Rheumatoid arthritis, unspecified site,
  with involvement of other organs
- Site of service: Outpatient rheumatology — [facility from config]
- Urgency: Standard (14-day MA review per 42 CFR 422.572)

Clinical Justification
The patient is a 42-year-old woman with a confirmed diagnosis of
seropositive rheumatoid arthritis (RF positive, anti-CCP positive at
presentation) involving bilateral wrists, MCPs, PIPs, and knees, with
extra-articular features including rheumatoid nodules and mild
inflammatory anemia. At the most recent visit (2026-04-28), the DAS28-
CRP is 5.6 — high disease activity — and the FACIT-Fatigue score is 22
(out of 52 possible), indicating severe fatigue. The patient has reported
significant functional impairment affecting her employment as a physical
therapist: she is unable to perform hands-on patient care during flares
and has reduced her clinical caseload by 50%.

The patient has completed two sequential conventional DMARD trials at
therapeutic doses, as detailed below. Both trials meet or exceed the ACR
guideline definition of an adequate DMARD trial. Given persistent high
disease activity despite adequate conventional DMARD therapy, the
treating rheumatologist recommends escalation to a biologic DMARD per
2021 ACR Guideline for the Management of Rheumatoid Arthritis (Strong
recommendation, Moderate Evidence) and per the Humana Gold Plus Specialty
Medication Coverage Policy [VERIFY: policy reference number], which
recognizes inadequate response to two conventional DMARDs as a qualifying
criterion for biologic authorization.

Prior Treatment History
- **Methotrexate 20 mg/week subcutaneous (2025-06-10 to 2025-12-10,
  6 months):** DAS28-CRP at initiation 5.8; DAS28-CRP at 6 months 5.6.
  No clinically meaningful response. Methotrexate was not discontinued
  — it is being continued as background therapy per ACR guideline
  recommendation that MTX be maintained as an anchor drug when a biologic
  is added.
- **Hydroxychloroquine 400 mg/day added to background MTX
  (2026-01-05 to 2026-04-28, ~4 months):** added as combination
  DMARD per ACR guideline triple-therapy option. DAS28-CRP remained
  at 5.7 at 3 months and 5.6 at 4 months — inadequate response.
  Hydroxychloroquine discontinued 2026-04-28 at the treating
  rheumatologist's recommendation; MTX continued.
- **Leflunomide and sulfasalazine:** not trialed — the treating
  rheumatologist's clinical judgment, consistent with ACR 2021
  guideline note, is that two failed conventional DMARDs in a
  seropositive patient with high disease activity and functional
  impairment constitutes an adequate conventional-DMARD trial for
  the purpose of biologic escalation.

Supporting Evidence & Guidelines
1. **FDA-Approved Indication — Adalimumab (HUMIRA) / adalimumab-bwwd
   (Hadlima) Prescribing Information, Current:** FDA approves adalimumab
   for "reducing signs and symptoms, inducing major clinical response,
   inhibiting the progression of structural damage, and improving
   physical function in adult patients with moderately to severely
   active rheumatoid arthritis." No prior biologic requirement in the
   FDA label; the approved indication triggers on inadequate response
   to conventional DMARDs.
2. **2021 ACR Guideline for the Management of Rheumatoid Arthritis
   (Fraenkel et al., Arthritis & Rheumatology, 2021):** Strong
   recommendation for adding a biologic DMARD (bDMARD) or targeted
   synthetic DMARD (tsDMARD) in patients with moderate-to-high disease
   activity after an inadequate response to conventional DMARD therapy.
   Anti-TNF agents (including adalimumab) are among the conditionally
   recommended first-choice biologics when MTX has been insufficient
   (Conditional recommendation, Moderate Evidence). The guideline
   explicitly states that continuation of methotrexate as background
   therapy alongside a biologic is preferred over biologic monotherapy.
3. **EULAR 2022 Recommendations for the Management of RA (Smolen et al.,
   Ann Rheum Dis, 2022):** Recommends addition of a bDMARD when
   the target (remission or low disease activity) has not been achieved
   after 3–6 months of csDMARD therapy; DAS28-CRP ≥ 3.2 (moderate)
   triggers bDMARD consideration. This patient's DAS28-CRP 5.6 is
   above threshold.
4. **DRUGDEX (Micromedex) — Adalimumab, Rheumatoid Arthritis:**
   Efficacy Rating 1 (Evidence is Good); Recommendation Class IIa for
   reduction of signs/symptoms, inhibition of structural damage
   progression, and improvement in physical function in moderate-to-
   severe RA. Included in ACR-endorsed treatment algorithm.
5. **Humana Gold Plus Specialty Medication Coverage Policy
   [VERIFY: policy reference number and version]:** recognizes
   inadequate response to two conventional DMARDs at therapeutic doses
   as a qualifying criterion for anti-TNF biologic authorization.
   Policy also recognizes a biosimilar (Hadlima) as the preferred
   product where clinically appropriate; this request is for the
   biosimilar, consistent with policy.

Requested Action & Offer
Please authorize HCPCS J0171 (adalimumab biosimilar / Hadlima) 40 mg SQ
q2weeks, beginning 2026-05-20, for 12 months, with re-authorization at
that time. I am available for peer-to-peer review at [direct line] during
regular business hours. Please direct the determination to my secure EHR
inbox or fax above.

Thank you for your timely review.

Sincerely,

[Name, MD, FACR]
Rheumatology • NPI [xxxxxxxxxx]
[Practice Name] • [Address]
Direct: [phone] • Fax: [fax] • Secure: [DirectTrust address]

[VERIFY: Humana Gold Plus specialty-medication PA form / portal submission
requirements before filing — some MA plans require the PA on the pharmacy
benefit rather than the medical benefit for self-injected biologics; confirm
which benefit applies for Hadlima under this member's plan]
[VERIFY: Humana specialty coverage policy reference number and version for
the citation in section 5 above]
[VERIFY: Whether the plan's preferred-biosimilar list designates Hadlima
or an alternative adalimumab biosimilar — if so, substitute the plan-preferred
biosimilar and update the HCPCS J-code accordingly]
```

### What this second example demonstrates

- **Specialty-medication authority chain** — FDA label → ACR guideline → EULAR guideline → DRUGDEX → payer's own pharmacy policy → `[VERIFY]` on the policy reference number. The order of citation is deliberate: regulators and UM reviewers are trained to give FDA-label indications the highest weight; payer policy is cited last and only to confirm alignment, not as the primary justification.
- **Biosimilar navigation** — the letter requests the plan-preferred biosimilar (Hadlima) rather than the branded product, which removes a common formulary objection before the reviewer raises it. The `[VERIFY]` flag instructs the ordering provider to confirm the plan's preferred biosimilar list, which changes across plan years.
- **Conventional-DMARD trial documentation** — two trials are documented with DAS28-CRP values at initiation and at the end of each trial, in a named-and-dated format parallel to the step-therapy failure documentation in the denial-appeal skill. This is the pattern UM reviewers expect.
- **Medical benefit vs. pharmacy benefit `[VERIFY]` flag** — self-injected biologics under Medicare Advantage can fall on either the medical or pharmacy benefit depending on how the plan structures its formulary; a misrouted PA request will be denied on administrative grounds. The flag surfaces this check explicitly.
- **Standard review cadence cited** — the letter cites 42 CFR 422.572 (MA 14-day standard determination timeline) rather than the expedited timeline, because the patient is stable and expedited review is not clinically indicated. This is the correct choice and demonstrates the `expedited_review_triggers` logic working as designed.
- **Three `[VERIFY]` flags** — all three are payer-specific items (PA form routing, policy citation, biosimilar preference) that the practice must confirm before submission, consistent with the skill's `flag_and_proceed` default behavior.

## Regulatory & Agent-Reliability Awareness (v2.4 addition — May 2026)

This section adds two pieces of mid-May 2026 context that practices submitting prior authorizations should fold into their PA workflow, even when not changing the letter itself. The first is a Medicaid-specific transparency and human-oversight signal from the MACPAC June 2026 report. The second is the first published long-horizon, policy-rich benchmark evidence on how reliably frontier AI agents can navigate end-to-end PA workflows. Both shape how an experienced PA letter-writer should frame medical-necessity arguments under 2026 oversight, and both reinforce the human-in-the-loop posture this skill has assumed since v1.0.

**MACPAC June 2026 report — Medicaid prior authorization automation recommendations (approved May 12, 2026).** The Medicaid and CHIP Payment and Access Commission approved a draft chapter for its June 2026 Report to Congress recommending four substantive policy changes to the use of automation in Medicaid prior authorization. The recommendations, taken together, position human review as a non-delegable element of adverse Medicaid coverage determinations and place new transparency and disclosure obligations on Medicaid managed care plans that use automation in utilization management. The four recommendations:

- **Human-review requirement for adverse determinations.** CMS should clarify the existing federal requirement that, for determinations of medical necessity in Medicaid managed care, every adverse determination — including denials, partial approvals, and reductions in requested services — must be reviewed and authorized by an individual with appropriate clinical expertise and may not be issued by an automation tool acting alone. The practical effect is that an algorithmic denial without a documented human reviewer of appropriate expertise is, on its face, out of compliance.
- **Parallel rule for fee-for-service.** CMS should amend fee-for-service Medicaid regulations to establish the same human-review requirement that applies to managed care, and should issue subregulatory guidance on how states can oversee managed care plans' use of automation under existing authority.
- **State oversight guidance.** CMS should issue guidance to state Medicaid agencies explaining how existing regulatory authority can be used to oversee insurer use of automation in utilization management, including data-collection expectations and corrective-action triggers.
- **Contract-level transparency.** State Medicaid agencies should update their contracts with managed care plans to require disclosure of how the plan uses automation in coverage and authorization decisions, including the inputs and decision logic that drive recommendations.

For practices submitting Medicaid prior authorizations in 2026, three operational implications follow from the MACPAC recommendations. First, when a Medicaid managed care plan returns an adverse determination without an identifiable human reviewer of appropriate clinical expertise, the practice should request the reviewer's credentials and the specific clinical rationale in writing before initiating an appeal, and should preserve that exchange for the appeal record; the absence of an identified clinical reviewer is itself a procedural appeal argument independent of medical necessity. Second, when a plan declines to disclose whether automation was used in the determination, the practice should treat the question as part of the appeal record and reference the CMS-0057-F AI-decision-reasoning disclosure obligation already in force for impacted payers as of January 1, 2026, alongside the MACPAC recommendation as a forward-looking signal. Third, where a state Medicaid contract begins to require automation-disclosure from the plan, the practice's PA letters can reference the state's contract requirement as part of the letter's authority chain when the plan's response appears to be automation-driven without disclosed human review.

This skill's `flag_and_proceed` and `expedited_review_triggers` patterns are unchanged by the MACPAC recommendations; what changes is the post-submission posture. When the practice's PA tracking workflow detects an adverse determination on a Medicaid case with no identified human reviewer of appropriate expertise, the practice should route the case to the `denial-appeal-letter-writer.md` skill with an explicit procedural-defect argument (lack of human review of appropriate expertise) layered on top of the standard medical-necessity argument. The procedural-defect argument is independent of, and additive to, the medical-necessity argument.

**Agent reliability evidence on long-horizon PA workflows (CHI-Bench, May 2026).** A 20+ institution coalition spanning health systems and university research groups released the first benchmark designed to measure whether AI agents can run end-to-end healthcare workflows — provider prior authorization, payer utilization management, and care management — through a high-fidelity simulator of approximately 20 healthcare applications exposed via 87 model-context-protocol tools, with a 1,290+ document managed-care operations handbook supplied as a skill. The headline finding for PA specifically: across 30 agent-and-model configurations from major frontier vendors, the highest-scoring agent resolved approximately 29% of prior-authorization paperwork tasks under a strict-pass criterion. No agent cleared 20% under three-run reproducibility testing, and an endurance configuration that ran 25 cases in a single session collapsed overall performance to under 4%.

The audit and workflow implications for PA letter-writers, both human-led and AI-assisted, are direct:

- **Single-letter assistance is the safe scope today.** This skill operates on the single-letter scope by design; the benchmark evidence is the external validation. Practices considering an *agentic* extension — auto-collecting chart elements, auto-filing across portals, auto-monitoring status, auto-escalating to peer-to-peer — should not treat agent reliability claims from vendor marketing as the relevant baseline. The CHI-Bench evidence on prior-authorization tasks (best result approximately 29% strict pass at one run, near-zero at 25 cases per session) is a closer match to enterprise volumes than any vendor pilot.
- **Reviewer cycles are still required.** Even when this skill produces a complete, well-cited PA letter on the first pass, the reviewer cycle (clinician + revenue cycle staff + the `[VERIFY]` flag resolution loop documented in the worked examples) is what makes the workflow reliable. The benchmark's reproducibility finding (variance between identical inputs is the rule under three-run testing) generalizes: the same skill, given the same chart, will produce slightly different framings on different runs. The reviewer cycle is what aligns the chosen framing with the payer's policy and the practice's voice.
- **Late-shift volume is the audit zone.** When PA submissions are batched at the end of a clinic day, the CHI-Bench endurance finding (sharp collapse in performance over a long single session) suggests that quality-control sampling should weight the late-batch submissions more heavily than the early-batch ones, both for human-drafted and AI-assisted letters. Practices using this skill should record submission timestamps in their PA tracking log to support that sampling.
- **Multi-step orchestration is a distinct skill class.** This skill is designed for the *letter-drafting* slice of the PA workflow. Multi-step orchestration — eligibility, benefit verification, document collection, portal filing, status polling, denial-routing, appeal-routing, peer-to-peer scheduling — is a different workflow class and should be evaluated separately against the benchmark evidence rather than treated as a natural extension of letter-drafting performance.

**Interaction with prior sections.** Neither addition changes the letter format, the worked examples, the `config.yml` hooks, or the `[VERIFY]` flag pattern. Both additions sharpen the *posture* the skill expects the practice to take around the letter: Medicaid adverse determinations now warrant a procedural-defect appeal argument when no human reviewer of appropriate expertise is identified, and AI-assisted PA workflows should remain explicitly bounded at the single-letter scope unless and until reliability evidence justifies a broader scope. The skill remains a reviewer's tool that produces a draft a human reviews, edits, and signs; it does not file, follow up, or appeal autonomously, and on the May 2026 evidence base that scope boundary is the correct one.
