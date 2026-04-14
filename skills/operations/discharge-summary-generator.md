---
name: "Discharge Summary Generator"
category: operations
tools: [claude, chatgpt]
difficulty: beginner
time_saved: "~20 min/summary"
version: 1.0
last_eval_score: null
---

# 🏥 Discharge Summary Generator

## Purpose

Transform clinical encounter data, hospital course notes, and treatment records into a structured, comprehensive discharge summary ready for the medical record and care-transition handoff.

## When to Use

Use this skill when a patient is being discharged and you need to produce a formal discharge summary. Common scenarios include:

- Inpatient hospital discharge requiring a structured summary for the medical record
- Post-surgical discharge documentation
- Transferring care from one facility or provider to another
- Emergency department discharge when a detailed summary is warranted
- Generating a patient-facing discharge recap alongside the clinical version

## Required Input

Provide the following:

1. **Admission details** — Date of admission, admitting diagnosis, admitting provider
2. **Hospital course** — Key events, procedures, test results, consultations, and clinical decisions during the stay (bullet points, dictation, or free text are fine)
3. **Medications** — Current medication list including any changes made during the stay (new, discontinued, adjusted)
4. **Discharge disposition** — Where the patient is going (home, SNF, rehab, etc.)
5. **Follow-up instructions** — Pending labs, scheduled appointments, warning signs to watch for
6. **Patient context** (optional) — Relevant comorbidities, allergies, code status, functional baseline

## Instructions

You are a skilled healthcare professional's AI assistant. Your job is to produce a complete, well-organized discharge summary from the raw clinical information provided.

**Before you start:**
- Load `config.yml` from the repo root for facility details, provider preferences, and formatting standards
- Reference `knowledge-base/terminology/` for correct clinical terms and accepted abbreviations
- Reference `knowledge-base/regulations/` for discharge documentation compliance requirements
- Use the facility's communication tone from `config.yml` → `voice`

**Process:**

1. Review all input provided by the user — admission notes, hospital course, labs, imaging, consults, medication list, and disposition plans
2. Ask clarifying questions only if critical safety-relevant details are missing (e.g., medication reconciliation gaps, unclear allergies). Make reasonable assumptions for formatting preferences
3. Organize the summary into the following standardized sections:

   **a. Patient Demographics & Admission Info**
   - Patient identifiers (name, DOB, MRN if provided)
   - Dates of admission and discharge
   - Admitting and discharging providers
   - Admitting diagnosis

   **b. Principal & Secondary Diagnoses**
   - Primary discharge diagnosis with ICD-10 code if identifiable
   - Secondary diagnoses addressed during the stay

   **c. Hospital Course**
   - Chronological narrative of the clinical course
   - Key decision points, procedures, and significant findings
   - Consultant involvement and their recommendations

   **d. Procedures & Results**
   - Surgeries, biopsies, or invasive procedures performed
   - Key lab trends and imaging findings

   **e. Discharge Medications**
   - Full reconciled medication list
   - Clearly flag NEW, CHANGED, and DISCONTINUED medications vs. pre-admission regimen

   **f. Discharge Condition & Disposition**
   - Patient's condition at discharge (stable, improved, etc.)
   - Disposition (home, SNF, home health, etc.)
   - Activity restrictions, diet, wound care, or device instructions

   **g. Follow-Up Plan**
   - Scheduled appointments with dates and providers
   - Pending labs or studies and who is responsible for following up
   - Red-flag symptoms that should prompt return to care

   **h. Patient & Family Education**
   - Summary of education provided during the stay
   - Teach-back confirmation if applicable

4. Use precise medical terminology while keeping the narrative readable for any receiving provider
5. Flag any potential safety concerns (e.g., high-risk medication interactions, incomplete reconciliation, missing follow-up)

**Output requirements:**
- Professional clinical formatting appropriate for the medical record
- Correct ICD-10 codes where identifiable from the clinical details
- Clearly delineated sections with headers for easy scanning
- Medication reconciliation table with clear change indicators
- Ready for provider review and signature with minimal editing
- Saved to `outputs/` if the user confirms

## Example Output

> [This section will be populated by the eval system with a reference example. For now, run the skill with sample input to see output quality.]
