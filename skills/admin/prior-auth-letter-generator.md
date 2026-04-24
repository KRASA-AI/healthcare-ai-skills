---
name: "Prior Auth Letter Generator"
category: admin
tools: [claude, chatgpt]
difficulty: intermediate
time_saved: "~25 min/letter"
version: 2.1
last_eval_score: 8.3
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

**Before you start:**
- Load `config.yml` from the repo root for facility details, provider credentials, NPI numbers, and payer mix
- Reference `knowledge-base/terminology/` for correct clinical and billing terminology
- Reference `knowledge-base/regulations/` for payer-specific prior auth requirements, CMS guidelines, and coverage criteria
- Use the facility's communication tone from `config.yml` → `voice`

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
