---
name: "Prior Auth Letter Generator"
category: admin
tools: [claude, chatgpt]
difficulty: intermediate
time_saved: "~25 min/letter"
version: 2.0
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

> [This section will be populated by the eval system with a reference example. For now, run the skill with sample input to see output quality.]
