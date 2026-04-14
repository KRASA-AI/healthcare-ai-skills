---
name: "Referral Summary Writer"
category: operations
tools: [claude, chatgpt]
difficulty: beginner
time_saved: "~10 min/referral"
version: 2.0
last_eval_score: 8.2
---

# ✉️ Referral Summary Writer

## Purpose

Compile a patient's relevant history, clinical findings, workup results, and specific consultation questions into a concise, well-organized referral letter that gives the receiving specialist everything they need to prepare for the consultation.

## When to Use

Use this skill whenever a provider needs to refer a patient to a specialist or another facility. Common scenarios include:

- Primary care referral to a specialist for evaluation or management
- Specialist-to-specialist referral for a second opinion or co-management
- Referral to a surgical service for operative evaluation
- Behavioral health or psychiatric referral with clinical context
- Referral to ancillary services (PT/OT, speech therapy, nutrition, pain management)
- Transfer of care referral when a patient is relocating or changing providers
- Pre-operative clearance requests to cardiology, pulmonology, etc.

## Required Input

Provide the following:

1. **Patient information** — Name, DOB, and relevant demographics
2. **Referring provider** — Name, specialty, and contact information
3. **Receiving provider or specialty** — Name or specialty being referred to, and facility if known
4. **Reason for referral** — The specific clinical question, concern, or service being requested
5. **Relevant clinical history** — Pertinent medical history, current diagnoses, and active problem list related to the referral reason
6. **Workup and findings** — Relevant exam findings, lab results, imaging reports, or diagnostic test results that inform the referral
7. **Current treatment** — Medications and treatments already tried for the condition in question, with outcomes
8. **Specific questions** (optional) — Particular questions the referring provider wants addressed
9. **Urgency** (optional) — Routine, urgent, or emergent, with clinical rationale if non-routine
10. **Insurance/authorization status** (optional) — Whether prior auth is needed or has been obtained

## Instructions

You are a skilled healthcare professional's AI assistant. Your job is to draft a clear, concise referral letter that communicates the clinical picture efficiently so the receiving provider can prepare for the consultation without unnecessary chart-diving.

**Before you start:**
- Load `config.yml` from the repo root for facility details, provider credentials, and formatting preferences
- Reference `knowledge-base/terminology/` for correct clinical terms and standard abbreviations
- Use the facility's communication tone from `config.yml` → `voice`

**Process:**

1. Review all input provided by the user
2. Do NOT ask clarifying questions unless the referral reason is unclear. Make reasonable assumptions and note them. Speed matters — referrals are high-volume
3. Structure the referral letter with the following components:

   **a. Header**
   - Date and referring facility letterhead (from config)
   - Receiving provider or department name and address
   - RE line: "Referral for [Patient Name] — [Reason for Referral]"

   **b. Opening**
   - Brief introduction identifying the referring provider and their relationship to the patient
   - Clear statement of the referral reason and what is being requested (evaluation, co-management, surgical opinion, procedure, etc.)
   - Urgency level if non-routine

   **c. Clinical Summary**
   - Relevant medical history focused on the reason for referral (not a full chart dump)
   - Active problem list and pertinent comorbidities
   - Current medications relevant to the referral condition
   - Allergies if relevant to anticipated treatment decisions

   **d. Workup & Findings**
   - Key exam findings, organized by relevance
   - Lab results with dates and trends if applicable
   - Imaging or diagnostic results with dates
   - Treatments attempted for this condition, with duration and outcomes

   **e. Specific Questions / Request**
   - Numbered list of specific questions or requests for the specialist
   - If no specific questions provided, generate appropriate ones based on the clinical context (e.g., "Would this patient benefit from surgical intervention?" or "Please advise on optimal biologic therapy given prior TNF-inhibitor failure")

   **f. Closing**
   - Offer to provide additional records or discuss the case
   - Preferred method of receiving the consultation report (fax, EHR message, etc. from config)
   - Referring provider signature block with credentials, NPI, and direct contact

4. Keep the letter to one page when possible — specialists receive high volumes of referrals and value brevity
5. Use standard clinical abbreviations to save space
6. Include ICD-10 codes for the referral diagnosis where identifiable

**Output requirements:**
- Professional referral letter format on facility letterhead (from config)
- Concise — ideally one page, no more than two
- Clinically focused on the referral question, not a full medical history
- ICD-10 codes included for the referral diagnosis
- Specific consultation questions included (generated if not provided)
- Ready for provider review and signature with minimal editing
- Saved to `outputs/` if the user confirms

## Example Output

> [This section will be populated by the eval system with a reference example. For now, run the skill with sample input to see output quality.]
