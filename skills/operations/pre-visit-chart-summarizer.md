---
name: "Pre-Visit Chart Summarizer"
category: operations
tools: [claude, chatgpt]
difficulty: beginner
time_saved: "~10 min/patient"
version: 1.0
last_eval_score: null
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

> [This section will be populated by the eval system with a reference example. For now, run the skill with sample input to see output quality.]
